# Documentation for `modules/core/include/opencv2/core/async.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/async.hpp`
- **File Name**: `async.hpp`
- **File Size**: 2,773 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/async.hpp](../../../../../modules/core/include/opencv2/core/async.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_CORE_ASYNC_HPP
#define OPENCV_CORE_ASYNC_HPP

#include <opencv2/core/mat.hpp>

//#include <future>
#include <chrono>

namespace cv {

/** @addtogroup core_async

@{
*/


/** @brief Returns result of asynchronous operations

Object has attached asynchronous state.
Assignment operator doesn't clone asynchronous state (it is shared between all instances).

Result can be fetched via get() method only once.

*/
class CV_EXPORTS_W AsyncArray
{
public:
    ~AsyncArray() CV_NOEXCEPT;
    CV_WRAP AsyncArray() CV_NOEXCEPT;
    AsyncArray(const AsyncArray& o) CV_NOEXCEPT;
    AsyncArray& operator=(const AsyncArray& o) CV_NOEXCEPT;
    CV_WRAP void release() CV_NOEXCEPT;

    /** Fetch the result.
    @param[out] dst destination array

    Waits for result until container has valid result.
    Throws exception if exception was stored as a result.

    Throws exception on invalid container state.

    @note Result or stored exception can be fetched only once.
    */
    CV_WRAP void get(OutputArray dst) const;

    /** Retrieving the result with timeout
    @param[out] dst destination array
    @param[in] timeoutNs timeout in nanoseconds, -1 for infinite wait

    @returns true if result is ready, false if the timeout has expired

    @note Result or stored exception can be fetched only once.
    */
    bool get(OutputArray dst, int64 timeoutNs) const;

    CV_WRAP inline
    bool get(OutputArray dst, double timeoutNs) const { return get(dst, (int64)timeoutNs); }

    bool wait_for(int64 timeoutNs) const;

    CV_WRAP inline
    bool wait_for(double timeoutNs) const { return wait_for((int64)timeoutNs); }

    CV_WRAP bool valid() const CV_NOEXCEPT;

    inline AsyncArray(AsyncArray&& o) { p = o.p; o.p = NULL; }
    inline AsyncArray& operator=(AsyncArray&& o) CV_NOEXCEPT { std::swap(p, o.p); return *this; }

    template<typename _Rep, typename _Period>
    inline bool get(OutputArray dst, const std::chrono::duration<_Rep, _Period>& timeout)
    {
        return get(dst, (int64)(std::chrono::nanoseconds(timeout).count()));
    }

    template<typename _Rep, typename _Period>
    inline bool wait_for(const std::chrono::duration<_Rep, _Period>& timeout)
    {
        return wait_for((int64)(std::chrono::nanoseconds(timeout).count()));
    }

#if 0
    std::future<Mat> getFutureMat() const;
    std::future<UMat> getFutureUMat() const;
#endif


    // PImpl
    struct Impl; friend struct Impl;
    inline void* _getImpl() const CV_NOEXCEPT { return p; }
protected:
    Impl* p;
};


//! @}
} // namespace
#endif // OPENCV_CORE_ASYNC_HPP
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

- **CV_EXPORTS_W**: A class/struct defined in this file
- **Impl**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_CORE_ASYNC_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `chrono`
- `opencv2/core/mat.hpp`
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

