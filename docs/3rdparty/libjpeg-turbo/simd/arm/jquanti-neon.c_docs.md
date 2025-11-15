# Documentation for `3rdparty/libjpeg-turbo/simd/arm/jquanti-neon.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/arm/jquanti-neon.c`
- **File Name**: `jquanti-neon.c`
- **File Size**: 9,096 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/arm/jquanti-neon.c](../../../../3rdparty/libjpeg-turbo/simd/arm/jquanti-neon.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/arm` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jquanti-neon.c - sample data conversion and quantization (Arm Neon)
 *
 * Copyright (C) 2020-2021, Arm Limited.  All Rights Reserved.
 * Copyright (C) 2024, D. R. Commander.  All Rights Reserved.
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

#define JPEG_INTERNALS
#include "../../src/jinclude.h"
#include "../../src/jpeglib.h"
#include "../../src/jsimd.h"
#include "../../src/jdct.h"
#include "../../src/jsimddct.h"
#include "../jsimd.h"
#include "neon-compat.h"

#include <arm_neon.h>


/* After downsampling, the resulting sample values are in the range [0, 255],
 * but the Discrete Cosine Transform (DCT) operates on values centered around
 * 0.
 *
 * To prepare sample values for the DCT, load samples into a DCT workspace,
 * subtracting CENTERJSAMPLE (128).  The samples, now in the range [-128, 127],
 * are also widened from 8- to 16-bit.
 *
 * The equivalent scalar C function convsamp() can be found in jcdctmgr.c.
 */

void jsimd_convsamp_neon(JSAMPARRAY sample_data, JDIMENSION start_col,
                         DCTELEM *workspace)
{
  uint8x8_t samp_row0 = vld1_u8(sample_data[0] + start_col);
  uint8x8_t samp_row1 = vld1_u8(sample_data[1] + start_col);
  uint8x8_t samp_row2 = vld1_u8(sample_data[2] + start_col);
  uint8x8_t samp_row3 = vld1_u8(sample_data[3] + start_col);
  uint8x8_t samp_row4 = vld1_u8(sample_data[4] + start_col);
  uint8x8_t samp_row5 = vld1_u8(sample_data[5] + start_col);
  uint8x8_t samp_row6 = vld1_u8(sample_data[6] + start_col);
  uint8x8_t samp_row7 = vld1_u8(sample_data[7] + start_col);

  int16x8_t row0 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row0, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row1 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row1, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row2 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row2, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row3 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row3, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row4 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row4, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row5 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row5, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row6 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row6, vdup_n_u8(CENTERJSAMPLE)));
  int16x8_t row7 =
    vreinterpretq_s16_u16(vsubl_u8(samp_row7, vdup_n_u8(CENTERJSAMPLE)));

  vst1q_s16(workspace + 0 * DCTSIZE, row0);
  vst1q_s16(workspace + 1 * DCTSIZE, row1);
  vst1q_s16(workspace + 2 * DCTSIZE, row2);
  vst1q_s16(workspace + 3 * DCTSIZE, row3);
  vst1q_s16(workspace + 4 * DCTSIZE, row4);
  vst1q_s16(workspace + 5 * DCTSIZE, row5);
  vst1q_s16(workspace + 6 * DCTSIZE, row6);
  vst1q_s16(workspace + 7 * DCTSIZE, row7);
}


/* After the DCT, the resulting array of coefficient values needs to be divided
 * by an array of quantization values.
 *
 * To avoid a slow division operation, the DCT coefficients are multiplied by
 * the (scaled) reciprocals of the quantization values and then right-shifted.
 *
 * The equivalent scalar C function quantize() can be found in jcdctmgr.c.
 */

void jsimd_quantize_neon(JCOEFPTR coef_block, DCTELEM *divisors,
                         DCTELEM *workspace)
{
  JCOEFPTR out_ptr = coef_block;
  UDCTELEM *recip_ptr = (UDCTELEM *)divisors;
  UDCTELEM *corr_ptr = (UDCTELEM *)divisors + DCTSIZE2;
  DCTELEM *shift_ptr = divisors + 3 * DCTSIZE2;
  int i;

#if defined(__clang__) && (defined(__aarch64__) || defined(_M_ARM64))
#pragma unroll
#endif
  for (i = 0; i < DCTSIZE; i += DCTSIZE / 2) {
    /* Load DCT coefficients. */
    int16x8_t row0 = vld1q_s16(workspace + (i + 0) * DCTSIZE);
    int16x8_t row1 = vld1q_s16(workspace + (i + 1) * DCTSIZE);
    int16x8_t row2 = vld1q_s16(workspace + (i + 2) * DCTSIZE);
    int16x8_t row3 = vld1q_s16(workspace + (i + 3) * DCTSIZE);
    /* Load reciprocals of quantization values. */
    uint16x8_t recip0 = vld1q_u16(recip_ptr + (i + 0) * DCTSIZE);
    uint16x8_t recip1 = vld1q_u16(recip_ptr + (i + 1) * DCTSIZE);
    uint16x8_t recip2 = vld1q_u16(recip_ptr + (i + 2) * DCTSIZE);
    uint16x8_t recip3 = vld1q_u16(recip_ptr + (i + 3) * DCTSIZE);
    uint16x8_t corr0 = vld1q_u16(corr_ptr + (i + 0) * DCTSIZE);
    uint16x8_t corr1 = vld1q_u16(corr_ptr + (i + 1) * DCTSIZE);
    uint16x8_t corr2 = vld1q_u16(corr_ptr + (i + 2) * DCTSIZE);
    uint16x8_t corr3 = vld1q_u16(corr_ptr + (i + 3) * DCTSIZE);
    int16x8_t shift0 = vld1q_s16(shift_ptr + (i + 0) * DCTSIZE);
    int16x8_t shift1 = vld1q_s16(shift_ptr + (i + 1) * DCTSIZE);
    int16x8_t shift2 = vld1q_s16(shift_ptr + (i + 2) * DCTSIZE);
    int16x8_t shift3 = vld1q_s16(shift_ptr + (i + 3) * DCTSIZE);

    /* Extract sign from coefficients. */
    int16x8_t sign_row0 = vshrq_n_s16(row0, 15);
    int16x8_t sign_row1 = vshrq_n_s16(row1, 15);
    int16x8_t sign_row2 = vshrq_n_s16(row2, 15);
    int16x8_t sign_row3 = vshrq_n_s16(row3, 15);
    /* Get absolute value of DCT coefficients. */
    uint16x8_t abs_row0 = vreinterpretq_u16_s16(vabsq_s16(row0));
    uint16x8_t abs_row1 = vreinterpretq_u16_s16(vabsq_s16(row1));
    uint16x8_t abs_row2 = vreinterpretq_u16_s16(vabsq_s16(row2));
    uint16x8_t abs_row3 = vreinterpretq_u16_s16(vabsq_s16(row3));
    /* Add correction. */
    abs_row0 = vaddq_u16(abs_row0, corr0);
    abs_row1 = vaddq_u16(abs_row1, corr1);
    abs_row2 = vaddq_u16(abs_row2, corr2);
    abs_row3 = vaddq_u16(abs_row3, corr3);

    /* Multiply DCT coefficients by quantization reciprocals. */
    int32x4_t row0_l = vreinterpretq_s32_u32(vmull_u16(vget_low_u16(abs_row0),
                                                       vget_low_u16(recip0)));
    int32x4_t row0_h = vreinterpretq_s32_u32(vmull_u16(vget_high_u16(abs_row0),
                                                       vget_high_u16(recip0)));
    int32x4_t row1_l = vreinterpretq_s32_u32(vmull_u16(vget_low_u16(abs_row1),
                                                       vget_low_u16(recip1)));
    int32x4_t row1_h = vreinterpretq_s32_u32(vmull_u16(vget_high_u16(abs_row1),
                                                       vget_high_u16(recip1)));
    int32x4_t row2_l = vreinterpretq_s32_u32(vmull_u16(vget_low_u16(abs_row2),
                                                       vget_low_u16(recip2)));
    int32x4_t row2_h = vreinterpretq_s32_u32(vmull_u16(vget_high_u16(abs_row2),
                                                       vget_high_u16(recip2)));
    int32x4_t row3_l = vreinterpretq_s32_u32(vmull_u16(vget_low_u16(abs_row3),
                                                       vget_low_u16(recip3)));
    int32x4_t row3_h = vreinterpretq_s32_u32(vmull_u16(vget_high_u16(abs_row3),
                                                       vget_high_u16(recip3)));
    /* Narrow back to 16-bit. */
    row0 = vcombine_s16(vshrn_n_s32(row0_l, 16), vshrn_n_s32(row0_h, 16));
    row1 = vcombine_s16(vshrn_n_s32(row1_l, 16), vshrn_n_s32(row1_h, 16));
    row2 = vcombine_s16(vshrn_n_s32(row2_l, 16), vshrn_n_s32(row2_h, 16));
    row3 = vcombine_s16(vshrn_n_s32(row3_l, 16), vshrn_n_s32(row3_h, 16));

    /* Since VSHR only supports an immediate as its second argument, negate the
     * shift value and shift left.
     */
    row0 = vreinterpretq_s16_u16(vshlq_u16(vreinterpretq_u16_s16(row0),
                                           vnegq_s16(shift0)));
    row1 = vreinterpretq_s16_u16(vshlq_u16(vreinterpretq_u16_s16(row1),
                                           vnegq_s16(shift1)));
    row2 = vreinterpretq_s16_u16(vshlq_u16(vreinterpretq_u16_s16(row2),
                                           vnegq_s16(shift2)));
    row3 = vreinterpretq_s16_u16(vshlq_u16(vreinterpretq_u16_s16(row3),
                                           vnegq_s16(shift3)));

    /* Restore sign to original product. */
    row0 = veorq_s16(row0, sign_row0);
    row0 = vsubq_s16(row0, sign_row0);
    row1 = veorq_s16(row1, sign_row1);
    row1 = vsubq_s16(row1, sign_row1);
    row2 = veorq_s16(row2, sign_row2);
    row2 = vsubq_s16(row2, sign_row2);
    row3 = veorq_s16(row3, sign_row3);
    row3 = vsubq_s16(row3, sign_row3);

    /* Store quantized coefficients to memory. */
    vst1q_s16(out_ptr + (i + 0) * DCTSIZE, row0);
    vst1q_s16(out_ptr + (i + 1) * DCTSIZE, row1);
    vst1q_s16(out_ptr + (i + 2) * DCTSIZE, row2);
    vst1q_s16(out_ptr + (i + 3) * DCTSIZE, row3);
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

### Functions and Methods

- **quantize()**: A function/method defined in this file
- **convsamp()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../src/jsimddct.h`
- `../../src/jdct.h`
- `arm_neon.h`
- `neon-compat.h`
- `../jsimd.h`
- `../../src/jpeglib.h`
- `../../src/jsimd.h`
- `../../src/jinclude.h`

**Python Imports:**
- `coefficients.`
- `any`
- `8`
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

