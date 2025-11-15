# Documentation for `modules/gapi/test/cpu/gapi_imgproc_tests_fluid.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/cpu/gapi_imgproc_tests_fluid.cpp`
- **File Name**: `gapi_imgproc_tests_fluid.cpp`
- **File Size**: 11,717 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/cpu/gapi_imgproc_tests_fluid.cpp](../../../../modules/gapi/test/cpu/gapi_imgproc_tests_fluid.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/cpu` directory and serves as part of the OpenCV library infrastructure.

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
#define IMGPROC_FLUID [] () { return cv::compile_args(cv::gapi::use_only{cv::gapi::imgproc::fluid::kernels()}); }
}  // anonymous namespace

namespace opencv_test
{

INSTANTIATE_TEST_CASE_P(ResizeTestFluid, ResizeTest,
                        Combine(Values(CV_8UC3/*CV_8UC1, CV_16UC1, CV_16SC1*/),
                                Values(cv::Size(1280, 720),
                                       cv::Size(30, 30)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_obj()),
                                Values(/*cv::INTER_NEAREST,*/ cv::INTER_LINEAR/*, cv::INTER_AREA*/),
                                Values(cv::Size(1280, 720),
                                       cv::Size(30, 30))));

INSTANTIATE_TEST_CASE_P(ResizeTestFxFyFluid, ResizeTestFxFy,
                        Combine(Values(CV_8UC3/*CV_8UC1, CV_16UC1, CV_16SC1*/),
                                Values(cv::Size(1280, 720),
                                       cv::Size(30, 30)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_obj()),
                                Values(/*cv::INTER_NEAREST,*/ cv::INTER_LINEAR/*, cv::INTER_AREA*/),
                                Values(0.5, 1, 2),
                                Values(0.5, 1, 2)));

INSTANTIATE_TEST_CASE_P(RGB2GrayTestFluid, RGB2GrayTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC1),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BGR2GrayTestFluid, BGR2GrayTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC1),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2YUVTestFluid, RGB2YUVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(YUV2RGBTestFluid, YUV2RGBTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2LabTestFluid, RGB2LabTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(AbsSimilarPoints(1, 0.05).to_compare_obj())));

// FIXME: Not supported by Fluid yet (no kernel implemented)
INSTANTIATE_TEST_CASE_P(BGR2LUVTestFluid, BGR2LUVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(5e-3, 6).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2HSVTestFluid, RGB2HSVTest,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BayerGR2RGBTestFluid, BayerGR2RGBTest,
                        Combine(Values(CV_8UC1),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC3),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceColor(1e-3).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(RGB2YUV422TestFluid, RGB2YUV422Test,
                        Combine(Values(CV_8UC3),
                                Values(cv::Size(1280, 720)),
                                Values(CV_8UC2),
                                Values(IMGPROC_FLUID),
                                Values(AbsTolerance(1).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(blurTestFluid, BlurTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(cv::BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(gaussBlurTestFluid, GaussianBlurTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-3f, 0.01).to_compare_obj()),
                                Values(3, 5)));

INSTANTIATE_TEST_CASE_P(medianBlurTestFluid, MedianBlurTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(AbsExact().to_compare_obj()),
                                Values(3))); // add kernel size=5 when implementation is ready

INSTANTIATE_TEST_CASE_P(erodeTestFluid, ErodeTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(AbsExact().to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE)));

INSTANTIATE_TEST_CASE_P(dilateTestFluid, DilateTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1),
                                Values(IMGPROC_FLUID),
                                Values(AbsExact().to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(cv::MorphShapes::MORPH_RECT,
                                       cv::MorphShapes::MORPH_CROSS,
                                       cv::MorphShapes::MORPH_ELLIPSE)));

INSTANTIATE_TEST_CASE_P(SobelTestFluid, SobelTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_16S, CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(AbsExact().to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(0, 1),
                                Values(1, 2)));

INSTANTIATE_TEST_CASE_P(SobelTestFluid32F, SobelTest,
                        Combine(Values(CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(0, 1),
                                Values(1, 2)));

INSTANTIATE_TEST_CASE_P(SobelXYTestFluid, SobelXYTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_16S, CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(AbsExact().to_compare_obj()),
                                Values(3),
                                Values(1, 2),
                                Values(BORDER_CONSTANT, BORDER_REPLICATE, BORDER_REFLECT_101),
                                Values(0, 1, 255)));

INSTANTIATE_TEST_CASE_P(SobelXYTestFluid32F, SobelXYTest,
                        Combine(Values(CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3),
                                Values(1, 2),
                                Values(BORDER_CONSTANT, BORDER_REPLICATE, BORDER_REFLECT_101),
                                Values(0, 1, 255)));

INSTANTIATE_TEST_CASE_P(boxFilterTestFluid32, BoxFilterTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3), // add kernel size=5 when implementation is ready
                                Values(cv::BORDER_DEFAULT)));

INSTANTIATE_TEST_CASE_P(sepFilterTestFluid, SepFilterTest,
                        Combine(Values(CV_32FC1),
                                Values(cv::Size(1280, 720)),
                                Values(-1, CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(3))); // add kernel size=5 when implementation is ready

INSTANTIATE_TEST_CASE_P(filter2DTestFluid, Filter2DTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                Values(cv::Size(1280, 720),
                                       cv::Size(128, 128)),
                                Values(-1, CV_32F),
                                Values(IMGPROC_FLUID),
                                Values(ToleranceFilter(1e-4f, 0.01).to_compare_obj()),
                                Values(cv::Size(3, 3)), // add kernel size=4x4,5x5,7x7 when implementation ready
                                Values(cv::BORDER_DEFAULT)));

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

