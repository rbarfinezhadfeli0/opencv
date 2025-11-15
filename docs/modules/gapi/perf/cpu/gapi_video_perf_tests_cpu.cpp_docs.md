# Documentation for `modules/gapi/perf/cpu/gapi_video_perf_tests_cpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/cpu/gapi_video_perf_tests_cpu.cpp`
- **File Name**: `gapi_video_perf_tests_cpu.cpp`
- **File Size**: 7,069 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/perf/cpu/gapi_video_perf_tests_cpu.cpp](../../../../modules/gapi/perf/cpu/gapi_video_perf_tests_cpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/cpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation


#include "../perf_precomp.hpp"

#include "../common/gapi_video_perf_tests.hpp"
#include <opencv2/gapi/cpu/video.hpp>

namespace
{
#define VIDEO_CPU cv::gapi::video::cpu::kernels()

#ifdef HAVE_OPENCV_VIDEO
#define WITH_VIDEO(X) X
#else
#define WITH_VIDEO(X) DISABLED_##X
#endif // HAVE_OPENCV_VIDEO

#define INSTANTIATE_TEST_CASE_MACRO_P(prefix, test_case_name, generator, ...) \
    INSTANTIATE_TEST_CASE_P(prefix, test_case_name, generator, __VA_ARGS__)
} // namespace


namespace opencv_test
{
INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildOptFlowPyramidPerfTestCPU),
                              BuildOptFlowPyramidPerfTest,
                              Combine(Values("cv/optflow/rock_1.bmp",
                                             "cv/optflow/frames/1080p_01.png"),
                                      Values(7, 11),
                                      Values(1000),
                                      testing::Bool(),
                                      Values(BORDER_DEFAULT, BORDER_TRANSPARENT),
                                      Values(BORDER_DEFAULT, BORDER_TRANSPARENT),
                                      testing::Bool(),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildOptFlowPyramidInternalPerfTestCPU),
                              BuildOptFlowPyramidPerfTest,
                              Combine(Values("cv/optflow/rock_1.bmp"),
                                      Values(15),
                                      Values(3),
                                      Values(true),
                                      Values(BORDER_REFLECT_101),
                                      Values(BORDER_CONSTANT),
                                      Values(true),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKPerfTestCPU), OptFlowLKPerfTest,
                              Combine(Values("cv/optflow/rock_%01d.bmp",
                                             "cv/optflow/frames/1080p_%02d.png"),
                                      Values(1, 3, 4),
                                      Values(std::make_tuple(9, 9), std::make_tuple(15, 15)),
                                      Values(7, 11),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              30, 0.01)),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKForPyrPerfTestCPU), OptFlowLKForPyrPerfTest,
                              Combine(Values("cv/optflow/rock_%01d.bmp",
                                             "cv/optflow/frames/1080p_%02d.png"),
                                      Values(1, 3, 4),
                                      Values(std::make_tuple(9, 9), std::make_tuple(15, 15)),
                                      Values(7, 11),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              30, 0.01)),
                                      testing::Bool(),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(OptFlowLKInternalPerfTestCPU),
                              OptFlowLKForPyrPerfTest,
                              Combine(Values("cv/optflow/rock_%01d.bmp"),
                                      Values(1),
                                      Values(std::make_tuple(10, 10)),
                                      Values(15),
                                      Values(cv::TermCriteria(cv::TermCriteria::COUNT |
                                                              cv::TermCriteria::EPS,
                                                              21, 0.05)),
                                      Values(true),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildPyr_CalcOptFlow_PipelinePerfTestCPU),
                              BuildPyr_CalcOptFlow_PipelinePerfTest,
                              Combine(Values("cv/optflow/frames/1080p_%02d.png"),
                                      Values(7, 11),
                                      Values(1000),
                                      testing::Bool(),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BuildPyr_CalcOptFlow_PipelineInternalTestPerfCPU),
                              BuildPyr_CalcOptFlow_PipelinePerfTest,
                              Combine(Values("cv/optflow/rock_%01d.bmp"),
                                      Values(15),
                                      Values(3),
                                      Values(true),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(BackgroundSubtractorPerfTestCPU),
                              BackgroundSubtractorPerfTest,
                              Combine(Values(cv::gapi::video::TYPE_BS_MOG2,
                                             cv::gapi::video::TYPE_BS_KNN),
                                      Values("cv/video/768x576.avi", "cv/video/1920x1080.avi"),
                                      testing::Bool(),
                                      Values(0., 0.5, 1.),
                                      Values(5),
                                      Values(cv::compile_args(VIDEO_CPU)),
                                      Values(AbsExact().to_compare_obj())));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(KalmanFilterControlPerfTestCPU),
                              KalmanFilterControlPerfTest,
                              Combine(Values(CV_32FC1, CV_64FC1),
                                      Values(2, 5),
                                      Values(2, 5),
                                      Values(5),
                                      testing::Bool(),
                                      Values(cv::compile_args(VIDEO_CPU))));

INSTANTIATE_TEST_CASE_MACRO_P(WITH_VIDEO(KalmanFilterNoControlPerfTestCPU),
                              KalmanFilterNoControlPerfTest,
                              Combine(Values(CV_32FC1, CV_64FC1),
                                      Values(2, 5),
                                      Values(2, 5),
                                      Values(5),
                                      testing::Bool(),
                                      Values(cv::compile_args(VIDEO_CPU))));
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
- `../common/gapi_video_perf_tests.hpp`
- `../perf_precomp.hpp`
- `opencv2/gapi/cpu/video.hpp`


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

