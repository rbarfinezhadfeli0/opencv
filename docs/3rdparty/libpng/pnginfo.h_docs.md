# Documentation for `3rdparty/libpng/pnginfo.h`

## File Metadata

- **Full Path**: `3rdparty/libpng/pnginfo.h`
- **File Name**: `pnginfo.h`
- **File Size**: 12,721 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libpng/pnginfo.h](../../3rdparty/libpng/pnginfo.h)

## Purpose and Role

This file is located in the `3rdparty/libpng` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* pnginfo.h - header file for PNG reference library
 *
 * Copyright (c) 2018 Cosmin Truta
 * Copyright (c) 1998-2002,2004,2006-2013,2018 Glenn Randers-Pehrson
 * Copyright (c) 1996-1997 Andreas Dilger
 * Copyright (c) 1995-1996 Guy Eric Schalnat, Group 42, Inc.
 *
 * This code is released under the libpng license.
 * For conditions of distribution and use, see the disclaimer
 * and license in png.h
 */

 /* png_info is a structure that holds the information in a PNG file so
 * that the application can find out the characteristics of the image.
 * If you are reading the file, this structure will tell you what is
 * in the PNG file.  If you are writing the file, fill in the information
 * you want to put into the PNG file, using png_set_*() functions, then
 * call png_write_info().
 *
 * The names chosen should be very close to the PNG specification, so
 * consult that document for information about the meaning of each field.
 *
 * With libpng < 0.95, it was only possible to directly set and read the
 * the values in the png_info_struct, which meant that the contents and
 * order of the values had to remain fixed.  With libpng 0.95 and later,
 * however, there are now functions that abstract the contents of
 * png_info_struct from the application, so this makes it easier to use
 * libpng with dynamic libraries, and even makes it possible to use
 * libraries that don't have all of the libpng ancillary chunk-handing
 * functionality.  In libpng-1.5.0 this was moved into a separate private
 * file that is not visible to applications.
 *
 * The following members may have allocated storage attached that should be
 * cleaned up before the structure is discarded: palette, trans, text,
 * pcal_purpose, pcal_units, pcal_params, hist, iccp_name, iccp_profile,
 * splt_palettes, scal_unit, row_pointers, and unknowns.   By default, these
 * are automatically freed when the info structure is deallocated, if they were
 * allocated internally by libpng.  This behavior can be changed by means
 * of the png_data_freer() function.
 *
 * More allocation details: all the chunk-reading functions that
 * change these members go through the corresponding png_set_*
 * functions.  A function to clear these members is available: see
 * png_free_data().  The png_set_* functions do not depend on being
 * able to point info structure members to any of the storage they are
 * passed (they make their own copies), EXCEPT that the png_set_text
 * functions use the same storage passed to them in the text_ptr or
 * itxt_ptr structure argument, and the png_set_rows and png_set_unknowns
 * functions do not make their own copies.
 */
#ifndef PNGINFO_H
#define PNGINFO_H

struct png_info_def
{
   /* The following are necessary for every PNG file */
   png_uint_32 width;       /* width of image in pixels (from IHDR) */
   png_uint_32 height;      /* height of image in pixels (from IHDR) */
   png_uint_32 valid;       /* valid chunk data (see PNG_INFO_ below) */
   size_t rowbytes;         /* bytes needed to hold an untransformed row */
   png_colorp palette;      /* array of color values (valid & PNG_INFO_PLTE) */
   png_uint_16 num_palette; /* number of color entries in "palette" (PLTE) */
   png_uint_16 num_trans;   /* number of transparent palette color (tRNS) */
   png_byte bit_depth;      /* 1, 2, 4, 8, or 16 bits/channel (from IHDR) */
   png_byte color_type;     /* see PNG_COLOR_TYPE_ below (from IHDR) */
   /* The following three should have been named *_method not *_type */
   png_byte compression_type; /* must be PNG_COMPRESSION_TYPE_BASE (IHDR) */
   png_byte filter_type;    /* must be PNG_FILTER_TYPE_BASE (from IHDR) */
   png_byte interlace_type; /* One of PNG_INTERLACE_NONE, PNG_INTERLACE_ADAM7 */

   /* The following are set by png_set_IHDR, called from the application on
    * write, but the are never actually used by the write code.
    */
   png_byte channels;       /* number of data channels per pixel (1, 2, 3, 4) */
   png_byte pixel_depth;    /* number of bits per pixel */
   png_byte spare_byte;     /* to align the data, and for future use */

#ifdef PNG_READ_SUPPORTED
   /* This is never set during write */
   png_byte signature[8];   /* magic bytes read by libpng from start of file */
#endif

