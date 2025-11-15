# Documentation for `modules/gapi/src/compiler/gmodel.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/gmodel.cpp`
- **File Name**: `gmodel.cpp`
- **File Size**: 9,190 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/gmodel.cpp](../../../../modules/gapi/src/compiler/gmodel.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#include "precomp.hpp"

#include <string>
#include <sstream> // used in GModel::log


#include <ade/util/zip_range.hpp>   // util::indexed
#include <ade/util/checked_cast.hpp>

#include <opencv2/gapi/gproto.hpp>
#include "api/gnode_priv.hpp"
#include "compiler/gobjref.hpp"
#include "compiler/gmodel.hpp"
#include "api/gorigin.hpp"
#include "compiler/gmodel_priv.hpp"

namespace cv { namespace gimpl {

ade::NodeHandle GModel::mkOpNode(GModel::Graph &g,
                                 const GKernel &k,
                                 const std::vector<GArg> &args,
                                 const cv::util::any &params,
                                 const std::string &island)
{
    ade::NodeHandle op_h = g.createNode();
    g.metadata(op_h).set(NodeType{NodeType::OP});
    //These extra empty {} are to please GCC (-Wmissing-field-initializers)
    g.metadata(op_h).set(Op{k, args, {}, {}, params});
    if (!island.empty())
        g.metadata(op_h).set(Island{island});
    return op_h;
}

ade::NodeHandle GModel::mkDataNode(GModel::Graph &g, const GOrigin& origin)
{
    ade::NodeHandle data_h = g.createNode();
    const auto id = g.metadata().get<DataObjectCounter>().GetNewId(origin.shape);
    g.metadata(data_h).set(NodeType{NodeType::DATA});

    GMetaArg meta;
    Data::Storage storage = Data::Storage::INTERNAL; // By default, all objects are marked INTERNAL

    if (origin.node.shape() == GNode::NodeShape::CONST_BOUNDED)
    {
        auto value = value_of(origin);
        meta       = descr_of(value);
        storage    = Data::Storage::CONST_VAL;
        g.metadata(data_h).set(ConstValue{value});
    }
    // FIXME: Sometimes a GArray-related node may be created w/o the
    // associated host-type constructor (e.g. when the array is
    // somewhere in the middle of the graph).
    auto ctor_copy = origin.ctor;
    g.metadata(data_h).set(Data{origin.shape, id, meta, ctor_copy, origin.kind, storage});
    return data_h;
}

ade::NodeHandle GModel::mkDataNode(GModel::Graph &g, const GShape shape)
{
    ade::NodeHandle data_h = g.createNode();
    g.metadata(data_h).set(NodeType{NodeType::DATA});

    const auto id = g.metadata().get<DataObjectCounter>().GetNewId(shape);
    GMetaArg meta;
    HostCtor ctor;
    Data::Storage storage = Data::Storage::INTERNAL; // By default, all objects are marked INTERNAL
    cv::detail::OpaqueKind kind = cv::detail::OpaqueKind::CV_UNKNOWN;

    g.metadata(data_h).set(Data{shape, id, meta, ctor, kind, storage});
    return data_h;
}

ade::EdgeHandle GModel::linkIn(Graph &g, ade::NodeHandle opH, ade::NodeHandle objH, std::size_t in_port)
{
    // Check if input is already connected
    for (const auto& in_e : opH->inEdges())
    {
        GAPI_Assert(g.metadata(in_e).get<Input>().port != in_port);
    }

    auto &op = g.metadata(opH).get<Op>();
    auto &gm = g.metadata(objH).get<Data>();

     // FIXME: check validity using kernel prototype
    GAPI_Assert(in_port < op.args.size());

    ade::EdgeHandle eh = g.link(objH, opH);
    g.metadata(eh).set(Input{in_port});

    // Replace an API object with a REF (G* -> GOBJREF)
    op.args[in_port] = cv::GArg(RcDesc{gm.rc, gm.shape, {}});

    return eh;
}

ade::EdgeHandle GModel::linkOut(Graph &g, ade::NodeHandle opH, ade::NodeHandle objH, std::size_t out_port)
{
    // FIXME: check validity using kernel prototype

    // Check if output is already connected
    for (const auto& out_e : opH->outEdges())
    {
        GAPI_Assert(g.metadata(out_e).get<Output>().port != out_port);
    }

    auto &op = g.metadata(opH).get<Op>();
    auto &gm = g.metadata(objH).get<Data>();

    GAPI_Assert(objH->inNodes().size() == 0u);

    ade::EdgeHandle eh = g.link(opH, objH);
    g.metadata(eh).set(Output{out_port});

    // TODO: outs must be allocated according to kernel protocol!
    const auto storage_with_port = ade::util::checked_cast<std::size_t>(out_port+1);
    const auto min_out_size = std::max(op.outs.size(), storage_with_port);
    op.outs.resize(min_out_size, RcDesc{-1,GShape::GMAT,{}}); // FIXME: Invalid shape instead?
    op.outs[out_port] = RcDesc{gm.rc, gm.shape, {}};

    return eh;
}

std::vector<ade::NodeHandle> GModel::orderedInputs(const ConstGraph &g, ade::NodeHandle nh)
{
    std::vector<ade::NodeHandle> sorted_in_nhs(nh->inEdges().size());
    for (const auto& in_eh : nh->inEdges())
    {
        const auto port = g.metadata(in_eh).get<cv::gimpl::Input>().port;
        GAPI_Assert(port < sorted_in_nhs.size());
        sorted_in_nhs[port] = in_eh->srcNode();
    }
    return sorted_in_nhs;
}

std::vector<ade::NodeHandle> GModel::orderedOutputs(const ConstGraph &g, ade::NodeHandle nh)
{
    std::vector<ade::NodeHandle> sorted_out_nhs(nh->outEdges().size());
    for (const auto& out_eh : nh->outEdges())
    {
        const auto port = g.metadata(out_eh).get<cv::gimpl::Output>().port;
        GAPI_Assert(port < sorted_out_nhs.size());
        sorted_out_nhs[port] = out_eh->dstNode();
    }
    return sorted_out_nhs;
}

void GModel::init(Graph& g)
{
    g.metadata().set(DataObjectCounter());
}

void GModel::log(Graph &g, ade::NodeHandle nh, std::string &&msg, ade::NodeHandle updater)
{
    std::string s = std::move(msg);
    if (updater != nullptr)
    {
        std::stringstream fmt;
        fmt << " (via " << updater << ")";
        s += fmt.str();
    }

    if (g.metadata(nh).contains<Journal>())
    {
        g.metadata(nh).get<Journal>().messages.push_back(s);
    }
    else
    {
        g.metadata(nh).set(Journal{{s}});
    }
}

// FIXME:
// Unify with GModel::log(.. ade::NodeHandle ..)
void GModel::log(Graph &g, ade::EdgeHandle eh, std::string &&msg, ade::NodeHandle updater)
{
    std::string s = std::move(msg);
    if (updater != nullptr)
    {
        std::stringstream fmt;
        fmt << " (via " << updater << ")";
        s += fmt.str();
    }

    if (g.metadata(eh).contains<Journal>())
    {
        g.metadata(eh).get<Journal>().messages.push_back(s);
    }
    else
    {
        g.metadata(eh).set(Journal{{s}});
    }
}

void GModel::log_clear(Graph &g, ade::NodeHandle node)
{
    if (g.metadata(node).contains<Journal>())
    {
        // according to documentation, clear() doesn't deallocate (__capacity__ of vector preserved)
        g.metadata(node).get<Journal>().messages.clear();
    }
}


ade::NodeHandle GModel::detail::dataNodeOf(const ConstLayoutGraph &g, const GOrigin &origin)
{
    // FIXME: Does it still work with graph transformations, e.g. redirectWriter()??
    return g.metadata().get<Layout>().object_nodes.at(origin);
}

std::vector<ade::EdgeHandle> GModel::redirectReaders(Graph &g, ade::NodeHandle from, ade::NodeHandle to)
{
    std::vector<ade::EdgeHandle> ehh(from->outEdges().begin(), from->outEdges().end());
    std::vector<ade::EdgeHandle> ohh;
    ohh.reserve(ehh.size());
    for (auto e : ehh)
    {
        auto dst = e->dstNode();
        auto input = g.metadata(e).get<Input>();
        g.erase(e);
        ohh.push_back(linkIn(g, dst, to, input.port));
    }
    return ohh;
}

ade::EdgeHandle GModel::redirectWriter(Graph &g, ade::NodeHandle from, ade::NodeHandle to)
{
    GAPI_Assert(from->inEdges().size() == 1);
    auto e = from->inEdges().front();
    auto op = e->srcNode();
    auto output = g.metadata(e).get<Output>();
    g.erase(e);
    return linkOut(g, op, to, output.port);
}

GMetaArgs GModel::collectInputMeta(const GModel::ConstGraph &cg, ade::NodeHandle node)
{
    GAPI_Assert(cg.metadata(node).get<NodeType>().t == NodeType::OP);
    GMetaArgs in_meta_args(cg.metadata(node).get<Op>().args.size());

    for (const auto &e : node->inEdges())
    {
        const auto& in_data = cg.metadata(e->srcNode()).get<Data>();
        in_meta_args[cg.metadata(e).get<Input>().port] = in_data.meta;
    }

    return in_meta_args;
}


ade::EdgeHandle GModel::getInEdgeByPort(const GModel::ConstGraph& cg,
                                        const ade::NodeHandle&    nh,
                                              std::size_t         in_port)
{
    auto inEdges = nh->inEdges();
    const auto& edge = ade::util::find_if(inEdges, [&](ade::EdgeHandle eh) {
        return cg.metadata(eh).get<Input>().port == in_port;
    });
    GAPI_Assert(edge != inEdges.end());
    return *edge;
}

GMetaArgs GModel::collectOutputMeta(const GModel::ConstGraph &cg, ade::NodeHandle node)
{
    GAPI_Assert(cg.metadata(node).get<NodeType>().t == NodeType::OP);
    GMetaArgs out_meta_args(cg.metadata(node).get<Op>().outs.size());

    for (const auto &e : node->outEdges())
    {
        const auto& out_data = cg.metadata(e->dstNode()).get<Data>();
        out_meta_args[cg.metadata(e).get<Output>().port] = out_data.meta;
    }

    return out_meta_args;
}

bool GModel::isActive(const GModel::Graph &cg, const cv::gapi::GBackend &backend)
{
    return ade::util::contains(cg.metadata().get<ActiveBackends>().backends,
                               backend);
}

}} // cv::gimpl
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
- `api/gorigin.hpp`
- `ade/util/zip_range.hpp`
- `ade/util/checked_cast.hpp`
- `compiler/gmodel_priv.hpp`
- `string`
- `sstream`
- `compiler/gobjref.hpp`
- `opencv2/gapi/gproto.hpp`
- `api/gnode_priv.hpp`
- `precomp.hpp`
- `compiler/gmodel.hpp`


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

