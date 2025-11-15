# Documentation for `3rdparty/libjpeg-turbo/simd/mips64/jquanti-mmi.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/mips64/jquanti-mmi.c`
- **File Name**: `jquanti-mmi.c`
- **File Size**: 4,498 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/mips64/jquanti-mmi.c](../../../../3rdparty/libjpeg-turbo/simd/mips64/jquanti-mmi.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/mips64` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Loongson MMI optimizations for libjpeg-turbo
 *
 * Copyright (C) 2016-2017, Loongson Technology Corporation Limited, BeiJing.
 *                          All Rights Reserved.
 * Authors:  ZhuChen     <zhuchen@loongson.cn>
 *           CaiWanwei   <caiwanwei@loongson.cn>
 *           SunZhangzhi <sunzhangzhi-cq@loongson.cn>
 * Copyright (C) 2018-2019, D. R. Commander.  All Rights Reserved.
 *
 * Based on the x86 SIMD extension for IJG JPEG library
 * Copyright (C) 1999-2006, MIYASAKA Masaru.
 *
 * This software is provided 'as-is', without any express or implied
 * warranty.  In no event will the authors be held liable for any damages
 * arising from the use of this software.
 *
 * Permission is granted to anyone to use this software for any purpose,
 * including commercial applications, and to alter it and redistribute it
 * freely, subject to the following restrictions:
 *
 * 1. The origin of this software must not be misrepresented; you must not
 *    claim that you wrote the original software. If you use this software
 *    in a product, an acknowledgment in the product documentation would be
 *    appreciated but is not required.
 * 2. Altered source versions must be plainly marked as such, and must not be
 *    misrepresented as being the original software.
 * 3. This notice may not be removed or altered from any source distribution.
 */

/* INTEGER QUANTIZATION AND SAMPLE CONVERSION */

#include "jsimd_mmi.h"


#define DO_QUANT() { \
  __m64 rowl, rowh, rowls, rowhs, rowlsave, rowhsave; \
  __m64 corrl, corrh, recipl, reciph, scalel, scaleh; \
  \
  rowl = _mm_load_si64((__m64 *)&workspace[0]); \
  rowh = _mm_load_si64((__m64 *)&workspace[4]); \
  \
  /* Branch-less absolute value */ \
  rowls = _mm_srai_pi16(rowl, (WORD_BIT - 1));  /* -1 if value < 0, */ \
                                                /* 0 otherwise */ \
  rowhs = _mm_srai_pi16(rowh, (WORD_BIT - 1)); \
  \
  rowl = _mm_xor_si64(rowl, rowls);           /* val = -val */ \
  rowh = _mm_xor_si64(rowh, rowhs); \
  rowl = _mm_sub_pi16(rowl, rowls); \
  rowh = _mm_sub_pi16(rowh, rowhs); \
  \
  corrl = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 1]);  /* correction */ \
  corrh = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 1 + 4]); \
  \
  rowlsave = rowl = _mm_add_pi16(rowl, corrl);  /* correction + roundfactor */ \
  rowhsave = rowh = _mm_add_pi16(rowh, corrh); \
  \
  recipl = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 0]);  /* reciprocal */ \
  reciph = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 0 + 4]); \
  \
  rowl = _mm_mulhi_pi16(rowl, recipl); \
  rowh = _mm_mulhi_pi16(rowh, reciph); \
  \
  /* reciprocal is always negative (MSB=1), so we always need to add the */ \
  /* initial value (input value is never negative as we inverted it at the */ \
  /* start of this routine) */ \
  rowlsave = rowl = _mm_add_pi16(rowl, rowlsave); \
  rowhsave = rowh = _mm_add_pi16(rowh, rowhsave); \
  \
  scalel = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 2]);  /* scale */ \
  scaleh = _mm_load_si64((__m64 *)&divisors[DCTSIZE2 * 2 + 4]); \
  \
  rowl = _mm_mulhi_pi16(rowl, scalel); \
  rowh = _mm_mulhi_pi16(rowh, scaleh); \
  \
  /* determine if scale is negative */ \
  scalel = _mm_srai_pi16(scalel, (WORD_BIT - 1)); \
  scaleh = _mm_srai_pi16(scaleh, (WORD_BIT - 1)); \
  \
  /* and add input if it is */ \
  scalel = _mm_and_si64(scalel, rowlsave); \
  scaleh = _mm_and_si64(scaleh, rowhsave); \
  rowl = _mm_add_pi16(rowl, scalel); \
  rowh = _mm_add_pi16(rowh, scaleh); \
  \
  /* then check if negative input */ \
  rowlsave = _mm_srai_pi16(rowlsave, (WORD_BIT - 1)); \
  rowhsave = _mm_srai_pi16(rowhsave, (WORD_BIT - 1)); \
  \
  /* and add scale if it is */ \
  rowlsave = _mm_and_si64(rowlsave, scalel); \
  rowhsave = _mm_and_si64(rowhsave, scaleh); \
  rowl = _mm_add_pi16(rowl, rowlsave); \
  rowh = _mm_add_pi16(rowh, rowhsave); \
  \
  rowl = _mm_xor_si64(rowl, rowls);           /* val = -val */ \
  rowh = _mm_xor_si64(rowh, rowhs); \
  rowl = _mm_sub_pi16(rowl, rowls); \
  rowh = _mm_sub_pi16(rowh, rowhs); \
  \
  _mm_store_si64((__m64 *)&output_ptr[0], rowl); \
  _mm_store_si64((__m64 *)&output_ptr[4], rowh); \
  \
  workspace += DCTSIZE; \
  divisors += DCTSIZE; \
  output_ptr += DCTSIZE; \
}


void jsimd_quantize_mmi(JCOEFPTR coef_block, DCTELEM *divisors,
                        DCTELEM *workspace)
{
  JCOEFPTR output_ptr = coef_block;

  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
  DO_QUANT()
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `jsimd_mmi.h`

**Python Imports:**
- `any`
- `the`


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

