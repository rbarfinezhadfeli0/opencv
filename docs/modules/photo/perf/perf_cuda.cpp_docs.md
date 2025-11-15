# Documentation for `modules/photo/perf/perf_cuda.cpp`

## File Metadata

- **Full Path**: `modules/photo/perf/perf_cuda.cpp`
- **File Name**: `perf_cuda.cpp`
- **File Size**: 6,033 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/perf/perf_cuda.cpp](../../../modules/photo/perf/perf_cuda.cpp)

## Purpose and Role

This file is located in the `modules/photo/perf` directory and serves as part of the OpenCV library infrastructure.

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

#include "perf_precomp.hpp"

#include "opencv2/photo/cuda.hpp"
#include "opencv2/ts/cuda_perf.hpp"

#include "opencv2/opencv_modules.hpp"

#if defined (HAVE_CUDA) && defined(HAVE_OPENCV_CUDAARITHM) && defined(HAVE_OPENCV_CUDAIMGPROC)

namespace opencv_test { namespace {
using namespace perf;

#define CUDA_DENOISING_IMAGE_SIZES testing::Values(perf::szVGA, perf::sz720p)

//////////////////////////////////////////////////////////////////////
// nonLocalMeans

DEF_PARAM_TEST(Sz_Depth_Cn_WinSz_BlockSz, cv::Size, MatDepth, MatCn, int, int);

PERF_TEST_P(Sz_Depth_Cn_WinSz_BlockSz, CUDA_NonLocalMeans,
            Combine(CUDA_DENOISING_IMAGE_SIZES,
                    Values<MatDepth>(CV_8U),
                    CUDA_CHANNELS_1_3,
                    Values(21),
                    Values(5)))
{
    declare.time(600.0);

    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int channels = GET_PARAM(2);
    const int search_window_size = GET_PARAM(3);
    const int block_size = GET_PARAM(4);

    const float h = 10;
    const int borderMode = cv::BORDER_REFLECT101;

    const int type = CV_MAKE_TYPE(depth, channels);

    cv::Mat src(size, type);
    declare.in(src, WARMUP_RNG);

    if (PERF_RUN_CUDA())
    {
        const cv::cuda::GpuMat d_src(src);
        cv::cuda::GpuMat dst;

        TEST_CYCLE() cv::cuda::nonLocalMeans(d_src, dst, h, search_window_size, block_size, borderMode);

        CUDA_SANITY_CHECK(dst);
    }
    else
    {
        FAIL_NO_CPU();
    }
}


//////////////////////////////////////////////////////////////////////
// fastNonLocalMeans

DEF_PARAM_TEST(Sz_Depth_Cn_WinSz_BlockSz, cv::Size, MatDepth, MatCn, int, int);

PERF_TEST_P(Sz_Depth_Cn_WinSz_BlockSz, CUDA_FastNonLocalMeans,
            Combine(CUDA_DENOISING_IMAGE_SIZES,
                    Values<MatDepth>(CV_8U),
                    CUDA_CHANNELS_1_3,
                    Values(21),
                    Values(7)))
{
    declare.time(60.0);

    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int search_window_size = GET_PARAM(2);
    const int block_size = GET_PARAM(3);

    const float h = 10;
    const int type = CV_MAKE_TYPE(depth, 1);

    cv::Mat src(size, type);
    declare.in(src, WARMUP_RNG);

    if (PERF_RUN_CUDA())
    {
        const cv::cuda::GpuMat d_src(src);
        cv::cuda::GpuMat dst;

        TEST_CYCLE() cv::cuda::fastNlMeansDenoising(d_src, dst, h, search_window_size, block_size);

        CUDA_SANITY_CHECK(dst);
    }
    else
    {
        cv::Mat dst;

        TEST_CYCLE() cv::fastNlMeansDenoising(src, dst, h, block_size, search_window_size);

        CPU_SANITY_CHECK(dst);
    }
}

//////////////////////////////////////////////////////////////////////
// fastNonLocalMeans (colored)

DEF_PARAM_TEST(Sz_Depth_WinSz_BlockSz, cv::Size, MatDepth, int, int);

PERF_TEST_P(Sz_Depth_WinSz_BlockSz, CUDA_FastNonLocalMeansColored,
            Combine(CUDA_DENOISING_IMAGE_SIZES,
                    Values<MatDepth>(CV_8U),
                    Values(21),
                    Values(7)))
{
    declare.time(60.0);

    const cv::Size size = GET_PARAM(0);
    const int depth = GET_PARAM(1);
    const int search_window_size = GET_PARAM(2);
    const int block_size = GET_PARAM(3);

    const float h = 10;
    const int type = CV_MAKE_TYPE(depth, 3);

    cv::Mat src(size, type);
    declare.in(src, WARMUP_RNG);

    if (PERF_RUN_CUDA())
    {
        const cv::cuda::GpuMat d_src(src);
        cv::cuda::GpuMat dst;

        TEST_CYCLE() cv::cuda::fastNlMeansDenoisingColored(d_src, dst, h, h, search_window_size, block_size);

        CUDA_SANITY_CHECK(dst);
    }
    else
    {
        cv::Mat dst;

        TEST_CYCLE() cv::fastNlMeansDenoisingColored(src, dst, h, h, block_size, search_window_size);

        CPU_SANITY_CHECK(dst);
    }
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/photo/cuda.hpp`
- `opencv2/opencv_modules.hpp`
- `opencv2/ts/cuda_perf.hpp`
- `perf_precomp.hpp`

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

