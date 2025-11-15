# Documentation for `modules/gapi/test/cpu/gapi_core_tests_fluid.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/cpu/gapi_core_tests_fluid.cpp`
- **File Name**: `gapi_core_tests_fluid.cpp`
- **File Size**: 15,749 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/cpu/gapi_core_tests_fluid.cpp](../../../../modules/gapi/test/cpu/gapi_core_tests_fluid.cpp)

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
#include "../common/gapi_core_tests.hpp"

namespace
{
#define CORE_FLUID [] () { return cv::compile_args(cv::gapi::use_only{cv::gapi::core::fluid::kernels()}); }
    const std::vector <cv::Size> in_sizes{ cv::Size(1280, 720), cv::Size(128, 128) };
}  // anonymous namespace

namespace opencv_test
{
// FIXME: Windows accuracy problems after recent update!
INSTANTIATE_TEST_CASE_P(MathOpTestFluid, MathOpTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1, CV_8U, CV_32F),
                                Values(CORE_FLUID),
                                Values(DIV, MUL),
                                testing::Bool(),
                                Values(1.0),
                                testing::Bool()));

// FIXME: Accuracy test for SUB math operation fails on FullHD and HD CV_16SC1 input cv::Mat,
//        double-presicion input cv::Scalar and CV_32FC1 output cv::Mat on Mac.
//        Accuracy test for ADD math operation fails on HD CV_16SC1 input cv::Mat,
//        double-presicion input cv::Scalar and CV_32FC1 output cv::Mat on Mac.
//        As failures are sporadic, disabling all instantiation cases for SUB and ADD.
//        Github ticket: https://github.com/opencv/opencv/issues/18373.
INSTANTIATE_TEST_CASE_P(DISABLED_MathOpTestFluid, MathOpTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1, CV_8U, CV_32F),
                                Values(CORE_FLUID),
                                Values(ADD, SUB),
                                testing::Bool(),
                                Values(1.0),
                                testing::Bool()));

// FIXME: Accuracy test for SUB math operation fails on CV_16SC1 input cv::Mat, double-presicion
//        input cv::Scalar and CV_32FC1 output cv::Mat on Mac.
//        As failures are sporadic, disabling all instantiation cases for SUB operation.
//        Github ticket: https://github.com/opencv/opencv/issues/18373.
INSTANTIATE_TEST_CASE_P(DISABLED_SubTestFluid, MathOpTest,
                        Combine(Values(CV_8UC1, CV_16SC1 , CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1, CV_8U, CV_32F),
                                Values(CORE_FLUID),
                                Values(SUB),
                                testing::Bool(),
                                Values (1.0),
                                testing::Bool()));

