# Documentation for `docs/3rdparty/libjpeg-turbo/src/jsamplecomp.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/src/jsamplecomp.h_docs.md`
- **File Name**: `jsamplecomp.h_docs.md`
- **File Size**: 16,022 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/src/jsamplecomp.h_docs.md](../../../../docs/3rdparty/libjpeg-turbo/src/jsamplecomp.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/src/jsamplecomp.h`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/src/jsamplecomp.h`
- **File Name**: `jsamplecomp.h`
- **File Size**: 12,702 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libjpeg-turbo/src/jsamplecomp.h](../../../3rdparty/libjpeg-turbo/src/jsamplecomp.h)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * jsamplecomp.h
 *
 * Copyright (C) 2022, D. R. Commander.
 * For conditions of distribution and use, see the accompanying README.ijg
 * file.
 */

/* In source files that must be compiled for multiple data precisions, we
 * prefix all precision-dependent data types, macros, methods, fields, and
 * function names with an underscore.  Including this file replaces those
 * precision-independent tokens with their precision-dependent equivalents,
 * based on the value of BITS_IN_JSAMPLE.
 */

#ifndef JSAMPLECOMP_H
#define JSAMPLECOMP_H

#if BITS_IN_JSAMPLE == 16

/* Sample data types and macros (jmorecfg.h) */
#define _JSAMPLE  J16SAMPLE

#define _MAXJSAMPLE  MAXJ16SAMPLE
#define _CENTERJSAMPLE   CENTERJ16SAMPLE

#define _JSAMPROW  J16SAMPROW
#define _JSAMPARRAY  J16SAMPARRAY
#define _JSAMPIMAGE  J16SAMPIMAGE

/* External functions (jpeglib.h) */
#define _jpeg_write_scanlines  jpeg16_write_scanlines
#define _jpeg_read_scanlines  jpeg16_read_scanlines

/* Internal methods (jpegint.h) */

#ifdef C_LOSSLESS_SUPPORTED
/* Use the 16-bit method in the jpeg_c_main_controller structure. */
#define _process_data  process_data_16
/* Use the 16-bit method in the jpeg_c_prep_controller structure. */
#define _pre_process_data  pre_process_data_16
/* Use the 16-bit method in the jpeg_c_coef_controller structure. */
#define _compress_data  compress_data_16
/* Use the 16-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert_16
/* Use the 16-bit method in the jpeg_downsampler structure. */
#define _downsample  downsample_16
#endif
#ifdef D_LOSSLESS_SUPPORTED
/* Use the 16-bit method in the jpeg_d_main_controller structure. */
#define _process_data  process_data_16
/* Use the 16-bit method in the jpeg_d_coef_controller structure. */
#define _decompress_data  decompress_data_16
/* Use the 16-bit method in the jpeg_d_post_controller structure. */
#define _post_process_data  post_process_data_16
/* Use the 16-bit method in the jpeg_upsampler structure. */
#define _upsample  upsample_16
/* Use the 16-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert_16
#endif

/* Global internal functions (jpegint.h) */
#ifdef C_LOSSLESS_SUPPORTED
#define _jinit_c_main_controller  j16init_c_main_controller
#define _jinit_c_prep_controller  j16init_c_prep_controller
#define _jinit_color_converter  j16init_color_converter
#define _jinit_downsampler  j16init_downsampler
#define _jinit_c_diff_controller  j16init_c_diff_controller
#define _jinit_lossless_compressor  j16init_lossless_compressor
#endif

#ifdef D_LOSSLESS_SUPPORTED
#define _jinit_d_main_controller  j16init_d_main_controller
#define _jinit_d_post_controller  j16init_d_post_controller
#define _jinit_upsampler  j16init_upsampler
#define _jinit_color_deconverter  j16init_color_deconverter
#define _jinit_merged_upsampler  j16init_merged_upsampler
#define _jinit_d_diff_controller  j16init_d_diff_controller
#define _jinit_lossless_decompressor  j16init_lossless_decompressor
#endif

#if defined(C_LOSSLESS_SUPPORTED) || defined(D_LOSSLESS_SUPPORTED)
#define _jcopy_sample_rows  j16copy_sample_rows
#endif

