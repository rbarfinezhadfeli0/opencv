# Documentation for `3rdparty/libtiff/tif_codec.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_codec.c`
- **File Name**: `tif_codec.c`
- **File Size**: 5,113 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_codec.c](../../3rdparty/libtiff/tif_codec.c)

## Purpose and Role

This file is located in the `3rdparty/libtiff` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Copyright (c) 1988-1997 Sam Leffler
 * Copyright (c) 1991-1997 Silicon Graphics, Inc.
 *
 * Permission to use, copy, modify, distribute, and sell this software and
 * its documentation for any purpose is hereby granted without fee, provided
 * that (i) the above copyright notices and this permission notice appear in
 * all copies of the software and related documentation, and (ii) the names of
 * Sam Leffler and Silicon Graphics may not be used in any advertising or
 * publicity relating to the software without the specific, prior written
 * permission of Sam Leffler and Silicon Graphics.
 *
 * THE SOFTWARE IS PROVIDED "AS-IS" AND WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS, IMPLIED OR OTHERWISE, INCLUDING WITHOUT LIMITATION, ANY
 * WARRANTY OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.
 *
 * IN NO EVENT SHALL SAM LEFFLER OR SILICON GRAPHICS BE LIABLE FOR
 * ANY SPECIAL, INCIDENTAL, INDIRECT OR CONSEQUENTIAL DAMAGES OF ANY KIND,
 * OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS,
 * WHETHER OR NOT ADVISED OF THE POSSIBILITY OF DAMAGE, AND ON ANY THEORY OF
 * LIABILITY, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE
 * OF THIS SOFTWARE.
 */

/*
 * TIFF Library
 *
 * Builtin Compression Scheme Configuration Support.
 */
#include "tiffiop.h"

static int NotConfigured(TIFF *, int);

#ifndef LZW_SUPPORT
#define TIFFInitLZW NotConfigured
#endif
#ifndef PACKBITS_SUPPORT
#define TIFFInitPackBits NotConfigured
#endif
#ifndef THUNDER_SUPPORT
#define TIFFInitThunderScan NotConfigured
#endif
#ifndef NEXT_SUPPORT
#define TIFFInitNeXT NotConfigured
#endif
#ifndef JPEG_SUPPORT
#define TIFFInitJPEG NotConfigured
#endif
#ifndef OJPEG_SUPPORT
#define TIFFInitOJPEG NotConfigured
#endif
#ifndef CCITT_SUPPORT
#define TIFFInitCCITTRLE NotConfigured
#define TIFFInitCCITTRLEW NotConfigured
#define TIFFInitCCITTFax3 NotConfigured
#define TIFFInitCCITTFax4 NotConfigured
#endif
#ifndef JBIG_SUPPORT
#define TIFFInitJBIG NotConfigured
#endif
#ifndef ZIP_SUPPORT
#define TIFFInitZIP NotConfigured
#endif
#ifndef PIXARLOG_SUPPORT
#define TIFFInitPixarLog NotConfigured
#endif
#ifndef LOGLUV_SUPPORT
#define TIFFInitSGILog NotConfigured
#endif
#ifndef LERC_SUPPORT
#define TIFFInitLERC NotConfigured
#endif
#ifndef LZMA_SUPPORT
#define TIFFInitLZMA NotConfigured
#endif
#ifndef ZSTD_SUPPORT
#define TIFFInitZSTD NotConfigured
#endif
#ifndef WEBP_SUPPORT
#define TIFFInitWebP NotConfigured
#endif

/*
 * Compression schemes statically built into the library.
 */
