# Documentation for `modules/core/include/opencv2/core/cv_cpu_dispatch.h`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/cv_cpu_dispatch.h`
- **File Name**: `cv_cpu_dispatch.h`
- **File Size**: 8,890 bytes
- **File Type**: .h
- **Link to Source**: [modules/core/include/opencv2/core/cv_cpu_dispatch.h](../../../../../modules/core/include/opencv2/core/cv_cpu_dispatch.h)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#if defined __OPENCV_BUILD \

#include "cv_cpu_config.h"
#include "cv_cpu_helper.h"

#ifdef CV_CPU_DISPATCH_MODE
#define CV_CPU_OPTIMIZATION_NAMESPACE __CV_CAT(opt_, CV_CPU_DISPATCH_MODE)
#define CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN namespace __CV_CAT(opt_, CV_CPU_DISPATCH_MODE) {
#define CV_CPU_OPTIMIZATION_NAMESPACE_END }
#else
#define CV_CPU_OPTIMIZATION_NAMESPACE cpu_baseline
#define CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN namespace cpu_baseline {
#define CV_CPU_OPTIMIZATION_NAMESPACE_END }
#define CV_CPU_BASELINE_MODE 1
#endif


#define __CV_CPU_DISPATCH_CHAIN_END(fn, args, mode, ...)  /* done */
#define __CV_CPU_DISPATCH(fn, args, mode, ...) __CV_EXPAND(__CV_CPU_DISPATCH_CHAIN_ ## mode(fn, args, __VA_ARGS__))
#define __CV_CPU_DISPATCH_EXPAND(fn, args, ...) __CV_EXPAND(__CV_CPU_DISPATCH(fn, args, __VA_ARGS__))
#define CV_CPU_DISPATCH(fn, args, ...) __CV_CPU_DISPATCH_EXPAND(fn, args, __VA_ARGS__, END) // expand macros


#if defined CV_ENABLE_INTRINSICS \
    && !defined CV_DISABLE_OPTIMIZATION \
    && !defined __CUDACC__ /* do not include SSE/AVX/NEON headers for NVCC compiler */ \

#ifdef CV_CPU_COMPILE_SSE2
#  include <emmintrin.h>
#  define CV_MMX 1
#  define CV_SSE 1
#  define CV_SSE2 1
#endif
#ifdef CV_CPU_COMPILE_SSE3
#  include <pmmintrin.h>
#  define CV_SSE3 1
#endif
#ifdef CV_CPU_COMPILE_SSSE3
#  include <tmmintrin.h>
#  define CV_SSSE3 1
#endif
#ifdef CV_CPU_COMPILE_SSE4_1
#  include <smmintrin.h>
#  define CV_SSE4_1 1
#endif
#ifdef CV_CPU_COMPILE_SSE4_2
#  include <nmmintrin.h>
#  define CV_SSE4_2 1
#endif
#ifdef CV_CPU_COMPILE_POPCNT
#  ifdef _MSC_VER
#    include <nmmintrin.h>
#    if defined(_M_X64)
#      define CV_POPCNT_U64 (int)_mm_popcnt_u64
#    endif
#    define CV_POPCNT_U32 _mm_popcnt_u32
#  else
#    include <popcntintrin.h>
#    if defined(__x86_64__)
#      define CV_POPCNT_U64 __builtin_popcountll
#    endif
#    define CV_POPCNT_U32 __builtin_popcount
#  endif
#  define CV_POPCNT 1
#endif
#ifdef CV_CPU_COMPILE_AVX
#  include <immintrin.h>
#  define CV_AVX 1
#endif
#ifdef CV_CPU_COMPILE_FP16
#  if defined(__arm__) || defined(__aarch64__) || defined(_M_ARM) || defined(_M_ARM64) || defined(_M_ARM64EC)
#    include <arm_neon.h>
#  else
#    include <immintrin.h>
#  endif
#  define CV_FP16 1
#endif
#ifdef CV_CPU_COMPILE_NEON_DOTPROD
#  include <arm_neon.h>
#  define CV_NEON_DOT 1
#endif
#ifdef CV_CPU_COMPILE_AVX2
#  include <immintrin.h>
#  define CV_AVX2 1
#endif
#ifdef CV_CPU_COMPILE_AVX_512F
#  include <immintrin.h>
#  define CV_AVX_512F 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_COMMON
#  define CV_AVX512_COMMON 1
#  define CV_AVX_512CD 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_KNL
#  define CV_AVX512_KNL 1
#  define CV_AVX_512ER 1
#  define CV_AVX_512PF 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_KNM
#  define CV_AVX512_KNM 1
#  define CV_AVX_5124FMAPS 1
#  define CV_AVX_5124VNNIW 1
#  define CV_AVX_512VPOPCNTDQ 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_SKX
#  define CV_AVX512_SKX 1
#  define CV_AVX_512VL 1
#  define CV_AVX_512BW 1
#  define CV_AVX_512DQ 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_CNL
#  define CV_AVX512_CNL 1
#  define CV_AVX_512IFMA 1
#  define CV_AVX_512VBMI 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_CLX
#  define CV_AVX512_CLX 1
#  define CV_AVX_512VNNI 1
#endif
#ifdef CV_CPU_COMPILE_AVX512_ICL
#  define CV_AVX512_ICL 1
#  undef CV_AVX_512IFMA
#  define CV_AVX_512IFMA 1
#  undef CV_AVX_512VBMI
#  define CV_AVX_512VBMI 1
#  undef CV_AVX_512VNNI
#  define CV_AVX_512VNNI 1
#  define CV_AVX_512VBMI2 1
#  define CV_AVX_512BITALG 1
#  define CV_AVX_512VPOPCNTDQ 1
#endif
#ifdef CV_CPU_COMPILE_FMA3
#  define CV_FMA3 1
#endif

