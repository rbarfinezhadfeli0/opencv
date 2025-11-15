# Documentation for `modules/gapi/test/gpu/gapi_core_tests_gpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gpu/gapi_core_tests_gpu.cpp`
- **File Name**: `gapi_core_tests_gpu.cpp`
- **File Size**: 17,456 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gpu/gapi_core_tests_gpu.cpp](../../../../modules/gapi/test/gpu/gapi_core_tests_gpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#include "../test_precomp.hpp"
#include "../common/gapi_core_tests.hpp"

namespace
{
#define CORE_GPU [] () { return cv::compile_args(cv::gapi::use_only{cv::gapi::core::gpu::kernels()}); }
    const std::vector <cv::Size> in_sizes{ cv::Size(1280, 720), cv::Size(128, 128) };
}  // anonymous namespace

namespace opencv_test
{

// FIXME: Wut? See MulTestGPU/MathOpTest below (duplicate?)
INSTANTIATE_TEST_CASE_P(AddTestGPU, MathOpTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU),
                                Values(ADD, MUL),
                                testing::Bool(),
                                Values(1.0),
                                Values(false)));

INSTANTIATE_TEST_CASE_P(MulTestGPU, MathOpTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU),
                                Values(MUL),
                                testing::Bool(),
                                Values(1.0, 0.5, 2.0),
                                Values(false)));

INSTANTIATE_TEST_CASE_P(SubTestGPU, MathOpTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU),
                                Values(SUB),
                                testing::Bool(),
                                Values (1.0),
                                testing::Bool()));

// FIXME: Accuracy test for DIV math operation fails on CV_8UC3 HD input cv::Mat, double-presicion
//        input cv::Scalar and CV_16U output cv::Mat when we also test reverse operation on Mac.
//        Accuracy test for DIV math operation fails on CV_8UC3 VGA input cv::Mat, double-presicion
//        input cv::Scalar and output cv::Mat having the SAME depth as input one when we also test
//        reverse operation on Mac.
//        It is oddly, but test doesn't fail if we have VGA CV_8UC3 input cv::Mat, double-precision
//        input cv::Scalar and output cv::Mat having explicitly specified CV_8U depth when we also
//        test reverse operation on Mac.
//        As failures are sporadic, disabling all instantiation cases for DIV operation.
//        Github ticket: https://github.com/opencv/opencv/issues/18373.
INSTANTIATE_TEST_CASE_P(DISABLED_DivTestGPU, MathOpTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU),
                                Values(DIV),
                                testing::Bool(),
                                Values (1.0, 0.5, 2.0),
                                testing::Bool()));

