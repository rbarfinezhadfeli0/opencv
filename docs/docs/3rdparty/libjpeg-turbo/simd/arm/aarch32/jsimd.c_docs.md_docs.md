# Documentation for `docs/3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c_docs.md`
- **File Name**: `jsimd.c_docs.md`
- **File Size**: 25,216 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c_docs.md](../../../../../../docs/3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/arm/aarch32` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c`
- **File Name**: `jsimd.c`
- **File Size**: 21,924 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c](../../../../../3rdparty/libjpeg-turbo/simd/arm/aarch32/jsimd.c)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/arm/aarch32` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jsimd_arm.c
 *
 * Copyright 2009 Pierre Ossman <ossman@cendio.se> for Cendio AB
 * Copyright (C) 2011, Nokia Corporation and/or its subsidiary(-ies).
 * Copyright (C) 2009-2011, 2013-2014, 2016, 2018, 2022, 2024, D. R. Commander.
 * Copyright (C) 2015-2016, 2018, 2022, Matthieu Darbois.
 * Copyright (C) 2019, Google LLC.
 * Copyright (C) 2020, Arm Limited.
 *
 * Based on the x86 SIMD extension for IJG JPEG library,
 * Copyright (C) 1999-2006, MIYASAKA Masaru.
 * For conditions of distribution and use, see copyright notice in jsimdext.inc
 *
 * This file contains the interface between the "normal" portions
 * of the library and the SIMD implementations when running on a
 * 32-bit Arm architecture.
 */

#define JPEG_INTERNALS
#include "../../../src/jinclude.h"
#include "../../../src/jpeglib.h"
#include "../../../src/jsimd.h"
#include "../../../src/jdct.h"
#include "../../../src/jsimddct.h"
#include "../../jsimd.h"

#include <ctype.h>

static THREAD_LOCAL unsigned int simd_support = ~0;
static THREAD_LOCAL unsigned int simd_huffman = 1;

#if !defined(__ARM_NEON__) && (defined(__linux__) || defined(ANDROID) || defined(__ANDROID__))

#define SOMEWHAT_SANE_PROC_CPUINFO_SIZE_LIMIT  (1024 * 1024)

LOCAL(int)
check_feature(char *buffer, char *feature)
{
  char *p;

  if (*feature == 0)
    return 0;
  if (strncmp(buffer, "Features", 8) != 0)
    return 0;
  buffer += 8;
  while (isspace(*buffer))
    buffer++;

  /* Check if 'feature' is present in the buffer as a separate word */
  while ((p = strstr(buffer, feature))) {
    if (p > buffer && !isspace(*(p - 1))) {
      buffer++;
      continue;
    }
    p += strlen(feature);
    if (*p != 0 && !isspace(*p)) {
      buffer++;
      continue;
    }
    return 1;
  }
  return 0;
}

LOCAL(int)
parse_proc_cpuinfo(int bufsize)
{
  char *buffer = (char *)malloc(bufsize);
  FILE *fd;

  simd_support = 0;

  if (!buffer)
    return 0;

  fd = fopen("/proc/cpuinfo", "r");
  if (fd) {
    while (fgets(buffer, bufsize, fd)) {
      if (!strchr(buffer, '\n') && !feof(fd)) {
        /* "impossible" happened - insufficient size of the buffer! */
        fclose(fd);
        free(buffer);
        return 0;
      }
      if (check_feature(buffer, "neon"))
        simd_support |= JSIMD_NEON;
    }
    fclose(fd);
  }
  free(buffer);
  return 1;
}

#endif

/*
 * Check what SIMD accelerations are supported.
 */
LOCAL(void)
init_simd(void)
{
#ifndef NO_GETENV
  char env[2] = { 0 };
#endif
#if !defined(__ARM_NEON__) && (defined(__linux__) || defined(ANDROID) || defined(__ANDROID__))
  int bufsize = 1024; /* an initial guess for the line buffer size limit */
#endif

  if (simd_support != ~0U)
    return;

  simd_support = 0;

#if defined(__ARM_NEON__)
  simd_support |= JSIMD_NEON;
#elif defined(__linux__) || defined(ANDROID) || defined(__ANDROID__)
  /* We still have a chance to use Neon regardless of globally used
   * -mcpu/-mfpu options passed to gcc by performing runtime detection via
   * /proc/cpuinfo parsing on linux/android */
  while (!parse_proc_cpuinfo(bufsize)) {
    bufsize *= 2;
    if (bufsize > SOMEWHAT_SANE_PROC_CPUINFO_SIZE_LIMIT)
      break;
  }
#endif

#ifndef NO_GETENV
  /* Force different settings through environment variables */
  if (!GETENV_S(env, 2, "JSIMD_FORCENEON") && !strcmp(env, "1"))
    simd_support = JSIMD_NEON;
  if (!GETENV_S(env, 2, "JSIMD_FORCENONE") && !strcmp(env, "1"))
    simd_support = 0;
  if (!GETENV_S(env, 2, "JSIMD_NOHUFFENC") && !strcmp(env, "1"))
    simd_huffman = 0;
#endif
}

GLOBAL(int)
jsimd_can_rgb_ycc(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if ((RGB_PIXELSIZE != 3) && (RGB_PIXELSIZE != 4))
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_rgb_gray(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if ((RGB_PIXELSIZE != 3) && (RGB_PIXELSIZE != 4))
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_ycc_rgb(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if ((RGB_PIXELSIZE != 3) && (RGB_PIXELSIZE != 4))
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_ycc_rgb565(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_rgb_ycc_convert(j_compress_ptr cinfo, JSAMPARRAY input_buf,
                      JSAMPIMAGE output_buf, JDIMENSION output_row,
                      int num_rows)
{
  void (*neonfct) (JDIMENSION, JSAMPARRAY, JSAMPIMAGE, JDIMENSION, int);

  switch (cinfo->in_color_space) {
  case JCS_EXT_RGB:
    neonfct = jsimd_extrgb_ycc_convert_neon;
    break;
  case JCS_EXT_RGBX:
  case JCS_EXT_RGBA:
    neonfct = jsimd_extrgbx_ycc_convert_neon;
    break;
  case JCS_EXT_BGR:
    neonfct = jsimd_extbgr_ycc_convert_neon;
    break;
  case JCS_EXT_BGRX:
  case JCS_EXT_BGRA:
    neonfct = jsimd_extbgrx_ycc_convert_neon;
    break;
  case JCS_EXT_XBGR:
  case JCS_EXT_ABGR:
    neonfct = jsimd_extxbgr_ycc_convert_neon;
    break;
  case JCS_EXT_XRGB:
  case JCS_EXT_ARGB:
    neonfct = jsimd_extxrgb_ycc_convert_neon;
    break;
  default:
    neonfct = jsimd_extrgb_ycc_convert_neon;
    break;
  }

  neonfct(cinfo->image_width, input_buf, output_buf, output_row, num_rows);
}

GLOBAL(void)
jsimd_rgb_gray_convert(j_compress_ptr cinfo, JSAMPARRAY input_buf,
                       JSAMPIMAGE output_buf, JDIMENSION output_row,
                       int num_rows)
{
  void (*neonfct) (JDIMENSION, JSAMPARRAY, JSAMPIMAGE, JDIMENSION, int);

  switch (cinfo->in_color_space) {
  case JCS_EXT_RGB:
    neonfct = jsimd_extrgb_gray_convert_neon;
    break;
  case JCS_EXT_RGBX:
  case JCS_EXT_RGBA:
    neonfct = jsimd_extrgbx_gray_convert_neon;
    break;
  case JCS_EXT_BGR:
    neonfct = jsimd_extbgr_gray_convert_neon;
    break;
  case JCS_EXT_BGRX:
  case JCS_EXT_BGRA:
    neonfct = jsimd_extbgrx_gray_convert_neon;
    break;
  case JCS_EXT_XBGR:
  case JCS_EXT_ABGR:
    neonfct = jsimd_extxbgr_gray_convert_neon;
    break;
  case JCS_EXT_XRGB:
  case JCS_EXT_ARGB:
    neonfct = jsimd_extxrgb_gray_convert_neon;
    break;
  default:
    neonfct = jsimd_extrgb_gray_convert_neon;
    break;
  }

  neonfct(cinfo->image_width, input_buf, output_buf, output_row, num_rows);
}

GLOBAL(void)
jsimd_ycc_rgb_convert(j_decompress_ptr cinfo, JSAMPIMAGE input_buf,
                      JDIMENSION input_row, JSAMPARRAY output_buf,
                      int num_rows)
{
  void (*neonfct) (JDIMENSION, JSAMPIMAGE, JDIMENSION, JSAMPARRAY, int);

  switch (cinfo->out_color_space) {
  case JCS_EXT_RGB:
    neonfct = jsimd_ycc_extrgb_convert_neon;
    break;
  case JCS_EXT_RGBX:
  case JCS_EXT_RGBA:
    neonfct = jsimd_ycc_extrgbx_convert_neon;
    break;
  case JCS_EXT_BGR:
    neonfct = jsimd_ycc_extbgr_convert_neon;
    break;
  case JCS_EXT_BGRX:
  case JCS_EXT_BGRA:
    neonfct = jsimd_ycc_extbgrx_convert_neon;
    break;
  case JCS_EXT_XBGR:
  case JCS_EXT_ABGR:
    neonfct = jsimd_ycc_extxbgr_convert_neon;
    break;
  case JCS_EXT_XRGB:
  case JCS_EXT_ARGB:
    neonfct = jsimd_ycc_extxrgb_convert_neon;
    break;
  default:
    neonfct = jsimd_ycc_extrgb_convert_neon;
    break;
  }

  neonfct(cinfo->output_width, input_buf, input_row, output_buf, num_rows);
}

GLOBAL(void)
jsimd_ycc_rgb565_convert(j_decompress_ptr cinfo, JSAMPIMAGE input_buf,
                         JDIMENSION input_row, JSAMPARRAY output_buf,
                         int num_rows)
{
  jsimd_ycc_rgb565_convert_neon(cinfo->output_width, input_buf, input_row,
                                output_buf, num_rows);
}

GLOBAL(int)
jsimd_can_h2v2_downsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_h2v1_downsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_h2v2_downsample(j_compress_ptr cinfo, jpeg_component_info *compptr,
                      JSAMPARRAY input_data, JSAMPARRAY output_data)
{
  jsimd_h2v2_downsample_neon(cinfo->image_width, cinfo->max_v_samp_factor,
                             compptr->v_samp_factor, compptr->width_in_blocks,
                             input_data, output_data);
}

GLOBAL(void)
jsimd_h2v1_downsample(j_compress_ptr cinfo, jpeg_component_info *compptr,
                      JSAMPARRAY input_data, JSAMPARRAY output_data)
{
  jsimd_h2v1_downsample_neon(cinfo->image_width, cinfo->max_v_samp_factor,
                             compptr->v_samp_factor, compptr->width_in_blocks,
                             input_data, output_data);
}

GLOBAL(int)
jsimd_can_h2v2_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_h2v1_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_h2v2_upsample(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                    JSAMPARRAY input_data, JSAMPARRAY *output_data_ptr)
{
  jsimd_h2v2_upsample_neon(cinfo->max_v_samp_factor, cinfo->output_width,
                           input_data, output_data_ptr);
}

GLOBAL(void)
jsimd_h2v1_upsample(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                    JSAMPARRAY input_data, JSAMPARRAY *output_data_ptr)
{
  jsimd_h2v1_upsample_neon(cinfo->max_v_samp_factor, cinfo->output_width,
                           input_data, output_data_ptr);
}

GLOBAL(int)
jsimd_can_h2v2_fancy_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_h2v1_fancy_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_h1v2_fancy_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_h2v2_fancy_upsample(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                          JSAMPARRAY input_data, JSAMPARRAY *output_data_ptr)
{
  jsimd_h2v2_fancy_upsample_neon(cinfo->max_v_samp_factor,
                                 compptr->downsampled_width, input_data,
                                 output_data_ptr);
}

GLOBAL(void)
jsimd_h2v1_fancy_upsample(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                          JSAMPARRAY input_data, JSAMPARRAY *output_data_ptr)
{
  jsimd_h2v1_fancy_upsample_neon(cinfo->max_v_samp_factor,
                                 compptr->downsampled_width, input_data,
                                 output_data_ptr);
}

GLOBAL(void)
jsimd_h1v2_fancy_upsample(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                          JSAMPARRAY input_data, JSAMPARRAY *output_data_ptr)
{
  jsimd_h1v2_fancy_upsample_neon(cinfo->max_v_samp_factor,
                                 compptr->downsampled_width, input_data,
                                 output_data_ptr);
}

GLOBAL(int)
jsimd_can_h2v2_merged_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_h2v1_merged_upsample(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_h2v2_merged_upsample(j_decompress_ptr cinfo, JSAMPIMAGE input_buf,
                           JDIMENSION in_row_group_ctr, JSAMPARRAY output_buf)
{
  void (*neonfct) (JDIMENSION, JSAMPIMAGE, JDIMENSION, JSAMPARRAY);

  switch (cinfo->out_color_space) {
    case JCS_EXT_RGB:
      neonfct = jsimd_h2v2_extrgb_merged_upsample_neon;
      break;
    case JCS_EXT_RGBX:
    case JCS_EXT_RGBA:
      neonfct = jsimd_h2v2_extrgbx_merged_upsample_neon;
      break;
    case JCS_EXT_BGR:
      neonfct = jsimd_h2v2_extbgr_merged_upsample_neon;
      break;
    case JCS_EXT_BGRX:
    case JCS_EXT_BGRA:
      neonfct = jsimd_h2v2_extbgrx_merged_upsample_neon;
      break;
    case JCS_EXT_XBGR:
    case JCS_EXT_ABGR:
      neonfct = jsimd_h2v2_extxbgr_merged_upsample_neon;
      break;
    case JCS_EXT_XRGB:
    case JCS_EXT_ARGB:
      neonfct = jsimd_h2v2_extxrgb_merged_upsample_neon;
      break;
    default:
      neonfct = jsimd_h2v2_extrgb_merged_upsample_neon;
      break;
  }

  neonfct(cinfo->output_width, input_buf, in_row_group_ctr, output_buf);
}

GLOBAL(void)
jsimd_h2v1_merged_upsample(j_decompress_ptr cinfo, JSAMPIMAGE input_buf,
                           JDIMENSION in_row_group_ctr, JSAMPARRAY output_buf)
{
  void (*neonfct) (JDIMENSION, JSAMPIMAGE, JDIMENSION, JSAMPARRAY);

  switch (cinfo->out_color_space) {
    case JCS_EXT_RGB:
      neonfct = jsimd_h2v1_extrgb_merged_upsample_neon;
      break;
    case JCS_EXT_RGBX:
    case JCS_EXT_RGBA:
      neonfct = jsimd_h2v1_extrgbx_merged_upsample_neon;
      break;
    case JCS_EXT_BGR:
      neonfct = jsimd_h2v1_extbgr_merged_upsample_neon;
      break;
    case JCS_EXT_BGRX:
    case JCS_EXT_BGRA:
      neonfct = jsimd_h2v1_extbgrx_merged_upsample_neon;
      break;
    case JCS_EXT_XBGR:
    case JCS_EXT_ABGR:
      neonfct = jsimd_h2v1_extxbgr_merged_upsample_neon;
      break;
    case JCS_EXT_XRGB:
    case JCS_EXT_ARGB:
      neonfct = jsimd_h2v1_extxrgb_merged_upsample_neon;
      break;
    default:
      neonfct = jsimd_h2v1_extrgb_merged_upsample_neon;
      break;
  }

  neonfct(cinfo->output_width, input_buf, in_row_group_ctr, output_buf);
}

GLOBAL(int)
jsimd_can_convsamp(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (sizeof(DCTELEM) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_convsamp_float(void)
{
  return 0;
}

GLOBAL(void)
jsimd_convsamp(JSAMPARRAY sample_data, JDIMENSION start_col,
               DCTELEM *workspace)
{
  jsimd_convsamp_neon(sample_data, start_col, workspace);
}

GLOBAL(void)
jsimd_convsamp_float(JSAMPARRAY sample_data, JDIMENSION start_col,
                     FAST_FLOAT *workspace)
{
}

GLOBAL(int)
jsimd_can_fdct_islow(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(DCTELEM) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_fdct_ifast(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(DCTELEM) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_fdct_float(void)
{
  return 0;
}

GLOBAL(void)
jsimd_fdct_islow(DCTELEM *data)
{
  jsimd_fdct_islow_neon(data);
}

GLOBAL(void)
jsimd_fdct_ifast(DCTELEM *data)
{
  jsimd_fdct_ifast_neon(data);
}

GLOBAL(void)
jsimd_fdct_float(FAST_FLOAT *data)
{
}

GLOBAL(int)
jsimd_can_quantize(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;
  if (sizeof(DCTELEM) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_quantize_float(void)
{
  return 0;
}

GLOBAL(void)
jsimd_quantize(JCOEFPTR coef_block, DCTELEM *divisors, DCTELEM *workspace)
{
  jsimd_quantize_neon(coef_block, divisors, workspace);
}

GLOBAL(void)
jsimd_quantize_float(JCOEFPTR coef_block, FAST_FLOAT *divisors,
                     FAST_FLOAT *workspace)
{
}

GLOBAL(int)
jsimd_can_idct_2x2(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (sizeof(ISLOW_MULT_TYPE) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_idct_4x4(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (sizeof(ISLOW_MULT_TYPE) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_idct_2x2(j_decompress_ptr cinfo, jpeg_component_info *compptr,
               JCOEFPTR coef_block, JSAMPARRAY output_buf,
               JDIMENSION output_col)
{
  jsimd_idct_2x2_neon(compptr->dct_table, coef_block, output_buf, output_col);
}

GLOBAL(void)
jsimd_idct_4x4(j_decompress_ptr cinfo, jpeg_component_info *compptr,
               JCOEFPTR coef_block, JSAMPARRAY output_buf,
               JDIMENSION output_col)
{
  jsimd_idct_4x4_neon(compptr->dct_table, coef_block, output_buf, output_col);
}

GLOBAL(int)
jsimd_can_idct_islow(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (sizeof(ISLOW_MULT_TYPE) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_idct_ifast(void)
{
  init_simd();

  /* The code is optimised for these values only */
  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;
  if (BITS_IN_JSAMPLE != 8)
    return 0;
  if (sizeof(JDIMENSION) != 4)
    return 0;
  if (sizeof(IFAST_MULT_TYPE) != 2)
    return 0;
  if (IFAST_SCALE_BITS != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_can_idct_float(void)
{
  return 0;
}

GLOBAL(void)
jsimd_idct_islow(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                 JCOEFPTR coef_block, JSAMPARRAY output_buf,
                 JDIMENSION output_col)
{
  jsimd_idct_islow_neon(compptr->dct_table, coef_block, output_buf,
                        output_col);
}

GLOBAL(void)
jsimd_idct_ifast(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                 JCOEFPTR coef_block, JSAMPARRAY output_buf,
                 JDIMENSION output_col)
{
  jsimd_idct_ifast_neon(compptr->dct_table, coef_block, output_buf,
                        output_col);
}

GLOBAL(void)
jsimd_idct_float(j_decompress_ptr cinfo, jpeg_component_info *compptr,
                 JCOEFPTR coef_block, JSAMPARRAY output_buf,
                 JDIMENSION output_col)
{
}

GLOBAL(int)
jsimd_can_huff_encode_one_block(void)
{
  init_simd();

  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;

  if (simd_support & JSIMD_NEON && simd_huffman)
    return 1;

  return 0;
}

GLOBAL(JOCTET *)
jsimd_huff_encode_one_block(void *state, JOCTET *buffer, JCOEFPTR block,
                            int last_dc_val, c_derived_tbl *dctbl,
                            c_derived_tbl *actbl)
{
  return jsimd_huff_encode_one_block_neon(state, buffer, block, last_dc_val,
                                          dctbl, actbl);
}

GLOBAL(int)
jsimd_can_encode_mcu_AC_first_prepare(void)
{
  init_simd();

  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(void)
jsimd_encode_mcu_AC_first_prepare(const JCOEF *block,
                                  const int *jpeg_natural_order_start, int Sl,
                                  int Al, UJCOEF *values, size_t *zerobits)
{
  jsimd_encode_mcu_AC_first_prepare_neon(block, jpeg_natural_order_start,
                                         Sl, Al, values, zerobits);
}

GLOBAL(int)
jsimd_can_encode_mcu_AC_refine_prepare(void)
{
  init_simd();

  if (DCTSIZE != 8)
    return 0;
  if (sizeof(JCOEF) != 2)
    return 0;

  if (simd_support & JSIMD_NEON)
    return 1;

  return 0;
}

GLOBAL(int)
jsimd_encode_mcu_AC_refine_prepare(const JCOEF *block,
                                   const int *jpeg_natural_order_start, int Sl,
                                   int Al, UJCOEF *absvalues, size_t *bits)
{
  return jsimd_encode_mcu_AC_refine_prepare_neon(block,
                                                 jpeg_natural_order_start, Sl,
                                                 Al, absvalues, bits);
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

- **between**: A class/struct defined in this file

### Functions and Methods

- **NO_GETENV()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../jsimd.h`
- `ctype.h`
- `../../../src/jinclude.h`
- `../../../src/jdct.h`
- `../../../src/jsimd.h`
- `../../../src/jpeglib.h`
- `../../../src/jsimddct.h`


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

