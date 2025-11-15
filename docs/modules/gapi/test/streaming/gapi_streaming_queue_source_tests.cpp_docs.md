# Documentation for `modules/gapi/test/streaming/gapi_streaming_queue_source_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/streaming/gapi_streaming_queue_source_tests.cpp`
- **File Name**: `gapi_streaming_queue_source_tests.cpp`
- **File Size**: 4,317 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/streaming/gapi_streaming_queue_source_tests.cpp](../../../../modules/gapi/test/streaming/gapi_streaming_queue_source_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2023 Intel Corporation


#include "../test_precomp.hpp"

#include <opencv2/gapi/gstreaming.hpp>
#include <opencv2/gapi/streaming/queue_source.hpp>
#include <opencv2/gapi/streaming/cap.hpp>

namespace opencv_test
{

TEST(GAPI_Streaming_Queue_Source, SmokeTest) {
    // This is more like an example on G-API Queue Source

    cv::GMat in;
    cv::GMat out = in + 1;
    cv::GStreamingCompiled comp = cv::GComputation(in, out).compileStreaming();

    // Queue source needs to know format information to maintain contracts
    auto src = std::make_shared<cv::gapi::wip::QueueSource<cv::Mat> >
        (cv::GMatDesc{CV_8U, 1, cv::Size{128, 128}});

    comp.setSource(cv::gin(src->ptr()));
    comp.start();

    // It is perfectly legal to start a pipeline at this point - the source was passed.
    // Now we can push data through the source and get the pipeline results.

    cv::Mat eye = cv::Mat::eye(cv::Size{128, 128}, CV_8UC1);
    src->push(eye);    // Push I (identity matrix)
    src->push(eye*2);  // Push I*2

    // Now its time to pop. The data could be already processed at this point.
    // Note the queue source queues are unbounded to avoid deadlocks

    cv::Mat result;
    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(eye + 1, result, NORM_INF));

    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(eye*2 + 1, result, NORM_INF));
}

TEST(GAPI_Streaming_Queue_Source, Mixed) {
    // Mixing a regular "live" source (which runs on its own) with a
    // manually controlled queue source may make a little sense, but
    // is perfectly legal and possible.

    cv::GMat in1;
    cv::GMat in2;
    cv::GMat out = in2 - in1;
    cv::GStreamingCompiled comp = cv::GComputation(in1, in2, out).compileStreaming();

    // Queue source needs to know format information to maintain contracts
    auto src1 = std::make_shared<cv::gapi::wip::QueueSource<cv::Mat> >
        (cv::GMatDesc{CV_8U, 3, cv::Size{768, 576}});

    std::shared_ptr<cv::gapi::wip::IStreamSource> src2;
    auto path = findDataFile("cv/video/768x576.avi");
    try {
        src2 = cv::gapi::wip::make_src<cv::gapi::wip::GCaptureSource>(path);
    } catch(...) {
        throw SkipTestException("Video file can not be opened");
    }

    comp.setSource(cv::gin(src1->ptr(), src2)); // FIXME: quite inconsistent
    comp.start();

    cv::Mat eye = cv::Mat::eye(cv::Size{768, 576}, CV_8UC3);
    src1->push(eye);    // Push I (identity matrix)
    src1->push(eye);    // Push I (again)

    cv::Mat ref, result;
    cv::VideoCapture cap(path);

    cap >> ref;
    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(ref - eye, result, NORM_INF));

    cap >> ref;
    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(ref - eye, result, NORM_INF));
}

TEST(GAPI_Streaming_Queue_Input, SmokeTest) {

    // Queue Input: a tiny wrapper atop of multiple queue sources.
    // Allows users to pass all input data at once.

    cv::GMat in1;
    cv::GScalar in2;
    cv::GMat out = in1 + in2;
    cv::GStreamingCompiled comp = cv::GComputation(cv::GIn(in1, in2), cv::GOut(out))
        .compileStreaming();

    // FIXME: This API is too raw
    cv::gapi::wip::QueueInput input({
            cv::GMetaArg{ cv::GMatDesc{CV_8U, 1, cv::Size{64,64} } },
            cv::GMetaArg{ cv::empty_scalar_desc() }
        });
    comp.setSource(input); // Implicit conversion allows it to be passed as-is.
    comp.start();

    // Push data via queue input
    cv::Mat eye = cv::Mat::eye(cv::Size{64, 64}, CV_8UC1);
    input.push(cv::gin(eye, cv::Scalar(1)));
    input.push(cv::gin(eye, cv::Scalar(2)));
    input.push(cv::gin(eye, cv::Scalar(3)));

    // Pop data and validate
    cv::Mat result;
    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(eye+1, result, NORM_INF));

    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(eye+2, result, NORM_INF));

    ASSERT_TRUE(comp.pull(cv::gout(result)));
    EXPECT_EQ(0, cvtest::norm(eye+3, result, NORM_INF));
}

} // namespace opencv_test
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
- `../test_precomp.hpp`
- `opencv2/gapi/gstreaming.hpp`
- `opencv2/gapi/streaming/cap.hpp`
- `opencv2/gapi/streaming/queue_source.hpp`


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