   /* The rest of the data is optional.  If you are reading, check the
    * valid field to see if the information in these are valid.  If you
    * are writing, set the valid field to those chunks you want written,
    * and initialize the appropriate fields below.
    */

#if defined(PNG_COLORSPACE_SUPPORTED) || defined(PNG_GAMMA_SUPPORTED)
   /* png_colorspace only contains 'flags' if neither GAMMA or COLORSPACE are
    * defined.  When COLORSPACE is switched on all the colorspace-defining
    * chunks should be enabled, when GAMMA is switched on all the gamma-defining
    * chunks should be enabled.  If this is not done it becomes possible to read
    * inconsistent PNG files and assign a probably incorrect interpretation to
    * the information.  (In other words, by carefully choosing which chunks to
    * recognize the system configuration can select an interpretation for PNG
    * files containing ambiguous data and this will result in inconsistent
    * behavior between different libpng builds!)
    */
   png_colorspace colorspace;
#endif

#ifdef PNG_cICP_SUPPORTED
   /* cICP chunk data */
   png_byte cicp_colour_primaries;
   png_byte cicp_transfer_function;
   png_byte cicp_matrix_coefficients;
   png_byte cicp_video_full_range_flag;
#endif

#ifdef PNG_iCCP_SUPPORTED
   /* iCCP chunk data. */
   png_charp iccp_name;     /* profile name */
   png_bytep iccp_profile;  /* International Color Consortium profile data */
   png_uint_32 iccp_proflen;  /* ICC profile data length */
#endif

#ifdef PNG_TEXT_SUPPORTED
   /* The tEXt, and zTXt chunks contain human-readable textual data in
    * uncompressed, compressed, and optionally compressed forms, respectively.
    * The data in "text" is an array of pointers to uncompressed,
    * null-terminated C strings. Each chunk has a keyword that describes the
    * textual data contained in that chunk.  Keywords are not required to be
    * unique, and the text string may be empty.  Any number of text chunks may
    * be in an image.
    */
   int num_text; /* number of comments read or comments to write */
   int max_text; /* current size of text array */
   png_textp text; /* array of comments read or comments to write */
#endif /* TEXT */

#ifdef PNG_tIME_SUPPORTED
   /* The tIME chunk holds the last time the displayed image data was
    * modified.  See the png_time struct for the contents of this struct.
    */
   png_time mod_time;
#endif

#ifdef PNG_sBIT_SUPPORTED
   /* The sBIT chunk specifies the number of significant high-order bits
    * in the pixel data.  Values are in the range [1, bit_depth], and are
    * only specified for the channels in the pixel data.  The contents of
    * the low-order bits is not specified.  Data is valid if
    * (valid & PNG_INFO_sBIT) is non-zero.
    */
   png_color_8 sig_bit; /* significant bits in color channels */
#endif

#if defined(PNG_tRNS_SUPPORTED) || defined(PNG_READ_EXPAND_SUPPORTED) || \
defined(PNG_READ_BACKGROUND_SUPPORTED)
   /* The tRNS chunk supplies transparency data for paletted images and
    * other image types that don't need a full alpha channel.  There are
    * "num_trans" transparency values for a paletted image, stored in the
    * same order as the palette colors, starting from index 0.  Values
    * for the data are in the range [0, 255], ranging from fully transparent
    * to fully opaque, respectively.  For non-paletted images, there is a
    * single color specified that should be treated as fully transparent.
    * Data is valid if (valid & PNG_INFO_tRNS) is non-zero.
    */
   png_bytep trans_alpha;    /* alpha values for paletted image */
   png_color_16 trans_color; /* transparent color for non-palette image */
#endif

#if defined(PNG_bKGD_SUPPORTED) || defined(PNG_READ_BACKGROUND_SUPPORTED)
   /* The bKGD chunk gives the suggested image background color if the
    * display program does not have its own background color and the image
    * is needs to composited onto a background before display.  The colors
    * in "background" are normally in the same color space/depth as the
    * pixel data.  Data is valid if (valid & PNG_INFO_bKGD) is non-zero.
    */
   png_color_16 background;
#endif

#ifdef PNG_oFFs_SUPPORTED
   /* The oFFs chunk gives the offset in "offset_unit_type" units rightwards
    * and downwards from the top-left corner of the display, page, or other
    * application-specific co-ordinate space.  See the PNG_OFFSET_ defines
    * below for the unit types.  Valid if (valid & PNG_INFO_oFFs) non-zero.
    */
   png_int_32 x_offset; /* x offset on page */
   png_int_32 y_offset; /* y offset on page */
   png_byte offset_unit_type; /* offset units type */
#endif

#ifdef PNG_pHYs_SUPPORTED
   /* The pHYs chunk gives the physical pixel density of the image for
    * display or printing in "phys_unit_type" units (see PNG_RESOLUTION_
    * defines below).  Data is valid if (valid & PNG_INFO_pHYs) is non-zero.
    */
   png_uint_32 x_pixels_per_unit; /* horizontal pixel density */
   png_uint_32 y_pixels_per_unit; /* vertical pixel density */
   png_byte phys_unit_type; /* resolution type (see PNG_RESOLUTION_ below) */
#endif

#ifdef PNG_eXIf_SUPPORTED
   int num_exif;  /* Added at libpng-1.6.31 */
   png_bytep exif;
# ifdef PNG_READ_eXIf_SUPPORTED
   png_bytep eXIf_buf;  /* Added at libpng-1.6.32 */
# endif
#endif

#ifdef PNG_hIST_SUPPORTED
   /* The hIST chunk contains the relative frequency or importance of the
    * various palette entries, so that a viewer can intelligently select a
    * reduced-color palette, if required.  Data is an array of "num_palette"
    * values in the range [0,65535]. Data valid if (valid & PNG_INFO_hIST)
    * is non-zero.
    */
   png_uint_16p hist;
#endif

#ifdef PNG_pCAL_SUPPORTED
   /* The pCAL chunk describes a transformation between the stored pixel
    * values and original physical data values used to create the image.
    * The integer range [0, 2^bit_depth - 1] maps to the floating-point
    * range given by [pcal_X0, pcal_X1], and are further transformed by a
    * (possibly non-linear) transformation function given by "pcal_type"
    * and "pcal_params" into "pcal_units".  Please see the PNG_EQUATION_
    * defines below, and the PNG-Group's PNG extensions document for a
    * complete description of the transformations and how they should be
    * implemented, and for a description of the ASCII parameter strings.
    * Data values are valid if (valid & PNG_INFO_pCAL) non-zero.
    */
   png_charp pcal_purpose;  /* pCAL chunk description string */
   png_int_32 pcal_X0;      /* minimum value */
   png_int_32 pcal_X1;      /* maximum value */
   png_charp pcal_units;    /* Latin-1 string giving physical units */
   png_charpp pcal_params;  /* ASCII strings containing parameter values */
   png_byte pcal_type;      /* equation type (see PNG_EQUATION_ below) */
   png_byte pcal_nparams;   /* number of parameters given in pcal_params */
#endif

/* New members added in libpng-1.0.6 */
   png_uint_32 free_me;     /* flags items libpng is responsible for freeing */

#ifdef PNG_STORE_UNKNOWN_CHUNKS_SUPPORTED
   /* Storage for unknown chunks that the library doesn't recognize. */
   png_unknown_chunkp unknown_chunks;

