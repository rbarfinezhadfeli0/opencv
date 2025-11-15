# Documentation for `modules/core/test/test_async.cpp`

## File Metadata

- **Full Path**: `modules/core/test/test_async.cpp`
- **File Name**: `test_async.cpp`
- **File Size**: 3,699 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/test/test_async.cpp](../../../modules/core/test/test_async.cpp)

## Purpose and Role

This file is located in the `modules/core/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
#include "test_precomp.hpp"
#include <opencv2/core/async.hpp>
#include <opencv2/core/detail/async_promise.hpp>

#include <opencv2/core/bindings_utils.hpp>

#if !defined(OPENCV_DISABLE_THREAD_SUPPORT)
#include <thread>
#include <chrono>
#endif

namespace opencv_test { namespace {

TEST(Core_Async, BasicCheck)
{
    Mat m(3, 3, CV_32FC1, Scalar::all(5.0f));
    AsyncPromise p;
    AsyncArray r = p.getArrayResult();
    EXPECT_TRUE(r.valid());

    // Follow the limitations of std::promise::get_future
    // https://en.cppreference.com/w/cpp/thread/promise/get_future
    EXPECT_THROW(AsyncArray r2 = p.getArrayResult(), cv::Exception);

    p.setValue(m);

    Mat m2;
    r.get(m2);
    EXPECT_EQ(0, cvtest::norm(m, m2, NORM_INF));

    // Follow the limitations of std::future::get
    // https://en.cppreference.com/w/cpp/thread/future/get
    EXPECT_FALSE(r.valid());
    Mat m3;
    EXPECT_THROW(r.get(m3), cv::Exception);
}

TEST(Core_Async, ExceptionCheck)
{
    Mat m(3, 3, CV_32FC1, Scalar::all(5.0f));
    AsyncPromise p;
    AsyncArray r = p.getArrayResult();
    EXPECT_TRUE(r.valid());

    try
    {
        CV_Error(Error::StsOk, "Test: Generated async error");
    }
    catch (const cv::Exception& e)
    {
        p.setException(e);
    }

    try {
        Mat m2;
        r.get(m2);
        FAIL() << "Exception is expected";
    }
    catch (const cv::Exception& e)
    {
        EXPECT_EQ(Error::StsOk, e.code) << e.what();
    }

    // Follow the limitations of std::future::get
    // https://en.cppreference.com/w/cpp/thread/future/get
    EXPECT_FALSE(r.valid());
}


TEST(Core_Async, LikePythonTest)
{
    Mat m(3, 3, CV_32FC1, Scalar::all(5.0f));
    AsyncArray r = cv::utils::testAsyncArray(m);
    EXPECT_TRUE(r.valid());
    Mat m2;
    r.get(m2);
    EXPECT_EQ(0, cvtest::norm(m, m2, NORM_INF));

    // Follow the limitations of std::future::get
    // https://en.cppreference.com/w/cpp/thread/future/get
    EXPECT_FALSE(r.valid());
}


#if !defined(OPENCV_DISABLE_THREAD_SUPPORT)

TEST(Core_Async, AsyncThread_Simple)
{
    Mat m(3, 3, CV_32FC1, Scalar::all(5.0f));
    AsyncPromise p;
    AsyncArray r = p.getArrayResult();

    std::thread t([&]{
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        try {
            p.setValue(m);
        } catch (const std::exception& e) {
            std::cout << e.what() << std::endl;
        } catch (...) {
            std::cout << "Unknown C++ exception" << std::endl;
        }
    });

    try
    {
        Mat m2;
        r.get(m2);
        EXPECT_EQ(0, cvtest::norm(m, m2, NORM_INF));

        t.join();
    }
    catch (...)
    {
        t.join();
        throw;
    }
}

TEST(Core_Async, AsyncThread_DetachedResult)
{
    Mat m(3, 3, CV_32FC1, Scalar::all(5.0f));
    AsyncPromise p;
    {
        AsyncArray r = p.getArrayResult();
        r.release();
    }

    bool exception_ok = false;

    std::thread t([&]{
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        try {
            p.setValue(m);
        } catch (const cv::Exception& e) {
            if (e.code == Error::StsError)
                exception_ok = true;
            else
                std::cout << e.what() << std::endl;
        } catch (const std::exception& e) {
            std::cout << e.what() << std::endl;
        } catch (...) {
            std::cout << "Unknown C++ exception" << std::endl;
        }
    });
    t.join();

    EXPECT_TRUE(exception_ok);
}

#endif

}} // namespace
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
- `opencv2/core/async.hpp`
- `test_precomp.hpp`
- `opencv2/core/detail/async_promise.hpp`
- `chrono`
- `thread`
- `opencv2/core/bindings_utils.hpp`


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