#if defined _WIN32 && (defined(_M_ARM) || defined(_M_ARM64) || defined(_M_ARM64EC)) && (defined(CV_CPU_COMPILE_NEON) || !defined(_MSC_VER))
# include <Intrin.h>
# include <arm_neon.h>
# define CV_NEON 1
#elif defined(__ARM_NEON)
#  include <arm_neon.h>
#  define CV_NEON 1
#endif

/* RVV-related macro states with different compiler
// +--------------------+----------+----------+
// | Macro              | Upstream | XuanTie  |
// +--------------------+----------+----------+
// | CV_CPU_COMPILE_RVV | defined  | defined  |
// | CV_RVV             | 1        | 0        |
// | CV_RVV071          | 0        | 1        |
// | CV_TRY_RVV         | 1        | 1        |
// +--------------------+----------+----------+
*/
#ifdef CV_CPU_COMPILE_RVV
#  ifdef __riscv_vector_071
#    define CV_RVV071 1
#  else
#    define CV_RVV 1
#  endif
#include <riscv_vector.h>
#endif

#ifdef CV_CPU_COMPILE_VSX
#  include <altivec.h>
#  undef vector
#  undef pixel
#  undef bool
#  define CV_VSX 1
#endif

#ifdef CV_CPU_COMPILE_VSX3
#  define CV_VSX3 1
#endif

#ifdef CV_CPU_COMPILE_MSA
#  include "hal/msa_macros.h"
#  define CV_MSA 1
#endif

#ifdef CV_CPU_COMPILE_LSX
#  include <lsxintrin.h>
#  define CV_LSX 1
#endif

#ifdef CV_CPU_COMPILE_LASX
#  include <lasxintrin.h>
#  define CV_LASX 1
#endif

#ifdef __EMSCRIPTEN__
#  define CV_WASM_SIMD 1
#  include <wasm_simd128.h>
#endif

#endif // CV_ENABLE_INTRINSICS && !CV_DISABLE_OPTIMIZATION && !__CUDACC__

#if defined CV_CPU_COMPILE_AVX && !defined CV_CPU_BASELINE_COMPILE_AVX
struct VZeroUpperGuard {
#ifdef __GNUC__
    __attribute__((always_inline))
#endif
    inline VZeroUpperGuard() { _mm256_zeroupper(); }
#ifdef __GNUC__
    __attribute__((always_inline))
#endif
    inline ~VZeroUpperGuard() { _mm256_zeroupper(); }
};
#define __CV_AVX_GUARD VZeroUpperGuard __vzeroupper_guard; CV_UNUSED(__vzeroupper_guard);
#endif

#ifdef __CV_AVX_GUARD
#define CV_AVX_GUARD __CV_AVX_GUARD
#else
#define CV_AVX_GUARD
#endif

#endif // __OPENCV_BUILD



#if !defined __OPENCV_BUILD /* Compatibility code */ \
    && !defined __CUDACC__ /* do not include SSE/AVX/NEON headers for NVCC compiler */
#if defined __SSE2__ || defined _M_X64 || (defined _M_IX86_FP && _M_IX86_FP >= 2)
#  include <emmintrin.h>
#  define CV_MMX 1
#  define CV_SSE 1
#  define CV_SSE2 1
#elif defined _WIN32 && (defined(_M_ARM) || defined(_M_ARM64) || defined(_M_ARM64EC)) && (defined(CV_CPU_COMPILE_NEON) || !defined(_MSC_VER))
# include <Intrin.h>
# include <arm_neon.h>
# define CV_NEON 1
#elif defined(__ARM_NEON)
#  include <arm_neon.h>
#  define CV_NEON 1
#elif defined(__VSX__) && defined(__PPC64__) && defined(__LITTLE_ENDIAN__)
#  include <altivec.h>
#  undef vector
#  undef pixel
#  undef bool
#  define CV_VSX 1
#endif

