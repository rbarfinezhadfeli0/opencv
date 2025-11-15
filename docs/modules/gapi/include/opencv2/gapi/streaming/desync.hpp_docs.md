# Documentation for `modules/gapi/include/opencv2/gapi/streaming/desync.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/desync.hpp`
- **File Name**: `desync.hpp`
- **File Size**: 3,056 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/desync.hpp](../../../../../../modules/gapi/include/opencv2/gapi/streaming/desync.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020-2021 Intel Corporation


#ifndef OPENCV_GAPI_GSTREAMING_DESYNC_HPP
#define OPENCV_GAPI_GSTREAMING_DESYNC_HPP

#include <tuple>

#include <opencv2/gapi/util/util.hpp>
#include <opencv2/gapi/gtype_traits.hpp>
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gcall.hpp>
#include <opencv2/gapi/gkernel.hpp>

namespace cv {
namespace gapi {
namespace streaming {

namespace detail {
struct GDesync {
    static const char *id() {
        return "org.opencv.streaming.desync";
    }

    // An universal yield for desync.
    // Yields output objects according to the input Types...
    // Reuses gkernel machinery.
    // FIXME: This function can be generic and declared in gkernel.hpp
    //        (it is there already, but a part of GKernelType[M]
    template<typename... R, int... IIs>
    static std::tuple<R...> yield(cv::GCall &call, cv::detail::Seq<IIs...>) {
        return std::make_tuple(cv::detail::Yield<R>::yield(call, IIs)...);
    }
};

template<typename G>
G desync(const G &g) {
    cv::GKernel k{
          GDesync::id()                                     // kernel id
        , ""                                                // kernel tag
        , [](const GMetaArgs &a, const GArgs &) {return a;} // outMeta callback
        , {cv::detail::GTypeTraits<G>::shape}               // output Shape
        , {cv::detail::GTypeTraits<G>::op_kind}             // input data kinds
        , {cv::detail::GObtainCtor<G>::get()}               // output template ctors
        , {cv::detail::GTypeTraits<G>::op_kind}             // output data kinds
    };
    cv::GCall call(std::move(k));
    call.pass(g);
    return std::get<0>(GDesync::yield<G>(call, cv::detail::MkSeq<1>::type()));
}
} // namespace detail

/**
 * @brief Starts a desynchronized branch in the graph.
 *
 * This operation takes a single G-API data object and returns a
 * graph-level "duplicate" of this object.
 *
 * Operations which use this data object can be desynchronized
 * from the rest of the graph.
 *
 * This operation has no effect when a GComputation is compiled with
 * regular cv::GComputation::compile(), since cv::GCompiled objects
 * always produce their full output vectors.
 *
 * This operation only makes sense when a GComputation is compiled in
 * streaming mode with cv::GComputation::compileStreaming(). If this
 * operation is used and there are desynchronized outputs, the user
 * should use a special version of cv::GStreamingCompiled::pull()
 * which produces an array of cv::util::optional<> objects.
 *
 * @note This feature is highly experimental now and is currently
 * limited to a single GMat/GFrame argument only.
 */
GAPI_EXPORTS GMat desync(const GMat &g);
GAPI_EXPORTS GFrame desync(const GFrame &f);

} // namespace streaming
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_GSTREAMING_DESYNC_HPP
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

- **GDesync**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GSTREAMING_DESYNC_HPP()**: A function/method defined in this file
- **can()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gcall.hpp`
- `opencv2/gapi/util/util.hpp`
- `opencv2/gapi/gtype_traits.hpp`
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/garg.hpp`
- `tuple`

**Python Imports:**
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

