# Documentation for `modules/core/include/opencv2/core/hal/intrin_sse_em.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/hal/intrin_sse_em.hpp`
- **File Name**: `intrin_sse_em.hpp`
- **File Size**: 5,424 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/hal/intrin_sse_em.hpp](../../../../../../modules/core/include/opencv2/core/hal/intrin_sse_em.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/hal` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#ifndef OPENCV_HAL_INTRIN_SSE_EM_HPP
#define OPENCV_HAL_INTRIN_SSE_EM_HPP

namespace cv
{

//! @cond IGNORED

CV_CPU_OPTIMIZATION_HAL_NAMESPACE_BEGIN

#define OPENCV_HAL_SSE_WRAP_1(fun, tp) \
    inline tp _v128_##fun(const tp& a) \
    { return _mm_##fun(a); }

#define OPENCV_HAL_SSE_WRAP_2(fun, tp) \
    inline tp _v128_##fun(const tp& a, const tp& b) \
    { return _mm_##fun(a, b); }

#define OPENCV_HAL_SSE_WRAP_3(fun, tp) \
    inline tp _v128_##fun(const tp& a, const tp& b, const tp& c) \
    { return _mm_##fun(a, b, c); }

///////////////////////////// XOP /////////////////////////////

// [todo] define CV_XOP
#if 1 // CV_XOP
inline __m128i _v128_comgt_epu32(const __m128i& a, const __m128i& b)
{
    const __m128i delta = _mm_set1_epi32((int)0x80000000);
    return _mm_cmpgt_epi32(_mm_xor_si128(a, delta), _mm_xor_si128(b, delta));
}
// wrapping XOP
#else
OPENCV_HAL_SSE_WRAP_2(_v128_comgt_epu32, __m128i)
#endif // !CV_XOP

///////////////////////////// SSE4.1 /////////////////////////////

#if !CV_SSE4_1

/** Swizzle **/
inline __m128i _v128_blendv_epi8(const __m128i& a, const __m128i& b, const __m128i& mask)
{ return _mm_xor_si128(a, _mm_and_si128(_mm_xor_si128(b, a), mask)); }

/** Convert **/
// 8 >> 16
inline __m128i _v128_cvtepu8_epi16(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpacklo_epi8(a, z);
}
inline __m128i _v128_cvtepi8_epi16(const __m128i& a)
{ return _mm_srai_epi16(_mm_unpacklo_epi8(a, a), 8); }
// 8 >> 32
inline __m128i _v128_cvtepu8_epi32(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpacklo_epi16(_mm_unpacklo_epi8(a, z), z);
}
inline __m128i _v128_cvtepi8_epi32(const __m128i& a)
{
    __m128i r = _mm_unpacklo_epi8(a, a);
    r = _mm_unpacklo_epi8(r, r);
    return _mm_srai_epi32(r, 24);
}
// 16 >> 32
inline __m128i _v128_cvtepu16_epi32(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpacklo_epi16(a, z);
}
inline __m128i _v128_cvtepi16_epi32(const __m128i& a)
{ return _mm_srai_epi32(_mm_unpacklo_epi16(a, a), 16); }
// 32 >> 64
inline __m128i _v128_cvtepu32_epi64(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpacklo_epi32(a, z);
}
inline __m128i _v128_cvtepi32_epi64(const __m128i& a)
{ return _mm_unpacklo_epi32(a, _mm_srai_epi32(a, 31)); }

/** Arithmetic **/
inline __m128i _v128_mullo_epi32(const __m128i& a, const __m128i& b)
{
    __m128i c0 = _mm_mul_epu32(a, b);
    __m128i c1 = _mm_mul_epu32(_mm_srli_epi64(a, 32), _mm_srli_epi64(b, 32));
    __m128i d0 = _mm_unpacklo_epi32(c0, c1);
    __m128i d1 = _mm_unpackhi_epi32(c0, c1);
    return _mm_unpacklo_epi64(d0, d1);
}

/** Math **/
inline __m128i _v128_min_epu32(const __m128i& a, const __m128i& b)
{ return _v128_blendv_epi8(a, b, _v128_comgt_epu32(a, b)); }

// wrapping SSE4.1
#else
OPENCV_HAL_SSE_WRAP_1(cvtepu8_epi16, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepi8_epi16, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepu8_epi32, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepi8_epi32, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepu16_epi32, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepi16_epi32, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepu32_epi64, __m128i)
OPENCV_HAL_SSE_WRAP_1(cvtepi32_epi64, __m128i)
OPENCV_HAL_SSE_WRAP_2(min_epu32, __m128i)
OPENCV_HAL_SSE_WRAP_2(mullo_epi32, __m128i)
OPENCV_HAL_SSE_WRAP_3(blendv_epi8, __m128i)
#endif // !CV_SSE4_1

///////////////////////////// Revolutionary /////////////////////////////

/** Convert **/
// 16 << 8
inline __m128i _v128_cvtepu8_epi16_high(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpackhi_epi8(a, z);
}
inline __m128i _v128_cvtepi8_epi16_high(const __m128i& a)
{ return _mm_srai_epi16(_mm_unpackhi_epi8(a, a), 8); }
// 32 << 16
inline __m128i _v128_cvtepu16_epi32_high(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpackhi_epi16(a, z);
}
inline __m128i _v128_cvtepi16_epi32_high(const __m128i& a)
{ return _mm_srai_epi32(_mm_unpackhi_epi16(a, a), 16); }
// 64 << 32
inline __m128i _v128_cvtepu32_epi64_high(const __m128i& a)
{
    const __m128i z = _mm_setzero_si128();
    return _mm_unpackhi_epi32(a, z);
}
inline __m128i _v128_cvtepi32_epi64_high(const __m128i& a)
{ return _mm_unpackhi_epi32(a, _mm_srai_epi32(a, 31)); }

/** Miscellaneous **/
inline __m128i _v128_packs_epu32(const __m128i& a, const __m128i& b)
{
    const __m128i m = _mm_set1_epi32(65535);
    __m128i am = _v128_min_epu32(a, m);
    __m128i bm = _v128_min_epu32(b, m);
#if CV_SSE4_1
    return _mm_packus_epi32(am, bm);
#else
    const __m128i d = _mm_set1_epi32(32768), nd = _mm_set1_epi16(-32768);
    am = _mm_sub_epi32(am, d);
    bm = _mm_sub_epi32(bm, d);
    am = _mm_packs_epi32(am, bm);
    return _mm_sub_epi16(am, nd);
#endif
}

template<int i>
inline int64 _v128_extract_epi64(const __m128i& a)
{
#if defined(CV__SIMD_HAVE_mm_extract_epi64) || (CV_SSE4_1 && (defined(__x86_64__)/*GCC*/ || defined(_M_X64)/*MSVC*/))
#define CV__SIMD_NATIVE_mm_extract_epi64 1
    return _mm_extract_epi64(a, i);
#else
    CV_DECL_ALIGNED(16) int64 tmp[2];
    _mm_store_si128((__m128i*)tmp, a);
    return tmp[i];
#endif
}

CV_CPU_OPTIMIZATION_HAL_NAMESPACE_END

//! @endcond

} // cv::

#endif // OPENCV_HAL_INTRIN_SSE_EM_HPP
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

- **OPENCV_HAL_INTRIN_SSE_EM_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