/* Internal fields (cdjpeg.h) */

#if defined(C_LOSSLESS_SUPPORTED) || defined(D_LOSSLESS_SUPPORTED)
/* Use the 16-bit buffer in the cjpeg_source_struct and djpeg_dest_struct
   structures. */
#define _buffer  buffer16
#endif

/* Image I/O functions (cdjpeg.h) */
#ifdef C_LOSSLESS_SUPPORTED
#define _jinit_read_ppm  j16init_read_ppm
#endif

#ifdef D_LOSSLESS_SUPPORTED
#define _jinit_write_ppm  j16init_write_ppm
#endif

#elif BITS_IN_JSAMPLE == 12

/* Sample data types and macros (jmorecfg.h) */
#define _JSAMPLE  J12SAMPLE

#define _MAXJSAMPLE  MAXJ12SAMPLE
#define _CENTERJSAMPLE   CENTERJ12SAMPLE

#define _JSAMPROW  J12SAMPROW
#define _JSAMPARRAY  J12SAMPARRAY
#define _JSAMPIMAGE  J12SAMPIMAGE

/* External functions (jpeglib.h) */
#define _jpeg_write_scanlines  jpeg12_write_scanlines
#define _jpeg_write_raw_data  jpeg12_write_raw_data
#define _jpeg_read_scanlines  jpeg12_read_scanlines
#define _jpeg_skip_scanlines  jpeg12_skip_scanlines
#define _jpeg_crop_scanline  jpeg12_crop_scanline
#define _jpeg_read_raw_data  jpeg12_read_raw_data

/* Internal methods (jpegint.h) */

/* Use the 12-bit method in the jpeg_c_main_controller structure. */
#define _process_data  process_data_12
/* Use the 12-bit method in the jpeg_c_prep_controller structure. */
#define _pre_process_data  pre_process_data_12
/* Use the 12-bit method in the jpeg_c_coef_controller structure. */
#define _compress_data  compress_data_12
/* Use the 12-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert_12
/* Use the 12-bit method in the jpeg_downsampler structure. */
#define _downsample  downsample_12
/* Use the 12-bit method in the jpeg_forward_dct structure. */
#define _forward_DCT  forward_DCT_12
/* Use the 12-bit method in the jpeg_d_main_controller structure. */
#define _process_data  process_data_12
/* Use the 12-bit method in the jpeg_d_coef_controller structure. */
#define _decompress_data  decompress_data_12
/* Use the 12-bit method in the jpeg_d_post_controller structure. */
#define _post_process_data  post_process_data_12
/* Use the 12-bit method in the jpeg_inverse_dct structure. */
#define _inverse_DCT_method_ptr  inverse_DCT_12_method_ptr
#define _inverse_DCT  inverse_DCT_12
/* Use the 12-bit method in the jpeg_upsampler structure. */
#define _upsample  upsample_12
/* Use the 12-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert_12
/* Use the 12-bit method in the jpeg_color_quantizer structure. */
#define _color_quantize  color_quantize_12

/* Global internal functions (jpegint.h) */
#define _jinit_c_main_controller  j12init_c_main_controller
#define _jinit_c_prep_controller  j12init_c_prep_controller
#define _jinit_c_coef_controller  j12init_c_coef_controller
#define _jinit_color_converter  j12init_color_converter
#define _jinit_downsampler  j12init_downsampler
#define _jinit_forward_dct  j12init_forward_dct
#ifdef C_LOSSLESS_SUPPORTED
#define _jinit_c_diff_controller  j12init_c_diff_controller
#define _jinit_lossless_compressor  j12init_lossless_compressor
#endif

#define _jinit_d_main_controller  j12init_d_main_controller
#define _jinit_d_coef_controller  j12init_d_coef_controller
#define _jinit_d_post_controller  j12init_d_post_controller
#define _jinit_inverse_dct  j12init_inverse_dct
#define _jinit_upsampler  j12init_upsampler
#define _jinit_color_deconverter  j12init_color_deconverter
#define _jinit_1pass_quantizer  j12init_1pass_quantizer
#define _jinit_2pass_quantizer  j12init_2pass_quantizer
#define _jinit_merged_upsampler  j12init_merged_upsampler
#ifdef D_LOSSLESS_SUPPORTED
#define _jinit_d_diff_controller  j12init_d_diff_controller
#define _jinit_lossless_decompressor  j12init_lossless_decompressor
#endif

