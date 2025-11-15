# Documentation for `docs/3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c_docs.md`
- **File Name**: `jcsample-neon.c_docs.md`
- **File Size**: 11,907 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c_docs.md](../../../../../docs/3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/arm` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c`
- **File Name**: `jcsample-neon.c`
- **File Size**: 8,733 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c](../../../../3rdparty/libjpeg-turbo/simd/arm/jcsample-neon.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/arm` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jcsample-neon.c - downsampling (Arm Neon)
 *
 * Copyright (C) 2020, Arm Limited.  All Rights Reserved.
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
#include "align.h"
#include "neon-compat.h"

#include <arm_neon.h>


ALIGN(16) static const uint8_t jsimd_h2_downsample_consts[] = {
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 0 */
  0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 1 */
  0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0E,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 2 */
  0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0D, 0x0D,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 3 */
  0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0C, 0x0C, 0x0C,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 4 */
  0x08, 0x09, 0x0A, 0x0B, 0x0B, 0x0B, 0x0B, 0x0B,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 5 */
  0x08, 0x09, 0x0A, 0x0A, 0x0A, 0x0A, 0x0A, 0x0A,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 6 */
  0x08, 0x09, 0x09, 0x09, 0x09, 0x09, 0x09, 0x09,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 7 */
  0x08, 0x08, 0x08, 0x08, 0x08, 0x08, 0x08, 0x08,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,   /* Pad 8 */
  0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x06,   /* Pad 9 */
  0x06, 0x06, 0x06, 0x06, 0x06, 0x06, 0x06, 0x06,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x05, 0x05,   /* Pad 10 */
  0x05, 0x05, 0x05, 0x05, 0x05, 0x05, 0x05, 0x05,
  0x00, 0x01, 0x02, 0x03, 0x04, 0x04, 0x04, 0x04,   /* Pad 11 */
  0x04, 0x04, 0x04, 0x04, 0x04, 0x04, 0x04, 0x04,
  0x00, 0x01, 0x02, 0x03, 0x03, 0x03, 0x03, 0x03,   /* Pad 12 */
  0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03,
  0x00, 0x01, 0x02, 0x02, 0x02, 0x02, 0x02, 0x02,   /* Pad 13 */
  0x02, 0x02, 0x02, 0x02, 0x02, 0x02, 0x02, 0x02,
  0x00, 0x01, 0x01, 0x01, 0x01, 0x01, 0x01, 0x01,   /* Pad 14 */
  0x01, 0x01, 0x01, 0x01, 0x01, 0x01, 0x01, 0x01,
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,   /* Pad 15 */
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};


/* Downsample pixel values of a single component.
 * This version handles the common case of 2:1 horizontal and 1:1 vertical,
 * without smoothing.
 */

void jsimd_h2v1_downsample_neon(JDIMENSION image_width, int max_v_samp_factor,
                                JDIMENSION v_samp_factor,
                                JDIMENSION width_in_blocks,
                                JSAMPARRAY input_data, JSAMPARRAY output_data)
{
  JSAMPROW inptr, outptr;
  /* Load expansion mask to pad remaining elements of last DCT block. */
  const int mask_offset = 16 * ((width_in_blocks * 2 * DCTSIZE) - image_width);
  const uint8x16_t expand_mask =
    vld1q_u8(&jsimd_h2_downsample_consts[mask_offset]);
  /* Load bias pattern (alternating every pixel.) */
  /* { 0, 1, 0, 1, 0, 1, 0, 1 } */
  const uint16x8_t bias = vreinterpretq_u16_u32(vdupq_n_u32(0x00010000));
  unsigned i, outrow;

  for (outrow = 0; outrow < v_samp_factor; outrow++) {
    outptr = output_data[outrow];
    inptr = input_data[outrow];

    /* Downsample all but the last DCT block of pixels. */
    for (i = 0; i < width_in_blocks - 1; i++) {
      uint8x16_t pixels = vld1q_u8(inptr + i * 2 * DCTSIZE);
      /* Add adjacent pixel values, widen to 16-bit, and add bias. */
      uint16x8_t samples_u16 = vpadalq_u8(bias, pixels);
      /* Divide total by 2 and narrow to 8-bit. */
      uint8x8_t samples_u8 = vshrn_n_u16(samples_u16, 1);
      /* Store samples to memory. */
      vst1_u8(outptr + i * DCTSIZE, samples_u8);
    }

    /* Load pixels in last DCT block into a table. */
    uint8x16_t pixels = vld1q_u8(inptr + (width_in_blocks - 1) * 2 * DCTSIZE);
#if defined(__aarch64__) || defined(_M_ARM64)
    /* Pad the empty elements with the value of the last pixel. */
    pixels = vqtbl1q_u8(pixels, expand_mask);
#else
    uint8x8x2_t table = { { vget_low_u8(pixels), vget_high_u8(pixels) } };
    pixels = vcombine_u8(vtbl2_u8(table, vget_low_u8(expand_mask)),
                         vtbl2_u8(table, vget_high_u8(expand_mask)));
#endif
    /* Add adjacent pixel values, widen to 16-bit, and add bias. */
    uint16x8_t samples_u16 = vpadalq_u8(bias, pixels);
    /* Divide total by 2, narrow to 8-bit, and store. */
    uint8x8_t samples_u8 = vshrn_n_u16(samples_u16, 1);
    vst1_u8(outptr + (width_in_blocks - 1) * DCTSIZE, samples_u8);
  }
}


/* Downsample pixel values of a single component.
 * This version handles the standard case of 2:1 horizontal and 2:1 vertical,
 * without smoothing.
 */

void jsimd_h2v2_downsample_neon(JDIMENSION image_width, int max_v_samp_factor,
                                JDIMENSION v_samp_factor,
                                JDIMENSION width_in_blocks,
                                JSAMPARRAY input_data, JSAMPARRAY output_data)
{
  JSAMPROW inptr0, inptr1, outptr;
  /* Load expansion mask to pad remaining elements of last DCT block. */
  const int mask_offset = 16 * ((width_in_blocks * 2 * DCTSIZE) - image_width);
  const uint8x16_t expand_mask =
    vld1q_u8(&jsimd_h2_downsample_consts[mask_offset]);
  /* Load bias pattern (alternating every pixel.) */
  /* { 1, 2, 1, 2, 1, 2, 1, 2 } */
  const uint16x8_t bias = vreinterpretq_u16_u32(vdupq_n_u32(0x00020001));
  unsigned i, outrow;

  for (outrow = 0; outrow < v_samp_factor; outrow++) {
    outptr = output_data[outrow];
    inptr0 = input_data[outrow];
    inptr1 = input_data[outrow + 1];

    /* Downsample all but the last DCT block of pixels. */
    for (i = 0; i < width_in_blocks - 1; i++) {
      uint8x16_t pixels_r0 = vld1q_u8(inptr0 + i * 2 * DCTSIZE);
      uint8x16_t pixels_r1 = vld1q_u8(inptr1 + i * 2 * DCTSIZE);
      /* Add adjacent pixel values in row 0, widen to 16-bit, and add bias. */
      uint16x8_t samples_u16 = vpadalq_u8(bias, pixels_r0);
      /* Add adjacent pixel values in row 1, widen to 16-bit, and accumulate.
       */
      samples_u16 = vpadalq_u8(samples_u16, pixels_r1);
      /* Divide total by 4 and narrow to 8-bit. */
      uint8x8_t samples_u8 = vshrn_n_u16(samples_u16, 2);
      /* Store samples to memory and increment pointers. */
      vst1_u8(outptr + i * DCTSIZE, samples_u8);
    }

    /* Load pixels in last DCT block into a table. */
    uint8x16_t pixels_r0 =
      vld1q_u8(inptr0 + (width_in_blocks - 1) * 2 * DCTSIZE);
    uint8x16_t pixels_r1 =
      vld1q_u8(inptr1 + (width_in_blocks - 1) * 2 * DCTSIZE);
#if defined(__aarch64__) || defined(_M_ARM64)
    /* Pad the empty elements with the value of the last pixel. */
    pixels_r0 = vqtbl1q_u8(pixels_r0, expand_mask);
    pixels_r1 = vqtbl1q_u8(pixels_r1, expand_mask);
#else
    uint8x8x2_t table_r0 =
      { { vget_low_u8(pixels_r0), vget_high_u8(pixels_r0) } };
    uint8x8x2_t table_r1 =
      { { vget_low_u8(pixels_r1), vget_high_u8(pixels_r1) } };
    pixels_r0 = vcombine_u8(vtbl2_u8(table_r0, vget_low_u8(expand_mask)),
                            vtbl2_u8(table_r0, vget_high_u8(expand_mask)));
    pixels_r1 = vcombine_u8(vtbl2_u8(table_r1, vget_low_u8(expand_mask)),
                            vtbl2_u8(table_r1, vget_high_u8(expand_mask)));
#endif
    /* Add adjacent pixel values in row 0, widen to 16-bit, and add bias. */
    uint16x8_t samples_u16 = vpadalq_u8(bias, pixels_r0);
    /* Add adjacent pixel values in row 1, widen to 16-bit, and accumulate. */
    samples_u16 = vpadalq_u8(samples_u16, pixels_r1);
    /* Divide total by 4, narrow to 8-bit, and store. */
    uint8x8_t samples_u8 = vshrn_n_u16(samples_u16, 2);
    vst1_u8(outptr + (width_in_blocks - 1) * DCTSIZE, samples_u8);
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
- `../../src/jsimddct.h`
- `../../src/jdct.h`
- `arm_neon.h`
- `neon-compat.h`
- `../jsimd.h`
- `../../src/jpeglib.h`
- `../../src/jsimd.h`
- `../../src/jinclude.h`
- `align.h`

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