const TIFFCodec _TIFFBuiltinCODECS[] = {
    {"None", COMPRESSION_NONE, TIFFInitDumpMode},
    {"LZW", COMPRESSION_LZW, TIFFInitLZW},
    {"PackBits", COMPRESSION_PACKBITS, TIFFInitPackBits},
    {"ThunderScan", COMPRESSION_THUNDERSCAN, TIFFInitThunderScan},
    {"NeXT", COMPRESSION_NEXT, TIFFInitNeXT},
    {"JPEG", COMPRESSION_JPEG, TIFFInitJPEG},
    {"Old-style JPEG", COMPRESSION_OJPEG, TIFFInitOJPEG},
    {"CCITT RLE", COMPRESSION_CCITTRLE, TIFFInitCCITTRLE},
    {"CCITT RLE/W", COMPRESSION_CCITTRLEW, TIFFInitCCITTRLEW},
    {"CCITT Group 3", COMPRESSION_CCITTFAX3, TIFFInitCCITTFax3},
    {"CCITT Group 4", COMPRESSION_CCITTFAX4, TIFFInitCCITTFax4},
    {"ISO JBIG", COMPRESSION_JBIG, TIFFInitJBIG},
    {"Deflate", COMPRESSION_DEFLATE, TIFFInitZIP},
    {"AdobeDeflate", COMPRESSION_ADOBE_DEFLATE, TIFFInitZIP},
    {"PixarLog", COMPRESSION_PIXARLOG, TIFFInitPixarLog},
    {"SGILog", COMPRESSION_SGILOG, TIFFInitSGILog},
    {"SGILog24", COMPRESSION_SGILOG24, TIFFInitSGILog},
    {"LZMA", COMPRESSION_LZMA, TIFFInitLZMA},
    {"ZSTD", COMPRESSION_ZSTD, TIFFInitZSTD},
    {"WEBP", COMPRESSION_WEBP, TIFFInitWebP},
    {"LERC", COMPRESSION_LERC, TIFFInitLERC},
    {NULL, 0, NULL}};

static int _notConfigured(TIFF *tif)
{
    const TIFFCodec *c = TIFFFindCODEC(tif->tif_dir.td_compression);
    char compression_code[20];

    snprintf(compression_code, sizeof(compression_code), "%" PRIu16,
             tif->tif_dir.td_compression);
    TIFFErrorExtR(tif, tif->tif_name,
                  "%s compression support is not configured",
                  c ? c->name : compression_code);
    return (0);
}

static int NotConfigured(TIFF *tif, int scheme)
{
    (void)scheme;

    tif->tif_fixuptags = _notConfigured;
    tif->tif_decodestatus = FALSE;
    tif->tif_setupdecode = _notConfigured;
    tif->tif_encodestatus = FALSE;
    tif->tif_setupencode = _notConfigured;
    return (1);
}

/************************************************************************/
/*                       TIFFIsCODECConfigured()                        */
/************************************************************************/

/**
 * Check whether we have working codec for the specific coding scheme.
 *
 * @return returns 1 if the codec is configured and working. Otherwise
 * 0 will be returned.
 */

int TIFFIsCODECConfigured(uint16_t scheme)
{
    const TIFFCodec *codec = TIFFFindCODEC(scheme);

    if (codec == NULL)
    {
        return 0;
    }
    if (codec->init == NULL)
    {
        return 0;
    }
    if (codec->init != NotConfigured)
    {
        return 1;
    }
    return 0;
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

- **JBIG_SUPPORT()**: A function/method defined in this file
- **CCITT_SUPPORT()**: A function/method defined in this file
- **NEXT_SUPPORT()**: A function/method defined in this file
- **THUNDER_SUPPORT()**: A function/method defined in this file
- **ZIP_SUPPORT()**: A function/method defined in this file
- **LZMA_SUPPORT()**: A function/method defined in this file
- **LOGLUV_SUPPORT()**: A function/method defined in this file
- **ZSTD_SUPPORT()**: A function/method defined in this file
- **PACKBITS_SUPPORT()**: A function/method defined in this file
- **LZW_SUPPORT()**: A function/method defined in this file
- **LERC_SUPPORT()**: A function/method defined in this file
- **JPEG_SUPPORT()**: A function/method defined in this file
- **OJPEG_SUPPORT()**: A function/method defined in this file
- **PIXARLOG_SUPPORT()**: A function/method defined in this file
- **WEBP_SUPPORT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `tiffiop.h`


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

