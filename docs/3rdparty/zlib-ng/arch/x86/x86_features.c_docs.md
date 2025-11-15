# Documentation for `3rdparty/zlib-ng/arch/x86/x86_features.c`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/x86/x86_features.c`
- **File Name**: `x86_features.c`
- **File Size**: 3,508 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/zlib-ng/arch/x86/x86_features.c](../../../../3rdparty/zlib-ng/arch/x86/x86_features.c)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/x86` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* x86_features.c - x86 feature check
 *
 * Copyright (C) 2013 Intel Corporation. All rights reserved.
 * Author:
 *  Jim Kukunas
 *
 * For conditions of distribution and use, see copyright notice in zlib.h
 */

#include "zbuild.h"
#include "x86_features.h"

#ifdef _MSC_VER
#  include <intrin.h>
#else
// Newer versions of GCC and clang come with cpuid.h
#  include <cpuid.h>
#  ifdef X86_HAVE_XSAVE_INTRIN
#    if __GNUC__ == 8
#      include <xsaveintrin.h>
#    else
#      include <immintrin.h>
#    endif
#  endif
#endif

#include <string.h>

static inline void cpuid(int info, unsigned* eax, unsigned* ebx, unsigned* ecx, unsigned* edx) {
#ifdef _MSC_VER
    unsigned int registers[4];
    __cpuid((int *)registers, info);

    *eax = registers[0];
    *ebx = registers[1];
    *ecx = registers[2];
    *edx = registers[3];
#else
    *eax = *ebx = *ecx = *edx = 0;
    __cpuid(info, *eax, *ebx, *ecx, *edx);
#endif
}

static inline void cpuidex(int info, int subinfo, unsigned* eax, unsigned* ebx, unsigned* ecx, unsigned* edx) {
#ifdef _MSC_VER
    unsigned int registers[4];
    __cpuidex((int *)registers, info, subinfo);

    *eax = registers[0];
    *ebx = registers[1];
    *ecx = registers[2];
    *edx = registers[3];
#else
    *eax = *ebx = *ecx = *edx = 0;
    __cpuid_count(info, subinfo, *eax, *ebx, *ecx, *edx);
#endif
}

static inline uint64_t xgetbv(unsigned int xcr) {
#if defined(_MSC_VER) || defined(X86_HAVE_XSAVE_INTRIN)
    return _xgetbv(xcr);
#else
    uint32_t eax, edx;
    __asm__ ( ".byte 0x0f, 0x01, 0xd0" : "=a"(eax), "=d"(edx) : "c"(xcr));
    return (uint64_t)(edx) << 32 | eax;
#endif
}

void Z_INTERNAL x86_check_features(struct x86_cpu_features *features) {
    unsigned eax, ebx, ecx, edx;
    unsigned maxbasic;

    cpuid(0, &maxbasic, &ebx, &ecx, &edx);
    cpuid(1 /*CPU_PROCINFO_AND_FEATUREBITS*/, &eax, &ebx, &ecx, &edx);

    features->has_sse2 = edx & 0x4000000;
    features->has_ssse3 = ecx & 0x200;
    features->has_sse42 = ecx & 0x100000;
    features->has_pclmulqdq = ecx & 0x2;

    if (ecx & 0x08000000) {
        uint64_t xfeature = xgetbv(0);

        features->has_os_save_ymm = ((xfeature & 0x06) == 0x06);
        features->has_os_save_zmm = ((xfeature & 0xe6) == 0xe6);
    }

    if (maxbasic >= 7) {
        cpuidex(7, 0, &eax, &ebx, &ecx, &edx);

        // check BMI1 bit
        // Reference: https://software.intel.com/sites/default/files/article/405250/how-to-detect-new-instruction-support-in-the-4th-generation-intel-core-processor-family.pdf
        features->has_vpclmulqdq = ecx & 0x400;

        // check AVX2 bit if the OS supports saving YMM registers
        if (features->has_os_save_ymm) {
            features->has_avx2 = ebx & 0x20;
        }

        // check AVX512 bits if the OS supports saving ZMM registers
        if (features->has_os_save_zmm) {
            features->has_avx512f = ebx & 0x00010000;
            if (features->has_avx512f) {
                // According to the Intel Software Developer's Manual, AVX512F must be enabled too in order to enable
                // AVX512(DQ,BW,VL).
                features->has_avx512dq = ebx & 0x00020000;
                features->has_avx512bw = ebx & 0x40000000;
                features->has_avx512vl = ebx & 0x80000000;
            }
            features->has_avx512_common = features->has_avx512f && features->has_avx512dq && features->has_avx512bw \
              && features->has_avx512vl;
            features->has_avx512vnni = ecx & 0x800;
        }
    }
}
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

- **x86_cpu_features**: A class/struct defined in this file

### Functions and Methods

- **X86_HAVE_XSAVE_INTRIN()**: A function/method defined in this file
- **_MSC_VER()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `zbuild.h`
- `x86_features.h`
- `string.h`


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

