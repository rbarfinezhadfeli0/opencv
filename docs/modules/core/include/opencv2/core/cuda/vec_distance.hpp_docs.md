# Documentation for `modules/core/include/opencv2/core/cuda/vec_distance.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cuda/vec_distance.hpp`
- **File Name**: `vec_distance.hpp`
- **File Size**: 7,863 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/cuda/vec_distance.hpp](../../../../../../modules/core/include/opencv2/core/cuda/vec_distance.hpp)

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

#ifndef OPENCV_CUDA_VEC_DISTANCE_HPP
#define OPENCV_CUDA_VEC_DISTANCE_HPP

#include "reduce.hpp"
#include "functional.hpp"
#include "detail/vec_distance_detail.hpp"

/** @file
 * @deprecated Use @ref cudev instead.
 */

//! @cond IGNORED

namespace cv { namespace cuda { namespace device
{
    template <typename T> struct L1Dist
    {
        typedef int value_type;
        typedef int result_type;

        __device__ __forceinline__ L1Dist() : mySum(0) {}

        __device__ __forceinline__ void reduceIter(int val1, int val2)
        {
            mySum = __sad(val1, val2, mySum);
        }

        template <int THREAD_DIM> __device__ __forceinline__ void reduceAll(int* smem, int tid)
        {
            reduce<THREAD_DIM>(smem, mySum, tid, plus<int>());
        }

        __device__ __forceinline__ operator int() const
        {
            return mySum;
        }

        int mySum;
    };
    template <> struct L1Dist<float>
    {
        typedef float value_type;
        typedef float result_type;

        __device__ __forceinline__ L1Dist() : mySum(0.0f) {}

        __device__ __forceinline__ void reduceIter(float val1, float val2)
        {
            mySum += ::fabs(val1 - val2);
        }

        template <int THREAD_DIM> __device__ __forceinline__ void reduceAll(float* smem, int tid)
        {
            reduce<THREAD_DIM>(smem, mySum, tid, plus<float>());
        }

        __device__ __forceinline__ operator float() const
        {
            return mySum;
        }

        float mySum;
    };

    struct L2Dist
    {
        typedef float value_type;
        typedef float result_type;

        __device__ __forceinline__ L2Dist() : mySum(0.0f) {}

        __device__ __forceinline__ void reduceIter(float val1, float val2)
        {
            float reg = val1 - val2;
            mySum += reg * reg;
        }

        template <int THREAD_DIM> __device__ __forceinline__ void reduceAll(float* smem, int tid)
        {
            reduce<THREAD_DIM>(smem, mySum, tid, plus<float>());
        }

        __device__ __forceinline__ operator float() const
        {
            return sqrtf(mySum);
        }

        float mySum;
    };

    struct HammingDist
    {
        typedef int value_type;
        typedef int result_type;

        __device__ __forceinline__ HammingDist() : mySum(0) {}

        __device__ __forceinline__ void reduceIter(int val1, int val2)
        {
            mySum += __popc(val1 ^ val2);
        }

        template <int THREAD_DIM> __device__ __forceinline__ void reduceAll(int* smem, int tid)
        {
            reduce<THREAD_DIM>(smem, mySum, tid, plus<int>());
        }

        __device__ __forceinline__ operator int() const
        {
            return mySum;
        }

        int mySum;
    };

    // calc distance between two vectors in global memory
    template <int THREAD_DIM, typename Dist, typename T1, typename T2>
    __device__ void calcVecDiffGlobal(const T1* vec1, const T2* vec2, int len, Dist& dist, typename Dist::result_type* smem, int tid)
    {
        for (int i = tid; i < len; i += THREAD_DIM)
        {
            T1 val1;
            ForceGlob<T1>::Load(vec1, i, val1);

            T2 val2;
            ForceGlob<T2>::Load(vec2, i, val2);

            dist.reduceIter(val1, val2);
        }

        dist.reduceAll<THREAD_DIM>(smem, tid);
    }

    // calc distance between two vectors, first vector is cached in register or shared memory, second vector is in global memory
    template <int THREAD_DIM, int MAX_LEN, bool LEN_EQ_MAX_LEN, typename Dist, typename T1, typename T2>
    __device__ __forceinline__ void calcVecDiffCached(const T1* vecCached, const T2* vecGlob, int len, Dist& dist, typename Dist::result_type* smem, int tid)
    {
        vec_distance_detail::VecDiffCachedCalculator<THREAD_DIM, MAX_LEN, LEN_EQ_MAX_LEN>::calc(vecCached, vecGlob, len, dist, tid);

        dist.reduceAll<THREAD_DIM>(smem, tid);
    }

    // calc distance between two vectors in global memory
    template <int THREAD_DIM, typename T1> struct VecDiffGlobal
    {
        explicit __device__ __forceinline__ VecDiffGlobal(const T1* vec1_, int = 0, void* = 0, int = 0, int = 0)
        {
            vec1 = vec1_;
        }

        template <typename T2, typename Dist>
        __device__ __forceinline__ void calc(const T2* vec2, int len, Dist& dist, typename Dist::result_type* smem, int tid) const
        {
            calcVecDiffGlobal<THREAD_DIM>(vec1, vec2, len, dist, smem, tid);
        }

        const T1* vec1;
    };

    // calc distance between two vectors, first vector is cached in register memory, second vector is in global memory
    template <int THREAD_DIM, int MAX_LEN, bool LEN_EQ_MAX_LEN, typename U> struct VecDiffCachedRegister
    {
        template <typename T1> __device__ __forceinline__ VecDiffCachedRegister(const T1* vec1, int len, U* smem, int glob_tid, int tid)
        {
            if (glob_tid < len)
                smem[glob_tid] = vec1[glob_tid];
            __syncthreads();

            U* vec1ValsPtr = vec1Vals;

            #pragma unroll
            for (int i = tid; i < MAX_LEN; i += THREAD_DIM)
                *vec1ValsPtr++ = smem[i];

            __syncthreads();
        }

        template <typename T2, typename Dist>
        __device__ __forceinline__ void calc(const T2* vec2, int len, Dist& dist, typename Dist::result_type* smem, int tid) const
        {
            calcVecDiffCached<THREAD_DIM, MAX_LEN, LEN_EQ_MAX_LEN>(vec1Vals, vec2, len, dist, smem, tid);
        }

        U vec1Vals[MAX_LEN / THREAD_DIM];
    };
}}} // namespace cv { namespace cuda { namespace cudev

//! @endcond

#endif // OPENCV_CUDA_VEC_DISTANCE_HPP
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

- **L2Dist**: A class/struct defined in this file
- **L1Dist**: A class/struct defined in this file
- **HammingDist**: A class/struct defined in this file
- **VecDiffGlobal**: A class/struct defined in this file
- **VecDiffCachedRegister**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_CUDA_VEC_DISTANCE_HPP()**: A function/method defined in this file
- **int()**: A function/method defined in this file
- **float()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `detail/vec_distance_detail.hpp`
- `functional.hpp`
- `reduce.hpp`

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

