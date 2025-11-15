# Documentation for `modules/stitching/src/warpers_cuda.cpp`

## File Metadata

- **Full Path**: `modules/stitching/src/warpers_cuda.cpp`
- **File Name**: `warpers_cuda.cpp`
- **File Size**: 10,554 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/stitching/src/warpers_cuda.cpp](../../../modules/stitching/src/warpers_cuda.cpp)

## Purpose and Role

This file is located in the `modules/stitching/src` directory and serves as part of the OpenCV library infrastructure.

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
//                          License Agreement
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
#include "opencv2/core/private.cuda.hpp"

using namespace cv;
using namespace cv::cuda;

#ifdef HAVE_CUDA

namespace cv { namespace cuda { namespace device
{
    namespace imgproc
    {
        void buildWarpPlaneMaps(int tl_u, int tl_v, PtrStepSzf map_x, PtrStepSzf map_y,
                                const float k_rinv[9], const float r_kinv[9], const float t[3], float scale,
                                cudaStream_t stream);

        void buildWarpSphericalMaps(int tl_u, int tl_v, PtrStepSzf map_x, PtrStepSzf map_y,
                                    const float k_rinv[9], const float r_kinv[9], float scale,
                                    cudaStream_t stream);

        void buildWarpCylindricalMaps(int tl_u, int tl_v, PtrStepSzf map_x, PtrStepSzf map_y,
                                      const float k_rinv[9], const float r_kinv[9], float scale,
                                      cudaStream_t stream);
    }
}}}

static void buildWarpPlaneMaps(Size src_size, Rect dst_roi, InputArray _K, InputArray _R, InputArray _T,
                               float scale, OutputArray _map_x, OutputArray _map_y, Stream& stream = Stream::Null())
{
    CV_UNUSED(src_size);

    Mat K = _K.getMat();
    Mat R = _R.getMat();
    Mat T = _T.getMat();

    CV_Assert( K.size() == Size(3,3) && K.type() == CV_32FC1 );
    CV_Assert( R.size() == Size(3,3) && R.type() == CV_32FC1 );
    CV_Assert( (T.size() == Size(3,1) || T.size() == Size(1,3)) && T.type() == CV_32FC1 && T.isContinuous() );

    Mat K_Rinv = K * R.t();
    Mat R_Kinv = R * K.inv();
    CV_Assert( K_Rinv.isContinuous() );
    CV_Assert( R_Kinv.isContinuous() );

    _map_x.create(dst_roi.size(), CV_32FC1);
    _map_y.create(dst_roi.size(), CV_32FC1);

    GpuMat map_x = _map_x.getGpuMat();
    GpuMat map_y = _map_y.getGpuMat();

    device::imgproc::buildWarpPlaneMaps(dst_roi.tl().x, dst_roi.tl().y, map_x, map_y, K_Rinv.ptr<float>(), R_Kinv.ptr<float>(),
                       T.ptr<float>(), scale, StreamAccessor::getStream(stream));
}

static void buildWarpSphericalMaps(Size src_size, Rect dst_roi, InputArray _K, InputArray _R, float scale,
                                   OutputArray _map_x, OutputArray _map_y, Stream& stream = Stream::Null())
{
    CV_UNUSED(src_size);

    Mat K = _K.getMat();
    Mat R = _R.getMat();

    CV_Assert( K.size() == Size(3,3) && K.type() == CV_32FC1 );
    CV_Assert( R.size() == Size(3,3) && R.type() == CV_32FC1 );

    Mat K_Rinv = K * R.t();
    Mat R_Kinv = R * K.inv();
    CV_Assert( K_Rinv.isContinuous() );
    CV_Assert( R_Kinv.isContinuous() );

    _map_x.create(dst_roi.size(), CV_32FC1);
    _map_y.create(dst_roi.size(), CV_32FC1);

    GpuMat map_x = _map_x.getGpuMat();
    GpuMat map_y = _map_y.getGpuMat();

    device::imgproc::buildWarpSphericalMaps(dst_roi.tl().x, dst_roi.tl().y, map_x, map_y, K_Rinv.ptr<float>(), R_Kinv.ptr<float>(), scale, StreamAccessor::getStream(stream));
}

static void buildWarpCylindricalMaps(Size src_size, Rect dst_roi, InputArray _K, InputArray _R, float scale,
                                     OutputArray _map_x, OutputArray _map_y, Stream& stream = Stream::Null())
{
    CV_UNUSED(src_size);

    Mat K = _K.getMat();
    Mat R = _R.getMat();

    CV_Assert( K.size() == Size(3,3) && K.type() == CV_32FC1 );
    CV_Assert( R.size() == Size(3,3) && R.type() == CV_32FC1 );

    Mat K_Rinv = K * R.t();
    Mat R_Kinv = R * K.inv();
    CV_Assert( K_Rinv.isContinuous() );
    CV_Assert( R_Kinv.isContinuous() );

    _map_x.create(dst_roi.size(), CV_32FC1);
    _map_y.create(dst_roi.size(), CV_32FC1);

    GpuMat map_x = _map_x.getGpuMat();
    GpuMat map_y = _map_y.getGpuMat();

    device::imgproc::buildWarpCylindricalMaps(dst_roi.tl().x, dst_roi.tl().y, map_x, map_y, K_Rinv.ptr<float>(), R_Kinv.ptr<float>(), scale, StreamAccessor::getStream(stream));
}

#endif

