# Documentation for `modules/gapi/perf/common/gapi_imgproc_perf_tests.hpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/common/gapi_imgproc_perf_tests.hpp`
- **File Name**: `gapi_imgproc_perf_tests.hpp`
- **File Size**: 8,578 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/perf/common/gapi_imgproc_perf_tests.hpp](../../../../modules/gapi/perf/common/gapi_imgproc_perf_tests.hpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_IMGPROC_PERF_TESTS_HPP
#define OPENCV_GAPI_IMGPROC_PERF_TESTS_HPP



#include "../../test/common/gapi_tests_common.hpp"
#include <opencv2/gapi/imgproc.hpp>

namespace opencv_test
{

  using namespace perf;

  //------------------------------------------------------------------------------

class SepFilterPerfTest       : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int, cv::GCompileArgs>> {};
class Filter2DPerfTest        : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int,int, cv::GCompileArgs>> {};
class BoxFilterPerfTest       : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int,int, cv::GCompileArgs>> {};
class BlurPerfTest            : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int, cv::GCompileArgs>> {};
class GaussianBlurPerfTest    : public TestPerfParams<tuple<compare_f, MatType,int, cv::Size, cv::GCompileArgs>> {};
class MedianBlurPerfTest      : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size, cv::GCompileArgs>> {};
class ErodePerfTest           : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int, cv::GCompileArgs>> {};
class Erode3x3PerfTest        : public TestPerfParams<tuple<compare_f, MatType,cv::Size,int, cv::GCompileArgs>> {};
class DilatePerfTest          : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int, cv::GCompileArgs>> {};
class Dilate3x3PerfTest       : public TestPerfParams<tuple<compare_f, MatType,cv::Size,int, cv::GCompileArgs>> {};
class MorphologyExPerfTest    : public TestPerfParams<tuple<compare_f,MatType,cv::Size,
                                                            cv::MorphTypes,cv::GCompileArgs>> {};
class SobelPerfTest           : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int,int,int, cv::GCompileArgs>> {};
class SobelXYPerfTest         : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int,int, cv::GCompileArgs>> {};
class LaplacianPerfTest       : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int,
                                                            cv::GCompileArgs>> {};
class BilateralFilterPerfTest : public TestPerfParams<tuple<compare_f, MatType,int,cv::Size,int, double,double,
                                                            cv::GCompileArgs>> {};
class CannyPerfTest           : public TestPerfParams<tuple<compare_f, MatType,cv::Size,double,double,int,bool,
                                                            cv::GCompileArgs>> {};
class GoodFeaturesPerfTest    : public TestPerfParams<tuple<compare_vector_f<cv::Point2f>, std::string,
                                                            int,int,double,double,int,bool,
                                                            cv::GCompileArgs>> {};
class FindContoursPerfTest    : public TestPerfParams<tuple<CompareMats, MatType,cv::Size,
                                                            cv::RetrievalModes,
                                                            cv::ContourApproximationModes,
                                                            cv::GCompileArgs>> {};
class FindContoursHPerfTest   : public TestPerfParams<tuple<CompareMats, MatType,cv::Size,
                                                            cv::RetrievalModes,
                                                            cv::ContourApproximationModes,
                                                            cv::GCompileArgs>> {};
class BoundingRectMatPerfTest       :
    public TestPerfParams<tuple<CompareRects, MatType,cv::Size,bool, cv::GCompileArgs>> {};
class BoundingRectVector32SPerfTest :
    public TestPerfParams<tuple<CompareRects, cv::Size, cv::GCompileArgs>> {};
class BoundingRectVector32FPerfTest :
    public TestPerfParams<tuple<CompareRects, cv::Size, cv::GCompileArgs>> {};
class FitLine2DMatVectorPerfTest : public TestPerfParams<tuple<CompareVecs<float, 4>,
                                                               MatType,cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine2DVector32SPerfTest : public TestPerfParams<tuple<CompareVecs<float, 4>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine2DVector32FPerfTest : public TestPerfParams<tuple<CompareVecs<float, 4>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine2DVector64FPerfTest : public TestPerfParams<tuple<CompareVecs<float, 4>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine3DMatVectorPerfTest : public TestPerfParams<tuple<CompareVecs<float, 6>,
                                                               MatType,cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine3DVector32SPerfTest : public TestPerfParams<tuple<CompareVecs<float, 6>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine3DVector32FPerfTest : public TestPerfParams<tuple<CompareVecs<float, 6>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class FitLine3DVector64FPerfTest : public TestPerfParams<tuple<CompareVecs<float, 6>,
                                                               cv::Size,cv::DistanceTypes,
                                                               cv::GCompileArgs>> {};
class EqHistPerfTest      : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BGR2RGBPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2GrayPerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BGR2GrayPerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2YUVPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class YUV2RGBPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BGR2I420PerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2I420PerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class I4202BGRPerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class I4202RGBPerfTest    : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2LabPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BGR2LUVPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class LUV2BGRPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BGR2YUVPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class YUV2BGRPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2HSVPerfTest     : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class BayerGR2RGBPerfTest : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class RGB2YUV422PerfTest  : public TestPerfParams<tuple<compare_f, cv::Size, cv::GCompileArgs>> {};
class ResizePerfTest      : public TestPerfParams<tuple<compare_f, MatType, int, cv::Size, cv::Size, cv::GCompileArgs>> {};
class ResizeFxFyPerfTest  : public TestPerfParams<tuple<compare_f, MatType, int, cv::Size, double, double, cv::GCompileArgs>> {};
class ResizeInSimpleGraphPerfTest : public TestPerfParams<tuple<compare_f, MatType, cv::Size, double, double,  cv::GCompileArgs>> {};
class BottleneckKernelsConstInputPerfTest : public TestPerfParams<tuple<compare_f, std::string, cv::GCompileArgs>> {};
} // opencv_test

#endif //OPENCV_GAPI_IMGPROC_PERF_TESTS_HPP
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

- **FindContoursHPerfTest**: A class/struct defined in this file
- **RGB2HSVPerfTest**: A class/struct defined in this file
- **EqHistPerfTest**: A class/struct defined in this file
- **SobelXYPerfTest**: A class/struct defined in this file
- **ErodePerfTest**: A class/struct defined in this file
- **BoundingRectMatPerfTest**: A class/struct defined in this file
- **FitLine3DVector64FPerfTest**: A class/struct defined in this file
- **BoxFilterPerfTest**: A class/struct defined in this file
- **SepFilterPerfTest**: A class/struct defined in this file
- **FitLine2DVector32FPerfTest**: A class/struct defined in this file
- **YUV2BGRPerfTest**: A class/struct defined in this file
- **RGB2YUVPerfTest**: A class/struct defined in this file
- **FindContoursPerfTest**: A class/struct defined in this file
- **Erode3x3PerfTest**: A class/struct defined in this file
- **BlurPerfTest**: A class/struct defined in this file
- **BoundingRectVector32SPerfTest**: A class/struct defined in this file
- **FitLine2DVector32SPerfTest**: A class/struct defined in this file
- **FitLine2DMatVectorPerfTest**: A class/struct defined in this file
- **I4202RGBPerfTest**: A class/struct defined in this file
- **ResizeFxFyPerfTest**: A class/struct defined in this file
- **MedianBlurPerfTest**: A class/struct defined in this file
- **CannyPerfTest**: A class/struct defined in this file
- **BGR2RGBPerfTest**: A class/struct defined in this file
- **BGR2GrayPerfTest**: A class/struct defined in this file
- **RGB2I420PerfTest**: A class/struct defined in this file
- **GoodFeaturesPerfTest**: A class/struct defined in this file
- **LaplacianPerfTest**: A class/struct defined in this file
- **BGR2I420PerfTest**: A class/struct defined in this file
- **BoundingRectVector32FPerfTest**: A class/struct defined in this file
- **I4202BGRPerfTest**: A class/struct defined in this file
- **MorphologyExPerfTest**: A class/struct defined in this file
- **Dilate3x3PerfTest**: A class/struct defined in this file
- **RGB2GrayPerfTest**: A class/struct defined in this file
- **GaussianBlurPerfTest**: A class/struct defined in this file
- **BGR2LUVPerfTest**: A class/struct defined in this file
- **RGB2YUV422PerfTest**: A class/struct defined in this file
- **SobelPerfTest**: A class/struct defined in this file
- **RGB2LabPerfTest**: A class/struct defined in this file
- **BayerGR2RGBPerfTest**: A class/struct defined in this file
- **FitLine3DVector32SPerfTest**: A class/struct defined in this file
- **BilateralFilterPerfTest**: A class/struct defined in this file
- **DilatePerfTest**: A class/struct defined in this file
- **BGR2YUVPerfTest**: A class/struct defined in this file
- **LUV2BGRPerfTest**: A class/struct defined in this file
- **ResizePerfTest**: A class/struct defined in this file
- **FitLine3DMatVectorPerfTest**: A class/struct defined in this file
- **FitLine3DVector32FPerfTest**: A class/struct defined in this file
- **YUV2RGBPerfTest**: A class/struct defined in this file
- **FitLine2DVector64FPerfTest**: A class/struct defined in this file
- **Filter2DPerfTest**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_IMGPROC_PERF_TESTS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/imgproc.hpp`
- `../../test/common/gapi_tests_common.hpp`


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

