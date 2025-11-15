# Documentation for `modules/gapi/include/opencv2/gapi/gproto.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gproto.hpp`
- **File Name**: `gproto.hpp`
- **File Size**: 5,173 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gproto.hpp](../../../../../modules/gapi/include/opencv2/gapi/gproto.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_GPROTO_HPP
#define OPENCV_GAPI_GPROTO_HPP

#include <type_traits>
#include <vector>
#include <ostream>

#include <opencv2/gapi/util/variant.hpp>

#include <opencv2/gapi/gmat.hpp>
#include <opencv2/gapi/gscalar.hpp>
#include <opencv2/gapi/garray.hpp>
#include <opencv2/gapi/gopaque.hpp>
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gmetaarg.hpp>

namespace cv {

// FIXME: user shouldn't deal with it - put to detail?
// GProtoArg is an union type over G-types which can serve as
// GComputation's in/output slots. In other words, GProtoArg
// wraps any type which can serve as G-API exchange type.
//
// In Runtime, GProtoArgs are substituted with appropriate GRunArgs.
//
// GProtoArg objects are constructed in-place when user describes
// (captures) computations, user doesn't interact with these types
// directly.
using GProtoArg = util::variant
    < GMat
    , GMatP
    , GFrame
    , GScalar
    , detail::GArrayU  // instead of GArray<T>
    , detail::GOpaqueU // instead of GOpaque<T>
    >;

using GProtoArgs = std::vector<GProtoArg>;

namespace detail
{
template<typename... Ts> inline GProtoArgs packArgs(Ts... args)
{
    return GProtoArgs{ GProtoArg(wrap_gapi_helper<Ts>::wrap(args))... };
}

}

template<class Tag>
struct GIOProtoArgs
{
public:
    // NB: Used by python wrapper
    GIOProtoArgs() = default;
    explicit GIOProtoArgs(const GProtoArgs& args) : m_args(args) {}
    explicit GIOProtoArgs(GProtoArgs &&args)      : m_args(std::move(args)) {}

    GProtoArgs m_args;

    // TODO: Think about the addition operator
    /**
     * @brief This operator allows to complement the proto vectors at runtime.
     *
     * It's an ordinary overload of addition assignment operator.
     *
     * Example of usage:
     * @snippet samples/cpp/tutorial_code/gapi/doc_snippets/dynamic_graph_snippets.cpp  GIOProtoArgs usage
     *
     */
    template<typename Tg>
    friend GIOProtoArgs<Tg>& operator += (GIOProtoArgs<Tg> &lhs, const GIOProtoArgs<Tg> &rhs);
};

template<typename Tg>
cv::GIOProtoArgs<Tg>& operator += (cv::GIOProtoArgs<Tg> &lhs, const cv::GIOProtoArgs<Tg> &rhs)
{
    lhs.m_args.reserve(lhs.m_args.size() + rhs.m_args.size());
    lhs.m_args.insert(lhs.m_args.end(), rhs.m_args.begin(), rhs.m_args.end());
    return lhs;
}

struct In_Tag{};
struct Out_Tag{};

using GProtoInputArgs  = GIOProtoArgs<In_Tag>;
using GProtoOutputArgs = GIOProtoArgs<Out_Tag>;

// Perfect forwarding
template<typename... Ts> inline GProtoInputArgs GIn(Ts&&... ts)
{
    return GProtoInputArgs(detail::packArgs(std::forward<Ts>(ts)...));
}

template<typename... Ts> inline GProtoOutputArgs GOut(Ts&&... ts)
{
    return GProtoOutputArgs(detail::packArgs(std::forward<Ts>(ts)...));
}

namespace detail
{
    // Extract elements form tuple
    // FIXME: Someday utilize a generic tuple_to_vec<> routine
    template<typename... Ts, int... Indexes>
    static GProtoOutputArgs getGOut_impl(const std::tuple<Ts...>& ts, detail::Seq<Indexes...>)
    {
        return GProtoOutputArgs{ detail::packArgs(std::get<Indexes>(ts)...)};
    }
}

template<typename... Ts> inline GProtoOutputArgs GOut(const std::tuple<Ts...>& ts)
{
    // TODO: think of std::forward(ts)
    return detail::getGOut_impl(ts, typename detail::MkSeq<sizeof...(Ts)>::type());
}

// Takes rvalue as input arg
template<typename... Ts> inline GProtoOutputArgs GOut(std::tuple<Ts...>&& ts)
{
    // TODO: think of std::forward(ts)
    return detail::getGOut_impl(ts, typename detail::MkSeq<sizeof...(Ts)>::type());
}

// Extract run-time arguments from node origin
// Can be used to extract constant values associated with G-objects
// (like GScalar) at graph construction time
GRunArg value_of(const GOrigin &origin);

// Transform run-time computation arguments into a collection of metadata
// extracted from that arguments
GMetaArg  GAPI_EXPORTS descr_of(const GRunArg  &arg );
GMetaArgs GAPI_EXPORTS descr_of(const GRunArgs &args);

// Transform run-time operation result argument into metadata extracted from that argument
// Used to compare the metadata, which generated at compile time with the metadata result operation in run time
GMetaArg GAPI_EXPORTS descr_of(const GRunArgP& argp);

// Checks if run-time computation argument can be described by metadata
bool GAPI_EXPORTS can_describe(const GMetaArg&  meta,  const GRunArg&  arg);
bool GAPI_EXPORTS can_describe(const GMetaArgs& metas, const GRunArgs& args);

// Checks if run-time computation result argument can be described by metadata.
// Used to check if the metadata generated at compile time
// coincides with output arguments passed to computation in cpu and ocl backends
bool GAPI_EXPORTS can_describe(const GMetaArg&  meta,  const GRunArgP& argp);

// Validates input arguments
void GAPI_EXPORTS validate_input_arg(const GRunArg& arg);
void GAPI_EXPORTS validate_input_args(const GRunArgs& args);

} // namespace cv

#endif // OPENCV_GAPI_GPROTO_HPP
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

- **GIOProtoArgs**: A class/struct defined in this file
- **Tag**: A class/struct defined in this file
- **Out_Tag**: A class/struct defined in this file
- **In_Tag**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GPROTO_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/util/variant.hpp`
- `opencv2/gapi/gmetaarg.hpp`
- `opencv2/gapi/garray.hpp`
- `vector`
- `opencv2/gapi/gmat.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi/gscalar.hpp`
- `ostream`
- `type_traits`
- `opencv2/gapi/gopaque.hpp`

**Python Imports:**
- `that`
- `node`


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

