# Documentation for `modules/gapi/test/gpu/gapi_imgproc_tests_gpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gpu/gapi_imgproc_tests_gpu.cpp`
- **File Name**: `gapi_imgproc_tests_gpu.cpp`
- **File Size**: 13,492 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gpu/gapi_imgproc_tests_gpu.cpp](../../../../modules/gapi/test/gpu/gapi_imgproc_tests_gpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2019 Intel Corporation


#include "../test_precomp.hpp"

#include "../common/gapi_imgproc_tests.hpp"

namespace
{
#define IMGPROC_GPU [] () { return cv::compile_args(cv::gapi::use_only{cv::gapi::imgproc::gpu::kernels()}); }
    const std::vector <cv::Size> in_sizes{ cv::Size(1280, 720), cv::Size(128, 128) };
}  // anonymous namespace

namespace opencv_test
{

INSTANTIATE_TEST_CASE_P(ResizeTestGPU, ResizeTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsSimilarPoints(2, 0.05).to_compare_obj()),
                                Values(cv::INTER_NEAREST, cv::INTER_LINEAR, cv::INTER_AREA),
                                Values(cv::Size(64,64),
                                       cv::Size(30,30))));

INSTANTIATE_TEST_CASE_P(ResizeTestGPU, ResizeTestFxFy,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsSimilarPoints(2, 0.05).to_compare_obj()),
                                Values(cv::INTER_NEAREST, cv::INTER_LINEAR, cv::INTER_AREA),
                                Values(0.5, 0.1),
                                Values(0.5, 0.1)));

INSTANTIATE_TEST_CASE_P(Filter2DTestGPU, Filter2DTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1, CV_32F),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 2).to_compare_obj()),
                                Values(cv::Size(3, 3),
                                       cv::Size(4, 4),
                                       cv::Size(5, 5),
                                       cv::Size(7, 7)),
                                Values(cv::BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(BoxFilterTestGPU, BoxFilterTest,
                        Combine(Values(/*CV_8UC1,*/ CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_32F),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 2).to_compare_obj()),
                                Values(3,5),
                                Values(cv::BORDER_DEFAULT)));  //TODO: 8UC1 doesn't work


INSTANTIATE_TEST_CASE_P(SepFilterTestGPU_8U, SepFilterTest,
                        Combine(Values(CV_8UC1, CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_16S, CV_32F),
                                Values(IMGPROC_GPU),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3)));

INSTANTIATE_TEST_CASE_P(SepFilterTestGPU_other, SepFilterTest,
                        Combine(Values(CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_32F),
                                Values(IMGPROC_GPU),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3)));

INSTANTIATE_TEST_CASE_P(BlurTestGPU, BlurTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-4, 2).to_compare_obj()),
                                Values(3,5),
                                Values(cv::BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(gaussBlurTestGPU, GaussianBlurTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(ToleranceFilter(1e-5f, 0.01).to_compare_obj()),
                                Values(3)));  // FIXIT 5

INSTANTIATE_TEST_CASE_P(MedianBlurTestGPU, MedianBlurTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(3, 5)));

INSTANTIATE_TEST_CASE_P(ErodeTestGPU, ErodeTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(3, 5),
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE)));

INSTANTIATE_TEST_CASE_P(Erode3x3TestGPU, Erode3x3Test,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(1,2,4)));

INSTANTIATE_TEST_CASE_P(DilateTestGPU, DilateTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(3, 5),
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE)));

INSTANTIATE_TEST_CASE_P(Dilate3x3TestGPU, Dilate3x3Test,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(1,2,4)));

INSTANTIATE_TEST_CASE_P(SobelTestGPU, SobelTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_16S, CV_32F),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-4, 2).to_compare_obj()),
                                Values(3, 5),
                                Values(0, 1),
                                Values(1, 2)));

INSTANTIATE_TEST_CASE_P(SobelTestGPU32F, SobelTest,
                        Combine(Values(CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(CV_32F),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-4, 2).to_compare_obj()),
                                Values(3, 5),
                                Values(0, 1),
                                Values(1, 2)));

INSTANTIATE_TEST_CASE_P(LaplacianTestGPU, LaplacianTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-4, 2).to_compare_obj()),
                                Values(5),
                                Values(3.0),
                                Values(BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(BilateralFilterTestGPU, BilateralFilterTest,
                        Combine(Values(CV_32FC1, CV_32FC3, CV_8UC1, CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-4, 2).to_compare_obj()),
                                Values(9),
                                Values(100),
                                Values(40),
                                Values(BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(EqHistTestGPU, EqHistTest,
                        Combine(Values(CV_8UC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_GPU),
                                Values(AbsExact().to_compare_obj())));  // FIXIT Non reliable check

INSTANTIATE_TEST_CASE_P(CannyTestGPU, CannyTest,
                        Combine(Values(CV_8UC1, CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC1),
                                Values(IMGPROC_GPU),
                                Values(AbsSimilarPoints(0, 0.05).to_compare_obj()),
                                Values(3.0, 120.0),
                                Values(125.0, 240.0),
                                Values(3, 5),
                                testing::Bool()));

INSTANTIATE_TEST_CASE_P(RGB2GrayTestGPU, RGB2GrayTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC1),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BGR2GrayTestGPU, BGR2GrayTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC1),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2YUVTestGPU, RGB2YUVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(YUV2RGBTestGPU, YUV2RGBTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2LabTestGPU, RGB2LabTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(AbsSimilarPoints(1, 0.05).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BGR2LUVTestGPU, BGR2LUVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(5e-3, 6).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(LUV2BGRTestGPU, LUV2BGRTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BGR2YUVTestGPU, BGR2YUVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(YUV2BGRTestGPU, YUV2BGRTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_GPU),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../common/gapi_imgproc_tests.hpp`
- `../test_precomp.hpp`


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