#ifdef __F16C__
#  include <immintrin.h>
#  define CV_FP16 1
#endif

#endif // !__OPENCV_BUILD && !__CUDACC (Compatibility code)



#ifndef CV_MMX
#  define CV_MMX 0
#endif
#ifndef CV_SSE
#  define CV_SSE 0
#endif
#ifndef CV_SSE2
#  define CV_SSE2 0
#endif
#ifndef CV_SSE3
#  define CV_SSE3 0
#endif
#ifndef CV_SSSE3
#  define CV_SSSE3 0
#endif
#ifndef CV_SSE4_1
#  define CV_SSE4_1 0
#endif
#ifndef CV_SSE4_2
#  define CV_SSE4_2 0
#endif
#ifndef CV_POPCNT
#  define CV_POPCNT 0
#endif
#ifndef CV_AVX
#  define CV_AVX 0
#endif
#ifndef CV_FP16
#  define CV_FP16 0
#endif
#ifndef CV_AVX2
#  define CV_AVX2 0
#endif
#ifndef CV_FMA3
#  define CV_FMA3 0
#endif
#ifndef CV_AVX_512F
#  define CV_AVX_512F 0
#endif
#ifndef CV_AVX_512BW
#  define CV_AVX_512BW 0
#endif
#ifndef CV_AVX_512CD
#  define CV_AVX_512CD 0
#endif
#ifndef CV_AVX_512DQ
#  define CV_AVX_512DQ 0
#endif
#ifndef CV_AVX_512ER
#  define CV_AVX_512ER 0
#endif
#ifndef CV_AVX_512IFMA
#  define CV_AVX_512IFMA 0
#endif
#define CV_AVX_512IFMA512 CV_AVX_512IFMA // deprecated
#ifndef CV_AVX_512PF
#  define CV_AVX_512PF 0
#endif
#ifndef CV_AVX_512VBMI
#  define CV_AVX_512VBMI 0
#endif
#ifndef CV_AVX_512VL
#  define CV_AVX_512VL 0
#endif
#ifndef CV_AVX_5124FMAPS
#  define CV_AVX_5124FMAPS 0
#endif
#ifndef CV_AVX_5124VNNIW
#  define CV_AVX_5124VNNIW 0
#endif
#ifndef CV_AVX_512VPOPCNTDQ
#  define CV_AVX_512VPOPCNTDQ 0
#endif
#ifndef CV_AVX_512VNNI
#  define CV_AVX_512VNNI 0
#endif
#ifndef CV_AVX_512VBMI2
#  define CV_AVX_512VBMI2 0
#endif
#ifndef CV_AVX_512BITALG
#  define CV_AVX_512BITALG 0
#endif
#ifndef CV_AVX512_COMMON
#  define CV_AVX512_COMMON 0
#endif
#ifndef CV_AVX512_KNL
#  define CV_AVX512_KNL 0
#endif
#ifndef CV_AVX512_KNM
#  define CV_AVX512_KNM 0
#endif
#ifndef CV_AVX512_SKX
#  define CV_AVX512_SKX 0
#endif
#ifndef CV_AVX512_CNL
#  define CV_AVX512_CNL 0
#endif
#ifndef CV_AVX512_CLX
#  define CV_AVX512_CLX 0
#endif
#ifndef CV_AVX512_ICL
#  define CV_AVX512_ICL 0
#endif

#ifndef CV_NEON
#  define CV_NEON 0
#endif

#ifndef CV_RVV071
#  define CV_RVV071 0
#endif

#ifndef CV_VSX
#  define CV_VSX 0
#endif

#ifndef CV_VSX3
#  define CV_VSX3 0
#endif

#ifndef CV_MSA
#  define CV_MSA 0
#endif

#ifndef CV_WASM_SIMD
#  define CV_WASM_SIMD 0
#endif

#ifndef CV_RVV
#  define CV_RVV 0
#endif

#ifndef CV_LSX
#  define CV_LSX 0
#endif

#ifndef CV_LASX
#  define CV_LASX 0
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

### Classes and Structures

- **VZeroUpperGuard**: A class/struct defined in this file

### Functions and Methods

