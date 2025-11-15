# Documentation for `modules/gapi/perf/gpu/gapi_imgproc_perf_tests_gpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/gpu/gapi_imgproc_perf_tests_gpu.cpp`
- **File Name**: `gapi_imgproc_perf_tests_gpu.cpp`
- **File Size**: 12,021 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/perf/gpu/gapi_imgproc_perf_tests_gpu.cpp](../../../../modules/gapi/perf/gpu/gapi_imgproc_perf_tests_gpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "../perf_precomp.hpp"
#include "../common/gapi_imgproc_perf_tests.hpp"

#define IMGPROC_GPU cv::gapi::imgproc::gpu::kernels()

namespace opencv_test
{


INSTANTIATE_TEST_CASE_P(SepFilterPerfTestGPU_8U, SepFilterPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3),
                                Values(3),
                                Values(szVGA, sz720p, sz1080p),
                                Values(-1, CV_16S, CV_32F),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(SepFilterPerfTestGPU_other, SepFilterPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3),
                                Values(szVGA, sz720p, sz1080p),
                                Values(-1, CV_32F),
                                Values(cv::compile_args(IMGPROC_GPU))));



INSTANTIATE_TEST_CASE_P(Filter2DPerfTestGPU, Filter2DPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3, 4, 5, 7),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::BORDER_DEFAULT),
                                Values(-1, CV_32F),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(BoxFilterPerfTestGPU, BoxFilterPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(/*CV_8UC1,*/ CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3,5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::BORDER_DEFAULT),
                                Values(-1, CV_32F),
                                Values(cv::compile_args(IMGPROC_GPU)))); //TODO: 8UC1 doesn't work

INSTANTIATE_TEST_CASE_P(BlurPerfTestGPU, BlurPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::BORDER_DEFAULT),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(GaussianBlurPerfTestGPU, GaussianBlurPerfTest,
                        Combine(Values(AbsSimilarPoints(1, 0.05).to_compare_f()), //TODO: too relaxed?
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(MedianBlurPerfTestGPU, MedianBlurPerfTest,
                         Combine(Values(AbsExact().to_compare_f()),
                                 Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                 Values(3, 5),
                                 Values(szVGA, sz720p, sz1080p),
                                 Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(ErodePerfTestGPU, ErodePerfTest,
                        Combine(Values(AbsExact().to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(Erode3x3PerfTestGPU, Erode3x3PerfTest,
                        Combine(Values(AbsExact().to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(szVGA, sz720p, sz1080p),
                                Values(1,2,4),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(DilatePerfTestGPU, DilatePerfTest,
                        Combine(Values(AbsExact().to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(Dilate3x3PerfTestGPU, Dilate3x3PerfTest,
                        Combine(Values(AbsExact().to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(szVGA, sz720p, sz1080p),
                                Values(1,2,4),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(SobelPerfTestGPU, SobelPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(-1, CV_16S, CV_32F),
                                Values(0, 1),
                                Values(1, 2),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(SobelPerfTestGPU32F, SobelPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_32FC1),
                                Values(3, 5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(CV_32F),
                                Values(0, 1),
                                Values(1, 2),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(LaplacianPerfTestGPU, LaplacianPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3),
                                Values(5),
                                Values(szVGA, sz720p, sz1080p),
                                Values(-1),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(BilateralFilterPerfTestGPU, BilateralFilterPerfTest,
                        Combine(Values(ToleranceFilter(1e-4f, 0.01).to_compare_f()),
                                Values(CV_32FC1, CV_32FC3),
                                Values(-1),
                                Values(szVGA, sz720p, sz1080p),
                                Values(5),
                                Values(100),
                                Values(40),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(CannyPerfTestGPU, CannyPerfTest,
                        Combine(Values(AbsSimilarPoints(1, 0.05).to_compare_f()),
                                Values(CV_8UC1, CV_8UC3),
                                Values(szVGA, sz720p, sz1080p),
                                Values(3.0, 120.0),
                                Values(125.0, 240.0),
                                Values(3, 5),
                                Values(true, false),
                                Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(EqHistPerfTestGPU, EqHistPerfTest,
                        Combine(Values(AbsExact().to_compare_f()),  // FIXIT unrealiable check
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(RGB2GrayPerfTestGPU, RGB2GrayPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(BGR2GrayPerfTestGPU, BGR2GrayPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(RGB2YUVPerfTestGPU, RGB2YUVPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(YUV2RGBPerfTestGPU, YUV2RGBPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(RGB2LabPerfTestGPU, RGB2LabPerfTest,
                        Combine(Values(AbsSimilarPoints(1, 0.05).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(BGR2LUVPerfTestGPU, BGR2LUVPerfTest,
                        Combine(Values(AbsSimilarPoints(1, 0.05).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(LUV2BGRPerfTestGPU, LUV2BGRPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(BGR2YUVPerfTestGPU, BGR2YUVPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(YUV2BGRPerfTestGPU, YUV2BGRPerfTest,
                        Combine(Values(ToleranceColor(1e-3).to_compare_f()),
                        Values(szVGA, sz720p, sz1080p),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(ResizePerfTestGPU, ResizePerfTest,
                        Combine(Values(AbsSimilarPoints(2, 0.05).to_compare_f()),
                        Values(CV_8UC1, CV_16UC1, CV_16SC1),
                        Values(cv::INTER_NEAREST, cv::INTER_LINEAR, cv::INTER_AREA),
                        Values( szSmall128, szVGA, sz720p, sz1080p ),
                        Values(cv::Size(64,64),
                               cv::Size(30,30)),
                        Values(cv::compile_args(IMGPROC_GPU))));

INSTANTIATE_TEST_CASE_P(ResizeFxFyPerfTestGPU, ResizeFxFyPerfTest,
                        Combine(Values(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()),
                        Values(CV_8UC1, CV_16UC1, CV_16SC1),
                        Values(cv::INTER_NEAREST, cv::INTER_LINEAR, cv::INTER_AREA),
                        Values(szSmall128, szVGA, sz720p, sz1080p),
                        Values(0.5, 0.1),
                        Values(0.5, 0.1),
                        Values(cv::compile_args(IMGPROC_GPU))));
}
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
- `../common/gapi_imgproc_perf_tests.hpp`
- `../perf_precomp.hpp`


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

