# Documentation for `docs/3rdparty/zlib-ng/functable.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/functable.c_docs.md`
- **File Name**: `functable.c_docs.md`
- **File Size**: 14,732 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/functable.c_docs.md](../../../docs/3rdparty/zlib-ng/functable.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/functable.c`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/functable.c`
- **File Name**: `functable.c`
- **File Size**: 10,227 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/zlib-ng/functable.c](../../3rdparty/zlib-ng/functable.c)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* functable.c -- Choose relevant optimized functions at runtime
 * Copyright (C) 2017 Hans Kristian Rosbach
 * For conditions of distribution and use, see copyright notice in zlib.h
 */
#ifndef DISABLE_RUNTIME_CPU_DETECTION

#include "zbuild.h"
#include "functable.h"
#include "cpu_features.h"
#include "arch_functions.h"

#if defined(_MSC_VER)
#  include <intrin.h>
#endif

/* Platform has pointer size atomic store */
#if defined(__GNUC__) || defined(__clang__)
#  define FUNCTABLE_ASSIGN(VAR, FUNC_NAME) \
    __atomic_store(&(functable.FUNC_NAME), &(VAR.FUNC_NAME), __ATOMIC_SEQ_CST)
#  define FUNCTABLE_BARRIER() __atomic_thread_fence(__ATOMIC_SEQ_CST)
#elif defined(_MSC_VER)
#  define FUNCTABLE_ASSIGN(VAR, FUNC_NAME) \
    _InterlockedExchangePointer((void * volatile *)&(functable.FUNC_NAME), (void *)(VAR.FUNC_NAME))
#  if defined(_M_ARM) || defined(_M_ARM64)
#    define FUNCTABLE_BARRIER() do { \
    _ReadWriteBarrier();  \
    __dmb(0xB); /* _ARM_BARRIER_ISH */ \
    _ReadWriteBarrier(); \
} while (0)
#  else
#    define FUNCTABLE_BARRIER() _ReadWriteBarrier()
#  endif
#else
#  warning Unable to detect atomic intrinsic support.
#  define FUNCTABLE_ASSIGN(VAR, FUNC_NAME) \
    *((void * volatile *)&(functable.FUNC_NAME)) = (void *)(VAR.FUNC_NAME)
#  define FUNCTABLE_BARRIER() do { /* Empty */ } while (0)
#endif

static void force_init_empty(void) {
    // empty
}

