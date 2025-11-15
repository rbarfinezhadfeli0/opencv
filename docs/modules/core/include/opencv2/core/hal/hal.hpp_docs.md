# Documentation for `modules/core/include/opencv2/core/hal/hal.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/hal/hal.hpp`
- **File Name**: `hal.hpp`
- **File Size**: 20,380 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/hal/hal.hpp](../../../../../../modules/core/include/opencv2/core/hal/hal.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/hal` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2015, Itseez Inc., all rights reserved.
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

#ifndef OPENCV_HAL_HPP
#define OPENCV_HAL_HPP

#include "opencv2/core/cvdef.h"
#include "opencv2/core/cvstd.hpp"
#include "opencv2/core/hal/interface.h"

namespace cv { namespace hal {

//! @addtogroup core_hal_functions
//! @{

CV_EXPORTS int normHamming(const uchar* a, int n);
CV_EXPORTS int normHamming(const uchar* a, const uchar* b, int n);

CV_EXPORTS int normHamming(const uchar* a, int n, int cellSize);
CV_EXPORTS int normHamming(const uchar* a, const uchar* b, int n, int cellSize);

CV_EXPORTS int LU32f(float* A, size_t astep, int m, float* b, size_t bstep, int n);
CV_EXPORTS int LU64f(double* A, size_t astep, int m, double* b, size_t bstep, int n);
CV_EXPORTS bool Cholesky32f(float* A, size_t astep, int m, float* b, size_t bstep, int n);
CV_EXPORTS bool Cholesky64f(double* A, size_t astep, int m, double* b, size_t bstep, int n);
CV_EXPORTS void SVD32f(float* At, size_t astep, float* W, float* U, size_t ustep, float* Vt, size_t vstep, int m, int n, int flags);
CV_EXPORTS void SVD64f(double* At, size_t astep, double* W, double* U, size_t ustep, double* Vt, size_t vstep, int m, int n, int flags);
CV_EXPORTS int QR32f(float* A, size_t astep, int m, int n, int k, float* b, size_t bstep, float* hFactors);
CV_EXPORTS int QR64f(double* A, size_t astep, int m, int n, int k, double* b, size_t bstep, double* hFactors);

CV_EXPORTS void gemm32f(const float* src1, size_t src1_step, const float* src2, size_t src2_step,
                        float alpha, const float* src3, size_t src3_step, float beta, float* dst, size_t dst_step,
                        int m_a, int n_a, int n_d, int flags);
CV_EXPORTS void gemm64f(const double* src1, size_t src1_step, const double* src2, size_t src2_step,
                        double alpha, const double* src3, size_t src3_step, double beta, double* dst, size_t dst_step,
                        int m_a, int n_a, int n_d, int flags);
CV_EXPORTS void gemm32fc(const float* src1, size_t src1_step, const float* src2, size_t src2_step,
                        float alpha, const float* src3, size_t src3_step, float beta, float* dst, size_t dst_step,
                        int m_a, int n_a, int n_d, int flags);
CV_EXPORTS void gemm64fc(const double* src1, size_t src1_step, const double* src2, size_t src2_step,
                        double alpha, const double* src3, size_t src3_step, double beta, double* dst, size_t dst_step,
                        int m_a, int n_a, int n_d, int flags);

CV_EXPORTS int normL1_(const uchar* a, const uchar* b, int n);
CV_EXPORTS float normL1_(const float* a, const float* b, int n);
CV_EXPORTS float normL2Sqr_(const float* a, const float* b, int n);

CV_EXPORTS void exp32f(const float* src, float* dst, int n);
CV_EXPORTS void exp64f(const double* src, double* dst, int n);
CV_EXPORTS void log32f(const float* src, float* dst, int n);
CV_EXPORTS void log64f(const double* src, double* dst, int n);

CV_EXPORTS void cartToPolar32f(const float* x, const float* y, float* mag, float* angle, int n, bool angleInDegrees);
CV_EXPORTS void cartToPolar64f(const double* x, const double* y, double* mag, double* angle, int n, bool angleInDegrees);
CV_EXPORTS void fastAtan32f(const float* y, const float* x, float* dst, int n, bool angleInDegrees);
CV_EXPORTS void fastAtan64f(const double* y, const double* x, double* dst, int n, bool angleInDegrees);
CV_EXPORTS void magnitude32f(const float* x, const float* y, float* dst, int n);
CV_EXPORTS void magnitude64f(const double* x, const double* y, double* dst, int n);
CV_EXPORTS void polarToCart32f(const float* mag, const float* angle, float* x, float* y, int n, bool angleInDegrees);
CV_EXPORTS void polarToCart64f(const double* mag, const double* angle, double* x, double* y, int n, bool angleInDegrees);
CV_EXPORTS void sqrt32f(const float* src, float* dst, int len);
CV_EXPORTS void sqrt64f(const double* src, double* dst, int len);
CV_EXPORTS void invSqrt32f(const float* src, float* dst, int len);
CV_EXPORTS void invSqrt64f(const double* src, double* dst, int len);

CV_EXPORTS void split8u(const uchar* src, uchar** dst, int len, int cn );
CV_EXPORTS void split16u(const ushort* src, ushort** dst, int len, int cn );
CV_EXPORTS void split32s(const int* src, int** dst, int len, int cn );
CV_EXPORTS void split64s(const int64* src, int64** dst, int len, int cn );

CV_EXPORTS void merge8u(const uchar** src, uchar* dst, int len, int cn );
CV_EXPORTS void merge16u(const ushort** src, ushort* dst, int len, int cn );
CV_EXPORTS void merge32s(const int** src, int* dst, int len, int cn );
CV_EXPORTS void merge64s(const int64** src, int64* dst, int len, int cn );

CV_EXPORTS void add8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* );
CV_EXPORTS void add64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* );

CV_EXPORTS void sub8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* );
CV_EXPORTS void sub64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* );

CV_EXPORTS void max8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* );
CV_EXPORTS void max64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* );

CV_EXPORTS void min8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* );
CV_EXPORTS void min64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* );

CV_EXPORTS void absdiff8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* );
CV_EXPORTS void absdiff64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* );

CV_EXPORTS void and8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void or8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void xor8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );
CV_EXPORTS void not8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* );

CV_EXPORTS void cmp8u(const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp8s(const schar* src1, size_t step1, const schar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp16u(const ushort* src1, size_t step1, const ushort* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp16s(const short* src1, size_t step1, const short* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp32s(const int* src1, size_t step1, const int* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp32f(const float* src1, size_t step1, const float* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);
CV_EXPORTS void cmp64f(const double* src1, size_t step1, const double* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _cmpop);

CV_EXPORTS void mul8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void mul64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* scale);

CV_EXPORTS void div8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void div64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* scale);

CV_EXPORTS void recip8u( const uchar *, size_t, const uchar * src2, size_t step2, uchar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip8s( const schar *, size_t, const schar * src2, size_t step2, schar* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip16u( const ushort *, size_t, const ushort * src2, size_t step2, ushort* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip16s( const short *, size_t, const short * src2, size_t step2, short* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip32s( const int *, size_t, const int * src2, size_t step2, int* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip32f( const float *, size_t, const float * src2, size_t step2, float* dst, size_t step, int width, int height, void* scale);
CV_EXPORTS void recip64f( const double *, size_t, const double * src2, size_t step2, double* dst, size_t step, int width, int height, void* scale);

CV_EXPORTS void addWeighted8u( const uchar* src1, size_t step1, const uchar* src2, size_t step2, uchar* dst, size_t step, int width, int height, void* _scalars );
CV_EXPORTS void addWeighted8s( const schar* src1, size_t step1, const schar* src2, size_t step2, schar* dst, size_t step, int width, int height, void* scalars );
CV_EXPORTS void addWeighted16u( const ushort* src1, size_t step1, const ushort* src2, size_t step2, ushort* dst, size_t step, int width, int height, void* scalars );
CV_EXPORTS void addWeighted16s( const short* src1, size_t step1, const short* src2, size_t step2, short* dst, size_t step, int width, int height, void* scalars );
CV_EXPORTS void addWeighted32s( const int* src1, size_t step1, const int* src2, size_t step2, int* dst, size_t step, int width, int height, void* scalars );
CV_EXPORTS void addWeighted32f( const float* src1, size_t step1, const float* src2, size_t step2, float* dst, size_t step, int width, int height, void* scalars );
CV_EXPORTS void addWeighted64f( const double* src1, size_t step1, const double* src2, size_t step2, double* dst, size_t step, int width, int height, void* scalars );

CV_EXPORTS void cvt16f32f( const hfloat* src, float* dst, int len );
CV_EXPORTS void cvt32f16f( const float* src, hfloat* dst, int len );

CV_EXPORTS void addRNGBias32f( float* arr, const float* scaleBiasPairs, int len );
CV_EXPORTS void addRNGBias64f( double* arr, const double* scaleBiasPairs, int len );

struct CV_EXPORTS DFT1D
{
    static Ptr<DFT1D> create(int len, int count, int depth, int flags, bool * useBuffer = 0);
    virtual void apply(const uchar *src, uchar *dst) = 0;
    virtual ~DFT1D() {}
};

struct CV_EXPORTS DFT2D
{
    static Ptr<DFT2D> create(int width, int height, int depth,
                             int src_channels, int dst_channels,
                             int flags, int nonzero_rows = 0);
    virtual void apply(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step) = 0;
    virtual ~DFT2D() {}
};

struct CV_EXPORTS DCT2D
{
    static Ptr<DCT2D> create(int width, int height, int depth, int flags);
    virtual void apply(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step) = 0;
    virtual ~DCT2D() {}
};

//! @} core_hal

//=============================================================================
// for binary compatibility with 3.0

//! @cond IGNORED

CV_EXPORTS int LU(float* A, size_t astep, int m, float* b, size_t bstep, int n);
CV_EXPORTS int LU(double* A, size_t astep, int m, double* b, size_t bstep, int n);
CV_EXPORTS bool Cholesky(float* A, size_t astep, int m, float* b, size_t bstep, int n);
CV_EXPORTS bool Cholesky(double* A, size_t astep, int m, double* b, size_t bstep, int n);

CV_EXPORTS void exp(const float* src, float* dst, int n);
CV_EXPORTS void exp(const double* src, double* dst, int n);
CV_EXPORTS void log(const float* src, float* dst, int n);
CV_EXPORTS void log(const double* src, double* dst, int n);

CV_EXPORTS void fastAtan2(const float* y, const float* x, float* dst, int n, bool angleInDegrees);
CV_EXPORTS void magnitude(const float* x, const float* y, float* dst, int n);
CV_EXPORTS void magnitude(const double* x, const double* y, double* dst, int n);
CV_EXPORTS void sqrt(const float* src, float* dst, int len);
CV_EXPORTS void sqrt(const double* src, double* dst, int len);
CV_EXPORTS void invSqrt(const float* src, float* dst, int len);
CV_EXPORTS void invSqrt(const double* src, double* dst, int len);

//! @endcond

}} //cv::hal

#endif //OPENCV_HAL_HPP
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

### Functions and Methods

- **OPENCV_HAL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/interface.h`
- `opencv2/core/cvstd.hpp`
- `opencv2/core/cvdef.h`

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

