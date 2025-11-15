# Documentation for `modules/core/include/opencv2/core/cuda/filters.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cuda/filters.hpp`
- **File Name**: `filters.hpp`
- **File Size**: 10,110 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/cuda/filters.hpp](../../../../../../modules/core/include/opencv2/core/cuda/filters.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/cuda` directory and serves as part of the OpenCV library infrastructure.

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

#ifndef OPENCV_CUDA_FILTERS_HPP
#define OPENCV_CUDA_FILTERS_HPP

#include "saturate_cast.hpp"
#include "vec_traits.hpp"
#include "vec_math.hpp"
#include "type_traits.hpp"
#include "nppdefs.h"

/** @file
 * @deprecated Use @ref cudev instead.
 */

//! @cond IGNORED

namespace cv { namespace cuda { namespace device
{
    template <typename Ptr2D> struct PointFilter
    {
        typedef typename Ptr2D::elem_type elem_type;
        typedef float index_type;

        explicit __host__ __device__ __forceinline__ PointFilter(const Ptr2D& src_, float fx = 0.f, float fy = 0.f)
        : src(src_)
        {
            CV_UNUSED(fx);
            CV_UNUSED(fy);
        }

        __device__ __forceinline__ elem_type operator ()(float y, float x) const
        {
            return src(__float2int_rz(y), __float2int_rz(x));
        }

        Ptr2D src;
    };

    template <typename Ptr2D> struct LinearFilter
    {
        typedef typename Ptr2D::elem_type elem_type;
        typedef float index_type;

        explicit __host__ __device__ __forceinline__ LinearFilter(const Ptr2D& src_, float fx = 0.f, float fy = 0.f)
        : src(src_)
        {
            CV_UNUSED(fx);
            CV_UNUSED(fy);
        }
        __device__ __forceinline__ elem_type operator ()(float y, float x) const
        {
            typedef typename TypeVec<float, VecTraits<elem_type>::cn>::vec_type work_type;

            work_type out = VecTraits<work_type>::all(0);

            const int x1 = __float2int_rd(x);
            const int y1 = __float2int_rd(y);
            if (x1 <= NPP_MIN_32S || x1 >= NPP_MAX_32S || y1 <= NPP_MIN_32S || y1 >= NPP_MAX_32S)
            {
                elem_type src_reg = src(y1, x1);
                out = out + src_reg * 1.0f;
                return saturate_cast<elem_type>(out);
            }
            const int x2 = x1 + 1;
            const int y2 = y1 + 1;

            elem_type src_reg = src(y1, x1);
            out = out + src_reg * ((x2 - x) * (y2 - y));

            src_reg = src(y1, x2);
            out = out + src_reg * ((x - x1) * (y2 - y));

            src_reg = src(y2, x1);
            out = out + src_reg * ((x2 - x) * (y - y1));

            src_reg = src(y2, x2);
            out = out + src_reg * ((x - x1) * (y - y1));

            return saturate_cast<elem_type>(out);
        }

        Ptr2D src;
    };

    template <typename Ptr2D> struct CubicFilter
    {
        typedef typename Ptr2D::elem_type elem_type;
        typedef float index_type;
        typedef typename TypeVec<float, VecTraits<elem_type>::cn>::vec_type work_type;

        explicit __host__ __device__ __forceinline__ CubicFilter(const Ptr2D& src_, float fx = 0.f, float fy = 0.f)
        : src(src_)
        {
            CV_UNUSED(fx);
            CV_UNUSED(fy);
        }

        static __device__ __forceinline__ float bicubicCoeff(float x_)
        {
            float x = fabsf(x_);
            if (x <= 1.0f)
            {
                return x * x * (1.5f * x - 2.5f) + 1.0f;
            }
            else if (x < 2.0f)
            {
                return x * (x * (-0.5f * x + 2.5f) - 4.0f) + 2.0f;
            }
            else
            {
                return 0.0f;
            }
        }

        __device__ elem_type operator ()(float y, float x) const
        {
            const float xmin = ::ceilf(x - 2.0f);
            const float xmax = ::floorf(x + 2.0f);

            const float ymin = ::ceilf(y - 2.0f);
            const float ymax = ::floorf(y + 2.0f);

            work_type sum = VecTraits<work_type>::all(0);
            float wsum = 0.0f;

            for (float cy = ymin; cy <= ymax; cy += 1.0f)
            {
                for (float cx = xmin; cx <= xmax; cx += 1.0f)
                {
                    const float w = bicubicCoeff(x - cx) * bicubicCoeff(y - cy);
                    sum = sum + w * src(__float2int_rd(cy), __float2int_rd(cx));
                    wsum += w;
                }
            }

            work_type res = (!wsum)? VecTraits<work_type>::all(0) : sum / wsum;

            return saturate_cast<elem_type>(res);
        }

        Ptr2D src;
    };
    // for integer scaling
    template <typename Ptr2D> struct IntegerAreaFilter
    {
        typedef typename Ptr2D::elem_type elem_type;
        typedef float index_type;

        explicit __host__ __device__ __forceinline__ IntegerAreaFilter(const Ptr2D& src_, float scale_x_, float scale_y_)
            : src(src_), scale_x(scale_x_), scale_y(scale_y_), scale(1.f / (scale_x * scale_y)) {}

        __device__ __forceinline__ elem_type operator ()(float y, float x) const
        {
            float fsx1 = x * scale_x;
            float fsx2 = fsx1 + scale_x;

            int sx1 = __float2int_ru(fsx1);
            int sx2 = __float2int_rd(fsx2);

            float fsy1 = y * scale_y;
            float fsy2 = fsy1 + scale_y;

            int sy1 = __float2int_ru(fsy1);
            int sy2 = __float2int_rd(fsy2);

            typedef typename TypeVec<float, VecTraits<elem_type>::cn>::vec_type work_type;
            work_type out = VecTraits<work_type>::all(0.f);

            for(int dy = sy1; dy < sy2; ++dy)
                for(int dx = sx1; dx < sx2; ++dx)
                {
                    out = out + src(dy, dx) * scale;
                }

            return saturate_cast<elem_type>(out);
        }

        Ptr2D src;
        float scale_x, scale_y ,scale;
    };

    template <typename Ptr2D> struct AreaFilter
    {
        typedef typename Ptr2D::elem_type elem_type;
        typedef float index_type;

        explicit __host__ __device__ __forceinline__ AreaFilter(const Ptr2D& src_, float scale_x_, float scale_y_)
            : src(src_), scale_x(scale_x_), scale_y(scale_y_){}

        __device__ __forceinline__ elem_type operator ()(float y, float x) const
        {
            float fsx1 = x * scale_x;
            float fsx2 = fsx1 + scale_x;

            int sx1 = __float2int_ru(fsx1);
            int sx2 = __float2int_rd(fsx2);

            float fsy1 = y * scale_y;
            float fsy2 = fsy1 + scale_y;

            int sy1 = __float2int_ru(fsy1);
            int sy2 = __float2int_rd(fsy2);

            float scale = 1.f / (fminf(scale_x, src.width - fsx1) * fminf(scale_y, src.height - fsy1));

            typedef typename TypeVec<float, VecTraits<elem_type>::cn>::vec_type work_type;
            work_type out = VecTraits<work_type>::all(0.f);

            for (int dy = sy1; dy < sy2; ++dy)
            {
                for (int dx = sx1; dx < sx2; ++dx)
                    out = out + src(dy, dx) * scale;

                if (sx1 > fsx1)
                    out = out + src(dy, (sx1 -1) ) * ((sx1 - fsx1) * scale);

                if (sx2 < fsx2)
                    out = out + src(dy, sx2) * ((fsx2 -sx2) * scale);
            }

            if (sy1 > fsy1)
                for (int dx = sx1; dx < sx2; ++dx)
                    out = out + src( (sy1 - 1) , dx) * ((sy1 -fsy1) * scale);

            if (sy2 < fsy2)
                for (int dx = sx1; dx < sx2; ++dx)
                    out = out + src(sy2, dx) * ((fsy2 -sy2) * scale);

            if ((sy1 > fsy1) &&  (sx1 > fsx1))
                out = out + src( (sy1 - 1) , (sx1 - 1)) * ((sy1 -fsy1) * (sx1 -fsx1) * scale);

            if ((sy1 > fsy1) &&  (sx2 < fsx2))
                out = out + src( (sy1 - 1) , sx2) * ((sy1 -fsy1) * (fsx2 -sx2) * scale);

            if ((sy2 < fsy2) &&  (sx2 < fsx2))
                out = out + src(sy2, sx2) * ((fsy2 -sy2) * (fsx2 -sx2) * scale);

            if ((sy2 < fsy2) &&  (sx1 > fsx1))
                out = out + src(sy2, (sx1 - 1)) * ((fsy2 -sy2) * (sx1 -fsx1) * scale);

            return saturate_cast<elem_type>(out);
        }

        Ptr2D src;
        float scale_x, scale_y;
        int width, haight;
    };
}}} // namespace cv { namespace cuda { namespace cudev

//! @endcond

#endif // OPENCV_CUDA_FILTERS_HPP
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

- **LinearFilter**: A class/struct defined in this file
- **AreaFilter**: A class/struct defined in this file
- **IntegerAreaFilter**: A class/struct defined in this file
- **PointFilter**: A class/struct defined in this file
- **CubicFilter**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_CUDA_FILTERS_HPP()**: A function/method defined in this file
- **float()**: A function/method defined in this file
- **typename()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `vec_traits.hpp`
- `saturate_cast.hpp`
- `vec_math.hpp`
- `nppdefs.h`
- `type_traits.hpp`

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

