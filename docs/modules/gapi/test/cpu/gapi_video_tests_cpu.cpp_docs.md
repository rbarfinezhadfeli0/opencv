# Documentation for `modules/gapi/test/cpu/gapi_video_tests_cpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/cpu/gapi_video_tests_cpu.cpp`
- **File Name**: `gapi_video_tests_cpu.cpp`
- **File Size**: 7,189 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/cpu/gapi_video_tests_cpu.cpp](../../../../modules/gapi/test/cpu/gapi_video_tests_cpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/cpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation


#include "../test_precomp.hpp"

#include "../common/gapi_video_tests.hpp"
#include <opencv2/gapi/cpu/video.hpp>

namespace
{
#define VIDEO_CPU [] () { return cv::compile_args(cv::gapi::video::cpu::kernels()); }

#ifdef HAVE_OPENCV_VIDEO
#define WITH_VIDEO(X) X
#else
#define WITH_VIDEO(X) DISABLED_##X
#endif // HAVE_OPENCV_VIDEO

#define INSTANTIATE_TEST_CASE_MACRO_P(prefix, test_case_name, generator, ...) \
    INSTANTIATE_TEST_CASE_P(prefix, test_case_name, generator, __VA_ARGS__)
}  // anonymous namespace

namespace opencv_test
{
INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildOptFlowPyramidTestCPU), BuildOptFlowPyramidTest,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_1.bmp",
                                             "cv/optflow/frames/1080p_01.png"),
                                      Values(7, 11),
                                      Values(1000),
                                      testing::Bool(),
                                      Values(BORDER_DEFAULT, BORDER_TRANSPARENT),
                                      Values(BORDER_DEFAULT, BORDER_TRANSPARENT),
                                      testing::Bool()));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildOptFlowPyramidInternalTestCPU),
                              BuildOptFlowPyramidTest,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_1.bmp"),
                                      Values(15),
                                      Values(3),
                                      Values(true),
                                      Values(BORDER_REFLECT_101),
                                      Values(BORDER_CONSTANT),
                                      Values(true)));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKTestCPU), OptFlowLKTest,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_%01d.bmp",
                                             "cv/optflow/frames/1080p_%02d.png"),
                                      Values(1, 3, 4),
                                      Values(std::make_tuple(9, 9), std::make_tuple(15, 15)),
                                      Values(7, 11),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              30, 0.01))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKTestForPyrCPU), OptFlowLKTestForPyr,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_%01d.bmp",
                                             "cv/optflow/frames/1080p_%02d.png"),
                                      Values(1, 3, 4),
                                      Values(std::make_tuple(9, 9), std::make_tuple(15, 15)),
                                      Values(7, 11),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              30, 0.01)),
                                      testing::Bool()));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKInternalTestCPU), OptFlowLKTestForPyr,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_%01d.bmp"),
                                      Values(1),
                                      Values(std::make_tuple(10, 10)),
                                      Values(15),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              21, 0.05)),
                                      Values(true)));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildPyr_CalcOptFlow_PipelineTestCPU),
                              BuildPyr_CalcOptFlow_PipelineTest,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/frames/1080p_%02d.png"),
                                      Values(7, 11),
                                      Values(1000),
                                      testing::Bool()));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildPyr_CalcOptFlow_PipelineInternalTestCPU),
                              BuildPyr_CalcOptFlow_PipelineTest,
                              Combine(Values(VIDEO_CPU),
                                      Values("cv/optflow/rock_%01d.bmp"),
                                      Values(15),
                                      Values(3),
                                      Values(true)));


INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BackgroundSubtractorTestCPU),
                              BackgroundSubtractorTest,
                              Combine(Values(VIDEO_CPU),
                                      Values(std::make_tuple(cv::gapi::video::TYPE_BS_MOG2, 16),
                                             std::make_tuple(cv::gapi::video::TYPE_BS_MOG2, 8),
                                             std::make_tuple(cv::gapi::video::TYPE_BS_KNN, 400),
                                             std::make_tuple(cv::gapi::video::TYPE_BS_KNN, 200)),
                                             Values(500, 50),
                                             testing::Bool(),
                                             Values(-1, 0, 0.5, 1),
                                             Values("cv/video/768x576.avi"),
                                             Values(3)));

INSTANTIATE_TEST_CASE_MACRO_P(KalmanFilterTestCPU,
                              KalmanFilterTest,
                              Combine(Values(VIDEO_CPU),
                                      Values(CV_32FC1, CV_64FC1),
                                      Values(2,5),
                                      Values(2,5),
                                      Values(2),
                                      Values(5)));

INSTANTIATE_TEST_CASE_MACRO_P(KalmanFilterTestCPU,
                              KalmanFilterNoControlTest,
                              Combine(Values(VIDEO_CPU),
                                      Values(CV_32FC1, CV_64FC1),
                                      Values(3),
                                      Values(4),
                                      Values(3)));

INSTANTIATE_TEST_CASE_MACRO_P(KalmanFilterTestCPU,
                              KalmanFilterCircleSampleTest,
                              Combine(Values(VIDEO_CPU),
                                      Values(CV_32FC1, CV_64FC1),
                                      Values(5)));

} // opencv_test
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

### Functions and Methods

- **HAVE_OPENCV_VIDEO()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../test_precomp.hpp`
- `opencv2/gapi/cpu/video.hpp`
- `../common/gapi_video_tests.hpp`


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

