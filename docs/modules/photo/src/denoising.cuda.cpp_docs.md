# Documentation for `modules/photo/src/denoising.cuda.cpp`

## File Metadata

- **Full Path**: `modules/photo/src/denoising.cuda.cpp`
- **File Name**: `denoising.cuda.cpp`
- **File Size**: 7,464 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/src/denoising.cuda.cpp](../../../modules/photo/src/denoising.cuda.cpp)

## Purpose and Role

This file is located in the `modules/photo/src` directory and serves as part of the OpenCV library infrastructure.

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

#include "precomp.hpp"

#include "opencv2/photo/cuda.hpp"

#include "opencv2/opencv_modules.hpp"

#if !defined (HAVE_CUDA) || !defined(HAVE_OPENCV_CUDAARITHM) || !defined(HAVE_OPENCV_CUDAIMGPROC)
#include "opencv2/core/private/cuda_stubs.hpp"
#else
#include "opencv2/core/private.cuda.hpp"
#endif

#ifdef HAVE_OPENCV_CUDAARITHM
#  include "opencv2/cudaarithm.hpp"
#endif

#ifdef HAVE_OPENCV_CUDAIMGPROC
#  include "opencv2/cudaimgproc.hpp"
#endif

using namespace cv;
using namespace cv::cuda;

#if !defined (HAVE_CUDA) || !defined(HAVE_OPENCV_CUDAARITHM) || !defined(HAVE_OPENCV_CUDAIMGPROC)

void cv::cuda::nonLocalMeans(InputArray, OutputArray, float, int, int, int, Stream&) { throw_no_cuda(); }
void cv::cuda::fastNlMeansDenoising(InputArray, OutputArray, float, int, int, Stream&) { throw_no_cuda(); }
void cv::cuda::fastNlMeansDenoisingColored(InputArray, OutputArray, float, float, int, int, Stream&) { throw_no_cuda(); }

#else

//////////////////////////////////////////////////////////////////////////////////
//// Non Local Means Denosing (brute force)

namespace cv { namespace cuda { namespace device
{
    namespace imgproc
    {
        template<typename T>
        void nlm_bruteforce_gpu(const PtrStepSzb& src, PtrStepSzb dst, int search_radius, int block_radius, float h, int borderMode, cudaStream_t stream);
    }
}}}

void cv::cuda::nonLocalMeans(InputArray _src, OutputArray _dst, float h, int search_window, int block_window, int borderMode, Stream& stream)
{
    using cv::cuda::device::imgproc::nlm_bruteforce_gpu;
    typedef void (*func_t)(const PtrStepSzb& src, PtrStepSzb dst, int search_radius, int block_radius, float h, int borderMode, cudaStream_t stream);

    static const func_t funcs[4] = { nlm_bruteforce_gpu<uchar>, nlm_bruteforce_gpu<uchar2>, nlm_bruteforce_gpu<uchar3>, 0/*nlm_bruteforce_gpu<uchar4>,*/ };

    const GpuMat src = _src.getGpuMat();

    CV_Assert(src.type() == CV_8U || src.type() == CV_8UC2 || src.type() == CV_8UC3);

    const func_t func = funcs[src.channels() - 1];
    CV_Assert(func != 0);

    int b = borderMode;
    CV_Assert(b == BORDER_REFLECT101 || b == BORDER_REPLICATE || b == BORDER_CONSTANT || b == BORDER_REFLECT || b == BORDER_WRAP);

    _dst.create(src.size(), src.type());
    GpuMat dst = _dst.getGpuMat();

    func(src, dst, search_window/2, block_window/2, h, borderMode, StreamAccessor::getStream(stream));
}

namespace cv { namespace cuda { namespace device
{
    namespace imgproc
    {
        void nln_fast_get_buffer_size(const PtrStepSzb& src, int search_window, int block_window, int& buffer_cols, int& buffer_rows);

        template<typename T>
        void nlm_fast_gpu(const PtrStepSzb& src, PtrStepSzb dst, PtrStepi buffer,
                          int search_window, int block_window, float h, cudaStream_t stream);

        void fnlm_split_channels(const PtrStepSz<uchar3>& lab, PtrStepb l, PtrStep<uchar2> ab, cudaStream_t stream);
        void fnlm_merge_channels(const PtrStepb& l, const PtrStep<uchar2>& ab, PtrStepSz<uchar3> lab, cudaStream_t stream);
     }
}}}

void cv::cuda::fastNlMeansDenoising(InputArray _src, OutputArray _dst, float h, int search_window, int block_window, Stream& stream)
{
    const GpuMat src = _src.getGpuMat();

    CV_Assert(src.depth() == CV_8U && src.channels() < 4);

    int border_size = search_window/2 + block_window/2;
    Size esize = src.size() + Size(border_size, border_size) * 2;

    BufferPool pool(stream);

    GpuMat extended_src = pool.getBuffer(esize, src.type());
    cv::cuda::copyMakeBorder(src, extended_src, border_size, border_size, border_size, border_size, cv::BORDER_DEFAULT, Scalar(), stream);
    GpuMat src_hdr = extended_src(Rect(Point2i(border_size, border_size), src.size()));

    int bcols, brows;
    device::imgproc::nln_fast_get_buffer_size(src_hdr, search_window, block_window, bcols, brows);
    GpuMat buffer = pool.getBuffer(brows, bcols, CV_32S);

    using namespace cv::cuda::device::imgproc;
    typedef void (*nlm_fast_t)(const PtrStepSzb&, PtrStepSzb, PtrStepi, int, int, float, cudaStream_t);
    static const nlm_fast_t funcs[] = { nlm_fast_gpu<uchar>, nlm_fast_gpu<uchar2>, nlm_fast_gpu<uchar3>, 0};

    _dst.create(src.size(), src.type());
    GpuMat dst = _dst.getGpuMat();

    funcs[src.channels()-1](src_hdr, dst, buffer, search_window, block_window, h, StreamAccessor::getStream(stream));
}

void cv::cuda::fastNlMeansDenoisingColored(InputArray _src, OutputArray _dst, float h_luminance, float h_color, int search_window, int block_window, Stream& stream)
{
    const GpuMat src = _src.getGpuMat();

    CV_Assert(src.type() == CV_8UC3);

    BufferPool pool(stream);

    GpuMat lab = pool.getBuffer(src.size(), src.type());
    cv::cuda::cvtColor(src, lab, cv::COLOR_BGR2Lab, 0, stream);

    GpuMat l = pool.getBuffer(src.size(), CV_8U);
    GpuMat ab = pool.getBuffer(src.size(), CV_8UC2);
    device::imgproc::fnlm_split_channels(lab, l, ab, StreamAccessor::getStream(stream));

    fastNlMeansDenoising(l, l, h_luminance, search_window, block_window, stream);
    fastNlMeansDenoising(ab, ab, h_color, search_window, block_window, stream);

    device::imgproc::fnlm_merge_channels(l, ab, lab, StreamAccessor::getStream(stream));
    cv::cuda::cvtColor(lab, _dst, cv::COLOR_Lab2BGR, 0, stream);
}

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

- **HAVE_OPENCV_CUDAIMGPROC()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **HAVE_OPENCV_CUDAARITHM()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/private.cuda.hpp`
- `opencv2/opencv_modules.hpp`
- `opencv2/photo/cuda.hpp`
- `precomp.hpp`
- `opencv2/core/private/cuda_stubs.hpp`

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

