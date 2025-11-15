# Documentation for `modules/imgproc/perf/opencl/perf_imgwarp.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/opencl/perf_imgwarp.cpp`
- **File Name**: `perf_imgwarp.cpp`
- **File Size**: 9,002 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/opencl/perf_imgwarp.cpp](../../../../modules/imgproc/perf/opencl/perf_imgwarp.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/perf/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2010-2012, Multicoreware, Inc., all rights reserved.
// Copyright (C) 2010-2012, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// @Authors
//    Fangfang Bai, fangfang@multicorewareinc.com
//    Jin Ma,       jin@multicorewareinc.com
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors as is and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "../perf_precomp.hpp"
#include "opencv2/ts/ocl_perf.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

///////////// WarpAffine ////////////////////////

CV_ENUM(InterType, INTER_NEAREST, INTER_LINEAR, INTER_CUBIC)

typedef tuple<Size, MatType, InterType> WarpAffineParams;
typedef TestBaseWithParam<WarpAffineParams> WarpAffineFixture;

OCL_PERF_TEST_P(WarpAffineFixture, WarpAffine,
            ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES_134, InterType::all()))
{
    static const double coeffs[2][3] =
    {
        { cos(CV_PI / 6), -sin(CV_PI / 6), 100.0  },
        { sin(CV_PI / 6), cos(CV_PI / 6) , -100.0 }
    };
    Mat M(2, 3, CV_64F, (void *)coeffs);

    const WarpAffineParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params), interpolation = get<2>(params);
    const double eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : interpolation == INTER_CUBIC ? 2e-3 : 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst(srcSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::warpAffine(src, dst, M, srcSize, interpolation);

    SANITY_CHECK(dst, eps);
}

///////////// WarpPerspective ////////////////////////

typedef WarpAffineParams WarpPerspectiveParams;
typedef TestBaseWithParam<WarpPerspectiveParams> WarpPerspectiveFixture;

OCL_PERF_TEST_P(WarpPerspectiveFixture, WarpPerspective,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES_134,
                                   OCL_PERF_ENUM(InterType(INTER_NEAREST), InterType(INTER_LINEAR))))
{
    static const double coeffs[3][3] =
    {
        {cos(CV_PI / 6), -sin(CV_PI / 6), 100.0},
        {sin(CV_PI / 6), cos(CV_PI / 6), -100.0},
        {0.0, 0.0, 1.0}
    };
    Mat M(3, 3, CV_64F, (void *)coeffs);

    const WarpPerspectiveParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params), interpolation = get<2>(params);
    const double eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst(srcSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::warpPerspective(src, dst, M, srcSize, interpolation);

    SANITY_CHECK(dst, eps);
}

///////////// Resize ////////////////////////

typedef tuple<Size, MatType, InterType, double> ResizeParams;
typedef TestBaseWithParam<ResizeParams> ResizeFixture;

OCL_PERF_TEST_P(ResizeFixture, Resize,
            ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES_134,
                               OCL_PERF_ENUM(InterType(INTER_NEAREST), InterType(INTER_LINEAR)),
                               ::testing::Values(0.5, 2.0)))
{
    const ResizeParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params), interType = get<2>(params);
    double scale = get<3>(params);
    const Size dstSize(cvRound(srcSize.width * scale), cvRound(srcSize.height * scale));
    const double eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(dstSize, type);

    UMat src(srcSize, type), dst(dstSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::resize(src, dst, Size(), scale, scale, interType);

    SANITY_CHECK(dst, eps);
}

typedef tuple<Size, MatType, double> ResizeAreaParams;
typedef TestBaseWithParam<ResizeAreaParams> ResizeAreaFixture;

OCL_PERF_TEST_P(ResizeAreaFixture, Resize,
            ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES_134, ::testing::Values(0.3, 0.5, 0.6)))
{
    const ResizeAreaParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);
    double scale = get<2>(params);
    const Size dstSize(cvRound(srcSize.width * scale), cvRound(srcSize.height * scale));
    const double eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(dstSize, type);

    UMat src(srcSize, type), dst(dstSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::resize(src, dst, Size(), scale, scale, cv::INTER_AREA);

    SANITY_CHECK(dst, eps);
}

typedef ResizeAreaParams ResizeLinearExactParams;
typedef TestBaseWithParam<ResizeLinearExactParams> ResizeLinearExactFixture;

OCL_PERF_TEST_P(ResizeLinearExactFixture, Resize,
            ::testing::Combine(OCL_TEST_SIZES, ::testing::Values(CV_8UC1, CV_8UC3, CV_8UC4), ::testing::Values(0.5, 2.0)))
{
    const ResizeAreaParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);
    double scale = get<2>(params);
    const Size dstSize(cvRound(srcSize.width * scale), cvRound(srcSize.height * scale));
    const double eps = 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(dstSize, type);

    UMat src(srcSize, type), dst(dstSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::resize(src, dst, Size(), scale, scale, cv::INTER_LINEAR_EXACT);

    SANITY_CHECK(dst, eps);
}

///////////// Remap ////////////////////////

typedef tuple<Size, MatType, InterType> RemapParams;
typedef TestBaseWithParam<RemapParams> RemapFixture;

OCL_PERF_TEST_P(RemapFixture, Remap,
            ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES_134,
                               OCL_PERF_ENUM(InterType(INTER_NEAREST), InterType(INTER_LINEAR))))
{
    const RemapParams params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params), interpolation = get<2>(params), borderMode = BORDER_CONSTANT;
    //const double eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : 1e-4;

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst(srcSize, type);
    UMat xmap(srcSize, CV_32FC1), ymap(srcSize, CV_32FC1);

    {
        Mat _xmap = xmap.getMat(ACCESS_WRITE), _ymap = ymap.getMat(ACCESS_WRITE);
        for (int i = 0; i < srcSize.height; ++i)
        {
            float * const xmap_row = _xmap.ptr<float>(i);
            float * const ymap_row = _ymap.ptr<float>(i);

            for (int j = 0; j < srcSize.width; ++j)
            {
                xmap_row[j] = (j - srcSize.width * 0.5f) * 0.75f + srcSize.width * 0.5f;
                ymap_row[j] = (i - srcSize.height * 0.5f) * 0.75f + srcSize.height * 0.5f;
            }
        }
    }
    declare.in(src, WARMUP_RNG).in(xmap, ymap, WARMUP_READ).out(dst);

    OCL_TEST_CYCLE() cv::remap(src, dst, xmap, ymap, interpolation, borderMode);

    SANITY_CHECK_NOTHING();
}

} } // namespace opencv_test::ocl

#endif // HAVE_OPENCL
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

- **TestBaseWithParam()**: A function/method defined in this file
- **WarpAffineParams()**: A function/method defined in this file
- **ResizeAreaParams()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_perf.hpp`
- `../perf_precomp.hpp`

**Python Imports:**
- `this`


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