#define _jcopy_sample_rows  j12copy_sample_rows

/* Global internal functions (jdct.h) */
#define _jpeg_fdct_islow  jpeg12_fdct_islow
#define _jpeg_fdct_ifast  jpeg12_fdct_ifast

#define _jpeg_idct_islow  jpeg12_idct_islow
#define _jpeg_idct_ifast  jpeg12_idct_ifast
#define _jpeg_idct_float  jpeg12_idct_float
#define _jpeg_idct_7x7  jpeg12_idct_7x7
#define _jpeg_idct_6x6  jpeg12_idct_6x6
#define _jpeg_idct_5x5  jpeg12_idct_5x5
#define _jpeg_idct_4x4  jpeg12_idct_4x4
#define _jpeg_idct_3x3  jpeg12_idct_3x3
#define _jpeg_idct_2x2  jpeg12_idct_2x2
#define _jpeg_idct_1x1  jpeg12_idct_1x1
#define _jpeg_idct_9x9  jpeg12_idct_9x9
#define _jpeg_idct_10x10  jpeg12_idct_10x10
#define _jpeg_idct_11x11  jpeg12_idct_11x11
#define _jpeg_idct_12x12  jpeg12_idct_12x12
#define _jpeg_idct_13x13  jpeg12_idct_13x13
#define _jpeg_idct_14x14  jpeg12_idct_14x14
#define _jpeg_idct_15x15  jpeg12_idct_15x15
#define _jpeg_idct_16x16  jpeg12_idct_16x16

/* Internal fields (cdjpeg.h) */

/* Use the 12-bit buffer in the cjpeg_source_struct and djpeg_dest_struct
   structures. */
#define _buffer  buffer12

/* Image I/O functions (cdjpeg.h) */
#define _jinit_write_gif  j12init_write_gif
#define _jinit_read_ppm  j12init_read_ppm
#define _jinit_write_ppm  j12init_write_ppm

#define _read_color_map  read_color_map_12

#else /* BITS_IN_JSAMPLE */

/* Sample data types and macros (jmorecfg.h) */
#define _JSAMPLE  JSAMPLE

#define _MAXJSAMPLE  MAXJSAMPLE
#define _CENTERJSAMPLE   CENTERJSAMPLE

#define _JSAMPROW  JSAMPROW
#define _JSAMPARRAY  JSAMPARRAY
#define _JSAMPIMAGE  JSAMPIMAGE

/* External functions (jpeglib.h) */
#define _jpeg_write_scanlines  jpeg_write_scanlines
#define _jpeg_write_raw_data  jpeg_write_raw_data
#define _jpeg_read_scanlines  jpeg_read_scanlines
#define _jpeg_skip_scanlines  jpeg_skip_scanlines
#define _jpeg_crop_scanline  jpeg_crop_scanline
#define _jpeg_read_raw_data  jpeg_read_raw_data

/* Internal methods (jpegint.h) */

/* Use the 8-bit method in the jpeg_c_main_controller structure. */
#define _process_data  process_data
/* Use the 8-bit method in the jpeg_c_prep_controller structure. */
#define _pre_process_data  pre_process_data
/* Use the 8-bit method in the jpeg_c_coef_controller structure. */
#define _compress_data  compress_data
/* Use the 8-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert
/* Use the 8-bit method in the jpeg_downsampler structure. */
#define _downsample  downsample
/* Use the 8-bit method in the jpeg_forward_dct structure. */
#define _forward_DCT  forward_DCT
/* Use the 8-bit method in the jpeg_d_main_controller structure. */
#define _process_data  process_data
/* Use the 8-bit method in the jpeg_d_coef_controller structure. */
#define _decompress_data  decompress_data
/* Use the 8-bit method in the jpeg_d_post_controller structure. */
#define _post_process_data  post_process_data
/* Use the 8-bit method in the jpeg_inverse_dct structure. */
#define _inverse_DCT_method_ptr  inverse_DCT_method_ptr
#define _inverse_DCT  inverse_DCT
/* Use the 8-bit method in the jpeg_upsampler structure. */
#define _upsample  upsample
/* Use the 8-bit method in the jpeg_color_converter structure. */
#define _color_convert  color_convert
/* Use the 8-bit method in the jpeg_color_quantizer structure. */
#define _color_quantize  color_quantize

