# Documentation for `modules/core/src/split.simd.hpp`

## File Metadata

- **Full Path**: `modules/core/src/split.simd.hpp`
- **File Name**: `split.simd.hpp`
- **File Size**: 5,967 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/src/split.simd.hpp](../../../modules/core/src/split.simd.hpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html


#include "precomp.hpp"

namespace cv { namespace hal {
CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

void split8u(const uchar* src, uchar** dst, int len, int cn);
void split16u(const ushort* src, ushort** dst, int len, int cn);
void split32s(const int* src, int** dst, int len, int cn);
void split64s(const int64* src, int64** dst, int len, int cn);

#ifndef CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY

#if (CV_SIMD || CV_SIMD_SCALABLE)
// see the comments for vecmerge_ in merge.cpp
template<typename T, typename VecT> static void
vecsplit_( const T* src, T** dst, int len, int cn )
{
    const int VECSZ = VTraits<VecT>::vlanes();
    int i, i0 = 0;
    T* dst0 = dst[0];
    T* dst1 = dst[1];

    int r0 = (int)((size_t)(void*)dst0 % (VECSZ*sizeof(T)));
    int r1 = (int)((size_t)(void*)dst1 % (VECSZ*sizeof(T)));
    int r2 = cn > 2 ? (int)((size_t)(void*)dst[2] % (VECSZ*sizeof(T))) : r0;
    int r3 = cn > 3 ? (int)((size_t)(void*)dst[3] % (VECSZ*sizeof(T))) : r0;

    hal::StoreMode mode = hal::STORE_ALIGNED_NOCACHE;
    if( (r0|r1|r2|r3) != 0 )
    {
        mode = hal::STORE_UNALIGNED;
        if( r0 == r1 && r0 == r2 && r0 == r3 && r0 % sizeof(T) == 0 && len > VECSZ*2 )
            i0 = VECSZ - (r0 / sizeof(T));
    }

    if( cn == 2 )
    {
        for( i = 0; i < len; i += VECSZ )
        {
            if( i > len - VECSZ )
            {
                i = len - VECSZ;
                mode = hal::STORE_UNALIGNED;
            }
            VecT a, b;
            v_load_deinterleave(src + i*cn, a, b);
            v_store(dst0 + i, a, mode);
            v_store(dst1 + i, b, mode);
            if( i < i0 )
            {
                i = i0 - VECSZ;
                mode = hal::STORE_ALIGNED_NOCACHE;
            }
        }
    }
    else if( cn == 3 )
    {
        T* dst2 = dst[2];
        for( i = 0; i < len; i += VECSZ )
        {
            if( i > len - VECSZ )
            {
                i = len - VECSZ;
                mode = hal::STORE_UNALIGNED;
            }
            VecT a, b, c;
            v_load_deinterleave(src + i*cn, a, b, c);
            v_store(dst0 + i, a, mode);
            v_store(dst1 + i, b, mode);
            v_store(dst2 + i, c, mode);
            if( i < i0 )
            {
                i = i0 - VECSZ;
                mode = hal::STORE_ALIGNED_NOCACHE;
            }
        }
    }
    else
    {
        CV_Assert( cn == 4 );
        T* dst2 = dst[2];
        T* dst3 = dst[3];
        for( i = 0; i < len; i += VECSZ )
        {
            if( i > len - VECSZ )
            {
                i = len - VECSZ;
                mode = hal::STORE_UNALIGNED;
            }
            VecT a, b, c, d;
            v_load_deinterleave(src + i*cn, a, b, c, d);
            v_store(dst0 + i, a, mode);
            v_store(dst1 + i, b, mode);
            v_store(dst2 + i, c, mode);
            v_store(dst3 + i, d, mode);
            if( i < i0 )
            {
                i = i0 - VECSZ;
                mode = hal::STORE_ALIGNED_NOCACHE;
            }
        }
    }
    vx_cleanup();
}
#endif

template<typename T> static void
split_( const T* src, T** dst, int len, int cn )
{
    int k = cn % 4 ? cn % 4 : 4;
    int i, j;
    if( k == 1 )
    {
        T* dst0 = dst[0];

        if(cn == 1)
        {
            memcpy(dst0, src, len * sizeof(T));
        }
        else
        {
            for( i = 0, j = 0 ; i < len; i++, j += cn )
                dst0[i] = src[j];
        }
    }
    else if( k == 2 )
    {
        T *dst0 = dst[0], *dst1 = dst[1];
        i = j = 0;

        for( ; i < len; i++, j += cn )
        {
            dst0[i] = src[j];
            dst1[i] = src[j+1];
        }
    }
    else if( k == 3 )
    {
        T *dst0 = dst[0], *dst1 = dst[1], *dst2 = dst[2];
        i = j = 0;

        for( ; i < len; i++, j += cn )
        {
            dst0[i] = src[j];
            dst1[i] = src[j+1];
            dst2[i] = src[j+2];
        }
    }
    else
    {
        T *dst0 = dst[0], *dst1 = dst[1], *dst2 = dst[2], *dst3 = dst[3];
        i = j = 0;

        for( ; i < len; i++, j += cn )
        {
            dst0[i] = src[j]; dst1[i] = src[j+1];
            dst2[i] = src[j+2]; dst3[i] = src[j+3];
        }
    }

    for( ; k < cn; k += 4 )
    {
        T *dst0 = dst[k], *dst1 = dst[k+1], *dst2 = dst[k+2], *dst3 = dst[k+3];
        for( i = 0, j = k; i < len; i++, j += cn )
        {
            dst0[i] = src[j]; dst1[i] = src[j+1];
            dst2[i] = src[j+2]; dst3[i] = src[j+3];
        }
    }
}

void split8u(const uchar* src, uchar** dst, int len, int cn )
{
    CV_INSTRUMENT_REGION();
#if (CV_SIMD || CV_SIMD_SCALABLE)
    if( len >= VTraits<v_uint8>::vlanes() && 2 <= cn && cn <= 4 )
        vecsplit_<uchar, v_uint8>(src, dst, len, cn);
    else
#endif
        split_(src, dst, len, cn);
}

void split16u(const ushort* src, ushort** dst, int len, int cn )
{
    CV_INSTRUMENT_REGION();
#if (CV_SIMD || CV_SIMD_SCALABLE)
    if( len >= VTraits<v_uint16>::vlanes() && 2 <= cn && cn <= 4 )
        vecsplit_<ushort, v_uint16>(src, dst, len, cn);
    else
#endif
        split_(src, dst, len, cn);
}

void split32s(const int* src, int** dst, int len, int cn )
{
    CV_INSTRUMENT_REGION();
#if (CV_SIMD || CV_SIMD_SCALABLE)
    if( len >= VTraits<v_uint32>::vlanes() && 2 <= cn && cn <= 4 )
        vecsplit_<int, v_int32>(src, dst, len, cn);
    else
#endif
        split_(src, dst, len, cn);
}

void split64s(const int64* src, int64** dst, int len, int cn )
{
    CV_INSTRUMENT_REGION();
#if (CV_SIMD || CV_SIMD_SCALABLE)
    if( len >= VTraits<v_int64>::vlanes() && 2 <= cn && cn <= 4 )
        vecsplit_<int64, v_int64>(src, dst, len, cn);
    else
#endif
        split_(src, dst, len, cn);
}

#endif
CV_CPU_OPTIMIZATION_NAMESPACE_END
}} // namespace
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

- **CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`


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

