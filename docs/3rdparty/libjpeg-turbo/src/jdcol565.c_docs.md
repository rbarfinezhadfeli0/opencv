# Documentation for `3rdparty/libjpeg-turbo/src/jdcol565.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/src/jdcol565.c`
- **File Name**: `jdcol565.c`
- **File Size**: 11,909 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/src/jdcol565.c](../../../3rdparty/libjpeg-turbo/src/jdcol565.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jdcol565.c
 *
 * This file was part of the Independent JPEG Group's software:
 * Copyright (C) 1991-1997, Thomas G. Lane.
 * Modifications:
 * Copyright (C) 2013, Linaro Limited.
 * Copyright (C) 2014-2015, 2022, D. R. Commander.
 * For conditions of distribution and use, see the accompanying README.ijg
 * file.
 *
 * This file contains output colorspace conversion routines.
 */

/* This file is included by jdcolor.c */


INLINE
LOCAL(void)
ycc_rgb565_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                            JDIMENSION input_row, _JSAMPARRAY output_buf,
                            int num_rows)
{
#if BITS_IN_JSAMPLE != 16
  my_cconvert_ptr cconvert = (my_cconvert_ptr)cinfo->cconvert;
  register int y, cb, cr;
  register _JSAMPROW outptr;
  register _JSAMPROW inptr0, inptr1, inptr2;
  register JDIMENSION col;
  JDIMENSION num_cols = cinfo->output_width;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  register int *Crrtab = cconvert->Cr_r_tab;
  register int *Cbbtab = cconvert->Cb_b_tab;
  register JLONG *Crgtab = cconvert->Cr_g_tab;
  register JLONG *Cbgtab = cconvert->Cb_g_tab;
  SHIFT_TEMPS

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int r, g, b;
    inptr0 = input_buf[0][input_row];
    inptr1 = input_buf[1][input_row];
    inptr2 = input_buf[2][input_row];
    input_row++;
    outptr = *output_buf++;

    if (PACK_NEED_ALIGNMENT(outptr)) {
      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[y + Crrtab[cr]];
      g = range_limit[y + ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                            SCALEBITS))];
      b = range_limit[y + Cbbtab[cb]];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[y + Crrtab[cr]];
      g = range_limit[y + ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                            SCALEBITS))];
      b = range_limit[y + Cbbtab[cb]];
      rgb = PACK_SHORT_565(r, g, b);

      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[y + Crrtab[cr]];
      g = range_limit[y + ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                            SCALEBITS))];
      b = range_limit[y + Cbbtab[cb]];
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      y  = *inptr0;
      cb = *inptr1;
      cr = *inptr2;
      r = range_limit[y + Crrtab[cr]];
      g = range_limit[y + ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                            SCALEBITS))];
      b = range_limit[y + Cbbtab[cb]];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
    }
  }
#else
  ERREXIT(cinfo, JERR_CONVERSION_NOTIMPL);
#endif
}


INLINE
LOCAL(void)
ycc_rgb565D_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                             JDIMENSION input_row, _JSAMPARRAY output_buf,
                             int num_rows)
{
#if BITS_IN_JSAMPLE != 16
  my_cconvert_ptr cconvert = (my_cconvert_ptr)cinfo->cconvert;
  register int y, cb, cr;
  register _JSAMPROW outptr;
  register _JSAMPROW inptr0, inptr1, inptr2;
  register JDIMENSION col;
  JDIMENSION num_cols = cinfo->output_width;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  register int *Crrtab = cconvert->Cr_r_tab;
  register int *Cbbtab = cconvert->Cb_b_tab;
  register JLONG *Crgtab = cconvert->Cr_g_tab;
  register JLONG *Cbgtab = cconvert->Cb_g_tab;
  JLONG d0 = dither_matrix[cinfo->output_scanline & DITHER_MASK];
  SHIFT_TEMPS

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int r, g, b;

    inptr0 = input_buf[0][input_row];
    inptr1 = input_buf[1][input_row];
    inptr2 = input_buf[2][input_row];
    input_row++;
    outptr = *output_buf++;
    if (PACK_NEED_ALIGNMENT(outptr)) {
      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[DITHER_565_R(y + Crrtab[cr], d0)];
      g = range_limit[DITHER_565_G(y +
                                   ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                                     SCALEBITS)), d0)];
      b = range_limit[DITHER_565_B(y + Cbbtab[cb], d0)];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[DITHER_565_R(y + Crrtab[cr], d0)];
      g = range_limit[DITHER_565_G(y +
                                   ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                                     SCALEBITS)), d0)];
      b = range_limit[DITHER_565_B(y + Cbbtab[cb], d0)];
      d0 = DITHER_ROTATE(d0);
      rgb = PACK_SHORT_565(r, g, b);

      y  = *inptr0++;
      cb = *inptr1++;
      cr = *inptr2++;
      r = range_limit[DITHER_565_R(y + Crrtab[cr], d0)];
      g = range_limit[DITHER_565_G(y +
                                   ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                                     SCALEBITS)), d0)];
      b = range_limit[DITHER_565_B(y + Cbbtab[cb], d0)];
      d0 = DITHER_ROTATE(d0);
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      y  = *inptr0;
      cb = *inptr1;
      cr = *inptr2;
      r = range_limit[DITHER_565_R(y + Crrtab[cr], d0)];
      g = range_limit[DITHER_565_G(y +
                                   ((int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr],
                                                     SCALEBITS)), d0)];
      b = range_limit[DITHER_565_B(y + Cbbtab[cb], d0)];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
    }
  }
