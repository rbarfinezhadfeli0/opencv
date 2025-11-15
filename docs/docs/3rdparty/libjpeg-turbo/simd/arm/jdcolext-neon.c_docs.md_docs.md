# Documentation for `docs/3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c_docs.md`
- **File Name**: `jdcolext-neon.c_docs.md`
- **File Size**: 19,411 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c_docs.md](../../../../../docs/3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/arm` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c`
- **File Name**: `jdcolext-neon.c`
- **File Size**: 16,425 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c](../../../../3rdparty/libjpeg-turbo/simd/arm/jdcolext-neon.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/arm` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jdcolext-neon.c - colorspace conversion (Arm Neon)
 *
 * Copyright (C) 2020, Arm Limited.  All Rights Reserved.
 * Copyright (C) 2020, D. R. Commander.  All Rights Reserved.
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

/* This file is included by jdcolor-neon.c. */


/* YCbCr -> RGB conversion is defined by the following equations:
 *    R = Y                        + 1.40200 * (Cr - 128)
 *    G = Y - 0.34414 * (Cb - 128) - 0.71414 * (Cr - 128)
 *    B = Y + 1.77200 * (Cb - 128)
 *
 * Scaled integer constants are used to avoid floating-point arithmetic:
 *    0.3441467 = 11277 * 2^-15
 *    0.7141418 = 23401 * 2^-15
 *    1.4020386 = 22971 * 2^-14
 *    1.7720337 = 29033 * 2^-14
 * These constants are defined in jdcolor-neon.c.
 *
 * To ensure correct results, rounding is used when descaling.
 */

/* Notes on safe memory access for YCbCr -> RGB conversion routines:
 *
 * Input memory buffers can be safely overread up to the next multiple of
 * ALIGN_SIZE bytes, since they are always allocated by alloc_sarray() in
 * jmemmgr.c.
 *
 * The output buffer cannot safely be written beyond output_width, since
 * output_buf points to a possibly unpadded row in the decompressed image
 * buffer allocated by the calling program.
 */

void jsimd_ycc_rgb_convert_neon(JDIMENSION output_width, JSAMPIMAGE input_buf,
                                JDIMENSION input_row, JSAMPARRAY output_buf,
                                int num_rows)
{
  JSAMPROW outptr;
  /* Pointers to Y, Cb, and Cr data */
  JSAMPROW inptr0, inptr1, inptr2;

  const int16x4_t consts = vld1_s16(jsimd_ycc_rgb_convert_neon_consts);
  const int16x8_t neg_128 = vdupq_n_s16(-128);

  while (--num_rows >= 0) {
    inptr0 = input_buf[0][input_row];
    inptr1 = input_buf[1][input_row];
    inptr2 = input_buf[2][input_row];
    input_row++;
    outptr = *output_buf++;
    int cols_remaining = output_width;
    for (; cols_remaining >= 16; cols_remaining -= 16) {
      uint8x16_t y  = vld1q_u8(inptr0);
      uint8x16_t cb = vld1q_u8(inptr1);
      uint8x16_t cr = vld1q_u8(inptr2);
      /* Subtract 128 from Cb and Cr. */
      int16x8_t cr_128_l =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128),
                                       vget_low_u8(cr)));
      int16x8_t cr_128_h =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128),
                                       vget_high_u8(cr)));
      int16x8_t cb_128_l =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128),
                                       vget_low_u8(cb)));
      int16x8_t cb_128_h =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128),
                                       vget_high_u8(cb)));
      /* Compute G-Y: - 0.34414 * (Cb - 128) - 0.71414 * (Cr - 128) */
      int32x4_t g_sub_y_ll = vmull_lane_s16(vget_low_s16(cb_128_l), consts, 0);
      int32x4_t g_sub_y_lh = vmull_lane_s16(vget_high_s16(cb_128_l),
                                            consts, 0);
      int32x4_t g_sub_y_hl = vmull_lane_s16(vget_low_s16(cb_128_h), consts, 0);
      int32x4_t g_sub_y_hh = vmull_lane_s16(vget_high_s16(cb_128_h),
                                            consts, 0);
      g_sub_y_ll = vmlsl_lane_s16(g_sub_y_ll, vget_low_s16(cr_128_l),
                                  consts, 1);
      g_sub_y_lh = vmlsl_lane_s16(g_sub_y_lh, vget_high_s16(cr_128_l),
                                  consts, 1);
      g_sub_y_hl = vmlsl_lane_s16(g_sub_y_hl, vget_low_s16(cr_128_h),
                                  consts, 1);
      g_sub_y_hh = vmlsl_lane_s16(g_sub_y_hh, vget_high_s16(cr_128_h),
                                  consts, 1);
      /* Descale G components: shift right 15, round, and narrow to 16-bit. */
      int16x8_t g_sub_y_l = vcombine_s16(vrshrn_n_s32(g_sub_y_ll, 15),
                                         vrshrn_n_s32(g_sub_y_lh, 15));
      int16x8_t g_sub_y_h = vcombine_s16(vrshrn_n_s32(g_sub_y_hl, 15),
                                         vrshrn_n_s32(g_sub_y_hh, 15));
      /* Compute R-Y: 1.40200 * (Cr - 128) */
      int16x8_t r_sub_y_l = vqrdmulhq_lane_s16(vshlq_n_s16(cr_128_l, 1),
                                               consts, 2);
      int16x8_t r_sub_y_h = vqrdmulhq_lane_s16(vshlq_n_s16(cr_128_h, 1),
                                               consts, 2);
      /* Compute B-Y: 1.77200 * (Cb - 128) */
      int16x8_t b_sub_y_l = vqrdmulhq_lane_s16(vshlq_n_s16(cb_128_l, 1),
                                               consts, 3);
      int16x8_t b_sub_y_h = vqrdmulhq_lane_s16(vshlq_n_s16(cb_128_h, 1),
                                               consts, 3);
      /* Add Y. */
      int16x8_t r_l =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(r_sub_y_l),
                                       vget_low_u8(y)));
      int16x8_t r_h =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(r_sub_y_h),
                                       vget_high_u8(y)));
      int16x8_t b_l =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(b_sub_y_l),
                                       vget_low_u8(y)));
      int16x8_t b_h =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(b_sub_y_h),
                                       vget_high_u8(y)));
      int16x8_t g_l =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(g_sub_y_l),
                                       vget_low_u8(y)));
      int16x8_t g_h =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(g_sub_y_h),
                                       vget_high_u8(y)));

#if RGB_PIXELSIZE == 4
      uint8x16x4_t rgba;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgba.val[RGB_RED] = vcombine_u8(vqmovun_s16(r_l), vqmovun_s16(r_h));
      rgba.val[RGB_GREEN] = vcombine_u8(vqmovun_s16(g_l), vqmovun_s16(g_h));
      rgba.val[RGB_BLUE] = vcombine_u8(vqmovun_s16(b_l), vqmovun_s16(b_h));
      /* Set alpha channel to opaque (0xFF). */
      rgba.val[RGB_ALPHA] = vdupq_n_u8(0xFF);
      /* Store RGBA pixel data to memory. */
      vst4q_u8(outptr, rgba);
#elif RGB_PIXELSIZE == 3
      uint8x16x3_t rgb;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgb.val[RGB_RED] = vcombine_u8(vqmovun_s16(r_l), vqmovun_s16(r_h));
      rgb.val[RGB_GREEN] = vcombine_u8(vqmovun_s16(g_l), vqmovun_s16(g_h));
      rgb.val[RGB_BLUE] = vcombine_u8(vqmovun_s16(b_l), vqmovun_s16(b_h));
      /* Store RGB pixel data to memory. */
      vst3q_u8(outptr, rgb);
#else
      /* Pack R, G, and B values in ratio 5:6:5. */
      uint16x8_t rgb565_l = vqshluq_n_s16(r_l, 8);
      rgb565_l = vsriq_n_u16(rgb565_l, vqshluq_n_s16(g_l, 8), 5);
      rgb565_l = vsriq_n_u16(rgb565_l, vqshluq_n_s16(b_l, 8), 11);
      uint16x8_t rgb565_h = vqshluq_n_s16(r_h, 8);
      rgb565_h = vsriq_n_u16(rgb565_h, vqshluq_n_s16(g_h, 8), 5);
      rgb565_h = vsriq_n_u16(rgb565_h, vqshluq_n_s16(b_h, 8), 11);
      /* Store RGB pixel data to memory. */
      vst1q_u16((uint16_t *)outptr, rgb565_l);
      vst1q_u16(((uint16_t *)outptr) + 8, rgb565_h);
#endif

      /* Increment pointers. */
      inptr0 += 16;
      inptr1 += 16;
      inptr2 += 16;
      outptr += (RGB_PIXELSIZE * 16);
    }

    if (cols_remaining >= 8) {
      uint8x8_t y  = vld1_u8(inptr0);
      uint8x8_t cb = vld1_u8(inptr1);
      uint8x8_t cr = vld1_u8(inptr2);
      /* Subtract 128 from Cb and Cr. */
      int16x8_t cr_128 =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128), cr));
      int16x8_t cb_128 =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128), cb));
      /* Compute G-Y: - 0.34414 * (Cb - 128) - 0.71414 * (Cr - 128) */
      int32x4_t g_sub_y_l = vmull_lane_s16(vget_low_s16(cb_128), consts, 0);
      int32x4_t g_sub_y_h = vmull_lane_s16(vget_high_s16(cb_128), consts, 0);
      g_sub_y_l = vmlsl_lane_s16(g_sub_y_l, vget_low_s16(cr_128), consts, 1);
      g_sub_y_h = vmlsl_lane_s16(g_sub_y_h, vget_high_s16(cr_128), consts, 1);
      /* Descale G components: shift right 15, round, and narrow to 16-bit. */
      int16x8_t g_sub_y = vcombine_s16(vrshrn_n_s32(g_sub_y_l, 15),
                                       vrshrn_n_s32(g_sub_y_h, 15));
      /* Compute R-Y: 1.40200 * (Cr - 128) */
      int16x8_t r_sub_y = vqrdmulhq_lane_s16(vshlq_n_s16(cr_128, 1),
                                             consts, 2);
      /* Compute B-Y: 1.77200 * (Cb - 128) */
      int16x8_t b_sub_y = vqrdmulhq_lane_s16(vshlq_n_s16(cb_128, 1),
                                             consts, 3);
      /* Add Y. */
      int16x8_t r =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(r_sub_y), y));
      int16x8_t b =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(b_sub_y), y));
      int16x8_t g =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(g_sub_y), y));

#if RGB_PIXELSIZE == 4
      uint8x8x4_t rgba;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgba.val[RGB_RED] = vqmovun_s16(r);
      rgba.val[RGB_GREEN] = vqmovun_s16(g);
      rgba.val[RGB_BLUE] = vqmovun_s16(b);
      /* Set alpha channel to opaque (0xFF). */
      rgba.val[RGB_ALPHA] = vdup_n_u8(0xFF);
      /* Store RGBA pixel data to memory. */
      vst4_u8(outptr, rgba);
#elif RGB_PIXELSIZE == 3
      uint8x8x3_t rgb;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgb.val[RGB_RED] = vqmovun_s16(r);
      rgb.val[RGB_GREEN] = vqmovun_s16(g);
      rgb.val[RGB_BLUE] = vqmovun_s16(b);
      /* Store RGB pixel data to memory. */
      vst3_u8(outptr, rgb);
#else
      /* Pack R, G, and B values in ratio 5:6:5. */
      uint16x8_t rgb565 = vqshluq_n_s16(r, 8);
      rgb565 = vsriq_n_u16(rgb565, vqshluq_n_s16(g, 8), 5);
      rgb565 = vsriq_n_u16(rgb565, vqshluq_n_s16(b, 8), 11);
      /* Store RGB pixel data to memory. */
      vst1q_u16((uint16_t *)outptr, rgb565);
#endif

      /* Increment pointers. */
      inptr0 += 8;
      inptr1 += 8;
      inptr2 += 8;
      outptr += (RGB_PIXELSIZE * 8);
      cols_remaining -= 8;
    }

    /* Handle the tail elements. */
    if (cols_remaining > 0) {
      uint8x8_t y  = vld1_u8(inptr0);
      uint8x8_t cb = vld1_u8(inptr1);
      uint8x8_t cr = vld1_u8(inptr2);
      /* Subtract 128 from Cb and Cr. */
      int16x8_t cr_128 =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128), cr));
      int16x8_t cb_128 =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(neg_128), cb));
      /* Compute G-Y: - 0.34414 * (Cb - 128) - 0.71414 * (Cr - 128) */
      int32x4_t g_sub_y_l = vmull_lane_s16(vget_low_s16(cb_128), consts, 0);
      int32x4_t g_sub_y_h = vmull_lane_s16(vget_high_s16(cb_128), consts, 0);
      g_sub_y_l = vmlsl_lane_s16(g_sub_y_l, vget_low_s16(cr_128), consts, 1);
      g_sub_y_h = vmlsl_lane_s16(g_sub_y_h, vget_high_s16(cr_128), consts, 1);
      /* Descale G components: shift right 15, round, and narrow to 16-bit. */
      int16x8_t g_sub_y = vcombine_s16(vrshrn_n_s32(g_sub_y_l, 15),
                                       vrshrn_n_s32(g_sub_y_h, 15));
      /* Compute R-Y: 1.40200 * (Cr - 128) */
      int16x8_t r_sub_y = vqrdmulhq_lane_s16(vshlq_n_s16(cr_128, 1),
                                             consts, 2);
      /* Compute B-Y: 1.77200 * (Cb - 128) */
      int16x8_t b_sub_y = vqrdmulhq_lane_s16(vshlq_n_s16(cb_128, 1),
                                             consts, 3);
      /* Add Y. */
      int16x8_t r =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(r_sub_y), y));
      int16x8_t b =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(b_sub_y), y));
      int16x8_t g =
        vreinterpretq_s16_u16(vaddw_u8(vreinterpretq_u16_s16(g_sub_y), y));

#if RGB_PIXELSIZE == 4
      uint8x8x4_t rgba;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgba.val[RGB_RED] = vqmovun_s16(r);
      rgba.val[RGB_GREEN] = vqmovun_s16(g);
      rgba.val[RGB_BLUE] = vqmovun_s16(b);
      /* Set alpha channel to opaque (0xFF). */
      rgba.val[RGB_ALPHA] = vdup_n_u8(0xFF);
      /* Store RGBA pixel data to memory. */
      switch (cols_remaining) {
      case 7:
        vst4_lane_u8(outptr + 6 * RGB_PIXELSIZE, rgba, 6);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 6:
        vst4_lane_u8(outptr + 5 * RGB_PIXELSIZE, rgba, 5);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 5:
        vst4_lane_u8(outptr + 4 * RGB_PIXELSIZE, rgba, 4);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 4:
        vst4_lane_u8(outptr + 3 * RGB_PIXELSIZE, rgba, 3);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 3:
        vst4_lane_u8(outptr + 2 * RGB_PIXELSIZE, rgba, 2);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 2:
        vst4_lane_u8(outptr + RGB_PIXELSIZE, rgba, 1);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 1:
        vst4_lane_u8(outptr, rgba, 0);
        FALLTHROUGH             /*FALLTHROUGH*/
      default:
        break;
      }
#elif RGB_PIXELSIZE == 3
      uint8x8x3_t rgb;
      /* Convert each component to unsigned and narrow, clamping to [0-255]. */
      rgb.val[RGB_RED] = vqmovun_s16(r);
      rgb.val[RGB_GREEN] = vqmovun_s16(g);
      rgb.val[RGB_BLUE] = vqmovun_s16(b);
      /* Store RGB pixel data to memory. */
      switch (cols_remaining) {
      case 7:
        vst3_lane_u8(outptr + 6 * RGB_PIXELSIZE, rgb, 6);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 6:
        vst3_lane_u8(outptr + 5 * RGB_PIXELSIZE, rgb, 5);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 5:
        vst3_lane_u8(outptr + 4 * RGB_PIXELSIZE, rgb, 4);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 4:
        vst3_lane_u8(outptr + 3 * RGB_PIXELSIZE, rgb, 3);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 3:
        vst3_lane_u8(outptr + 2 * RGB_PIXELSIZE, rgb, 2);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 2:
        vst3_lane_u8(outptr + RGB_PIXELSIZE, rgb, 1);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 1:
        vst3_lane_u8(outptr, rgb, 0);
        FALLTHROUGH             /*FALLTHROUGH*/
      default:
        break;
      }
#else
      /* Pack R, G, and B values in ratio 5:6:5. */
      uint16x8_t rgb565 = vqshluq_n_s16(r, 8);
      rgb565 = vsriq_n_u16(rgb565, vqshluq_n_s16(g, 8), 5);
      rgb565 = vsriq_n_u16(rgb565, vqshluq_n_s16(b, 8), 11);
      /* Store RGB565 pixel data to memory. */
      switch (cols_remaining) {
      case 7:
        vst1q_lane_u16((uint16_t *)(outptr + 6 * RGB_PIXELSIZE), rgb565, 6);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 6:
        vst1q_lane_u16((uint16_t *)(outptr + 5 * RGB_PIXELSIZE), rgb565, 5);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 5:
        vst1q_lane_u16((uint16_t *)(outptr + 4 * RGB_PIXELSIZE), rgb565, 4);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 4:
        vst1q_lane_u16((uint16_t *)(outptr + 3 * RGB_PIXELSIZE), rgb565, 3);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 3:
        vst1q_lane_u16((uint16_t *)(outptr + 2 * RGB_PIXELSIZE), rgb565, 2);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 2:
        vst1q_lane_u16((uint16_t *)(outptr + RGB_PIXELSIZE), rgb565, 1);
        FALLTHROUGH             /*FALLTHROUGH*/
      case 1:
        vst1q_lane_u16((uint16_t *)outptr, rgb565, 0);
        FALLTHROUGH             /*FALLTHROUGH*/
      default:
        break;
      }
#endif
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

**Python Imports:**
- `any`
- `the`
- `Cb`


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

