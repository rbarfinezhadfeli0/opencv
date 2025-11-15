# Documentation for `docs/3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c_docs.md`
- **File Name**: `jcsample-mmi.c_docs.md`
- **File Size**: 6,876 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c_docs.md](../../../../../docs/3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/mips64` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c`
- **File Name**: `jcsample-mmi.c`
- **File Size**: 3,838 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c](../../../../3rdparty/libjpeg-turbo/simd/mips64/jcsample-mmi.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/mips64` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Loongson MMI optimizations for libjpeg-turbo
 *
 * Copyright (C) 2015, 2018-2019, D. R. Commander.  All Rights Reserved.
 * Copyright (C) 2016-2017, Loongson Technology Corporation Limited, BeiJing.
 *                          All Rights Reserved.
 * Authors:  ZhuChen     <zhuchen@loongson.cn>
 *           CaiWanwei   <caiwanwei@loongson.cn>
 *           SunZhangzhi <sunzhangzhi-cq@loongson.cn>
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

/* CHROMA DOWNSAMPLING */

#include "jsimd_mmi.h"
#include "jcsample.h"


void jsimd_h2v2_downsample_mmi(JDIMENSION image_width, int max_v_samp_factor,
                               JDIMENSION v_samp_factor,
                               JDIMENSION width_in_blocks,
                               JSAMPARRAY input_data, JSAMPARRAY output_data)
{
  int inrow, outrow, outcol;
  JDIMENSION output_cols = width_in_blocks * DCTSIZE;
  JSAMPROW inptr0, inptr1, outptr;
  __m64 bias, mask = 0.0, thisavg, nextavg, avg;
  __m64 this0o, this0e, this0, this0sum, next0o, next0e, next0, next0sum;
  __m64 this1o, this1e, this1, this1sum, next1o, next1e, next1, next1sum;

  expand_right_edge(input_data, max_v_samp_factor, image_width,
                    output_cols * 2);

  bias = _mm_set1_pi32((1 << 17) + 1);   /* 0x00020001 (32-bit bias pattern) */
                                         /* bias={1, 2, 1, 2} (16-bit) */
  mask = _mm_cmpeq_pi16(mask, mask);
  mask = _mm_srli_pi16(mask, BYTE_BIT);  /* {0xFF 0x00 0xFF 0x00 ..} */

  for (inrow = 0, outrow = 0; outrow < v_samp_factor;
       inrow += 2, outrow++) {

    inptr0 = input_data[inrow];
    inptr1 = input_data[inrow + 1];
    outptr = output_data[outrow];

    for (outcol = output_cols; outcol > 0;
         outcol -= 8, inptr0 += 16, inptr1 += 16, outptr += 8) {

      this0 = _mm_load_si64((__m64 *)&inptr0[0]);
      this1 = _mm_load_si64((__m64 *)&inptr1[0]);
      next0 = _mm_load_si64((__m64 *)&inptr0[8]);
      next1 = _mm_load_si64((__m64 *)&inptr1[8]);

      this0o = _mm_and_si64(this0, mask);
      this0e = _mm_srli_pi16(this0, BYTE_BIT);
      this1o = _mm_and_si64(this1, mask);
      this1e = _mm_srli_pi16(this1, BYTE_BIT);
      this0sum = _mm_add_pi16(this0o, this0e);
      this1sum = _mm_add_pi16(this1o, this1e);

      next0o = _mm_and_si64(next0, mask);
      next0e = _mm_srli_pi16(next0, BYTE_BIT);
      next1o = _mm_and_si64(next1, mask);
      next1e = _mm_srli_pi16(next1, BYTE_BIT);
      next0sum = _mm_add_pi16(next0o, next0e);
      next1sum = _mm_add_pi16(next1o, next1e);

      thisavg = _mm_add_pi16(this0sum, this1sum);
      nextavg = _mm_add_pi16(next0sum, next1sum);
      thisavg = _mm_add_pi16(thisavg, bias);
      nextavg = _mm_add_pi16(nextavg, bias);
      thisavg = _mm_srli_pi16(thisavg, 2);
      nextavg = _mm_srli_pi16(nextavg, 2);

      avg = _mm_packs_pu16(thisavg, nextavg);

      _mm_store_si64((__m64 *)&outptr[0], avg);
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `jcsample.h`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