#else
  ERREXIT(cinfo, JERR_CONVERSION_NOTIMPL);
#endif
}


INLINE
LOCAL(void)
rgb_rgb565_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                            JDIMENSION input_row, _JSAMPARRAY output_buf,
                            int num_rows)
{
  register _JSAMPROW outptr;
  register _JSAMPROW inptr0, inptr1, inptr2;
  register JDIMENSION col;
  JDIMENSION num_cols = cinfo->output_width;
  SHIFT_TEMPS

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int r, g, b;

    inptr0 = input_buf[0][input_row];
    inptr1 = input_buf[1][input_row];
    inptr2 = input_buf[2][input_row];
    input_row++;
    outptr = *output_buf++;
    if (PACK_NEED_ALIGNMENT(outptr)) {
      r = *inptr0++;
      g = *inptr1++;
      b = *inptr2++;
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      r = *inptr0++;
      g = *inptr1++;
      b = *inptr2++;
      rgb = PACK_SHORT_565(r, g, b);

      r = *inptr0++;
      g = *inptr1++;
      b = *inptr2++;
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      r = *inptr0;
      g = *inptr1;
      b = *inptr2;
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
    }
  }
}


INLINE
LOCAL(void)
rgb_rgb565D_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                             JDIMENSION input_row, _JSAMPARRAY output_buf,
                             int num_rows)
{
  register _JSAMPROW outptr;
  register _JSAMPROW inptr0, inptr1, inptr2;
  register JDIMENSION col;
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  JDIMENSION num_cols = cinfo->output_width;
  JLONG d0 = dither_matrix[cinfo->output_scanline & DITHER_MASK];
  SHIFT_TEMPS

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int r, g, b;

    inptr0 = input_buf[0][input_row];
    inptr1 = input_buf[1][input_row];
    inptr2 = input_buf[2][input_row];
    input_row++;
    outptr = *output_buf++;
    if (PACK_NEED_ALIGNMENT(outptr)) {
      r = range_limit[DITHER_565_R(*inptr0++, d0)];
      g = range_limit[DITHER_565_G(*inptr1++, d0)];
      b = range_limit[DITHER_565_B(*inptr2++, d0)];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      r = range_limit[DITHER_565_R(*inptr0++, d0)];
      g = range_limit[DITHER_565_G(*inptr1++, d0)];
      b = range_limit[DITHER_565_B(*inptr2++, d0)];
      d0 = DITHER_ROTATE(d0);
      rgb = PACK_SHORT_565(r, g, b);

      r = range_limit[DITHER_565_R(*inptr0++, d0)];
      g = range_limit[DITHER_565_G(*inptr1++, d0)];
      b = range_limit[DITHER_565_B(*inptr2++, d0)];
      d0 = DITHER_ROTATE(d0);
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      r = range_limit[DITHER_565_R(*inptr0, d0)];
      g = range_limit[DITHER_565_G(*inptr1, d0)];
      b = range_limit[DITHER_565_B(*inptr2, d0)];
      rgb = PACK_SHORT_565(r, g, b);
      *(INT16 *)outptr = (INT16)rgb;
    }
  }
}


INLINE
LOCAL(void)
gray_rgb565_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                             JDIMENSION input_row, _JSAMPARRAY output_buf,
                             int num_rows)
{
  register _JSAMPROW inptr, outptr;
  register JDIMENSION col;
  JDIMENSION num_cols = cinfo->output_width;

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int g;

    inptr = input_buf[0][input_row++];
    outptr = *output_buf++;
    if (PACK_NEED_ALIGNMENT(outptr)) {
      g = *inptr++;
      rgb = PACK_SHORT_565(g, g, g);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      g = *inptr++;
      rgb = PACK_SHORT_565(g, g, g);
      g = *inptr++;
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(g, g, g));
      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      g = *inptr;
      rgb = PACK_SHORT_565(g, g, g);
      *(INT16 *)outptr = (INT16)rgb;
    }
  }
}


INLINE
LOCAL(void)
gray_rgb565D_convert_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                              JDIMENSION input_row, _JSAMPARRAY output_buf,
                              int num_rows)
{
  register _JSAMPROW inptr, outptr;
  register JDIMENSION col;
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  JDIMENSION num_cols = cinfo->output_width;
  JLONG d0 = dither_matrix[cinfo->output_scanline & DITHER_MASK];

  while (--num_rows >= 0) {
    JLONG rgb;
    unsigned int g;

    inptr = input_buf[0][input_row++];
    outptr = *output_buf++;
    if (PACK_NEED_ALIGNMENT(outptr)) {
      g = *inptr++;
      g = range_limit[DITHER_565_R(g, d0)];
      rgb = PACK_SHORT_565(g, g, g);
      *(INT16 *)outptr = (INT16)rgb;
      outptr += 2;
      num_cols--;
    }
    for (col = 0; col < (num_cols >> 1); col++) {
      g = *inptr++;
      g = range_limit[DITHER_565_R(g, d0)];
      rgb = PACK_SHORT_565(g, g, g);
      d0 = DITHER_ROTATE(d0);

      g = *inptr++;
      g = range_limit[DITHER_565_R(g, d0)];
      rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(g, g, g));
      d0 = DITHER_ROTATE(d0);

      WRITE_TWO_ALIGNED_PIXELS(outptr, rgb);
      outptr += 4;
    }
    if (num_cols & 1) {
      g = *inptr;
      g = range_limit[DITHER_565_R(g, d0)];
      rgb = PACK_SHORT_565(g, g, g);
      *(INT16 *)outptr = (INT16)rgb;
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