static void init_functable(void) {
    struct functable_s ft;
    struct cpu_features cf;

    cpu_check_features(&cf);

    // Generic code
    ft.force_init = &force_init_empty;
    ft.adler32 = &adler32_c;
    ft.adler32_fold_copy = &adler32_fold_copy_c;
    ft.chunkmemset_safe = &chunkmemset_safe_c;
    ft.chunksize = &chunksize_c;
    ft.crc32 = &PREFIX(crc32_braid);
    ft.crc32_fold = &crc32_fold_c;
    ft.crc32_fold_copy = &crc32_fold_copy_c;
    ft.crc32_fold_final = &crc32_fold_final_c;
    ft.crc32_fold_reset = &crc32_fold_reset_c;
    ft.inflate_fast = &inflate_fast_c;
    ft.slide_hash = &slide_hash_c;
    ft.longest_match = &longest_match_generic;
    ft.longest_match_slow = &longest_match_slow_generic;
    ft.compare256 = &compare256_generic;

    // Select arch-optimized functions

    // X86 - SSE2
#ifdef X86_SSE2
#  if !defined(__x86_64__) && !defined(_M_X64) && !defined(X86_NOCHECK_SSE2)
    if (cf.x86.has_sse2)
#  endif
    {
        ft.chunkmemset_safe = &chunkmemset_safe_sse2;
        ft.chunksize = &chunksize_sse2;
        ft.inflate_fast = &inflate_fast_sse2;
        ft.slide_hash = &slide_hash_sse2;
#  ifdef HAVE_BUILTIN_CTZ
        ft.compare256 = &compare256_sse2;
        ft.longest_match = &longest_match_sse2;
        ft.longest_match_slow = &longest_match_slow_sse2;
#  endif
    }
#endif
    // X86 - SSSE3
#ifdef X86_SSSE3
    if (cf.x86.has_ssse3) {
        ft.adler32 = &adler32_ssse3;
        ft.chunkmemset_safe = &chunkmemset_safe_ssse3;
        ft.inflate_fast = &inflate_fast_ssse3;
    }
#endif
    // X86 - SSE4.2
#ifdef X86_SSE42
    if (cf.x86.has_sse42) {
        ft.adler32_fold_copy = &adler32_fold_copy_sse42;
    }
#endif
    // X86 - PCLMUL
#ifdef X86_PCLMULQDQ_CRC
    if (cf.x86.has_pclmulqdq) {
        ft.crc32 = &crc32_pclmulqdq;
        ft.crc32_fold = &crc32_fold_pclmulqdq;
        ft.crc32_fold_copy = &crc32_fold_pclmulqdq_copy;
        ft.crc32_fold_final = &crc32_fold_pclmulqdq_final;
        ft.crc32_fold_reset = &crc32_fold_pclmulqdq_reset;
    }
#endif
    // X86 - AVX
#ifdef X86_AVX2
    if (cf.x86.has_avx2) {
        ft.adler32 = &adler32_avx2;
        ft.adler32_fold_copy = &adler32_fold_copy_avx2;
        ft.chunkmemset_safe = &chunkmemset_safe_avx2;
        ft.chunksize = &chunksize_avx2;
        ft.inflate_fast = &inflate_fast_avx2;
        ft.slide_hash = &slide_hash_avx2;
#  ifdef HAVE_BUILTIN_CTZ
        ft.compare256 = &compare256_avx2;
        ft.longest_match = &longest_match_avx2;
        ft.longest_match_slow = &longest_match_slow_avx2;
#  endif
    }
#endif
    // X86 - AVX512 (F,DQ,BW,Vl)
#ifdef X86_AVX512
    if (cf.x86.has_avx512_common) {
        ft.adler32 = &adler32_avx512;
        ft.adler32_fold_copy = &adler32_fold_copy_avx512;
    }
#endif
#ifdef X86_AVX512VNNI
    if (cf.x86.has_avx512vnni) {
        ft.adler32 = &adler32_avx512_vnni;
        ft.adler32_fold_copy = &adler32_fold_copy_avx512_vnni;
    }
#endif
    // X86 - VPCLMULQDQ
#ifdef X86_VPCLMULQDQ_CRC
    if (cf.x86.has_pclmulqdq && cf.x86.has_avx512_common && cf.x86.has_vpclmulqdq) {
        ft.crc32 = &crc32_vpclmulqdq;
        ft.crc32_fold = &crc32_fold_vpclmulqdq;
        ft.crc32_fold_copy = &crc32_fold_vpclmulqdq_copy;
        ft.crc32_fold_final = &crc32_fold_vpclmulqdq_final;
        ft.crc32_fold_reset = &crc32_fold_vpclmulqdq_reset;
    }
#endif


    // ARM - SIMD
#ifdef ARM_SIMD
#  ifndef ARM_NOCHECK_SIMD
    if (cf.arm.has_simd)
#  endif
    {
        ft.slide_hash = &slide_hash_armv6;
    }
#endif
    // ARM - NEON
#ifdef ARM_NEON
#  ifndef ARM_NOCHECK_NEON
    if (cf.arm.has_neon)
#  endif
    {
        ft.adler32 = &adler32_neon;
        ft.chunkmemset_safe = &chunkmemset_safe_neon;
        ft.chunksize = &chunksize_neon;
        ft.inflate_fast = &inflate_fast_neon;
        ft.slide_hash = &slide_hash_neon;
#  ifdef HAVE_BUILTIN_CTZLL
        ft.compare256 = &compare256_neon;
        ft.longest_match = &longest_match_neon;
        ft.longest_match_slow = &longest_match_slow_neon;
#  endif
    }
#endif
    // ARM - ACLE
#ifdef ARM_ACLE
    if (cf.arm.has_crc32) {
        ft.crc32 = &crc32_acle;
    }
#endif


    // Power - VMX
#ifdef PPC_VMX
    if (cf.power.has_altivec) {
        ft.adler32 = &adler32_vmx;
        ft.slide_hash = &slide_hash_vmx;
    }
#endif
    // Power8 - VSX
#ifdef POWER8_VSX
    if (cf.power.has_arch_2_07) {
        ft.adler32 = &adler32_power8;
        ft.chunkmemset_safe = &chunkmemset_safe_power8;
        ft.chunksize = &chunksize_power8;
        ft.inflate_fast = &inflate_fast_power8;
        ft.slide_hash = &slide_hash_power8;
    }
#endif
#ifdef POWER8_VSX_CRC32
    if (cf.power.has_arch_2_07)
        ft.crc32 = &crc32_power8;
#endif
    // Power9
#ifdef POWER9
    if (cf.power.has_arch_3_00) {
        ft.compare256 = &compare256_power9;
        ft.longest_match = &longest_match_power9;
        ft.longest_match_slow = &longest_match_slow_power9;
    }
#endif


    // RISCV - RVV
#ifdef RISCV_RVV
    if (cf.riscv.has_rvv) {
        ft.adler32 = &adler32_rvv;
        ft.adler32_fold_copy = &adler32_fold_copy_rvv;
        ft.chunkmemset_safe = &chunkmemset_safe_rvv;
        ft.chunksize = &chunksize_rvv;
        ft.compare256 = &compare256_rvv;
        ft.inflate_fast = &inflate_fast_rvv;
        ft.longest_match = &longest_match_rvv;
        ft.longest_match_slow = &longest_match_slow_rvv;
        ft.slide_hash = &slide_hash_rvv;
    }
#endif


    // S390
#ifdef S390_CRC32_VX
    if (cf.s390.has_vx)
        ft.crc32 = crc32_s390_vx;
#endif

    // Assign function pointers individually for atomic operation
    FUNCTABLE_ASSIGN(ft, force_init);
    FUNCTABLE_ASSIGN(ft, adler32);
    FUNCTABLE_ASSIGN(ft, adler32_fold_copy);
    FUNCTABLE_ASSIGN(ft, chunkmemset_safe);
    FUNCTABLE_ASSIGN(ft, chunksize);
    FUNCTABLE_ASSIGN(ft, compare256);
    FUNCTABLE_ASSIGN(ft, crc32);
    FUNCTABLE_ASSIGN(ft, crc32_fold);
    FUNCTABLE_ASSIGN(ft, crc32_fold_copy);
    FUNCTABLE_ASSIGN(ft, crc32_fold_final);
    FUNCTABLE_ASSIGN(ft, crc32_fold_reset);
    FUNCTABLE_ASSIGN(ft, inflate_fast);
    FUNCTABLE_ASSIGN(ft, longest_match);
    FUNCTABLE_ASSIGN(ft, longest_match_slow);
    FUNCTABLE_ASSIGN(ft, slide_hash);

    // Memory barrier for weak memory order CPUs
    FUNCTABLE_BARRIER();
}

