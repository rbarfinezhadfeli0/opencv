# Documentation for `modules/gapi/src/compiler/passes/transformations.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/passes/transformations.cpp`
- **File Name**: `transformations.cpp`
- **File Size**: 5,609 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/passes/transformations.cpp](../../../../../modules/gapi/src/compiler/passes/transformations.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler/passes` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation

#include "precomp.hpp"

#include <ade/util/zip_range.hpp>
#include <ade/graph.hpp>

#include "api/gcomputation_priv.hpp"

#include "compiler/gmodel.hpp"
#include "compiler/gmodelbuilder.hpp"
#include "compiler/passes/passes.hpp"
#include "compiler/passes/pattern_matching.hpp"

#include <sstream>

namespace cv { namespace gimpl { namespace passes {
namespace
{
using Graph = GModel::Graph;
using Metadata = typename Graph::CMetadataT;

// Checks pairs of {pattern node, substitute node} and asserts if there are any incompatibilities
void checkDataNodes(const Graph& pattern,
                    const Graph& substitute,
                    const std::vector<ade::NodeHandle>& patternNodes,
                    const std::vector<ade::NodeHandle>& substituteNodes)
{
    for (auto it : ade::util::zip(patternNodes, substituteNodes)) {
        auto pNodeMeta = pattern.metadata(std::get<0>(it));
        auto sNodeMeta = substitute.metadata(std::get<1>(it));
        GAPI_Assert(pNodeMeta.get<NodeType>().t == NodeType::DATA);
        GAPI_Assert(pNodeMeta.get<NodeType>().t == sNodeMeta.get<NodeType>().t);
        GAPI_Assert(pNodeMeta.get<Data>().shape == sNodeMeta.get<Data>().shape);
    }
}

// Checks compatibility of pattern and substitute graphs based on in/out nodes
void checkCompatibility(const Graph& pattern,
                        const Graph& substitute,
                        const Protocol& patternP,
                        const Protocol& substituteP)
{
    const auto& patternDataInputs = patternP.in_nhs;
    const auto& patternDataOutputs = patternP.out_nhs;

    const auto& substituteDataInputs = substituteP.in_nhs;
    const auto& substituteDataOutputs = substituteP.out_nhs;

    // number of data nodes must be the same
    GAPI_Assert(patternDataInputs.size() == substituteDataInputs.size());
    GAPI_Assert(patternDataOutputs.size() == substituteDataOutputs.size());

    // for each pattern input node, verify a corresponding substitute input node
    checkDataNodes(pattern, substitute, patternDataInputs, substituteDataInputs);

    // for each pattern output node, verify a corresponding substitute output node
    checkDataNodes(pattern, substitute, patternDataOutputs, substituteDataOutputs);
}

// Tries to substitute __single__ pattern with substitute in the given graph
bool tryToSubstitute(ade::Graph& main,
                     const std::unique_ptr<ade::Graph>& patternG,
                     const cv::GComputation& substitute)
{
    GModel::Graph gm(main);

    // 1. find a pattern in main graph
    auto match1 = findMatches(*patternG, gm);
    if (!match1.ok()) {
        return false;
    }

    // 2. build substitute graph inside the main graph
    cv::gimpl::GModelBuilder builder(main);
    auto expr = cv::util::get<cv::GComputation::Priv::Expr>(substitute.priv().m_shape);
    const auto& proto_slots = builder.put(expr.m_ins, expr.m_outs);
    Protocol substituteP;
    std::tie(substituteP.inputs, substituteP.outputs, substituteP.in_nhs, substituteP.out_nhs) =
        proto_slots;

    const Protocol& patternP = GModel::Graph(*patternG).metadata().get<Protocol>();

    // 3. check that pattern and substitute are compatible
    // FIXME: in theory, we should always have compatible pattern/substitute. if not, we're in
    //        half-completed state where some transformations are already applied - what can we do
    //        to handle the situation better?  -- use transactional API as in fuse_islands pass?
    checkCompatibility(*patternG, gm, patternP, substituteP);

    // 4. make substitution
    performSubstitution(gm, patternP, substituteP, match1);

    return true;
}
}  // anonymous namespace

void applyTransformations(ade::passes::PassContext& ctx,
                          const GKernelPackage& pkg,
                          const std::vector<std::unique_ptr<ade::Graph>>& patterns)
{
    const auto& transforms = pkg.get_transformations();
    const auto size = transforms.size();
    if (0u == size) return;
    // Note: patterns are already generated at this point
    GAPI_Assert(patterns.size() == transforms.size());

    // transform as long as it is possible
    bool canTransform = true;
    while (canTransform)
    {
        canTransform = false;

        // iterate through every transformation and try to transform graph parts
        for (auto it : ade::util::zip(ade::util::toRange(transforms), ade::util::toRange(patterns)))
        {
            const auto& t = std::get<0>(it);
            auto& pattern = std::get<1>(it);  // Note: using pre-created graphs
            GAPI_Assert(nullptr != pattern);

            // if transformation is successful (pattern found and substituted), it is possible that
            // other transformations will also be successful, so set canTransform to the returned
            // value from tryToSubstitute
            canTransform = tryToSubstitute(ctx.graph, pattern, t.substitute());

            // Note: apply the __same__ substitution as many times as possible and only after go to
            //       the next one. BUT it can happen that after applying some substitution, some
            //       _previous_ patterns will also be found and these will be applied first
            if (canTransform) {
                break;
            }
        }
    }
}
}  // namespace passes
}  // namespace gimpl
}  // namespace cv
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `compiler/passes/passes.hpp`
- `ade/util/zip_range.hpp`
- `sstream`
- `compiler/gmodel.hpp`
- `precomp.hpp`
- `api/gcomputation_priv.hpp`
- `ade/graph.hpp`
- `compiler/passes/pattern_matching.hpp`
- `compiler/gmodelbuilder.hpp`

**Python Imports:**
- `tryToSubstitute`


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