INSTANTIATE_TEST_CASE_P(MulTestGPU, MulDoubleTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(DivTestGPU, DivTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(DivCTestGPU, DivCTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(MeanTestGPU, MeanTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

//TODO: mask test doesn't work
INSTANTIATE_TEST_CASE_P(DISABLED_MaskTestGPU, MaskTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(SelectTestGPU, SelectTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Polar2CartGPU, Polar2CartTest,
                        Combine(Values(CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_32FC1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Cart2PolarGPU, Cart2PolarTest,
                        Combine(Values(CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_32FC1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(CompareTestGPU, CmpTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(CV_8U),
                                Values(CORE_GPU),
                                Values(CMP_EQ, CMP_GE, CMP_NE, CMP_GT, CMP_LT, CMP_LE),
                                testing::Bool(),
                                Values(AbsExact().to_compare_obj())));

INSTANTIATE_TEST_CASE_P(BitwiseTestGPU, BitwiseTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(AND, OR, XOR),
                                testing::Bool()));

INSTANTIATE_TEST_CASE_P(BitwiseNotTestGPU, NotTest,
                        Combine(Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(DISABLED_MinTestGPU, MinTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(DISABLED_MaxTestGPU, MaxTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(SumTestGPU, SumTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(AbsToleranceScalar(1e-5).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(CountNonZeroTestGPU, CountNonZeroTest,
                        Combine(Values( CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(AbsToleranceScalar(1e-5).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(AbsDiffTestGPU, AbsDiffTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(AbsDiffCTestGPU, AbsDiffCTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(AddWeightedTestGPU, AddWeightedTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values( -1, CV_8U, CV_16U, CV_32F ),
                                Values(CORE_GPU),
                                Values(Tolerance_FloatRel_IntAbs(1e-6, 1).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(NormTestGPU, NormTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(AbsToleranceScalar(1e-3).to_compare_obj()), //TODO: too relaxed?
                                Values(NORM_INF, NORM_L1, NORM_L2)));

INSTANTIATE_TEST_CASE_P(IntegralTestGPU, IntegralTest,
                        Combine(Values( CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(ThresholdTestGPU, ThresholdTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(cv::THRESH_BINARY, cv::THRESH_BINARY_INV, cv::THRESH_TRUNC,
                                       cv::THRESH_TOZERO, cv::THRESH_TOZERO_INV),
                                Values(cv::Scalar(0, 0, 0, 0),
                                       cv::Scalar(100, 100, 100, 100),
                                       cv::Scalar(255, 255, 255, 255))));

INSTANTIATE_TEST_CASE_P(ThresholdTestGPU, ThresholdOTTest,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(cv::THRESH_OTSU, cv::THRESH_TRIANGLE)));


INSTANTIATE_TEST_CASE_P(InRangeTestGPU, InRangeTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Split3TestGPU, Split3Test,
                        Combine(Values(CV_8UC3),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Split4TestGPU, Split4Test,
                        Combine(Values(CV_8UC4),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Merge3TestGPU, Merge3Test,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(CV_8UC3),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(Merge4TestGPU, Merge4Test,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(CV_8UC4),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(RemapTestGPU, RemapTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(FlipTestGPU, FlipTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(0,1,-1)));

INSTANTIATE_TEST_CASE_P(CropTestGPU, CropTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(cv::Rect(10, 8, 20, 35), cv::Rect(4, 10, 37, 50))));

INSTANTIATE_TEST_CASE_P(LUTTestGPU, LUTTest,
                        Combine(Values(CV_8UC1, CV_8UC3),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(LUTTestCustomGPU, LUTTest,
                        Combine(Values(CV_8UC3),
                                ValuesIn(in_sizes),
                                Values(CV_8UC3),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(ConvertToGPU, ConvertToTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(CV_8U, CV_16U, CV_16S, CV_32F),
                                Values(CORE_GPU),
                                Values(AbsExact().to_compare_obj()),
                                Values(2.5, 1.0, -1.0),
                                Values(250.0, 0.0, -128.0)));

INSTANTIATE_TEST_CASE_P(ConcatHorTestGPU, ConcatHorTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(ConcatVertTestGPU, ConcatVertTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(TransposeTestGPU, TransposeTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1,
                                       CV_8UC2, CV_16UC2, CV_16SC2, CV_32FC2,
                                       CV_8UC3, CV_16UC3, CV_16SC3, CV_32FC3),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(AbsExact().to_compare_obj())));
// PLEASE DO NOT PUT NEW ACCURACY TESTS BELOW THIS POINT! //////////////////////

INSTANTIATE_TEST_CASE_P(BackendOutputAllocationTestGPU, BackendOutputAllocationTest,
                        Combine(Values(CV_8UC3, CV_16SC2, CV_32FC1),
                                Values(cv::Size(50, 50)),
                                Values(-1),
                                Values(CORE_GPU)));

// FIXME: there's an issue in OCL backend with matrix reallocation that shouldn't happen
INSTANTIATE_TEST_CASE_P(DISABLED_BackendOutputAllocationLargeSizeWithCorrectSubmatrixTestGPU,
                        BackendOutputAllocationLargeSizeWithCorrectSubmatrixTest,
                        Combine(Values(CV_8UC3, CV_16SC2, CV_32FC1),
                                Values(cv::Size(50, 50)),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(ReInitOutTestGPU, ReInitOutTest,
                        Combine(Values(CV_8UC3, CV_16SC4, CV_32FC1),
                                Values(cv::Size(640, 480)),
                                Values(-1),
                                Values(CORE_GPU),
                                Values(cv::Size(640, 400),
                                       cv::Size(10, 480))));

//TODO: fix this backend to allow ConcatVertVec ConcatHorVec
INSTANTIATE_TEST_CASE_P(DISABLED_ConcatVertVecTestGPU, ConcatVertVecTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));

INSTANTIATE_TEST_CASE_P(DISABLED_ConcatHorVecTestGPU, ConcatHorVecTest,
                        Combine(Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_GPU)));
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
- `../test_precomp.hpp`
- `../common/gapi_core_tests.hpp`


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

