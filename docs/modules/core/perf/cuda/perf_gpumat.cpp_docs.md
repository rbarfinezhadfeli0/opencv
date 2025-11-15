# Documentation for `modules/core/perf/cuda/perf_gpumat.cpp`

## File Metadata

- **Full Path**: `modules/core/perf/cuda/perf_gpumat.cpp`
- **File Name**: `perf_gpumat.cpp`
- **File Size**: 5,843 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/perf/cuda/perf_gpumat.cpp](../../../../modules/core/perf/cuda/perf_gpumat.cpp)

## Purpose and Role

This file is located in the `modules/core/perf/cuda` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
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
// This software is provided by the copyright holders and contributors "as is" and
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

#ifdef HAVE_CUDA

#include "opencv2/core/cuda.hpp"
#include "opencv2/ts/cuda_perf.hpp"

namespace opencv_test { namespace {
using namespace testing;
using namespace perf;

//////////////////////////////////////////////////////////////////////
// SetTo

PERF_TEST_P(Sz_Depth_Cn, CUDA_GpuMat_SetTo,
            Combine(CUDA_TYPICAL_MAT_SIZES,
                    Values(CV_8U, CV_16U, CV_32F, CV_64F),
                    CUDA_CHANNELS_1_3_4))
{
    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int channels = GET_PARAM(2);

    const int type = CV_MAKE_TYPE(depth, channels);

    const cv::Scalar val(1, 2, 3, 4);

    if (PERF_RUN_CUDA())
    {
        cv::cuda::GpuMat dst(size, type);

        TEST_CYCLE() dst.setTo(val);
    }
    else
    {
        cv::Mat dst(size, type);

        TEST_CYCLE() dst.setTo(val);
    }

    SANITY_CHECK_NOTHING();
}

//////////////////////////////////////////////////////////////////////
// SetToMasked

PERF_TEST_P(Sz_Depth_Cn, CUDA_GpuMat_SetToMasked,
            Combine(CUDA_TYPICAL_MAT_SIZES,
                    Values(CV_8U, CV_16U, CV_32F, CV_64F),
                    CUDA_CHANNELS_1_3_4))
{
    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int channels = GET_PARAM(2);

    const int type = CV_MAKE_TYPE(depth, channels);

    cv::Mat src(size, type);
    cv::Mat mask(size, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG);

    const cv::Scalar val(1, 2, 3, 4);

    if (PERF_RUN_CUDA())
    {
        cv::cuda::GpuMat dst(src);
        const cv::cuda::GpuMat d_mask(mask);

        TEST_CYCLE() dst.setTo(val, d_mask);
    }
    else
    {
        cv::Mat dst = src;

        TEST_CYCLE() dst.setTo(val, mask);
    }

    SANITY_CHECK_NOTHING();
}

//////////////////////////////////////////////////////////////////////
// CopyToMasked

PERF_TEST_P(Sz_Depth_Cn, CUDA_GpuMat_CopyToMasked,
            Combine(CUDA_TYPICAL_MAT_SIZES,
                    Values(CV_8U, CV_16U, CV_32F, CV_64F),
                    CUDA_CHANNELS_1_3_4))
{
    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int channels = GET_PARAM(2);

    const int type = CV_MAKE_TYPE(depth, channels);

    cv::Mat src(size, type);
    cv::Mat mask(size, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG);

    if (PERF_RUN_CUDA())
    {
        const cv::cuda::GpuMat d_src(src);
        const cv::cuda::GpuMat d_mask(mask);
        cv::cuda::GpuMat dst(d_src.size(), d_src.type(), cv::Scalar::all(0));

        TEST_CYCLE() d_src.copyTo(dst, d_mask);
    }
    else
    {
        cv::Mat dst(src.size(), src.type(), cv::Scalar::all(0));

        TEST_CYCLE() src.copyTo(dst, mask);
    }

    SANITY_CHECK_NOTHING();
}

//////////////////////////////////////////////////////////////////////
// ConvertTo

DEF_PARAM_TEST(Sz_2Depth, cv::Size, MatDepth, MatDepth);

PERF_TEST_P(Sz_2Depth, CUDA_GpuMat_ConvertTo,
            Combine(CUDA_TYPICAL_MAT_SIZES,
                    Values(CV_8U, CV_16U, CV_32F, CV_64F),
                    Values(CV_8U, CV_16U, CV_32F, CV_64F)))
{
    const cv::Size size = GET_PARAM(0);
    const int depth1 = GET_PARAM(1);
    const int depth2 = GET_PARAM(2);

    cv::Mat src(size, depth1);
    declare.in(src, WARMUP_RNG);

    const double a = 0.5;
    const double b = 1.0;

    if (PERF_RUN_CUDA())
    {
        const cv::cuda::GpuMat d_src(src);
        cv::cuda::GpuMat dst;

        TEST_CYCLE() d_src.convertTo(dst, depth2, a, b);
    }
    else
    {
        cv::Mat dst;

        TEST_CYCLE() src.convertTo(dst, depth2, a, b);
    }

    SANITY_CHECK_NOTHING();
}

}} // namespace
#endif
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

- **HAVE_CUDA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../perf_precomp.hpp`
- `opencv2/ts/cuda_perf.hpp`
- `opencv2/core/cuda.hpp`

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

