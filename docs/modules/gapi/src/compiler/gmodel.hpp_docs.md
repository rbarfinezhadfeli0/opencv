# Documentation for `modules/gapi/src/compiler/gmodel.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/gmodel.hpp`
- **File Name**: `gmodel.hpp`
- **File Size**: 11,819 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/compiler/gmodel.hpp](../../../../modules/gapi/src/compiler/gmodel.hpp)

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


#ifndef OPENCV_GAPI_GMODEL_HPP
#define OPENCV_GAPI_GMODEL_HPP

#include <memory>           // shared_ptr
#include <unordered_map>
#include <functional>       // std::function

#include <ade/graph.hpp>
#include <ade/typed_graph.hpp>
#include <ade/passes/topological_sort.hpp>

// /!\ ATTENTION:
//
// No API includes like GMat, GNode, GCall here!
// This part of the system is API-unaware by its design.
//

#include <opencv2/gapi/util/any.hpp>

#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gkernel.hpp>
#include <opencv2/gapi/gcommon.hpp>

#include "compiler/gobjref.hpp"
#include "compiler/gislandmodel.hpp"

namespace cv { namespace gimpl {

// TODO: Document all metadata types

struct NodeType
{
    static const char *name() { return "NodeType"; }
    enum { OP, DATA } t;
};

struct Input
{
    static const char *name() { return "Input"; }
    std::size_t port;
};

struct Output
{
    static const char *name() { return "Output"; }
    std::size_t port;
};

struct Op
{
    static const char *name() { return "Op"; }
    cv::GKernel         k;
    std::vector<GArg>   args; // TODO: Introduce a new type for internal args?
    std::vector<RcDesc> outs; // TODO: Introduce a new type for resource references

    cv::gapi::GBackend  backend;
    cv::util::any params; // Operation specific information
};

struct Data
{
    static const char *name() { return "Data"; }

    // FIXME: This is a _pure_ duplication of RcDesc now! (except storage)
    GShape   shape; // FIXME: Probably to be replaced by GMetaArg?
    int      rc;    // rc is unique but local to shape
    GMetaArg meta;
    HostCtor ctor;  // T-specific helper to deal with unknown types in our code
    cv::detail::OpaqueKind kind; // FIXME: is needed to store GArray/GOpaque type
    // FIXME: Why rc+shape+meta is not represented as RcDesc here?

    enum class Storage: int
    {
        INTERNAL,   // data object is not listed in GComputation protocol
        INPUT,      // data object is listed in GComputation protocol as Input
        OUTPUT,     // data object is listed in GComputation protocol as Output
        CONST_VAL,  // data object is constant.
                    // Note: CONST is sometimes defined in Win sys headers
    };
    Storage storage;
};

struct ConstValue
{
    static const char *name() { return "ConstValue"; }
    GRunArg arg;
};

// This metadata is valid for both DATA and OP kinds of nodes
// FIXME: Rename to IslandTag
struct Island
{
    static const char *name() { return "Island"; }
    std::string island; // can be set by user, otherwise is set by fusion
};

struct Protocol
{
    static const char *name() { return "Protocol"; }
    // TODO: Replace the whole thing with a "Protocol" object
    std::vector<RcDesc> inputs;
    std::vector<RcDesc> outputs;

    std::vector<ade::NodeHandle> in_nhs;
    std::vector<ade::NodeHandle> out_nhs;
};

// The original metadata the graph has been compiled for.
// - For regular GCompiled, this information always present and
//   is NOT updated on reshape()
// - For GStreamingCompiled, this information may be missing.
//   It means that compileStreaming() was called without meta.
struct OriginalInputMeta
{
    static const char *name() { return "OriginalInputMeta"; }
    GMetaArgs inputMeta;
};

struct OutputMeta
{
    static const char *name() { return "OutputMeta"; }
    GMetaArgs outMeta;
};

struct Journal
{
    static const char *name() { return "Journal"; }
    std::vector<std::string> messages;
};

// Unique data object counter (per-type)
class DataObjectCounter
{
public:
    static const char* name() { return "DataObjectCounter"; }
    int GetNewId(GShape shape) { return m_next_data_id[shape]++; }

