# Documentation for `modules/gapi/include/opencv2/gapi/gcomputation_async.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcomputation_async.hpp`
- **File Name**: `gcomputation_async.hpp`
- **File Size**: 3,396 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcomputation_async.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcomputation_async.hpp)

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

#ifndef OPENCV_GAPI_GCOMPUTATION_ASYNC_HPP
#define OPENCV_GAPI_GCOMPUTATION_ASYNC_HPP


#include <future>                           //for std::future
#include <exception>                        //for std::exception_ptr
#include <functional>                       //for std::function
#include <opencv2/gapi/garg.hpp>            //for GRunArgs, GRunArgsP
#include <opencv2/gapi/gcommon.hpp>         //for GCompileArgs
#include <opencv2/gapi/own/exports.hpp>


namespace cv {
    //fwd declaration
    class GComputation;
namespace gapi {
namespace wip  {
    class GAsyncContext;
    /** In contrast to async() functions, these do call GComputation::apply() member function of the GComputation passed in.

    @param gcomp        Computation (graph) to run asynchronously
    @param callback     Callback to be called when execution of gcomp is done
    @param ins          Input parameters for gcomp
    @param outs         Output parameters for gcomp
    @param args         Compile arguments to pass to GComputation::apply()
    @see                async
    */
    GAPI_EXPORTS void                async_apply(GComputation& gcomp, std::function<void(std::exception_ptr)>&& callback, GRunArgs &&ins, GRunArgsP &&outs, GCompileArgs &&args = {});
    /** @overload
    @param gcomp        Computation (graph) to run asynchronously
    @param callback     Callback to be called when execution of gcomp is done
    @param ins          Input parameters for gcomp
    @param outs         Output parameters for gcomp
    @param args         Compile arguments to pass to GComputation::apply()
    @param ctx          Context this request belongs to
    @see                async_apply async GAsyncContext
    */
    GAPI_EXPORTS void                async_apply(GComputation& gcomp, std::function<void(std::exception_ptr)>&& callback, GRunArgs &&ins, GRunArgsP &&outs, GCompileArgs &&args, GAsyncContext& ctx);
    /** @overload
    @param gcomp        Computation (graph) to run asynchronously
    @param ins          Input parameters for gcomp
    @param outs         Output parameters for gcomp
    @param args         Compile arguments to pass to GComputation::apply()
    @return             std::future<void> object to wait for completion of async operation
    @see                async_apply async
    */
    GAPI_EXPORTS std::future<void>   async_apply(GComputation& gcomp, GRunArgs &&ins, GRunArgsP &&outs, GCompileArgs &&args = {});
    /** @overload
    @param gcomp        Computation (graph) to run asynchronously
    @param ins          Input parameters for gcomp
    @param outs         Output parameters for gcomp
    @param args         Compile arguments to pass to GComputation::apply()
    @param ctx          Context this request belongs to
    @return             std::future<void> object to wait for completion of async operation
    @see                async_apply async GAsyncContext
    */
    GAPI_EXPORTS std::future<void>   async_apply(GComputation& gcomp, GRunArgs &&ins, GRunArgsP &&outs, GCompileArgs &&args,  GAsyncContext& ctx);
} // namespace wip
} // namespace gapi
} // namespace cv


#endif //OPENCV_GAPI_GCOMPUTATION_ASYNC_HPP
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
- **GComputation**: A class/struct defined in this file

### Functions and Methods

- **of()**: A function/method defined in this file
- **OPENCV_GAPI_GCOMPUTATION_ASYNC_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `functional`
- `exception`
- `opencv2/gapi/own/exports.hpp`
- `opencv2/gapi/garg.hpp`
- `future`
- `opencv2/gapi/gcommon.hpp`


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

