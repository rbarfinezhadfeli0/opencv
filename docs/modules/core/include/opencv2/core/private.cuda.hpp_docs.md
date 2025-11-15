# Documentation for `modules/core/include/opencv2/core/private.cuda.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/private.cuda.hpp`
- **File Name**: `private.cuda.hpp`
- **File Size**: 8,490 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/private.cuda.hpp](../../../../../modules/core/include/opencv2/core/private.cuda.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
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

#ifndef OPENCV_CORE_PRIVATE_CUDA_HPP
#define OPENCV_CORE_PRIVATE_CUDA_HPP

#ifndef __OPENCV_BUILD
#  error this is a private header which should not be used from outside of the OpenCV library
#endif

#include "cvconfig.h"

#include "opencv2/core/cvdef.h"
#include "opencv2/core/base.hpp"

#include "opencv2/core/cuda.hpp"
#include "opencv2/core/private/cuda_stubs.hpp"

#ifdef HAVE_CUDA
#  include <cuda.h>
#  include <cuda_runtime.h>
#  if defined(__CUDACC_VER_MAJOR__) && (8 <= __CUDACC_VER_MAJOR__)
#    if defined (__GNUC__) && !defined(__CUDACC__)
#     pragma GCC diagnostic push
#     pragma GCC diagnostic ignored "-Wstrict-aliasing"
#     include <cuda_fp16.h>
#     pragma GCC diagnostic pop
#    else
#     include <cuda_fp16.h>
#    endif
#  endif // defined(__CUDACC_VER_MAJOR__) && (8 <= __CUDACC_VER_MAJOR__)
#  include <npp.h>
#  include "opencv2/core/cuda_stream_accessor.hpp"
#  include "opencv2/core/cuda/common.hpp"

# ifndef NPP_VERSION
#  define NPP_VERSION (NPP_VERSION_MAJOR * 1000 + NPP_VERSION_MINOR * 100 + NPP_VERSION_BUILD)
# endif

#  define CUDART_MINIMUM_REQUIRED_VERSION 6050

#  if (CUDART_VERSION < CUDART_MINIMUM_REQUIRED_VERSION)
#    error "Insufficient Cuda Runtime library version, please update it."
#  endif

#endif

//! @cond IGNORED

namespace cv { namespace cuda {
    CV_EXPORTS cv::String getNppErrorMessage(int code);
    CV_EXPORTS cv::String getCudaDriverApiErrorMessage(int code);

    CV_EXPORTS GpuMat getInputMat(InputArray _src, Stream& stream);

    CV_EXPORTS GpuMat getOutputMat(OutputArray _dst, int rows, int cols, int type, Stream& stream);
    static inline GpuMat getOutputMat(OutputArray _dst, Size size, int type, Stream& stream)
    {
        return getOutputMat(_dst, size.height, size.width, type, stream);
    }

    CV_EXPORTS void syncOutput(const GpuMat& dst, OutputArray _dst, Stream& stream);
}}

#ifdef HAVE_CUDA

#define nppSafeSetStream(oldStream, newStream) { if(oldStream != newStream) { cudaStreamSynchronize(oldStream); nppSetStream(newStream); } }

namespace cv { namespace cuda
{
    static inline void checkNppError(int code, const char* file, const int line, const char* func)
    {
        if (code < 0)
            cv::error(cv::Error::GpuApiCallError, getNppErrorMessage(code), func, file, line);
    }

    static inline void checkCudaDriverApiError(int code, const char* file, const int line, const char* func)
    {
        if (code != CUDA_SUCCESS)
            cv::error(cv::Error::GpuApiCallError, getCudaDriverApiErrorMessage(code), func, file, line);
    }

    template<int n> struct NPPTypeTraits;
    template<> struct NPPTypeTraits<CV_8U>  { typedef Npp8u npp_type; };
    template<> struct NPPTypeTraits<CV_8S>  { typedef Npp8s npp_type; };
    template<> struct NPPTypeTraits<CV_16U> { typedef Npp16u npp_type; };
    template<> struct NPPTypeTraits<CV_16S> { typedef Npp16s npp_type; };
    template<> struct NPPTypeTraits<CV_32S> { typedef Npp32s npp_type; };
    template<> struct NPPTypeTraits<CV_32F> { typedef Npp32f npp_type; };
    template<> struct NPPTypeTraits<CV_64F> { typedef Npp64f npp_type; };

#define nppSafeCall(expr)  cv::cuda::checkNppError(expr, __FILE__, __LINE__, CV_Func)
// NppStreamContext is introduced in NPP version 10100 included in CUDA toolkit 10.1 (CUDA_VERSION == 10010) however not all of the NPP functions called internally by OpenCV
// - have an NppStreamContext argument (e.g. nppiHistogramEvenGetBufferSize_8u_C1R_Ctx in CUDA 12.3) and/or
// - have a corresponding function in the supplied library (e.g. nppiEvenLevelsHost_32s_Ctx is not present in nppist.lib or libnppist.so as of CUDA 12.6)
// Because support for these functions has gradually been introduced without being mentioned in the release notes this flag is set to a version of NPP (version 12205 included in CUDA toolkit 12.4) which is known to work.
#define USE_NPP_STREAM_CTX NPP_VERSION >= 12205
#if USE_NPP_STREAM_CTX
    class NppStreamHandler
    {
    public:
        inline explicit NppStreamHandler(cudaStream_t newStream)
        {
            nppStreamContext = {};
            #if CUDA_VERSION < 12090
                nppSafeCall(nppGetStreamContext(&nppStreamContext));
            #else
                int device = 0;
                cudaSafeCall(cudaGetDevice(&device));

                cudaDeviceProp prop{};
                cudaSafeCall(cudaGetDeviceProperties(&prop, device));

                nppStreamContext.nCudaDeviceId = device;
                nppStreamContext.nMultiProcessorCount = prop.multiProcessorCount;
                nppStreamContext.nMaxThreadsPerMultiProcessor = prop.maxThreadsPerMultiProcessor;
                nppStreamContext.nMaxThreadsPerBlock = prop.maxThreadsPerBlock;
                nppStreamContext.nSharedMemPerBlock = prop.sharedMemPerBlock;
                nppStreamContext.nCudaDevAttrComputeCapabilityMajor = prop.major;
                nppStreamContext.nCudaDevAttrComputeCapabilityMinor = prop.minor;
            #endif
            nppStreamContext.hStream = newStream;
            cudaSafeCall(cudaStreamGetFlags(nppStreamContext.hStream, &nppStreamContext.nStreamFlags));
        }

        inline explicit NppStreamHandler(Stream& newStream) : NppStreamHandler(StreamAccessor::getStream(newStream)) {}

        inline operator NppStreamContext() const {
            return nppStreamContext;
        }

        inline NppStreamContext get() { return nppStreamContext; }

    private:
        NppStreamContext nppStreamContext;
    };
#else
    class NppStreamHandler
    {
    public:
        inline explicit NppStreamHandler(Stream& newStream)
        {
            oldStream = nppGetStream();
            nppSafeSetStream(oldStream, StreamAccessor::getStream(newStream));
        }

        inline explicit NppStreamHandler(cudaStream_t newStream)
        {
            oldStream = nppGetStream();
            nppSafeSetStream(oldStream, newStream);
        }

        inline ~NppStreamHandler()
        {
            nppSafeSetStream(nppGetStream(), oldStream);
        }

    private:
        cudaStream_t oldStream;
    };
#endif
}}

#define cuSafeCall(expr)  cv::cuda::checkCudaDriverApiError(expr, __FILE__, __LINE__, CV_Func)

#endif // HAVE_CUDA

//! @endcond

#endif // OPENCV_CORE_PRIVATE_CUDA_HPP
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

- **NPPTypeTraits**: A class/struct defined in this file
- **NppStreamHandler**: A class/struct defined in this file

### Functions and Methods

- **in()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **OPENCV_CORE_PRIVATE_CUDA_HPP()**: A function/method defined in this file
- **__OPENCV_BUILD()**: A function/method defined in this file
- **Npp8u()**: A function/method defined in this file
- **Npp16s()**: A function/method defined in this file
- **NPP_VERSION()**: A function/method defined in this file
- **Npp32f()**: A function/method defined in this file
- **Npp32s()**: A function/method defined in this file
- **Npp8s()**: A function/method defined in this file
- **Npp64f()**: A function/method defined in this file
- **Npp16u()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cvconfig.h`
- `opencv2/core/cuda.hpp`
- `opencv2/core/base.hpp`
- `opencv2/core/private/cuda_stubs.hpp`
- `opencv2/core/cvdef.h`

**Python Imports:**
- `outside`
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