    // NB: private!!! but used in the serialization
    // couldn't get the `friend` stuff working correctly -- DM
    std::unordered_map<cv::GShape, int> m_next_data_id;
};

// A projected graph of Islands (generated from graph of Operations)
struct IslandModel
{
    static const char* name() { return "IslandModel"; }
    std::shared_ptr<ade::Graph> model;
};

// List of backends selected for current graph execution
struct ActiveBackends
{
    static const char *name() { return "ActiveBackends"; }
    std::unordered_set<cv::gapi::GBackend> backends;
};

// This is a graph-global flag indicating this graph is compiled for
// the streaming case.  Streaming-neutral passes (i.e. nearly all of
// them) can ignore this flag safely.
//
// FIXME: Probably a better design can be suggested.
struct Streaming
{
    static const char *name() { return "StreamingFlag"; }
};


// This is a graph-global flag indicating this graph is compiled
// after the deserialization. Some bits of information may be
// unavailable (mainly callbacks) so let sensitive passes obtain
// the required information in their special way.
//
// FIXME: Probably a better design can be suggested.
struct Deserialized
{
    static const char *name() { return "DeserializedFlag"; }
};


// Backend-specific inference parameters for a neural network.
// Since these parameters are set on compilation stage (not
// on a construction stage), these parameters are bound lately
// to the operation node.
// NB: These parameters are not included into GModel by default
// since it is not used regularly by all parties.
struct NetworkParams
{
    static const char *name() { return "NetworkParams"; }
    cv::util::any opaque;
};

// This is a custom metadata handling operator.
// Sometimes outMeta() can't be bound to input parameters only
// so several backends (today -- mainly inference) may find this useful.
// If provided, the meta inference pass uses this function instead of
// OP.k.outMeta.
struct CustomMetaFunction
{
    static const char *name() { return "CustomMetaFunction"; }
    using CM = std::function< cv::GMetaArgs( const ade::Graph      &,
                                             const ade::NodeHandle &,
                                             const cv::GMetaArgs   &,
                                             const cv::GArgs       &)>;
    CM customOutMeta;
};

// This is a general flag indicating that this GModel has intrinsics.
// In the beginning of the compilation, it is a quick check to
// indicate there are intrinsics.
//
// In the end of the compilation, having this flag is fatal -- all
// intrinsics must be resolved.
struct HasIntrinsics
{
    static const char *name() { return "HasIntrinsicsFlag"; }
};

// This is a special tag for both DATA and OP nodes indicating
// which desynchronized path this node belongs to.
// This tag is set by a special complex pass intrinDesync/accept.
struct DesyncPath
{
    static const char *name() { return "DesynchronizedPath"; }

    // A zero-based index of the desynchronized path in the graph.
    // Set by intrinDesync() compiler pass
    int index;
};

// This is a special tag for graph Edges indicating that this
// particular edge starts a desynchronized path in the graph.
// At the execution stage, the data coming "through" these edges
// (virtually, of course, since our GModel edges never transfer the
// actual data, they just represent these transfers) is desynchronized
// from the rest of the pipeline, i.e. may be "lost" (stay unconsumed
// and then overwritten with some new data when streaming).
struct DesyncEdge
{
    static const char *name() { return "DesynchronizedEdge"; }

    // A zero-based index of the desynchronized path in the graph.
    // Set by intrinDesync/apply() compiler pass
    int index;
};

// This flag marks the island graph as "desynchronized"
struct Desynchronized
{
    static const char *name() { return "Desynchronized"; }
};

// Reference to compile args of the computation
struct CompileArgs
{
    static const char *name() { return "CompileArgs"; }
    GCompileArgs args;
};

namespace GModel
{
    using Graph = ade::TypedGraph
        < NodeType
        , Input
        , Output
        , Op
        , Data
        , ConstValue
        , Island
        , Protocol
        , OriginalInputMeta
        , OutputMeta
        , Journal
        , ade::passes::TopologicalSortData
        , DataObjectCounter
        , IslandModel
        , ActiveBackends
        , CustomMetaFunction
        , Streaming
        , Deserialized
        , HasIntrinsics
        , DesyncPath
        , DesyncEdge
        , Desynchronized
        , CompileArgs
        >;

    // FIXME: How to define it based on GModel???
    using ConstGraph = ade::ConstTypedGraph
        < NodeType
        , Input
        , Output
        , Op
        , Data
        , ConstValue
        , Island
        , Protocol
        , OriginalInputMeta
        , OutputMeta
        , Journal
        , ade::passes::TopologicalSortData
        , DataObjectCounter
        , IslandModel
        , ActiveBackends
        , CustomMetaFunction
        , Streaming
        , Deserialized
        , HasIntrinsics
        , DesyncPath
        , DesyncEdge
        , Desynchronized
        , CompileArgs
        >;

    // FIXME:
    // Export a single class, not a bunch of functions inside a namespace

    // User should initialize graph before using it
    // GAPI_EXPORTS for tests
    GAPI_EXPORTS void init (Graph& g);

