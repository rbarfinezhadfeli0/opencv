# Documentation for `docs/hal/carotene/src/saturate_cast.hpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/carotene/src/saturate_cast.hpp_docs.md`
- **File Name**: `saturate_cast.hpp_docs.md`
- **File Size**: 13,692 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/carotene/src/saturate_cast.hpp_docs.md](../../../../docs/hal/carotene/src/saturate_cast.hpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/carotene/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/carotene/src/saturate_cast.hpp`

## File Metadata

- **Full Path**: `hal/carotene/src/saturate_cast.hpp`
- **File Name**: `saturate_cast.hpp`
- **File Size**: 10,438 bytes
- **File Type**: .hpp
- **Link to Source**: [hal/carotene/src/saturate_cast.hpp](../../../hal/carotene/src/saturate_cast.hpp)

## Purpose and Role

This file is located in the `hal/carotene/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * By downloading, copying, installing or using the software you agree to this license.
 * If you do not agree to this license, do not download, install,
 * copy or use the software.
 *
 *
 *                           License Agreement
 *                For Open Source Computer Vision Library
 *                        (3-clause BSD License)
 *
 * Copyright (C) 2012-2015, NVIDIA Corporation, all rights reserved.
 * Third party copyrights are property of their respective owners.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 *   * Redistributions of source code must retain the above copyright notice,
 *     this list of conditions and the following disclaimer.
 *
 *   * Redistributions in binary form must reproduce the above copyright notice,
 *     this list of conditions and the following disclaimer in the documentation
 *     and/or other materials provided with the distribution.
 *
 *   * Neither the names of the copyright holders nor the names of the contributors
 *     may be used to endorse or promote products derived from this software
 *     without specific prior written permission.
 *
 * This software is provided by the copyright holders and contributors "as is" and
 * any express or implied warranties, including, but not limited to, the implied
 * warranties of merchantability and fitness for a particular purpose are disclaimed.
 * In no event shall copyright holders or contributors be liable for any direct,
 * indirect, incidental, special, exemplary, or consequential damages
 * (including, but not limited to, procurement of substitute goods or services;
 * loss of use, data, or profits; or business interruption) however caused
 * and on any theory of liability, whether in contract, strict liability,
 * or tort (including negligence or otherwise) arising in any way out of
 * the use of this software, even if advised of the possibility of such damage.
 */

#ifndef CAROTENE_SATURATE_CAST_HPP
#define CAROTENE_SATURATE_CAST_HPP

#include <algorithm>
#include <climits>
#include <cmath>

#if defined _MSC_VER && defined _M_ARM
# include <intrin.h>
#endif

#include <carotene/definitions.hpp>
#include <carotene/types.hpp>

namespace CAROTENE_NS { namespace internal {

#if defined _MSC_VER && defined _M_ARM

__declspec(naked) static void vcvtr_s32_f64_imp(f64 d)
{
    (void)d;
    __emit(0xEEBD);  // vcvtr.s32.f64 s0, d0
    __emit(0x0B40);
    __emit(0xEE10);  // vmov r0, s0
    __emit(0x0A10);
    __emit(0x4770);  // bx lr
}

# define CAROTENE_ROUND_FLT(x) return ((s32 (*)(f64))vcvtr_s32_f64_imp)((f64)x);
# define CAROTENE_ROUND_DBL(x) return ((s32 (*)(f64))vcvtr_s32_f64_imp)(x);

#elif defined CV_ICC || defined __GNUC__

# if defined(__VFP_FP__) && !defined(__SOFTFP__) && !(defined _DEBUG || defined DEBUG) && !defined(__CUDACC__)
#  define CAROTENE_ROUND_FLT(value) {                              \
    union { f32 f; s32 i; } result; \
    asm ("ftosis  %0, %1 \n" : "=w" (result.f) : "w" (value) ); \
    return result.i; }
#  define CAROTENE_ROUND_DBL(value) {                      \
    union {f32 f; s32 i;} __tegra_result; \
    asm (                                               \
        "ftosid  %0, %P1\n"                             \
        : "=w" (__tegra_result.f)                       \
        : "w" (value)                                   \
    );                                                  \
    return __tegra_result.i;                            \
    }
# else
#  define CAROTENE_ROUND_FLT(x) return (s32)lrintf(value);
#  define CAROTENE_ROUND_DBL(value) return (s32)lrint(value);
# endif

#endif

inline s32 round(f32 value)
{
#ifdef CAROTENE_ROUND_FLT
    CAROTENE_ROUND_FLT(value)
#else
    s32 intpart = (s32)(value);
    f32 fractpart = value - intpart;
    if ((fractpart != 0.5 && fractpart != -0.5) || ((intpart % 2) != 0))
        return (s32)(value + (value >= 0 ? 0.5 : -0.5));
    else
        return intpart;
#endif
}

inline s32 round(f64 value)
{
#ifdef CAROTENE_ROUND_DBL
    CAROTENE_ROUND_DBL(value)
#else
    s32 intpart = (s32)(value);
    f64 fractpart = value - intpart;
    if ((fractpart != 0.5 && fractpart != -0.5) || ((intpart % 2) != 0))
        return (s32)(value + (value >= 0 ? 0.5 : -0.5));
    else
        return intpart;
#endif
}
/////////////// saturate_cast (used in image & signal processing) ///////////////////

template<typename _Tp> inline _Tp saturate_cast(u8 v)    { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(s8 v)    { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(u16 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(s16 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(u32 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(s32 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(s64 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(u64 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(f32 v)   { return _Tp(v); }
template<typename _Tp> inline _Tp saturate_cast(f64 v)   { return _Tp(v); }

template<> inline u8 saturate_cast<u8>(s8 v)      { return (u8)std::max((s32)v, 0); }
template<> inline u8 saturate_cast<u8>(u16 v)     { return (u8)std::min((u32)v, (u32)UCHAR_MAX); }
template<> inline u8 saturate_cast<u8>(s32 v)     { return (u8)((u32)v <= UCHAR_MAX ? v : v > 0 ? UCHAR_MAX : 0); }
template<> inline u8 saturate_cast<u8>(s16 v)     { return saturate_cast<u8>((s32)v); }
template<> inline u8 saturate_cast<u8>(u32 v)     { return (u8)std::min(v, (u32)UCHAR_MAX); }
template<> inline u8 saturate_cast<u8>(s64 v)     { return (u8)((u64)v <= UCHAR_MAX ? v : v > 0 ? UCHAR_MAX : 0); }
template<> inline u8 saturate_cast<u8>(u64 v)     { return (u8)std::min(v, (u64)UCHAR_MAX); }
template<> inline u8 saturate_cast<u8>(f32 v)     { return saturate_cast<u8>(round(v)); }
template<> inline u8 saturate_cast<u8>(f64 v)     { return saturate_cast<u8>(round(v)); }

template<> inline s8 saturate_cast<s8>(u8 v)      { return (s8)std::min((s32)v, SCHAR_MAX); }
template<> inline s8 saturate_cast<s8>(u16 v)     { return (s8)std::min((u32)v, (u32)SCHAR_MAX); }
template<> inline s8 saturate_cast<s8>(s32 v)     { return (s8)((u32)(v-SCHAR_MIN) <= (u32)UCHAR_MAX ? v : v > 0 ? SCHAR_MAX : SCHAR_MIN); }
template<> inline s8 saturate_cast<s8>(s16 v)     { return saturate_cast<s8>((s32)v); }
template<> inline s8 saturate_cast<s8>(u32 v)     { return (s8)std::min(v, (u32)SCHAR_MAX); }
template<> inline s8 saturate_cast<s8>(s64 v)     { return (s8)((u64)(v-SCHAR_MIN) <= (u64)UCHAR_MAX ? v : v > 0 ? SCHAR_MAX : SCHAR_MIN); }
template<> inline s8 saturate_cast<s8>(u64 v)     { return (s8)std::min(v, (u64)SCHAR_MAX); }
template<> inline s8 saturate_cast<s8>(f32 v)     { return saturate_cast<s8>(round(v)); }
template<> inline s8 saturate_cast<s8>(f64 v)     { return saturate_cast<s8>(round(v)); }

template<> inline u16 saturate_cast<u16>(s8 v)    { return (u16)std::max((s32)v, 0); }
template<> inline u16 saturate_cast<u16>(s16 v)   { return (u16)std::max((s32)v, 0); }
template<> inline u16 saturate_cast<u16>(s32 v)   { return (u16)((u32)v <= (u32)USHRT_MAX ? v : v > 0 ? USHRT_MAX : 0); }
template<> inline u16 saturate_cast<u16>(u32 v)   { return (u16)std::min(v, (u32)USHRT_MAX); }
template<> inline u16 saturate_cast<u16>(s64 v)   { return (u16)((u64)v <= (u64)USHRT_MAX ? v : v > 0 ? USHRT_MAX : 0); }
template<> inline u16 saturate_cast<u16>(u64 v)   { return (u16)std::min(v, (u64)USHRT_MAX); }
template<> inline u16 saturate_cast<u16>(f32 v)   { return saturate_cast<u16>(round(v)); }
template<> inline u16 saturate_cast<u16>(f64 v)   { return saturate_cast<u16>(round(v)); }

template<> inline s16 saturate_cast<s16>(u16 v)   { return (s16)std::min((s32)v, SHRT_MAX); }
template<> inline s16 saturate_cast<s16>(s32 v)   { return (s16)((u32)(v - SHRT_MIN) <= (u32)USHRT_MAX ? v : v > 0 ? SHRT_MAX : SHRT_MIN); }
template<> inline s16 saturate_cast<s16>(u32 v)   { return (s16)std::min(v, (u32)SHRT_MAX); }
template<> inline s16 saturate_cast<s16>(s64 v)   { return (s16)((u64)(v - SHRT_MIN) <= (u64)USHRT_MAX ? v : v > 0 ? SHRT_MAX : SHRT_MIN); }
template<> inline s16 saturate_cast<s16>(u64 v)   { return (s16)std::min(v, (u64)SHRT_MAX); }
template<> inline s16 saturate_cast<s16>(f32 v)   { return saturate_cast<s16>(round(v)); }
template<> inline s16 saturate_cast<s16>(f64 v)   { return saturate_cast<s16>(round(v)); }

template<> inline u32 saturate_cast<u32>(s8 v)    { return (u32)std::max(v, (s8)0); }
template<> inline u32 saturate_cast<u32>(s16 v)   { return (u32)std::max(v, (s16)0); }
template<> inline u32 saturate_cast<u32>(s32 v)   { return (u32)std::max(v, (s32)0); }
template<> inline u32 saturate_cast<u32>(s64 v)   { return (u32)((u64)v <= (u64)UINT_MAX ? v : v > 0 ? UINT_MAX : 0); }
template<> inline u32 saturate_cast<u32>(u64 v)   { return (u32)std::min(v, (u64)UINT_MAX); }
//OpenCV like f32/f64 -> u32 conversion
//we intentionally do not clip negative numbers, to make -1 become 0xffffffff etc.
template<> inline u32 saturate_cast<u32>(f32 v)   { return round(v); }
template<> inline u32 saturate_cast<u32>(f64 v)   { return round(v); }
//Negative clipping implementation
//template<> inline u32 saturate_cast<u32>(f32 v)   { return saturate_cast<u32>(round(v)); }
//template<> inline u32 saturate_cast<u32>(f64 v)   { return saturate_cast<u32>(round(v)); }

template<> inline s32 saturate_cast<s32>(u32 v)   { return (s32)std::min(v, (u32)INT_MAX); }
template<> inline s32 saturate_cast<s32>(s64 v)   { return (s32)((u64)(v - INT_MIN) <= (u64)UINT_MAX ? v : v > 0 ? INT_MAX : INT_MIN); }
template<> inline s32 saturate_cast<s32>(u64 v)   { return (s32)std::min(v, (u64)INT_MAX); }
template<> inline s32 saturate_cast<s32>(f32 v)   { return round(v); }
template<> inline s32 saturate_cast<s32>(f64 v)   { return round(v); }

template<> inline u64 saturate_cast<u64>(s8 v)    { return (u64)std::max(v, (s8)0); }
template<> inline u64 saturate_cast<u64>(s16 v)   { return (u64)std::max(v, (s16)0); }
template<> inline u64 saturate_cast<u64>(s32 v)   { return (u64)std::max(v, (s32)0); }
template<> inline u64 saturate_cast<u64>(s64 v)   { return (u64)std::max(v, (s64)0); }

template<> inline s64 saturate_cast<s64>(u64 v)   { return (s64)std::min(v, (u64)LLONG_MAX); }

} }

#endif
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

- **CAROTENE_SATURATE_CAST_HPP()**: A function/method defined in this file
- **CAROTENE_ROUND_DBL()**: A function/method defined in this file
- **CAROTENE_ROUND_FLT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cmath`
- `climits`
- `carotene/definitions.hpp`
- `carotene/types.hpp`
- `algorithm`

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

