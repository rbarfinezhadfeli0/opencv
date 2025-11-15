# Documentation for `modules/gapi/src/compiler/passes/kernels.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/passes/kernels.cpp`
- **File Name**: `kernels.cpp`
- **File Size**: 10,059 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/passes/kernels.cpp](../../../../../modules/gapi/src/compiler/passes/kernels.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler/passes` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "precomp.hpp"

#include <ade/util/zip_range.hpp>   // util::indexed
#include <ade/graph.hpp>
#include <ade/passes/check_cycles.hpp>

#include <opencv2/gapi/gcompoundkernel.hpp> // compound::backend()
#include <opencv2/gapi/gkernel.hpp>         // GKernelPackage
#include <opencv2/gapi/infer.hpp>           // GNetPackage
#include <opencv2/gapi/streaming/desync.hpp>// GDesync intrinsic

#include "compiler/gmodel.hpp"
#include "compiler/passes/passes.hpp"

#include "api/gbackend_priv.hpp"
#include "backends/common/gbackend.hpp"
#include "compiler/gmodelbuilder.hpp"
#include "logger.hpp"    // GAPI_LOG
#include "api/gproto_priv.hpp" // is_dynamic, rewrap

static
const std::vector<std::string>& getKnownIntrinsics()
{
    // FIXME: This may be not the right design choice, but so far it works
    static const std::vector<std::string> known_intrinsics = {
        cv::gapi::streaming::detail::GDesync::id()
    };
    return known_intrinsics;
}
bool cv::gimpl::is_intrinsic(const std::string &s) {
    // FIXME: This search might be better in time once we start using string
    const std::vector<std::string>& known_intrinsics = getKnownIntrinsics();
    return std::find(known_intrinsics.begin(),
                     known_intrinsics.end(),
                     s) != known_intrinsics.end();
}

namespace
{
    struct ImplInfo
    {
        cv::GKernelImpl impl;
        cv::GArgs       in_args;
    };

    // Generally the algorithm is following
    //
    // 1. Get GCompoundKernel implementation
    // 2. Create GCompoundContext
    // 3. Run GCompoundKernel with GCompoundContext
    // 4. Build subgraph from inputs/outputs GCompoundKernel
    // 5. Replace compound node to subgraph

    void expand(ade::Graph& g, ade::NodeHandle nh, const ImplInfo& impl_info)
    {
        cv::gimpl::GModel::Graph gr(g);
        auto compound_impl = cv::util::any_cast<cv::detail::GCompoundKernel>(impl_info.impl.opaque);

        // GCompoundContext instantiates its own objects
        // in accordance with the RcDescs from in_args
        cv::detail::GCompoundContext context(impl_info.in_args);
        compound_impl.apply(context);

        cv::GProtoArgs ins, outs;
        ins.reserve(context.m_args.size());
        outs.reserve(context.m_results.size());

        // Inputs can be non-dynamic types.
        // Such inputs are not used when building a graph
        for (const auto& arg : context.m_args)
        {
            if (cv::gimpl::proto::is_dynamic(arg))
            {
                ins.emplace_back(cv::gimpl::proto::rewrap(arg));
            }
        }

        ade::util::transform(context.m_results, std::back_inserter(outs), &cv::gimpl::proto::rewrap);

        cv::gimpl::GModelBuilder builder(g);

        // Build the subgraph graph which will need to replace the compound node
        const auto& proto_slots = builder.put(ins, outs);

        const auto& in_nhs  = std::get<2>(proto_slots);
        const auto& out_nhs = std::get<3>(proto_slots);

        auto sorted_in_nhs  = cv::gimpl::GModel::orderedInputs(gr, nh);
        auto sorted_out_nhs = cv::gimpl::GModel::orderedOutputs(gr, nh);

        // Reconnect expanded kernels from graph data objects
        // to subgraph data objects, then drop that graph data objects
        for (const auto it : ade::util::zip(in_nhs, sorted_in_nhs))
        {
            const auto& subgr_in_nh = std::get<0>(it);
            const auto& comp_in_nh  = std::get<1>(it);

            cv::gimpl::GModel::redirectReaders(gr, subgr_in_nh, comp_in_nh);
            gr.erase(subgr_in_nh);
        }

        gr.erase(nh);

        for (const auto it : ade::util::zip(out_nhs, sorted_out_nhs))
        {
            const auto& subgr_out_nh = std::get<0>(it);
            const auto& comp_out_nh  = std::get<1>(it);

            cv::gimpl::GModel::redirectWriter(gr, subgr_out_nh, comp_out_nh);
            gr.erase(subgr_out_nh);
        }
    }
} // anonymous namespace

// This pass, given the network package, associates every infer[list] node
// with particular inference backend and its parameters.
void cv::gimpl::passes::bindNetParams(ade::passes::PassContext &ctx,
                                      const gapi::GNetPackage  &pkg)
{
    GModel::Graph gr(ctx.graph);
    ade::TypedGraph<NetworkParams> pgr(ctx.graph);

    for (const auto &nh : gr.nodes())
    {
        if (gr.metadata(nh).get<NodeType>().t == NodeType::OP)
        {
            auto &op = gr.metadata(nh).get<Op>();
            if (op.k.tag.empty())
                continue;

            // FIXME: What if there's more than one???
            const auto it = ade::util::find_if(pkg.networks,
                                               [&](const cv::gapi::GNetParam &p) {
                                                   return p.tag == op.k.tag;
                                               });
            if (it == std::end(pkg.networks))
                continue;

            pgr.metadata(nh).set(NetworkParams{it->params});
            op.backend = it->backend;
        }
    }
}

// This pass, given the kernel package, selects a kernel
// implementation for every operation in the graph
//
// Starting OpenCV 4.3, G-API may have some special "intrinsic"
// operations.  Those can be implemented by backends as regular
// kernels, but if not, they are handled by the framework itself in
// its optimization/execution passes.
void cv::gimpl::passes::resolveKernels(ade::passes::PassContext   &ctx,
                                       const GKernelPackage &kernels)
{
    std::unordered_set<cv::gapi::GBackend> active_backends;

    GModel::Graph gr(ctx.graph);
    for (const auto &nh : gr.nodes())
    {
        if (gr.metadata(nh).get<NodeType>().t == NodeType::OP)
        {
            // If the operation is known to be intrinsic and is NOT
            // implemented in the package, just skip it - there should
            // be some pass which handles it.
            auto &op = gr.metadata(nh).get<Op>();
            if (is_intrinsic(op.k.name) && !kernels.includesAPI(op.k.name)) {
                gr.metadata().set(HasIntrinsics{});
                continue;
            }
            // FIXME: And this logic is terribly wrong. The right
            // thing is to assign an intrinsic to a particular island
            // if and only if it is:
            // (a) surrounded by nodes of backend X, AND
            // (b) is supported by backend X.
            // Here we may have multiple backends supporting an
            // intrinsic but only one of those gets selected.  And
            // this is exactly a situation we need multiple versions
            // of the same kernel to be presented in the kernel
            // package (as it was designed originally).

            cv::GKernelImpl selected_impl;

            if (op.backend == cv::gapi::GBackend()) {
                std::tie(op.backend, selected_impl) = kernels.lookup(op.k.name);
            } else {
                // FIXME: This needs to be reworked properly
                // Lookup for implementation from the pre-assinged backend
                cv::gapi::GBackend dummy;
                std::tie(dummy, selected_impl) = op.backend.priv()
                    .auxiliaryKernels().lookup(op.k.name);
                // FIXME: Warning here!
                // This situation may happen when NN (infer) backend was assigned
                // by tag in bindNetParams (see above) but at this stage the operation
                // lookup resulted in another backend (and it is perfectly valid when
                // we have multiple NN backends available).
            }

            op.backend.priv().unpackKernel(ctx.graph, nh, selected_impl);
            active_backends.insert(op.backend);

            if (gr.metadata().contains<Deserialized>())
            {
                // Trick: in this case, the op.k.outMeta is by default
                // missing. Take it from the resolved kernel
                GAPI_Assert(op.k.outMeta == nullptr);
                const_cast<cv::GKernel::M&>(op.k.outMeta) = selected_impl.outMeta;
            } else {
                // Sanity check: the metadata function must be present
                GAPI_Assert(op.k.outMeta != nullptr);
            }
        }
    }
    gr.metadata().set(ActiveBackends{active_backends});
}

void cv::gimpl::passes::expandKernels(ade::passes::PassContext &ctx, const GKernelPackage &kernels)
{
    GModel::Graph gr(ctx.graph);

    // Repeat the loop while there are compound kernels.
    // Restart procedure after every successful unrolling
    bool has_compound_kernel = true;
    while (has_compound_kernel)
    {
        has_compound_kernel = false;
        for (const auto& nh : gr.nodes())
        {
            if (gr.metadata(nh).get<NodeType>().t == NodeType::OP)
            {
                const auto& op = gr.metadata(nh).get<Op>();
                // FIXME: Essentially the same problem as in the above resolveKernels
                if (is_intrinsic(op.k.name) && !kernels.includesAPI(op.k.name)) {
                    // Note: There's no need to set HasIntrinsics flag here
                    // since resolveKernels would do it later.
                    continue;
                }

                cv::gapi::GBackend selected_backend;
                cv::GKernelImpl    selected_impl;
                std::tie(selected_backend, selected_impl) = kernels.lookup(op.k.name);

                if (selected_backend == cv::gapi::compound::backend())
                {
                    has_compound_kernel = true;
                    expand(ctx.graph, nh, ImplInfo{selected_impl, op.args});
                    break;
                }
            }
        }
    }
    GAPI_LOG_INFO(NULL, "Final graph: " << ctx.graph.nodes().size() << " nodes" << std::endl);
}
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **ImplInfo**: A class/struct defined in this file

### Functions and Methods

- **must()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gcompoundkernel.hpp`
- `logger.hpp`
- `opencv2/gapi/infer.hpp`
- `compiler/passes/passes.hpp`
- `ade/util/zip_range.hpp`
- `backends/common/gbackend.hpp`
- `opencv2/gapi/gkernel.hpp`
- `ade/passes/check_cycles.hpp`
- `api/gbackend_priv.hpp`
- `opencv2/gapi/streaming/desync.hpp`
- `api/gproto_priv.hpp`
- `precomp.hpp`
- `ade/graph.hpp`
- `compiler/gmodel.hpp`
- `compiler/gmodelbuilder.hpp`

**Python Imports:**
- `graph`
- `inputs`
- `the`
- `in_args`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