INSTANTIATE_TEST_CASE_P(MulSTestFluid, MulDoubleTest,
                        Combine(Values(CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1), // FIXME: extend with more types
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DivCTestFluid, DivCTest,
                        Combine(Values(CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_8U, CV_32F),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DISABLED_MeanTestFluid, MeanTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(MaskTestFluid, MaskTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(AbsDiffTestFluid, AbsDiffTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(AbsDiffCTestFluid, AbsDiffCTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1, CV_8UC2,
                                       CV_16UC2, CV_16SC2, CV_8UC3, CV_16UC3,
                                       CV_16SC3, CV_8UC4, CV_16UC4, CV_16SC4),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(BitwiseTestFluid, BitwiseTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(AND, OR, XOR),
                                testing::Bool()));

INSTANTIATE_TEST_CASE_P(BitwiseNotTestFluid, NotTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(MinTestFluid, MinTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(MaxTestFluid, MaxTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DISABLED_SumTestFluid, SumTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(AbsToleranceScalar(1e-5).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(CompareTestFluid, CmpTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_8U),
                                Values(CORE_FLUID),
                                Values(CMP_EQ, CMP_GE, CMP_NE, CMP_GT, CMP_LT, CMP_LE),
                                Values(false),
                                Values(AbsExact().to_compare_obj())));

// FIXME: solve comparison error to unite with the test above
INSTANTIATE_TEST_CASE_P(CompareTestFluidScalar, CmpTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_8U),
                                Values(CORE_FLUID),
                                Values(CMP_EQ, CMP_GE, CMP_NE, CMP_GT, CMP_LT, CMP_LE),
                                Values(true),
                                Values(AbsSimilarPoints(1, 0.01).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(AddWeightedTestFluid, AddWeightedTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1, CV_8U, CV_32F),
                                Values(CORE_FLUID),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 2).to_compare_obj())));

INSTANTIATE_TEST_CASE_P(DISABLED_NormTestFluid, NormTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(AbsToleranceScalar(1e-5).to_compare_obj()),
                                Values(NORM_INF, NORM_L1, NORM_L2)));

INSTANTIATE_TEST_CASE_P(DISABLED_IntegralTestFluid, IntegralTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(LUTTestFluid, LUTTest,
                        Combine(Values(CV_8UC1, CV_8UC3),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(ConvertToFluid, ConvertToTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_8U, CV_16U, CV_16S, CV_32F),
                                Values(CORE_FLUID),
                                Values(Tolerance_FloatRel_IntAbs(1e-5, 2).to_compare_obj()),
                                Values(2.5, 1.0, -1.0),
                                Values(250.0, 0.0, -128.0)));

INSTANTIATE_TEST_CASE_P(DISABLED_ConcatHorTestFluid, ConcatHorTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DISABLED_ConcatVertTestFluid, ConcatVertTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Split3TestFluid, Split3Test,
                        Combine(Values(CV_8UC3),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Split4TestFluid, Split4Test,
                        Combine(Values(CV_8UC4),
                                ValuesIn(in_sizes),
                                Values(CV_8UC1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Merge3TestFluid, Merge3Test,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(CV_8UC3),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Merge4TestFluid, Merge4Test,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(CV_8UC4),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DISABLED_RemapTestFluid, RemapTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(DISABLED_FlipTestFluid, FlipTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(0,1,-1)));

INSTANTIATE_TEST_CASE_P(DISABLED_CropTestFluid, CropTest,
                        Combine(Values(CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(cv::Rect(10, 8, 20, 35), cv::Rect(4, 10, 37, 50))));

INSTANTIATE_TEST_CASE_P(SelectTestFluid, SelectTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Polar2CartFluid, Polar2CartTest,
                        Combine(Values(CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_32FC1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(Cart2PolarFluid, Cart2PolarTest,
                        Combine(Values(CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(CV_32FC1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(PhaseFluid, PhaseTest,
                        Combine(Values(CV_32F, CV_32FC3),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
         /* angle_in_degrees */ testing::Bool()));

INSTANTIATE_TEST_CASE_P(SqrtFluid, SqrtTest,
                        Combine(Values(CV_32F, CV_32FC3),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(ThresholdTestFluid, ThresholdTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(cv::THRESH_BINARY, cv::THRESH_BINARY_INV,
                                       cv::THRESH_TRUNC,
                                       cv::THRESH_TOZERO, cv::THRESH_TOZERO_INV),
                                Values(cv::Scalar(0, 0, 0, 0),
                                       cv::Scalar(100, 100, 100, 100),
                                       cv::Scalar(255, 255, 255, 255))));

INSTANTIATE_TEST_CASE_P(DISABLED_ThresholdTestFluid, ThresholdOTTest,
                        Combine(Values(CV_8UC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(cv::THRESH_OTSU, cv::THRESH_TRIANGLE)));

INSTANTIATE_TEST_CASE_P(InRangeTestFluid, InRangeTest,
                        Combine(Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1),
                                ValuesIn(in_sizes),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(BackendOutputAllocationTestFluid, BackendOutputAllocationTest,
                        Combine(Values(CV_8UC3, CV_16SC2, CV_32FC1),
                                Values(cv::Size(50, 50)),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(BackendOutputAllocationLargeSizeWithCorrectSubmatrixTestFluid,
                        BackendOutputAllocationLargeSizeWithCorrectSubmatrixTest,
                        Combine(Values(CV_8UC3, CV_16SC2, CV_32FC1),
                                Values(cv::Size(50, 50)),
                                Values(-1),
                                Values(CORE_FLUID)));

INSTANTIATE_TEST_CASE_P(ReInitOutTestFluid, ReInitOutTest,
                        Combine(Values(CV_8UC3, CV_16SC4, CV_32FC1),
                                Values(cv::Size(640, 480)),
                                Values(-1),
                                Values(CORE_FLUID),
                                Values(cv::Size(640, 400),
                                       cv::Size(10, 480))));
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

