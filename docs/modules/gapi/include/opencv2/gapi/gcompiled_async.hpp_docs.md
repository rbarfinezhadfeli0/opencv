# Documentation for `modules/gapi/include/opencv2/gapi/gcompiled_async.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcompiled_async.hpp`
- **File Name**: `gcompiled_async.hpp`
- **File Size**: 3,528 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcompiled_async.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcompiled_async.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation


#ifndef OPENCV_GAPI_GCOMPILED_ASYNC_HPP
#define OPENCV_GAPI_GCOMPILED_ASYNC_HPP

#include <future>           //for std::future
#include <exception>        //for std::exception_ptr
#include <functional>       //for std::function
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/own/exports.hpp>

namespace cv {
    //fwd declaration
    class GCompiled;

namespace gapi{
namespace wip {
    class GAsyncContext;
    /**
    These functions asynchronously (i.e. probably on a separate thread of execution) call GCompiled::operator() member function of their first argument with copies of rest of arguments (except callback) passed in.
    The difference between the function is the way to get the completion notification (via callback or a waiting on std::future object)
    If exception is occurred during execution of apply it is transferred to the callback (via function parameter) or passed to future (and will be thrown on call to std::future::get)

    N.B. :
    Input arguments are copied on call to async function (actually on call to cv::gin) and thus do not have to outlive the actual completion of asynchronous activity.
    While output arguments are "captured" by reference(pointer) and therefore _must_ outlive the asynchronous activity
    (i.e. live at least until callback is called or future is unblocked)

    @param gcmpld       Compiled computation (graph) to start asynchronously
    @param callback     Callback to be called when execution of gcmpld is done
    @param ins          Input parameters for gcmpld
    @param outs         Output parameters for gcmpld
    */
    GAPI_EXPORTS void                async(GCompiled& gcmpld, std::function<void(std::exception_ptr)>&& callback, GRunArgs &&ins, GRunArgsP &&outs);

    /** @overload
    @param gcmpld       Compiled computation (graph) to run asynchronously
    @param callback     Callback to be called when execution of gcmpld is done
    @param ins          Input parameters for gcmpld
    @param outs         Output parameters for gcmpld
    @param ctx          Context this request belongs to
    @see   async GAsyncContext
    */
    GAPI_EXPORTS void                async(GCompiled& gcmpld, std::function<void(std::exception_ptr)>&& callback, GRunArgs &&ins, GRunArgsP &&outs, GAsyncContext& ctx);

    /** @overload
    @param gcmpld       Compiled computation (graph) to run asynchronously
    @param ins          Input parameters for gcmpld
    @param outs         Output parameters for gcmpld
    @return             std::future<void> object to wait for completion of async operation
    @see async
    */
    GAPI_EXPORTS std::future<void>   async(GCompiled& gcmpld, GRunArgs &&ins, GRunArgsP &&outs);

    /**
    @param gcmpld       Compiled computation (graph) to run asynchronously
    @param ins          Input parameters for gcmpld
    @param outs         Output parameters for gcmpld
    @param ctx          Context this request belongs to
    @return             std::future<void> object to wait for completion of async operation
    @see   async GAsyncContext
    */
    GAPI_EXPORTS std::future<void>   async(GCompiled& gcmpld, GRunArgs &&ins, GRunArgsP &&outs, GAsyncContext& ctx);
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_GCOMPILED_ASYNC_HPP
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

- **GAsyncContext**: A class/struct defined in this file
- **GCompiled**: A class/struct defined in this file

### Functions and Methods

- **of()**: A function/method defined in this file
- **parameter()**: A function/method defined in this file
- **is()**: A function/method defined in this file
- **OPENCV_GAPI_GCOMPILED_ASYNC_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `functional`
- `exception`
- `opencv2/gapi/own/exports.hpp`
- `opencv2/gapi/garg.hpp`
- `future`


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

