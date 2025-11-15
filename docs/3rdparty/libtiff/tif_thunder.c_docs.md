# Documentation for `3rdparty/libtiff/tif_thunder.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_thunder.c`
- **File Name**: `tif_thunder.c`
- **File Size**: 7,418 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_thunder.c](../../3rdparty/libtiff/tif_thunder.c)

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

#include "tiffiop.h"
#include <assert.h>
#ifdef THUNDER_SUPPORT
/*
 * TIFF Library.
 *
 * ThunderScan 4-bit Compression Algorithm Support
 */

/*
 * ThunderScan uses an encoding scheme designed for
 * 4-bit pixel values.  Data is encoded in bytes, with
 * each byte split into a 2-bit code word and a 6-bit
 * data value.  The encoding gives raw data, runs of
 * pixels, or pixel values encoded as a delta from the
 * previous pixel value.  For the latter, either 2-bit
 * or 3-bit delta values are used, with the deltas packed
 * into a single byte.
 */
#define THUNDER_DATA 0x3f /* mask for 6-bit data */
#define THUNDER_CODE 0xc0 /* mask for 2-bit code word */
/* code values */
#define THUNDER_RUN 0x00        /* run of pixels w/ encoded count */
#define THUNDER_2BITDELTAS 0x40 /* 3 pixels w/ encoded 2-bit deltas */
#define DELTA2_SKIP 2           /* skip code for 2-bit deltas */
#define THUNDER_3BITDELTAS 0x80 /* 2 pixels w/ encoded 3-bit deltas */
#define DELTA3_SKIP 4           /* skip code for 3-bit deltas */
#define THUNDER_RAW 0xc0        /* raw data encoded */

static const int twobitdeltas[4] = {0, 1, 0, -1};
static const int threebitdeltas[8] = {0, 1, 2, 3, 0, -3, -2, -1};

#define SETPIXEL(op, v)                                                        \
    {                                                                          \
        lastpixel = (v)&0xf;                                                   \
        if (npixels < maxpixels)                                               \
        {                                                                      \
            if (npixels++ & 1)                                                 \
                *op++ |= lastpixel;                                            \
            else                                                               \
                op[0] = (uint8_t)(lastpixel << 4);                             \
        }                                                                      \
    }

static int ThunderSetupDecode(TIFF *tif)
{
    static const char module[] = "ThunderSetupDecode";

    if (tif->tif_dir.td_bitspersample != 4)
    {
        TIFFErrorExtR(tif, module,
                      "Wrong bitspersample value (%d), Thunder decoder only "
                      "supports 4bits per sample.",
                      (int)tif->tif_dir.td_bitspersample);
        return 0;
    }

    return (1);
}

static int ThunderDecode(TIFF *tif, uint8_t *op0, tmsize_t maxpixels)
{
    static const char module[] = "ThunderDecode";
    register unsigned char *bp;
    register tmsize_t cc;
    unsigned int lastpixel;
    tmsize_t npixels;
    uint8_t *op = op0;

    bp = (unsigned char *)tif->tif_rawcp;
    cc = tif->tif_rawcc;
    lastpixel = 0;
    npixels = 0;
    while (cc > 0 && npixels < maxpixels)
    {
        int n, delta;

        n = *bp++;
        cc--;
        switch (n & THUNDER_CODE)
        {
            case THUNDER_RUN: /* pixel run */
                /*
                 * Replicate the last pixel n times,
                 * where n is the lower-order 6 bits.
                 */
                if (n == 0)
                    break;
                if (npixels & 1)
                {
                    op[0] |= lastpixel;
                    lastpixel = *op++;
                    npixels++;
                    n--;
                }
                else
                    lastpixel |= lastpixel << 4;
                npixels += n;
                if (npixels > maxpixels)
                    break;
                for (; n > 0; n -= 2)
                    *op++ = (uint8_t)lastpixel;
                if (n == -1)
                    *--op &= 0xf0;
                lastpixel &= 0xf;
                break;
            case THUNDER_2BITDELTAS: /* 2-bit deltas */
                if ((delta = ((n >> 4) & 3)) != DELTA2_SKIP)
                    SETPIXEL(op,
                             (unsigned)((int)lastpixel + twobitdeltas[delta]));
                if ((delta = ((n >> 2) & 3)) != DELTA2_SKIP)
                    SETPIXEL(op,
                             (unsigned)((int)lastpixel + twobitdeltas[delta]));
                if ((delta = (n & 3)) != DELTA2_SKIP)
                    SETPIXEL(op,
                             (unsigned)((int)lastpixel + twobitdeltas[delta]));
                break;
            case THUNDER_3BITDELTAS: /* 3-bit deltas */
                if ((delta = ((n >> 3) & 7)) != DELTA3_SKIP)
                    SETPIXEL(
                        op, (unsigned)((int)lastpixel + threebitdeltas[delta]));
                if ((delta = (n & 7)) != DELTA3_SKIP)
                    SETPIXEL(
                        op, (unsigned)((int)lastpixel + threebitdeltas[delta]));
                break;
            case THUNDER_RAW: /* raw data */
                SETPIXEL(op, n);
                break;
        }
    }
    tif->tif_rawcp = (uint8_t *)bp;
    tif->tif_rawcc = cc;
    if (npixels != maxpixels)
    {
        uint8_t *op_end = op0 + (maxpixels + 1) / 2;
        memset(op, 0, (size_t)(op_end - op));
        TIFFErrorExtR(tif, module,
                      "%s data at scanline %lu (%" PRIu64 " != %" PRIu64 ")",
                      npixels < maxpixels ? "Not enough" : "Too much",
                      (unsigned long)tif->tif_row, (uint64_t)npixels,
                      (uint64_t)maxpixels);
        return (0);
    }

    return (1);
}

static int ThunderDecodeRow(TIFF *tif, uint8_t *buf, tmsize_t occ, uint16_t s)
{
    static const char module[] = "ThunderDecodeRow";
    uint8_t *row = buf;

    (void)s;
    if (occ % tif->tif_scanlinesize)
    {
        TIFFErrorExtR(tif, module, "Fractional scanlines cannot be read");
        return (0);
    }
    while (occ > 0)
    {
        if (!ThunderDecode(tif, row, tif->tif_dir.td_imagewidth))
            return (0);
        occ -= tif->tif_scanlinesize;
        row += tif->tif_scanlinesize;
    }
    return (1);
}

int TIFFInitThunderScan(TIFF *tif, int scheme)
{
    (void)scheme;

    tif->tif_setupdecode = ThunderSetupDecode;
    tif->tif_decoderow = ThunderDecodeRow;
    tif->tif_decodestrip = ThunderDecodeRow;
    return (1);
}
#endif /* THUNDER_SUPPORT */
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

- **THUNDER_SUPPORT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `tiffiop.h`
- `assert.h`

**Python Imports:**
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

