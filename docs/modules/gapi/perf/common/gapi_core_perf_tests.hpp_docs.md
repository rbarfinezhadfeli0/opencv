# Documentation for `modules/gapi/perf/common/gapi_core_perf_tests.hpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/common/gapi_core_perf_tests.hpp`
- **File Name**: `gapi_core_perf_tests.hpp`
- **File Size**: 7,145 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/perf/common/gapi_core_perf_tests.hpp](../../../../modules/gapi/perf/common/gapi_core_perf_tests.hpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2021 Intel Corporation


#ifndef OPENCV_GAPI_CORE_PERF_TESTS_HPP
#define OPENCV_GAPI_CORE_PERF_TESTS_HPP


#include "../../test/common/gapi_tests_common.hpp"
#include "../../test/common/gapi_parsers_tests_common.hpp"
#include <opencv2/gapi/core.hpp>

namespace opencv_test
{
  using namespace perf;

  enum bitwiseOp
  {
      AND = 0,
      OR = 1,
      XOR = 2,
      NOT = 3
  };

//------------------------------------------------------------------------------
    class PhasePerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class SqrtPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class AddPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class AddCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class SubPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class SubCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class SubRCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class MulPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, double, cv::GCompileArgs>> {};
    class MulDoublePerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class MulCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class DivPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, double, cv::GCompileArgs>> {};
    class DivCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, double, cv::GCompileArgs>> {};
    class DivRCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, double, cv::GCompileArgs>> {};
    class MaskPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class MeanPerfTest : public TestPerfParams<tuple<compare_scalar_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class Polar2CartPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
    class Cart2PolarPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
    class CmpPerfTest : public TestPerfParams<tuple<compare_f, CmpTypes, cv::Size, MatType, cv::GCompileArgs>> {};
    class CmpWithScalarPerfTest : public TestPerfParams<tuple<compare_f, CmpTypes, cv::Size, MatType, cv::GCompileArgs>> {};
    class BitwisePerfTest : public TestPerfParams<tuple<compare_f, bitwiseOp, bool, cv::Size, MatType, cv::GCompileArgs>> {};
    class BitwiseNotPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class SelectPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class MinPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class MaxPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class AbsDiffPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class AbsDiffCPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class SumPerfTest : public TestPerfParams<tuple<compare_scalar_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class CountNonZeroPerfTest : public TestPerfParams<tuple<compare_scalar_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class AddWeightedPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class NormPerfTest : public TestPerfParams<tuple<compare_scalar_f, NormTypes, cv::Size, MatType, cv::GCompileArgs>> {};
    class IntegralPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ThresholdPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class ThresholdOTPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class InRangePerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class Split3PerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
    class Split4PerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
    class Merge3PerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class Merge4PerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
    class RemapPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class FlipPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, int, cv::GCompileArgs>> {};
    class CropPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::Rect, cv::GCompileArgs>> {};
    class CopyPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ConcatHorPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ConcatHorVecPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ConcatVertPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ConcatVertVecPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class LUTPerfTest : public TestPerfParams<tuple<compare_f, MatType, MatType, cv::Size, cv::GCompileArgs>> {};
    class ConvertToPerfTest : public TestPerfParams<tuple<compare_f, MatType, int, cv::Size, double, double, cv::GCompileArgs>> {};
    class KMeansNDPerfTest : public TestPerfParams<tuple<cv::Size, CompareMats, int, cv::KmeansFlags, cv::GCompileArgs>> {};
    class KMeans2DPerfTest : public TestPerfParams<tuple<int, int, cv::KmeansFlags, cv::GCompileArgs>> {};
    class KMeans3DPerfTest : public TestPerfParams<tuple<int, int, cv::KmeansFlags, cv::GCompileArgs>> {};
    class TransposePerfTest : public TestPerfParams<tuple<compare_f, cv::Size, MatType, cv::GCompileArgs>> {};
    class ParseSSDBLPerfTest : public TestPerfParams<tuple<cv::Size, float, int, cv::GCompileArgs>>, public ParserSSDTest {};
    class ParseSSDPerfTest   : public TestPerfParams<tuple<cv::Size, float, bool, bool, cv::GCompileArgs>>, public ParserSSDTest {};
    class ParseYoloPerfTest  : public TestPerfParams<tuple<cv::Size, float, float, int, cv::GCompileArgs>>, public ParserYoloTest {};
    class SizePerfTest       : public TestPerfParams<tuple<MatType, cv::Size, cv::GCompileArgs>> {};
    class SizeRPerfTest      : public TestPerfParams<tuple<cv::Size, cv::GCompileArgs>> {};
}
#endif // OPENCV_GAPI_CORE_PERF_TESTS_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **CmpPerfTest**: A class/struct defined in this file
- **LUTPerfTest**: A class/struct defined in this file
- **DivPerfTest**: A class/struct defined in this file
- **MaskPerfTest**: A class/struct defined in this file
- **BitwisePerfTest**: A class/struct defined in this file
- **FlipPerfTest**: A class/struct defined in this file
- **CropPerfTest**: A class/struct defined in this file
- **MulDoublePerfTest**: A class/struct defined in this file
- **ConcatVertPerfTest**: A class/struct defined in this file
- **SubCPerfTest**: A class/struct defined in this file
- **InRangePerfTest**: A class/struct defined in this file
- **ConcatVertVecPerfTest**: A class/struct defined in this file
- **Polar2CartPerfTest**: A class/struct defined in this file
- **Merge3PerfTest**: A class/struct defined in this file
- **PhasePerfTest**: A class/struct defined in this file
- **NormPerfTest**: A class/struct defined in this file
- **ConcatHorVecPerfTest**: A class/struct defined in this file
- **KMeans2DPerfTest**: A class/struct defined in this file
- **ConcatHorPerfTest**: A class/struct defined in this file
- **SelectPerfTest**: A class/struct defined in this file
- **SubPerfTest**: A class/struct defined in this file
- **ConvertToPerfTest**: A class/struct defined in this file
- **BitwiseNotPerfTest**: A class/struct defined in this file
- **IntegralPerfTest**: A class/struct defined in this file
- **AbsDiffPerfTest**: A class/struct defined in this file
- **MeanPerfTest**: A class/struct defined in this file
- **RemapPerfTest**: A class/struct defined in this file
- **CmpWithScalarPerfTest**: A class/struct defined in this file
- **SqrtPerfTest**: A class/struct defined in this file
- **CountNonZeroPerfTest**: A class/struct defined in this file
- **SumPerfTest**: A class/struct defined in this file
- **CopyPerfTest**: A class/struct defined in this file
- **Split4PerfTest**: A class/struct defined in this file
- **Cart2PolarPerfTest**: A class/struct defined in this file
- **MinPerfTest**: A class/struct defined in this file
- **AbsDiffCPerfTest**: A class/struct defined in this file
- **AddWeightedPerfTest**: A class/struct defined in this file
- **ThresholdPerfTest**: A class/struct defined in this file
- **DivCPerfTest**: A class/struct defined in this file
- **Split3PerfTest**: A class/struct defined in this file
- **DivRCPerfTest**: A class/struct defined in this file
- **KMeansNDPerfTest**: A class/struct defined in this file
- **MaxPerfTest**: A class/struct defined in this file
- **AddPerfTest**: A class/struct defined in this file
- **MulPerfTest**: A class/struct defined in this file
- **SubRCPerfTest**: A class/struct defined in this file
- **ThresholdOTPerfTest**: A class/struct defined in this file
- **AddCPerfTest**: A class/struct defined in this file
- **Merge4PerfTest**: A class/struct defined in this file
- **MulCPerfTest**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_CORE_PERF_TESTS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../test/common/gapi_tests_common.hpp`
- `../../test/common/gapi_parsers_tests_common.hpp`
- `opencv2/gapi/core.hpp`


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