/* stub functions */
static void force_init_stub(void) {
    init_functable();
}

static uint32_t adler32_stub(uint32_t adler, const uint8_t* buf, size_t len) {
    init_functable();
    return functable.adler32(adler, buf, len);
}

static uint32_t adler32_fold_copy_stub(uint32_t adler, uint8_t* dst, const uint8_t* src, size_t len) {
    init_functable();
    return functable.adler32_fold_copy(adler, dst, src, len);
}

static uint8_t* chunkmemset_safe_stub(uint8_t* out, unsigned dist, unsigned len, unsigned left) {
    init_functable();
    return functable.chunkmemset_safe(out, dist, len, left);
}

static uint32_t chunksize_stub(void) {
    init_functable();
    return functable.chunksize();
}

static uint32_t compare256_stub(const uint8_t* src0, const uint8_t* src1) {
    init_functable();
    return functable.compare256(src0, src1);
}

static uint32_t crc32_stub(uint32_t crc, const uint8_t* buf, size_t len) {
    init_functable();
    return functable.crc32(crc, buf, len);
}

static void crc32_fold_stub(crc32_fold* crc, const uint8_t* src, size_t len, uint32_t init_crc) {
    init_functable();
    functable.crc32_fold(crc, src, len, init_crc);
}

static void crc32_fold_copy_stub(crc32_fold* crc, uint8_t* dst, const uint8_t* src, size_t len) {
    init_functable();
    functable.crc32_fold_copy(crc, dst, src, len);
}

