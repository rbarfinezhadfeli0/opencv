# Documentation for `modules/gapi/perf/gpu/gapi_core_perf_tests_gpu.cpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/gpu/gapi_core_perf_tests_gpu.cpp`
- **File Name**: `gapi_core_perf_tests_gpu.cpp`
- **File Size**: 15,091 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/perf/gpu/gapi_core_perf_tests_gpu.cpp](../../../../modules/gapi/perf/gpu/gapi_core_perf_tests_gpu.cpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#include "../perf_precomp.hpp"
#include "../common/gapi_core_perf_tests.hpp"

#define CORE_GPU cv::gapi::core::gpu::kernels()

namespace opencv_test
{

INSTANTIATE_TEST_CASE_P(AddPerfTestGPU, AddPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(AddCPerfTestGPU, AddCPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(SubPerfTestGPU, SubPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(SubCPerfTestGPU, SubCPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(SubRCPerfTestGPU, SubRCPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MulPerfTestGPU, MulPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(2.0),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MulDoublePerfTestGPU, MulDoublePerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MulCPerfTestGPU, MulCPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(DivPerfTestGPU, DivPerfTest,
    Combine(Values(AbsTolerance(2).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(2.3),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(DivCPerfTestGPU, DivCPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-6, 1).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(1.0),
            Values(cv::compile_args(CORE_GPU))));

// FIXIT: CV_16SC1 test case doesn't work with OpenCL [3031]
INSTANTIATE_TEST_CASE_P(DivRCPerfTestGPU, DivRCPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-6, 1).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, /*CV_16SC1,*/ CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(1.0),
            Values(cv::compile_args(CORE_GPU))));

// FIXIT: mask test on GPU doesn't work [3032]
INSTANTIATE_TEST_CASE_P(DISABLED_MaskPerfTestGPU, MaskPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_16UC1, CV_16SC1),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MeanPerfTestGPU, MeanPerfTest,
    Combine(Values(AbsToleranceScalar(0.0).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Polar2CartPerfTestGPU, Polar2CartPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-5, 2).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Cart2PolarPerfTestGPU, Cart2PolarPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-2, 2).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(CmpPerfTestGPU, CmpPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(CMP_EQ, CMP_GE, CMP_NE, CMP_GT, CMP_LT, CMP_LE),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(CmpWithScalarPerfTestGPU, CmpWithScalarPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(CMP_EQ, CMP_GE, CMP_NE, CMP_GT, CMP_LT, CMP_LE),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(BitwisePerfTestGPU, BitwisePerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(AND, OR, XOR),
            testing::Bool(),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(BitwiseNotPerfTestGPU, BitwiseNotPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(SelectPerfTestGPU, SelectPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MinPerfTestGPU, MinPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(MaxPerfTestGPU, MaxPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(AbsDiffPerfTestGPU, AbsDiffPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(AbsDiffCPerfTestGPU, AbsDiffCPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(SumPerfTestGPU, SumPerfTest,
    Combine(Values(AbsToleranceScalar(1e-5).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(CountNonZeroPerfTestGPU, CountNonZeroPerfTest,
    Combine(Values(AbsToleranceScalar(0.0).to_compare_f()),
            Values(szSmall128, szVGA, sz720p, sz1080p),
            Values(CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(AddWeightedPerfTestGPU, AddWeightedPerfTest,
    Combine(Values(Tolerance_FloatRel_IntAbs(1e-6, 1).to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values( -1, CV_8U, CV_16U, CV_32F ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(NormPerfTestGPU, NormPerfTest,
    Combine(Values(AbsToleranceScalar(1e-5).to_compare_f()),
            Values(NORM_INF, NORM_L1, NORM_L2),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(IntegralPerfTestGPU, IntegralPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ThresholdPerfTestGPU, ThresholdPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::THRESH_BINARY, cv::THRESH_BINARY_INV,
                   cv::THRESH_TRUNC, cv::THRESH_TOZERO, cv::THRESH_TOZERO_INV),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ThresholdPerfTestGPU, ThresholdOTPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1 ),
            Values(cv::THRESH_OTSU, cv::THRESH_TRIANGLE),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(InRangePerfTestGPU, InRangePerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Split3PerfTestGPU, Split3PerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Split4PerfTestGPU, Split4PerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Merge3PerfTestGPU, Merge3PerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(CV_8U),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(Merge4PerfTestGPU, Merge4PerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(RemapPerfTestGPU, RemapPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(FlipPerfTestGPU, FlipPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(0,1,-1),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(CropPerfTestGPU, CropPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::Rect(10, 8, 20, 35), cv::Rect(4, 10, 37, 50)),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ConcatHorPerfTestGPU, ConcatHorPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ConcatVertPerfTestGPU, ConcatVertPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ConcatHorVecPerfTestGPU, ConcatHorVecPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(szSmall128, szVGA, sz720p, sz1080p),
            Values(CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ConcatVertVecPerfTestGPU, ConcatVertVecPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values( CV_8UC1, CV_8UC3, CV_16UC1, CV_16SC1, CV_32FC1 ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(LUTPerfTestGPU, LUTPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(CV_8UC1, CV_8UC3),
            Values(CV_8UC1),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(LUTPerfTestCustomGPU, LUTPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(CV_8UC3),
            Values(CV_8UC3),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(ConvertToPerfTestGPU, ConvertToPerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(CV_8UC3, CV_8UC1, CV_16UC1, CV_32FC1),
            Values(CV_8U, CV_16U, CV_16S, CV_32F),
            Values( szSmall128, szVGA, sz720p, sz1080p ),
            Values(2.5, 1.0),
            Values(0.0),
            Values(cv::compile_args(CORE_GPU))));

INSTANTIATE_TEST_CASE_P(TransposePerfTestGPU, TransposePerfTest,
    Combine(Values(AbsExact().to_compare_f()),
            Values(szSmall128, szVGA, sz720p, sz1080p),
            Values(CV_8UC1, CV_16UC1, CV_16SC1, CV_32FC1,
                   CV_8UC2, CV_16UC2, CV_16SC2, CV_32FC2,
                   CV_8UC3, CV_16UC3, CV_16SC3, CV_32FC3),
            Values(cv::compile_args(CORE_GPU))));
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
- `../common/gapi_core_perf_tests.hpp`
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

