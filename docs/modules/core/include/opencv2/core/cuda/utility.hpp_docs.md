# Documentation for `modules/core/include/opencv2/core/cuda/utility.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cuda/utility.hpp`
- **File Name**: `utility.hpp`
- **File Size**: 7,989 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/cuda/utility.hpp](../../../../../../modules/core/include/opencv2/core/cuda/utility.hpp)

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

#ifndef OPENCV_CUDA_UTILITY_HPP
#define OPENCV_CUDA_UTILITY_HPP

#include "saturate_cast.hpp"
#include "datamov_utils.hpp"

/** @file
 * @deprecated Use @ref cudev instead.
 */

//! @cond IGNORED

namespace cv { namespace cuda { namespace device
{
    struct CV_EXPORTS ThrustAllocator
    {
        typedef uchar value_type;
        virtual ~ThrustAllocator();
        virtual __device__ __host__ uchar* allocate(size_t numBytes) = 0;
        virtual __device__ __host__ void deallocate(uchar* ptr, size_t numBytes) = 0;
        static ThrustAllocator& getAllocator();
        static void setAllocator(ThrustAllocator* allocator);
    };
    #define OPENCV_CUDA_LOG_WARP_SIZE        (5)
    #define OPENCV_CUDA_WARP_SIZE            (1 << OPENCV_CUDA_LOG_WARP_SIZE)
    #define OPENCV_CUDA_LOG_MEM_BANKS        ((__CUDA_ARCH__ >= 200) ? 5 : 4) // 32 banks on fermi, 16 on tesla
    #define OPENCV_CUDA_MEM_BANKS            (1 << OPENCV_CUDA_LOG_MEM_BANKS)

    ///////////////////////////////////////////////////////////////////////////////
    // swap

    template <typename T> void __device__ __host__ __forceinline__ swap(T& a, T& b)
    {
        const T temp = a;
        a = b;
        b = temp;
    }

    ///////////////////////////////////////////////////////////////////////////////
    // Mask Reader

    struct SingleMask
    {
        explicit __host__ __device__ __forceinline__ SingleMask(PtrStepb mask_) : mask(mask_) {}
        __host__ __device__ __forceinline__ SingleMask(const SingleMask& mask_): mask(mask_.mask){}

        __device__ __forceinline__ bool operator()(int y, int x) const
        {
            return mask.ptr(y)[x] != 0;
        }

        PtrStepb mask;
    };

    struct SingleMaskChannels
    {
        __host__ __device__ __forceinline__ SingleMaskChannels(PtrStepb mask_, int channels_)
        : mask(mask_), channels(channels_) {}
        __host__ __device__ __forceinline__ SingleMaskChannels(const SingleMaskChannels& mask_)
            :mask(mask_.mask), channels(mask_.channels){}

        __device__ __forceinline__ bool operator()(int y, int x) const
        {
            return mask.ptr(y)[x / channels] != 0;
        }

        PtrStepb mask;
        int channels;
    };

    struct MaskCollection
    {
        explicit __host__ __device__ __forceinline__ MaskCollection(PtrStepb* maskCollection_)
            : maskCollection(maskCollection_) {}

        __device__ __forceinline__ MaskCollection(const MaskCollection& masks_)
            : maskCollection(masks_.maskCollection), curMask(masks_.curMask){}

        __device__ __forceinline__ void next()
        {
            curMask = *maskCollection++;
        }
        __device__ __forceinline__ void setMask(int z)
        {
            curMask = maskCollection[z];
        }

        __device__ __forceinline__ bool operator()(int y, int x) const
        {
            uchar val;
            return curMask.data == 0 || (ForceGlob<uchar>::Load(curMask.ptr(y), x, val), (val != 0));
        }

        const PtrStepb* maskCollection;
        PtrStepb curMask;
    };

    struct WithOutMask
    {
        __host__ __device__ __forceinline__ WithOutMask(){}
        __host__ __device__ __forceinline__ WithOutMask(const WithOutMask&){}

        __device__ __forceinline__ void next() const
        {
        }
        __device__ __forceinline__ void setMask(int) const
        {
        }

        __device__ __forceinline__ bool operator()(int, int) const
        {
            return true;
        }

        __device__ __forceinline__ bool operator()(int, int, int) const
        {
            return true;
        }

        static __device__ __forceinline__ bool check(int, int)
        {
            return true;
        }

        static __device__ __forceinline__ bool check(int, int, int)
        {
            return true;
        }
    };

    ///////////////////////////////////////////////////////////////////////////////
    // Solve linear system

    // solve 2x2 linear system Ax=b
    template <typename T> __device__ __forceinline__ bool solve2x2(const T A[2][2], const T b[2], T x[2])
    {
        T det = A[0][0] * A[1][1] - A[1][0] * A[0][1];

        if (det != 0)
        {
            double invdet = 1.0 / det;

            x[0] = saturate_cast<T>(invdet * (b[0] * A[1][1] - b[1] * A[0][1]));

            x[1] = saturate_cast<T>(invdet * (A[0][0] * b[1] - A[1][0] * b[0]));

            return true;
        }

        return false;
    }

    // solve 3x3 linear system Ax=b
    template <typename T> __device__ __forceinline__ bool solve3x3(const T A[3][3], const T b[3], T x[3])
    {
        T det = A[0][0] * (A[1][1] * A[2][2] - A[1][2] * A[2][1])
              - A[0][1] * (A[1][0] * A[2][2] - A[1][2] * A[2][0])
              + A[0][2] * (A[1][0] * A[2][1] - A[1][1] * A[2][0]);

        if (det != 0)
        {
            double invdet = 1.0 / det;

            x[0] = saturate_cast<T>(invdet *
                (b[0]    * (A[1][1] * A[2][2] - A[1][2] * A[2][1]) -
                 A[0][1] * (b[1]    * A[2][2] - A[1][2] * b[2]   ) +
                 A[0][2] * (b[1]    * A[2][1] - A[1][1] * b[2]   )));

            x[1] = saturate_cast<T>(invdet *
                (A[0][0] * (b[1]    * A[2][2] - A[1][2] * b[2]   ) -
                 b[0]    * (A[1][0] * A[2][2] - A[1][2] * A[2][0]) +
                 A[0][2] * (A[1][0] * b[2]    - b[1]    * A[2][0])));

            x[2] = saturate_cast<T>(invdet *
                (A[0][0] * (A[1][1] * b[2]    - b[1]    * A[2][1]) -
                 A[0][1] * (A[1][0] * b[2]    - b[1]    * A[2][0]) +
                 b[0]    * (A[1][0] * A[2][1] - A[1][1] * A[2][0])));

            return true;
        }

        return false;
    }
}}} // namespace cv { namespace cuda { namespace cudev

//! @endcond

#endif // OPENCV_CUDA_UTILITY_HPP
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

- **CV_EXPORTS**: A class/struct defined in this file
- **MaskCollection**: A class/struct defined in this file
- **WithOutMask**: A class/struct defined in this file
- **SingleMask**: A class/struct defined in this file
- **SingleMaskChannels**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_CUDA_UTILITY_HPP()**: A function/method defined in this file
- **uchar()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `saturate_cast.hpp`
- `datamov_utils.hpp`

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