    GAPI_EXPORTS ade::NodeHandle mkOpNode(Graph &g,
                                          const GKernel &k,
                                          const std::vector<GArg>& args,
                                          const cv::util::any& params,
                                          const std::string &island);
    // Isn't used by the framework or default backends, required for external backend development
    GAPI_EXPORTS ade::NodeHandle mkDataNode(Graph &g, const GShape shape);

    // Adds a string message to a node. Any node can be subject of log, messages then
    // appear in the dumped .dot file.x
    GAPI_EXPORTS void log(Graph &g, ade::NodeHandle op, std::string &&message, ade::NodeHandle updater = ade::NodeHandle());
    GAPI_EXPORTS void log(Graph &g, ade::EdgeHandle op, std::string &&message, ade::NodeHandle updater = ade::NodeHandle());
    // Clears logged messages of a node.
    GAPI_EXPORTS void log_clear(Graph &g, ade::NodeHandle node);

    GAPI_EXPORTS ade::EdgeHandle linkIn   (Graph &g, ade::NodeHandle op,     ade::NodeHandle obj, std::size_t in_port);
    GAPI_EXPORTS ade::EdgeHandle linkOut  (Graph &g, ade::NodeHandle op,     ade::NodeHandle obj, std::size_t out_port);

    GAPI_EXPORTS std::vector<ade::EdgeHandle> redirectReaders(Graph &g, ade::NodeHandle from, ade::NodeHandle to);
    GAPI_EXPORTS             ade::EdgeHandle  redirectWriter (Graph &g, ade::NodeHandle from, ade::NodeHandle to);

    GAPI_EXPORTS std::vector<ade::NodeHandle> orderedInputs (const ConstGraph &g, ade::NodeHandle nh);
    GAPI_EXPORTS std::vector<ade::NodeHandle> orderedOutputs(const ConstGraph &g, ade::NodeHandle nh);

    // Returns input meta array for given op node
    // Array is sparse, as metadata for non-gapi input objects is empty
    // TODO:
    // Cover with tests!!
    GAPI_EXPORTS GMetaArgs collectInputMeta(const GModel::ConstGraph &cg, ade::NodeHandle node);
    GAPI_EXPORTS GMetaArgs collectOutputMeta(const GModel::ConstGraph &cg, ade::NodeHandle node);

    GAPI_EXPORTS ade::EdgeHandle getInEdgeByPort(const GModel::ConstGraph& cg, const ade::NodeHandle& nh, std::size_t in_port);

    // Returns true if the given backend participates in the execution
    GAPI_EXPORTS bool isActive(const GModel::Graph &cg, const cv::gapi::GBackend &backend);
} // namespace GModel


}} // namespace cv::gimpl

#endif // OPENCV_GAPI_GMODEL_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **Storage**: A class/struct defined in this file
- **NodeType**: A class/struct defined in this file
- **DataObjectCounter**: A class/struct defined in this file
- **Deserialized**: A class/struct defined in this file
- **Island**: A class/struct defined in this file
- **Data**: A class/struct defined in this file
- **Op**: A class/struct defined in this file
- **NetworkParams**: A class/struct defined in this file
- **OriginalInputMeta**: A class/struct defined in this file
- **Input**: A class/struct defined in this file
- **CompileArgs**: A class/struct defined in this file
- **Journal**: A class/struct defined in this file
- **DesyncPath**: A class/struct defined in this file
- **IslandModel**: A class/struct defined in this file
- **Output**: A class/struct defined in this file
- **ConstValue**: A class/struct defined in this file
- **OutputMeta**: A class/struct defined in this file
- **Streaming**: A class/struct defined in this file
- **DesyncEdge**: A class/struct defined in this file
- **CustomMetaFunction**: A class/struct defined in this file
- **Desynchronized**: A class/struct defined in this file
- **ActiveBackends**: A class/struct defined in this file
- **HasIntrinsics**: A class/struct defined in this file
- **Protocol**: A class/struct defined in this file

### Functions and Methods

- **instead()**: A function/method defined in this file
- **OPENCV_GAPI_GMODEL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ade/typed_graph.hpp`
- `ade/passes/topological_sort.hpp`
- `unordered_map`
- `functional`
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/garg.hpp`
- `compiler/gislandmodel.hpp`
- `opencv2/gapi/util/any.hpp`
- `memory`
- `opencv2/gapi/gcommon.hpp`
- `compiler/gobjref.hpp`
- `ade/graph.hpp`

**Python Imports:**
- `graph`
- `the`


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