static uint32_t crc32_fold_final_stub(crc32_fold* crc) {
    init_functable();
    return functable.crc32_fold_final(crc);
}

static uint32_t crc32_fold_reset_stub(crc32_fold* crc) {
    init_functable();
    return functable.crc32_fold_reset(crc);
}

static void inflate_fast_stub(PREFIX3(stream) *strm, uint32_t start) {
    init_functable();
    functable.inflate_fast(strm, start);
}

static uint32_t longest_match_stub(deflate_state* const s, Pos cur_match) {
    init_functable();
    return functable.longest_match(s, cur_match);
}

static uint32_t longest_match_slow_stub(deflate_state* const s, Pos cur_match) {
    init_functable();
    return functable.longest_match_slow(s, cur_match);
}

static void slide_hash_stub(deflate_state* s) {
    init_functable();
    functable.slide_hash(s);
}

/* functable init */
Z_INTERNAL struct functable_s functable = {
    force_init_stub,
    adler32_stub,
    adler32_fold_copy_stub,
    chunkmemset_safe_stub,
    chunksize_stub,
    compare256_stub,
    crc32_stub,
    crc32_fold_stub,
    crc32_fold_copy_stub,
    crc32_fold_final_stub,
    crc32_fold_reset_stub,
    inflate_fast_stub,
    longest_match_stub,
    longest_match_slow_stub,
    slide_hash_stub,
};

#endif
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **cpu_features**: A class/struct defined in this file
- **functable_s**: A class/struct defined in this file

### Functions and Methods

- **ARM_NOCHECK_SIMD()**: A function/method defined in this file
- **ARM_ACLE()**: A function/method defined in this file
- **PPC_VMX()**: A function/method defined in this file
- **POWER8_VSX_CRC32()**: A function/method defined in this file
- **X86_AVX512()**: A function/method defined in this file
- **DISABLE_RUNTIME_CPU_DETECTION()**: A function/method defined in this file
- **X86_SSE42()**: A function/method defined in this file
- **HAVE_BUILTIN_CTZ()**: A function/method defined in this file
- **HAVE_BUILTIN_CTZLL()**: A function/method defined in this file
- **POWER9()**: A function/method defined in this file
- **X86_AVX2()**: A function/method defined in this file
- **pointers()**: A function/method defined in this file
- **RISCV_RVV()**: A function/method defined in this file
- **X86_PCLMULQDQ_CRC()**: A function/method defined in this file
- **ARM_NEON()**: A function/method defined in this file
- **ARM_SIMD()**: A function/method defined in this file
- **X86_SSE2()**: A function/method defined in this file
- **X86_SSSE3()**: A function/method defined in this file
- **S390_CRC32_VX()**: A function/method defined in this file
- **X86_AVX512VNNI()**: A function/method defined in this file
- **ARM_NOCHECK_NEON()**: A function/method defined in this file
- **POWER8_VSX()**: A function/method defined in this file
- **X86_VPCLMULQDQ_CRC()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `zbuild.h`
- `cpu_features.h`
- `functable.h`
- `arch_functions.h`


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