- **CV_CPU_COMPILE_VSX3()**: A function/method defined in this file
- **CV_CPU_COMPILE_LSX()**: A function/method defined in this file
- **CV_CPU_COMPILE_SSE3()**: A function/method defined in this file
- **CV_AVX_512VNNI()**: A function/method defined in this file
- **CV_WASM_SIMD()**: A function/method defined in this file
- **CV_AVX512_KNM()**: A function/method defined in this file
- **CV_AVX_512VBMI()**: A function/method defined in this file
- **CV_CPU_COMPILE_FMA3()**: A function/method defined in this file
- **CV_AVX512_ICL()**: A function/method defined in this file
- **CV_AVX_512VBMI2()**: A function/method defined in this file
- **CV_CPU_COMPILE_SSE2()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_CNL()**: A function/method defined in this file
- **pixel()**: A function/method defined in this file
- **CV_AVX_512BITALG()**: A function/method defined in this file
- **CV_CPU_COMPILE_SSE4_1()**: A function/method defined in this file
- **vector()**: A function/method defined in this file
- **CV_POPCNT()**: A function/method defined in this file
- **CV_CPU_DISPATCH_MODE()**: A function/method defined in this file
- **CV_RVV()**: A function/method defined in this file
- **CV_AVX_512DQ()**: A function/method defined in this file
- **CV_SSE2()**: A function/method defined in this file
- **CV_SSSE3()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_SKX()**: A function/method defined in this file
- **CV_AVX2()**: A function/method defined in this file
- **CV_SSE3()**: A function/method defined in this file
- **CV_CPU_COMPILE_POPCNT()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_KNL()**: A function/method defined in this file
- **CV_SSE4_2()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_KNM()**: A function/method defined in this file
- **CV_CPU_COMPILE_VSX()**: A function/method defined in this file
- **CV_AVX_512BW()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_COMMON()**: A function/method defined in this file
- **CV_CPU_COMPILE_MSA()**: A function/method defined in this file
- **CV_CPU_COMPILE_RVV()**: A function/method defined in this file
- **CV_AVX_512ER()**: A function/method defined in this file
- **CV_CPU_COMPILE_LASX()**: A function/method defined in this file
- **__CV_AVX_GUARD()**: A function/method defined in this file
- **CV_AVX_512F()**: A function/method defined in this file
- **CV_AVX_512PF()**: A function/method defined in this file
- **CV_AVX512_SKX()**: A function/method defined in this file
- **CV_CPU_COMPILE_NEON_DOTPROD()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_CLX()**: A function/method defined in this file
- **CV_CPU_COMPILE_SSE4_2()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX_512F()**: A function/method defined in this file
- **CV_VSX3()**: A function/method defined in this file
- **CV_CPU_COMPILE_SSSE3()**: A function/method defined in this file
- **CV_LSX()**: A function/method defined in this file
- **CV_AVX512_KNL()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX()**: A function/method defined in this file
- **CV_MMX()**: A function/method defined in this file
- **CV_AVX512_CNL()**: A function/method defined in this file
- **CV_LASX()**: A function/method defined in this file
- **CV_FMA3()**: A function/method defined in this file
- **CV_AVX_5124VNNIW()**: A function/method defined in this file
- **__F16C__()**: A function/method defined in this file
- **CV_RVV071()**: A function/method defined in this file
- **CV_AVX_512VL()**: A function/method defined in this file
- **bool()**: A function/method defined in this file
- **CV_CPU_COMPILE_FP16()**: A function/method defined in this file
- **CV_FP16()**: A function/method defined in this file
- **_MSC_VER()**: A function/method defined in this file
- **CV_NEON()**: A function/method defined in this file
- **CV_AVX512_COMMON()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX2()**: A function/method defined in this file
- **CV_AVX_512CD()**: A function/method defined in this file
- **CV_SSE()**: A function/method defined in this file
- **CV_AVX_512VPOPCNTDQ()**: A function/method defined in this file
- **CV_CPU_COMPILE_AVX512_ICL()**: A function/method defined in this file
- **CV_AVX()**: A function/method defined in this file
- **CV_SSE4_1()**: A function/method defined in this file
- **__riscv_vector_071()**: A function/method defined in this file
- **__EMSCRIPTEN__()**: A function/method defined in this file
- **__GNUC__()**: A function/method defined in this file
- **CV_MSA()**: A function/method defined in this file
- **CV_AVX_5124FMAPS()**: A function/method defined in this file
- **CV_AVX512_CLX()**: A function/method defined in this file
- **CV_VSX()**: A function/method defined in this file
- **CV_AVX_512IFMA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cv_cpu_helper.h`
- `cv_cpu_config.h`
- `riscv_vector.h`


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

