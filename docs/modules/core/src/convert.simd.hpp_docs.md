# Documentation for `modules/core/src/convert.simd.hpp`

## File Metadata

- **Full Path**: `modules/core/src/convert.simd.hpp`
- **File Name**: `convert.simd.hpp`
- **File Size**: 11,123 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/src/convert.simd.hpp](../../../modules/core/src/convert.simd.hpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include "precomp.hpp"
#include "convert.hpp"

#if !defined(OPENCV_SUPRESS_WARNING_AVX2_WITHOUT_FP16C) && \
    (defined(__GNUC__) && defined(__AVX2__) && !defined(__F16C__))
#warning "Non-optimal compiler flags: AVX2 without FP16. Generated code is very slow. Consider adding '-mf16c' compiler option."
#endif

namespace cv {
namespace hal {
CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

void cvt16f32f(const hfloat* src, float* dst, int len);
void cvt32f16f(const float* src, hfloat* dst, int len);
void addRNGBias32f(float* arr, const float* scaleBiasPairs, int len);
void addRNGBias64f(double* arr, const double* scaleBiasPairs, int len);

CV_CPU_OPTIMIZATION_NAMESPACE_END
} // namespace cv::hal

CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

BinaryFunc getConvertFunc(int sdepth, int ddepth);

CV_CPU_OPTIMIZATION_NAMESPACE_END

#ifndef CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY

namespace hal {
CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

BinaryFunc getConvertFunc(int sdepth, int ddepth);

void cvt16f32f( const hfloat* src, float* dst, int len )
{
    CV_INSTRUMENT_REGION();
    int j = 0;
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_float32>::vlanes();
    for( ; j < len; j += VECSZ )
    {
        if( j > len - VECSZ )
        {
            if( j == 0 )
                break;
            j = len - VECSZ;
        }
        v_store(dst + j, vx_load_expand(src + j));
    }
#endif
    for( ; j < len; j++ )
        dst[j] = (float)src[j];
}

void cvt32f16f( const float* src, hfloat* dst, int len )
{
    CV_INSTRUMENT_REGION();
    int j = 0;
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_float32>::vlanes();
    for( ; j < len; j += VECSZ )
    {
        if( j > len - VECSZ )
        {
            if( j == 0 )
                break;
            j = len - VECSZ;
        }
        v_pack_store(dst + j, vx_load(src + j));
    }
#endif
    for( ; j < len; j++ )
        dst[j] = hfloat(src[j]);
}

void addRNGBias32f( float* arr, const float* scaleBiasPairs, int len )
{
    CV_INSTRUMENT_REGION();
    // the loop is simple enough, so we let the compiler to vectorize it
    for( int i = 0; i < len; i++ )
        arr[i] += scaleBiasPairs[i*2 + 1];
}

void addRNGBias64f( double* arr, const double* scaleBiasPairs, int len )
{
    CV_INSTRUMENT_REGION();
    // the loop is simple enough, so we let the compiler to vectorize it
    for( int i = 0; i < len; i++ )
        arr[i] += scaleBiasPairs[i*2 + 1];
}

CV_CPU_OPTIMIZATION_NAMESPACE_END
} // namespace cv::hal

// cv::
CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

template<typename _Ts, typename _Td, typename _Twvec> static inline void
cvt_( const _Ts* src, size_t sstep, _Td* dst, size_t dstep, Size size )
{
    sstep /= sizeof(src[0]);
    dstep /= sizeof(dst[0]);

    for( int i = 0; i < size.height; i++, src += sstep, dst += dstep )
    {
        int j = 0;
// Excluding GNU in CV_SIMD_SCALABLE because of "opencv/issues/26936"
#if (CV_SIMD || (CV_SIMD_SCALABLE && !(defined(__GNUC__) && !defined(__clang__))) )
        const int VECSZ = VTraits<_Twvec>::vlanes()*2;
        for( ; j < size.width; j += VECSZ )
        {
            if( j > size.width - VECSZ )
            {
                if( j == 0 || src == (_Ts*)dst )
                    break;
                j = size.width - VECSZ;
            }
            _Twvec v0, v1;
            vx_load_pair_as(src + j, v0, v1);
            v_store_pair_as(dst + j, v0, v1);
        }
#endif
        for( ; j < size.width; j++ )
            dst[j] = saturate_cast<_Td>(src[j]);
    }
}

// in order to reduce the code size, for (16f <-> ...) conversions
// we add a conversion function without loop unrolling
template<typename _Ts, typename _Td, typename _Twvec> static inline void
cvt1_( const _Ts* src, size_t sstep, _Td* dst, size_t dstep, Size size )
{
    sstep /= sizeof(src[0]);
    dstep /= sizeof(dst[0]);

    for( int i = 0; i < size.height; i++, src += sstep, dst += dstep )
    {
        int j = 0;
#if (CV_SIMD || CV_SIMD_SCALABLE)
        const int VECSZ = VTraits<_Twvec>::vlanes();
        for( ; j < size.width; j += VECSZ )
        {
            if( j > size.width - VECSZ )
            {
                if( j == 0 || src == (_Ts*)dst )
                    break;
                j = size.width - VECSZ;
            }
            _Twvec v;
            vx_load_as(src + j, v);
            v_store_as(dst + j, v);
        }
        vx_cleanup();
#endif
        for( ; j < size.width; j++ )
            dst[j] = saturate_cast<_Td>(src[j]);
    }
}

static void cvtCopy( const uchar* src, size_t sstep,
                     uchar* dst, size_t dstep, Size size, size_t elemsize)
{
    size_t len = size.width*elemsize;
    for( int i = 0; i < size.height; i++, src += sstep, dst += dstep )
    {
        memcpy( dst, src, len );
    }
}

#define DEF_CVT_FUNC(suffix, cvtfunc, _Ts, _Td, _Twvec) \
static void cvt##suffix(const uchar* src_, size_t sstep, const uchar*, size_t, \
                        uchar* dst_, size_t dstep, Size size, void*) \
{ \
    CV_INSTRUMENT_REGION(); \
    const _Ts* src = (const _Ts*)src_; \
    _Td* dst = (_Td*)dst_; \
    cvtfunc<_Ts, _Td, _Twvec>(src, sstep, dst, dstep, size); \
}

////////////////////// 8u -> ... ////////////////////////

DEF_CVT_FUNC(8u8s,  cvt_,  uchar, schar,    v_int16)
DEF_CVT_FUNC(8u16u, cvt_,  uchar, ushort,   v_uint16)
DEF_CVT_FUNC(8u16s, cvt_,  uchar, short,    v_int16)
DEF_CVT_FUNC(8u32s, cvt_,  uchar, int,      v_int32)
DEF_CVT_FUNC(8u32f, cvt_,  uchar, float,    v_float32)
DEF_CVT_FUNC(8u64f, cvt_,  uchar, double,   v_int32)
DEF_CVT_FUNC(8u16f, cvt1_, uchar, hfloat, v_float32)

////////////////////// 8s -> ... ////////////////////////

DEF_CVT_FUNC(8s8u,  cvt_,  schar, uchar,    v_int16)
DEF_CVT_FUNC(8s16u, cvt_,  schar, ushort,   v_uint16)
DEF_CVT_FUNC(8s16s, cvt_,  schar, short,    v_int16)
DEF_CVT_FUNC(8s32s, cvt_,  schar, int,      v_int32)
DEF_CVT_FUNC(8s32f, cvt_,  schar, float,    v_float32)
DEF_CVT_FUNC(8s64f, cvt_,  schar, double,   v_int32)
DEF_CVT_FUNC(8s16f, cvt1_, schar, hfloat, v_float32)

////////////////////// 16u -> ... ////////////////////////

DEF_CVT_FUNC(16u8u,  cvt_, ushort, uchar,  v_uint16)
DEF_CVT_FUNC(16u8s,  cvt_, ushort, schar,  v_uint16)
DEF_CVT_FUNC(16u16s, cvt_, ushort, short,  v_int32)
DEF_CVT_FUNC(16u32s, cvt_, ushort, int,    v_int32)
DEF_CVT_FUNC(16u32f, cvt_, ushort, float,  v_float32)
DEF_CVT_FUNC(16u64f, cvt_, ushort, double, v_int32)
DEF_CVT_FUNC(16u16f, cvt1_,ushort, hfloat, v_float32)

////////////////////// 16s -> ... ////////////////////////

DEF_CVT_FUNC(16s8u,  cvt_, short, uchar,  v_int16)
DEF_CVT_FUNC(16s8s,  cvt_, short, schar,  v_int16)
DEF_CVT_FUNC(16s16u, cvt_, short, ushort, v_int32)
DEF_CVT_FUNC(16s32s, cvt_, short, int,    v_int32)
DEF_CVT_FUNC(16s32f, cvt_, short, float,  v_float32)
DEF_CVT_FUNC(16s64f, cvt_, short, double, v_int32)
DEF_CVT_FUNC(16s16f, cvt1_,short, hfloat, v_float32)

////////////////////// 32s -> ... ////////////////////////

DEF_CVT_FUNC(32s8u,  cvt_, int, uchar,  v_int32)
DEF_CVT_FUNC(32s8s,  cvt_, int, schar,  v_int32)
DEF_CVT_FUNC(32s16u, cvt_, int, ushort, v_int32)
DEF_CVT_FUNC(32s16s, cvt_, int, short,  v_int32)
DEF_CVT_FUNC(32s32f, cvt_, int, float,  v_float32)
DEF_CVT_FUNC(32s64f, cvt_, int, double, v_int32)
DEF_CVT_FUNC(32s16f, cvt1_,int, hfloat, v_float32)

////////////////////// 32f -> ... ////////////////////////

DEF_CVT_FUNC(32f8u,  cvt_, float, uchar,  v_float32)
DEF_CVT_FUNC(32f8s,  cvt_, float, schar,  v_float32)
DEF_CVT_FUNC(32f16u, cvt_, float, ushort, v_float32)
DEF_CVT_FUNC(32f16s, cvt_, float, short,  v_float32)
DEF_CVT_FUNC(32f32s, cvt_, float, int,    v_float32)
DEF_CVT_FUNC(32f64f, cvt_, float, double, v_float32)
DEF_CVT_FUNC(32f16f, cvt1_,float, hfloat, v_float32)

////////////////////// 64f -> ... ////////////////////////

DEF_CVT_FUNC(64f8u,  cvt_, double, uchar,  v_int32)
DEF_CVT_FUNC(64f8s,  cvt_, double, schar,  v_int32)
DEF_CVT_FUNC(64f16u, cvt_, double, ushort, v_int32)
DEF_CVT_FUNC(64f16s, cvt_, double, short,  v_int32)
DEF_CVT_FUNC(64f32s, cvt_, double, int,    v_int32)
DEF_CVT_FUNC(64f32f, cvt_, double, float,  v_float32)
DEF_CVT_FUNC(64f16f, cvt1_,double, hfloat, v_float32)

////////////////////// 16f -> ... ////////////////////////

DEF_CVT_FUNC(16f8u,  cvt_,  hfloat, uchar,  v_float32)
DEF_CVT_FUNC(16f8s,  cvt_,  hfloat, schar,  v_float32)
DEF_CVT_FUNC(16f16u, cvt1_, hfloat, ushort, v_float32)
DEF_CVT_FUNC(16f16s, cvt1_, hfloat, short,  v_float32)
DEF_CVT_FUNC(16f32s, cvt1_, hfloat, int,    v_float32)
DEF_CVT_FUNC(16f32f, cvt1_, hfloat, float,  v_float32)
DEF_CVT_FUNC(16f64f, cvt1_, hfloat, double, v_float32)

///////////// "conversion" w/o conversion ///////////////

static void cvt8u(const uchar* src, size_t sstep, const uchar*, size_t, uchar* dst, size_t dstep, Size size, void*)
{ CV_INSTRUMENT_REGION(); cvtCopy(src, sstep, dst, dstep, size, 1); }

static void cvt16u(const uchar* src, size_t sstep, const uchar*, size_t, uchar* dst, size_t dstep, Size size, void*)
{ CV_INSTRUMENT_REGION(); cvtCopy((const uchar*)src, sstep, (uchar*)dst, dstep, size, 2); }

static void cvt32s(const uchar* src, size_t sstep, const uchar*, size_t, uchar* dst, size_t dstep, Size size, void*)
{ CV_INSTRUMENT_REGION(); cvtCopy((const uchar*)src, sstep, (uchar*)dst, dstep, size, 4); }

static void cvt64s(const uchar* src, size_t sstep, const uchar*, size_t, uchar* dst, size_t dstep, Size size, void*)
{ CV_INSTRUMENT_REGION(); cvtCopy((const uchar*)src, sstep, (uchar*)dst, dstep, size, 8); }

BinaryFunc getConvertFunc(int sdepth, int ddepth)
{
    static BinaryFunc cvtTab[CV_DEPTH_MAX][CV_DEPTH_MAX] =
    {
        {
            (cvt8u), (cvt8s8u), (cvt16u8u),
            (cvt16s8u), (cvt32s8u), (cvt32f8u),
            (cvt64f8u), (cvt16f8u)
        },
        {
            (cvt8u8s), cvt8u, (cvt16u8s),
            (cvt16s8s), (cvt32s8s), (cvt32f8s),
            (cvt64f8s), (cvt16f8s)
        },
        {
            (cvt8u16u), (cvt8s16u), cvt16u,
            (cvt16s16u), (cvt32s16u), (cvt32f16u),
            (cvt64f16u), (cvt16f16u)
        },
        {
            (cvt8u16s), (cvt8s16s), (cvt16u16s),
            cvt16u, (cvt32s16s), (cvt32f16s),
            (cvt64f16s), (cvt16f16s)
        },
        {
            (cvt8u32s), (cvt8s32s), (cvt16u32s),
            (cvt16s32s), cvt32s, (cvt32f32s),
            (cvt64f32s), (cvt16f32s)
        },
        {
            (cvt8u32f), (cvt8s32f), (cvt16u32f),
            (cvt16s32f), (cvt32s32f), cvt32s,
            (cvt64f32f), (cvt16f32f)
        },
        {
            (cvt8u64f), (cvt8s64f), (cvt16u64f),
            (cvt16s64f), (cvt32s64f), (cvt32f64f),
            (cvt64s), (cvt16f64f)
        },
        {
            (cvt8u16f), (cvt8s16f), (cvt16u16f), (cvt16s16f),
            (cvt32s16f), (cvt32f16f), (cvt64f16f), (cvt16u)
        }
    };
    return cvtTab[CV_MAT_DEPTH(ddepth)][CV_MAT_DEPTH(sdepth)];
}

CV_CPU_OPTIMIZATION_NAMESPACE_END
#endif
} // namespace
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

### Functions and Methods

- **without()**: A function/method defined in this file
- **CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `convert.hpp`


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