Rect cv::detail::PlaneWarperGpu::buildMaps(Size src_size, InputArray K, InputArray R,
                                           cuda::GpuMat & xmap, cuda::GpuMat & ymap)
{
    return buildMaps(src_size, K, R, Mat::zeros(3, 1, CV_32F), xmap, ymap);
}

Rect cv::detail::PlaneWarperGpu::buildMaps(Size src_size, InputArray K, InputArray R, InputArray T,
                                           cuda::GpuMat & xmap, cuda::GpuMat & ymap)
{
#ifndef HAVE_CUDA
    CV_UNUSED(src_size);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(T);
    CV_UNUSED(xmap);
    CV_UNUSED(ymap);
    throw_no_cuda();
#else
    projector_.setCameraParams(K, R, T);

    Point dst_tl, dst_br;
    detectResultRoi(src_size, dst_tl, dst_br);

    ::buildWarpPlaneMaps(src_size, Rect(dst_tl, Point(dst_br.x + 1, dst_br.y + 1)),
                         K, R, T, projector_.scale, xmap, ymap);

    return Rect(dst_tl, dst_br);
#endif
}

Point cv::detail::PlaneWarperGpu::warp(const cuda::GpuMat & src, InputArray K, InputArray R,
                                       int interp_mode, int border_mode,
                                       cuda::GpuMat & dst)
{
    return warp(src, K, R, Mat::zeros(3, 1, CV_32F), interp_mode, border_mode, dst);
}


Point cv::detail::PlaneWarperGpu::warp(const cuda::GpuMat & src, InputArray K, InputArray R, InputArray T,
                                       int interp_mode, int border_mode,
                                       cuda::GpuMat & dst)
{
#ifndef HAVE_OPENCV_CUDAWARPING
    CV_UNUSED(src);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(T);
    CV_UNUSED(interp_mode);
    CV_UNUSED(border_mode);
    CV_UNUSED(dst);
    throw_no_cuda();
#else
    Rect dst_roi = buildMaps(src.size(), K, R, T, d_xmap_, d_ymap_);
    dst.create(dst_roi.height + 1, dst_roi.width + 1, src.type());
    cuda::remap(src, dst, d_xmap_, d_ymap_, interp_mode, border_mode);
    return dst_roi.tl();
#endif
}

Rect cv::detail::SphericalWarperGpu::buildMaps(Size src_size, InputArray K, InputArray R, cuda::GpuMat & xmap, cuda::GpuMat & ymap)
{
#ifndef HAVE_CUDA
    CV_UNUSED(src_size);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(xmap);
    CV_UNUSED(ymap);
    throw_no_cuda();
#else
    projector_.setCameraParams(K, R);

    Point dst_tl, dst_br;
    detectResultRoi(src_size, dst_tl, dst_br);

    ::buildWarpSphericalMaps(src_size, Rect(dst_tl, Point(dst_br.x + 1, dst_br.y + 1)),
                             K, R, projector_.scale, xmap, ymap);

    return Rect(dst_tl, dst_br);
#endif
}

Point cv::detail::SphericalWarperGpu::warp(const cuda::GpuMat & src, InputArray K, InputArray R,
                                           int interp_mode, int border_mode,
                                           cuda::GpuMat & dst)
{
#ifndef HAVE_OPENCV_CUDAWARPING
    CV_UNUSED(src);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(interp_mode);
    CV_UNUSED(border_mode);
    CV_UNUSED(dst);
    throw_no_cuda();
#else
    Rect dst_roi = buildMaps(src.size(), K, R, d_xmap_, d_ymap_);
    dst.create(dst_roi.height + 1, dst_roi.width + 1, src.type());
    cuda::remap(src, dst, d_xmap_, d_ymap_, interp_mode, border_mode);
    return dst_roi.tl();
#endif
}


Rect cv::detail::CylindricalWarperGpu::buildMaps(Size src_size, InputArray K, InputArray R,
                                                 cuda::GpuMat & xmap, cuda::GpuMat & ymap)
{
#ifndef HAVE_CUDA
    CV_UNUSED(src_size);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(xmap);
    CV_UNUSED(ymap);
    throw_no_cuda();
#else
    projector_.setCameraParams(K, R);

    Point dst_tl, dst_br;
    detectResultRoi(src_size, dst_tl, dst_br);

    ::buildWarpCylindricalMaps(src_size, Rect(dst_tl, Point(dst_br.x + 1, dst_br.y + 1)),
                               K, R, projector_.scale, xmap, ymap);

    return Rect(dst_tl, dst_br);
#endif
}

Point cv::detail::CylindricalWarperGpu::warp(const cuda::GpuMat & src, InputArray K, InputArray R,
                                             int interp_mode, int border_mode,
                                             cuda::GpuMat & dst)
{
#ifndef HAVE_OPENCV_CUDAWARPING
    CV_UNUSED(src);
    CV_UNUSED(K);
    CV_UNUSED(R);
    CV_UNUSED(interp_mode);
    CV_UNUSED(border_mode);
    CV_UNUSED(dst);
    throw_no_cuda();
#else
    Rect dst_roi = buildMaps(src.size(), K, R, d_xmap_, d_ymap_);
    dst.create(dst_roi.height + 1, dst_roi.width + 1, src.type());
    cuda::remap(src, dst, d_xmap_, d_ymap_, interp_mode, border_mode);
    return dst_roi.tl();
#endif
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

### Functions and Methods

- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_OPENCV_CUDAWARPING()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/private.cuda.hpp`
- `precomp.hpp`

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