/* Global internal functions (jpegint.h) */
#define _jinit_c_main_controller  jinit_c_main_controller
#define _jinit_c_prep_controller  jinit_c_prep_controller
#define _jinit_c_coef_controller  jinit_c_coef_controller
#define _jinit_color_converter  jinit_color_converter
#define _jinit_downsampler  jinit_downsampler
#define _jinit_forward_dct  jinit_forward_dct
#ifdef C_LOSSLESS_SUPPORTED
#define _jinit_c_diff_controller  jinit_c_diff_controller
#define _jinit_lossless_compressor  jinit_lossless_compressor
#endif

#define _jinit_d_main_controller  jinit_d_main_controller
#define _jinit_d_coef_controller  jinit_d_coef_controller
#define _jinit_d_post_controller  jinit_d_post_controller
#define _jinit_inverse_dct  jinit_inverse_dct
#define _jinit_upsampler  jinit_upsampler
#define _jinit_color_deconverter  jinit_color_deconverter
#define _jinit_1pass_quantizer  jinit_1pass_quantizer
#define _jinit_2pass_quantizer  jinit_2pass_quantizer
#define _jinit_merged_upsampler  jinit_merged_upsampler
#ifdef D_LOSSLESS_SUPPORTED
#define _jinit_d_diff_controller  jinit_d_diff_controller
#define _jinit_lossless_decompressor  jinit_lossless_decompressor
#endif

#define _jcopy_sample_rows  jcopy_sample_rows

/* Global internal functions (jdct.h) */
#define _jpeg_fdct_islow  jpeg_fdct_islow
#define _jpeg_fdct_ifast  jpeg_fdct_ifast

#define _jpeg_idct_islow  jpeg_idct_islow
#define _jpeg_idct_ifast  jpeg_idct_ifast
#define _jpeg_idct_float  jpeg_idct_float
#define _jpeg_idct_7x7  jpeg_idct_7x7
#define _jpeg_idct_6x6  jpeg_idct_6x6
#define _jpeg_idct_5x5  jpeg_idct_5x5
#define _jpeg_idct_4x4  jpeg_idct_4x4
#define _jpeg_idct_3x3  jpeg_idct_3x3
#define _jpeg_idct_2x2  jpeg_idct_2x2
#define _jpeg_idct_1x1  jpeg_idct_1x1
#define _jpeg_idct_9x9  jpeg_idct_9x9
#define _jpeg_idct_10x10  jpeg_idct_10x10
#define _jpeg_idct_11x11  jpeg_idct_11x11
#define _jpeg_idct_12x12  jpeg_idct_12x12
#define _jpeg_idct_13x13  jpeg_idct_13x13
#define _jpeg_idct_14x14  jpeg_idct_14x14
#define _jpeg_idct_15x15  jpeg_idct_15x15
#define _jpeg_idct_16x16  jpeg_idct_16x16

/* Internal fields (cdjpeg.h) */

/* Use the 8-bit buffer in the cjpeg_source_struct and djpeg_dest_struct
   structures. */
#define _buffer  buffer

/* Image I/O functions (cdjpeg.h) */
#define _jinit_write_gif  jinit_write_gif
#define _jinit_read_ppm  jinit_read_ppm
#define _jinit_write_ppm  jinit_write_ppm

#define _read_color_map  read_color_map

#endif /* BITS_IN_JSAMPLE */

#endif /* JSAMPLECOMP_H */
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **structures**: A class/struct defined in this file
- **and**: A class/struct defined in this file

### Functions and Methods

- **names()**: A function/method defined in this file
- **JSAMPLECOMP_H()**: A function/method defined in this file
- **C_LOSSLESS_SUPPORTED()**: A function/method defined in this file
- **D_LOSSLESS_SUPPORTED()**: A function/method defined in this file


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

