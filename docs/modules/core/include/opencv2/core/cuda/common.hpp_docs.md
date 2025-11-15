# Documentation for `modules/core/include/opencv2/core/cuda/common.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cuda/common.hpp`
- **File Name**: `common.hpp`
- **File Size**: 4,889 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/cuda/common.hpp](../../../../../../modules/core/include/opencv2/core/cuda/common.hpp)

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

#ifndef OPENCV_CUDA_COMMON_HPP
#define OPENCV_CUDA_COMMON_HPP

#include <cuda_runtime.h>
#include "opencv2/core/cuda_types.hpp"
#include "opencv2/core/cvdef.h"
#include "opencv2/core/base.hpp"

/** @file
 * @deprecated Use @ref cudev instead.
 */

//! @cond IGNORED

#ifndef CV_PI_F
    #ifndef CV_PI
        #define CV_PI_F 3.14159265f
    #else
        #define CV_PI_F ((float)CV_PI)
    #endif
#endif

namespace cv { namespace cuda {
    static inline void checkCudaError(cudaError_t err, const char* file, const int line, const char* func)
    {
        if (cudaSuccess != err) {
            cudaGetLastError(); // reset the last stored error to cudaSuccess
            cv::error(cv::Error::GpuApiCallError, cudaGetErrorString(err), func, file, line);
        }
    }
}}

#ifndef cudaSafeCall
    #define cudaSafeCall(expr)  cv::cuda::checkCudaError(expr, __FILE__, __LINE__, CV_Func)
#endif

namespace cv { namespace cuda
{
    template <typename T> static inline bool isAligned(const T* ptr, size_t size)
    {
        return reinterpret_cast<size_t>(ptr) % size == 0;
    }

    static inline bool isAligned(size_t step, size_t size)
    {
        return step % size == 0;
    }
}}

namespace cv { namespace cuda
{
    namespace device
    {
        __host__ __device__ __forceinline__ int divUp(int total, int grain)
        {
            return (total + grain - 1) / grain;
        }

#if (CUDART_VERSION >= 12000)
        template<class T> inline void createTextureObjectPitch2D(cudaTextureObject_t*, PtrStepSz<T>&, const cudaTextureDesc&) {
            CV_Error(cv::Error::GpuNotSupported, "Function removed in CUDA SDK 12"); }
#else
        //TODO: remove from OpenCV 5.x
        template<class T> inline void bindTexture(const textureReference* tex, const PtrStepSz<T>& img)
        {
            cudaChannelFormatDesc desc = cudaCreateChannelDesc<T>();
            cudaSafeCall( cudaBindTexture2D(0, tex, img.ptr(), &desc, img.cols, img.rows, img.step) );
        }

        template<class T> inline void createTextureObjectPitch2D(cudaTextureObject_t* tex, PtrStepSz<T>& img, const cudaTextureDesc& texDesc)
        {
            cudaResourceDesc resDesc;
            memset(&resDesc, 0, sizeof(resDesc));
            resDesc.resType = cudaResourceTypePitch2D;
            resDesc.res.pitch2D.devPtr = static_cast<void*>(img.ptr());
            resDesc.res.pitch2D.height = img.rows;
            resDesc.res.pitch2D.width = img.cols;
            resDesc.res.pitch2D.pitchInBytes = img.step;
            resDesc.res.pitch2D.desc = cudaCreateChannelDesc<T>();

            cudaSafeCall( cudaCreateTextureObject(tex, &resDesc, &texDesc, NULL) );
        }
#endif
    }
}}

//! @endcond

#endif // OPENCV_CUDA_COMMON_HPP
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

- **T**: A class/struct defined in this file

### Functions and Methods

- **CV_PI()**: A function/method defined in this file
- **cudaSafeCall()**: A function/method defined in this file
- **CV_PI_F()**: A function/method defined in this file
- **OPENCV_CUDA_COMMON_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/base.hpp`
- `cuda_runtime.h`
- `opencv2/core/cuda_types.hpp`
- `opencv2/core/cvdef.h`

**Python Imports:**
- `OpenCV`
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

