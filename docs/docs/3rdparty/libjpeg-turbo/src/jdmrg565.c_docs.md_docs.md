# Documentation for `docs/3rdparty/libjpeg-turbo/src/jdmrg565.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/src/jdmrg565.c_docs.md`
- **File Name**: `jdmrg565.c_docs.md`
- **File Size**: 14,027 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/src/jdmrg565.c_docs.md](../../../../docs/3rdparty/libjpeg-turbo/src/jdmrg565.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/src/jdmrg565.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/src/jdmrg565.c`
- **File Name**: `jdmrg565.c`
- **File Size**: 11,138 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/src/jdmrg565.c](../../../3rdparty/libjpeg-turbo/src/jdmrg565.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jdmrg565.c
 *
 * This file was part of the Independent JPEG Group's software:
 * Copyright (C) 1994-1996, Thomas G. Lane.
 * libjpeg-turbo Modifications:
 * Copyright (C) 2013, Linaro Limited.
 * Copyright (C) 2014-2015, 2018, 2020, 2022, D. R. Commander.
 * For conditions of distribution and use, see the accompanying README.ijg
 * file.
 *
 * This file contains code for merged upsampling/color conversion.
 */


INLINE
LOCAL(void)
h2v1_merged_upsample_565_internal(j_decompress_ptr cinfo,
                                  _JSAMPIMAGE input_buf,
                                  JDIMENSION in_row_group_ctr,
                                  _JSAMPARRAY output_buf)
{
  my_merged_upsample_ptr upsample = (my_merged_upsample_ptr)cinfo->upsample;
  register int y, cred, cgreen, cblue;
  int cb, cr;
  register _JSAMPROW outptr;
  _JSAMPROW inptr0, inptr1, inptr2;
  JDIMENSION col;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  int *Crrtab = upsample->Cr_r_tab;
  int *Cbbtab = upsample->Cb_b_tab;
  JLONG *Crgtab = upsample->Cr_g_tab;
  JLONG *Cbgtab = upsample->Cb_g_tab;
  unsigned int r, g, b;
  JLONG rgb;
  SHIFT_TEMPS

  inptr0 = input_buf[0][in_row_group_ctr];
  inptr1 = input_buf[1][in_row_group_ctr];
  inptr2 = input_buf[2][in_row_group_ctr];
  outptr = output_buf[0];

  /* Loop for each pair of output pixels */
  for (col = cinfo->output_width >> 1; col > 0; col--) {
    /* Do the chroma part of the calculation */
    cb = *inptr1++;
    cr = *inptr2++;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    /* Fetch 2 Y values and emit 2 pixels */
    y  = *inptr0++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr0++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr, rgb);
    outptr += 4;
  }

  /* If image width is odd, do the last output column separately */
  if (cinfo->output_width & 1) {
    cb = *inptr1;
    cr = *inptr2;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];
    y  = *inptr0;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr = (INT16)rgb;
  }
}


INLINE
LOCAL(void)
h2v1_merged_upsample_565D_internal(j_decompress_ptr cinfo,
                                   _JSAMPIMAGE input_buf,
                                   JDIMENSION in_row_group_ctr,
                                   _JSAMPARRAY output_buf)
{
  my_merged_upsample_ptr upsample = (my_merged_upsample_ptr)cinfo->upsample;
  register int y, cred, cgreen, cblue;
  int cb, cr;
  register _JSAMPROW outptr;
  _JSAMPROW inptr0, inptr1, inptr2;
  JDIMENSION col;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  int *Crrtab = upsample->Cr_r_tab;
  int *Cbbtab = upsample->Cb_b_tab;
  JLONG *Crgtab = upsample->Cr_g_tab;
  JLONG *Cbgtab = upsample->Cb_g_tab;
  JLONG d0 = dither_matrix[cinfo->output_scanline & DITHER_MASK];
  unsigned int r, g, b;
  JLONG rgb;
  SHIFT_TEMPS

  inptr0 = input_buf[0][in_row_group_ctr];
  inptr1 = input_buf[1][in_row_group_ctr];
  inptr2 = input_buf[2][in_row_group_ctr];
  outptr = output_buf[0];

  /* Loop for each pair of output pixels */
  for (col = cinfo->output_width >> 1; col > 0; col--) {
    /* Do the chroma part of the calculation */
    cb = *inptr1++;
    cr = *inptr2++;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    /* Fetch 2 Y values and emit 2 pixels */
    y  = *inptr0++;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    d0 = DITHER_ROTATE(d0);
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr0++;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    d0 = DITHER_ROTATE(d0);
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr, rgb);
    outptr += 4;
  }

  /* If image width is odd, do the last output column separately */
  if (cinfo->output_width & 1) {
    cb = *inptr1;
    cr = *inptr2;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];
    y  = *inptr0;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr = (INT16)rgb;
  }
}


INLINE
LOCAL(void)
h2v2_merged_upsample_565_internal(j_decompress_ptr cinfo, _JSAMPIMAGE input_buf,
                                  JDIMENSION in_row_group_ctr,
                                  _JSAMPARRAY output_buf)
{
  my_merged_upsample_ptr upsample = (my_merged_upsample_ptr)cinfo->upsample;
  register int y, cred, cgreen, cblue;
  int cb, cr;
  register _JSAMPROW outptr0, outptr1;
  _JSAMPROW inptr00, inptr01, inptr1, inptr2;
  JDIMENSION col;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  int *Crrtab = upsample->Cr_r_tab;
  int *Cbbtab = upsample->Cb_b_tab;
  JLONG *Crgtab = upsample->Cr_g_tab;
  JLONG *Cbgtab = upsample->Cb_g_tab;
  unsigned int r, g, b;
  JLONG rgb;
  SHIFT_TEMPS

  inptr00 = input_buf[0][in_row_group_ctr * 2];
  inptr01 = input_buf[0][in_row_group_ctr * 2 + 1];
  inptr1 = input_buf[1][in_row_group_ctr];
  inptr2 = input_buf[2][in_row_group_ctr];
  outptr0 = output_buf[0];
  outptr1 = output_buf[1];

  /* Loop for each group of output pixels */
  for (col = cinfo->output_width >> 1; col > 0; col--) {
    /* Do the chroma part of the calculation */
    cb = *inptr1++;
    cr = *inptr2++;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    /* Fetch 4 Y values and emit 4 pixels */
    y  = *inptr00++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr00++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr0, rgb);
    outptr0 += 4;

    y  = *inptr01++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr01++;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr1, rgb);
    outptr1 += 4;
  }

  /* If image width is odd, do the last output column separately */
  if (cinfo->output_width & 1) {
    cb = *inptr1;
    cr = *inptr2;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    y  = *inptr00;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr0 = (INT16)rgb;

    y  = *inptr01;
    r = range_limit[y + cred];
    g = range_limit[y + cgreen];
    b = range_limit[y + cblue];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr1 = (INT16)rgb;
  }
}


INLINE
LOCAL(void)
h2v2_merged_upsample_565D_internal(j_decompress_ptr cinfo,
                                   _JSAMPIMAGE input_buf,
                                   JDIMENSION in_row_group_ctr,
                                   _JSAMPARRAY output_buf)
{
  my_merged_upsample_ptr upsample = (my_merged_upsample_ptr)cinfo->upsample;
  register int y, cred, cgreen, cblue;
  int cb, cr;
  register _JSAMPROW outptr0, outptr1;
  _JSAMPROW inptr00, inptr01, inptr1, inptr2;
  JDIMENSION col;
  /* copy these pointers into registers if possible */
  register _JSAMPLE *range_limit = (_JSAMPLE *)cinfo->sample_range_limit;
  int *Crrtab = upsample->Cr_r_tab;
  int *Cbbtab = upsample->Cb_b_tab;
  JLONG *Crgtab = upsample->Cr_g_tab;
  JLONG *Cbgtab = upsample->Cb_g_tab;
  JLONG d0 = dither_matrix[cinfo->output_scanline & DITHER_MASK];
  JLONG d1 = dither_matrix[(cinfo->output_scanline + 1) & DITHER_MASK];
  unsigned int r, g, b;
  JLONG rgb;
  SHIFT_TEMPS

  inptr00 = input_buf[0][in_row_group_ctr * 2];
  inptr01 = input_buf[0][in_row_group_ctr * 2 + 1];
  inptr1 = input_buf[1][in_row_group_ctr];
  inptr2 = input_buf[2][in_row_group_ctr];
  outptr0 = output_buf[0];
  outptr1 = output_buf[1];

  /* Loop for each group of output pixels */
  for (col = cinfo->output_width >> 1; col > 0; col--) {
    /* Do the chroma part of the calculation */
    cb = *inptr1++;
    cr = *inptr2++;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    /* Fetch 4 Y values and emit 4 pixels */
    y  = *inptr00++;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    d0 = DITHER_ROTATE(d0);
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr00++;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    d0 = DITHER_ROTATE(d0);
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr0, rgb);
    outptr0 += 4;

    y  = *inptr01++;
    r = range_limit[DITHER_565_R(y + cred, d1)];
    g = range_limit[DITHER_565_G(y + cgreen, d1)];
    b = range_limit[DITHER_565_B(y + cblue, d1)];
    d1 = DITHER_ROTATE(d1);
    rgb = PACK_SHORT_565(r, g, b);

    y  = *inptr01++;
    r = range_limit[DITHER_565_R(y + cred, d1)];
    g = range_limit[DITHER_565_G(y + cgreen, d1)];
    b = range_limit[DITHER_565_B(y + cblue, d1)];
    d1 = DITHER_ROTATE(d1);
    rgb = PACK_TWO_PIXELS(rgb, PACK_SHORT_565(r, g, b));

    WRITE_TWO_PIXELS(outptr1, rgb);
    outptr1 += 4;
  }

  /* If image width is odd, do the last output column separately */
  if (cinfo->output_width & 1) {
    cb = *inptr1;
    cr = *inptr2;
    cred = Crrtab[cr];
    cgreen = (int)RIGHT_SHIFT(Cbgtab[cb] + Crgtab[cr], SCALEBITS);
    cblue = Cbbtab[cb];

    y  = *inptr00;
    r = range_limit[DITHER_565_R(y + cred, d0)];
    g = range_limit[DITHER_565_G(y + cgreen, d0)];
    b = range_limit[DITHER_565_B(y + cblue, d0)];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr0 = (INT16)rgb;

    y  = *inptr01;
    r = range_limit[DITHER_565_R(y + cred, d1)];
    g = range_limit[DITHER_565_G(y + cgreen, d1)];
    b = range_limit[DITHER_565_B(y + cblue, d1)];
    rgb = PACK_SHORT_565(r, g, b);
    *(INT16 *)outptr1 = (INT16)rgb;
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