   /* The type of this field is limited by the type of
    * png_struct::user_chunk_cache_max, else overflow can occur.
    */
   int                unknown_chunks_num;
#endif

#ifdef PNG_sPLT_SUPPORTED
   /* Data on sPLT chunks (there may be more than one). */
   png_sPLT_tp splt_palettes;
   int         splt_palettes_num; /* Match type returned by png_get API */
#endif

#ifdef PNG_sCAL_SUPPORTED
   /* The sCAL chunk describes the actual physical dimensions of the
    * subject matter of the graphic.  The chunk contains a unit specification
    * a byte value, and two ASCII strings representing floating-point
    * values.  The values are width and height corresponding to one pixel
    * in the image.  Data values are valid if (valid & PNG_INFO_sCAL) is
    * non-zero.
    */
   png_byte scal_unit;         /* unit of physical scale */
   png_charp scal_s_width;     /* string containing height */
   png_charp scal_s_height;    /* string containing width */
#endif

#ifdef PNG_INFO_IMAGE_SUPPORTED
   /* Memory has been allocated if (valid & PNG_ALLOCATED_INFO_ROWS)
      non-zero */
   /* Data valid if (valid & PNG_INFO_IDAT) non-zero */
   png_bytepp row_pointers;        /* the image bits */
#endif

};
#endif /* PNGINFO_H */
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

- **png_info_def**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **from**: A class/struct defined in this file

### Functions and Methods

- **given()**: A function/method defined in this file
- **PNG_sPLT_SUPPORTED()**: A function/method defined in this file
- **PNG_INFO_IMAGE_SUPPORTED()**: A function/method defined in this file
- **PNG_tIME_SUPPORTED()**: A function/method defined in this file
- **PNG_pHYs_SUPPORTED()**: A function/method defined in this file
- **PNG_STORE_UNKNOWN_CHUNKS_SUPPORTED()**: A function/method defined in this file
- **PNG_iCCP_SUPPORTED()**: A function/method defined in this file
- **PNGINFO_H()**: A function/method defined in this file
- **PNG_TEXT_SUPPORTED()**: A function/method defined in this file
- **PNG_oFFs_SUPPORTED()**: A function/method defined in this file
- **PNG_eXIf_SUPPORTED()**: A function/method defined in this file
- **PNG_cICP_SUPPORTED()**: A function/method defined in this file
- **PNG_hIST_SUPPORTED()**: A function/method defined in this file
- **PNG_READ_SUPPORTED()**: A function/method defined in this file
- **PNG_sCAL_SUPPORTED()**: A function/method defined in this file
- **to()**: A function/method defined in this file
- **PNG_pCAL_SUPPORTED()**: A function/method defined in this file
- **PNG_READ_eXIf_SUPPORTED()**: A function/method defined in this file
- **PNG_sBIT_SUPPORTED()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `index`
- `the`
- `start`
- `IHDR`
- `fully`


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

