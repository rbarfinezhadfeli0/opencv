# Documentation for `docs/3rdparty/libtiff/tif_color.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libtiff/tif_color.c_docs.md`
- **File Name**: `tif_color.c_docs.md`
- **File Size**: 14,464 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libtiff/tif_color.c_docs.md](../../../docs/3rdparty/libtiff/tif_color.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libtiff` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libtiff/tif_color.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_color.c`
- **File Name**: `tif_color.c`
- **File Size**: 10,918 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_color.c](../../3rdparty/libtiff/tif_color.c)

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
 * CIE L*a*b* to CIE XYZ and CIE XYZ to RGB conversion routines are taken
 * from the VIPS library (http://www.vips.ecs.soton.ac.uk) with
 * the permission of John Cupitt, the VIPS author.
 */

/*
 * TIFF Library.
 *
 * Color space conversion routines.
 */

#include "tiffiop.h"
#include <math.h>

/*
 * Convert color value from the CIE L*a*b* 1976 space to CIE XYZ.
 */
void TIFFCIELabToXYZ(TIFFCIELabToRGB *cielab, uint32_t l, int32_t a, int32_t b,
                     float *X, float *Y, float *Z)
{
    TIFFCIELab16ToXYZ(cielab, l * 257, a * 256, b * 256, X, Y, Z);
}

/*
 * For CIELab encoded in 16 bits, L is an unsigned integer range [0,65535].
 * The a* and b* components are signed integers range [-32768,32767]. The 16
 * bit chrominance values are encoded as 256 times the 1976 CIE a* and b*
 * values
 */
void TIFFCIELab16ToXYZ(TIFFCIELabToRGB *cielab, uint32_t l, int32_t a,
                       int32_t b, float *X, float *Y, float *Z)
{
    float L = (float)l * 100.0F / 65535.0F;
    float cby, tmp;

    if (L < 8.856F)
    {
        *Y = (L * cielab->Y0) / 903.292F;
        cby = 7.787F * (*Y / cielab->Y0) + 16.0F / 116.0F;
    }
    else
    {
        cby = (L + 16.0F) / 116.0F;
        *Y = cielab->Y0 * cby * cby * cby;
    }

    tmp = (float)a / 256.0F / 500.0F + cby;
    if (tmp < 0.2069F)
        *X = cielab->X0 * (tmp - 0.13793F) / 7.787F;
    else
        *X = cielab->X0 * tmp * tmp * tmp;

    tmp = cby - (float)b / 256.0F / 200.0F;
    if (tmp < 0.2069F)
        *Z = cielab->Z0 * (tmp - 0.13793F) / 7.787F;
    else
        *Z = cielab->Z0 * tmp * tmp * tmp;
}

#define RINT(R) ((uint32_t)((R) > 0 ? ((R) + 0.5) : ((R)-0.5)))
/*
 * Convert color value from the XYZ space to RGB.
 */
void TIFFXYZToRGB(TIFFCIELabToRGB *cielab, float X, float Y, float Z,
                  uint32_t *r, uint32_t *g, uint32_t *b)
{
    size_t i;
    float Yr, Yg, Yb;
    float *matrix = &cielab->display.d_mat[0][0];

    /* Multiply through the matrix to get luminosity values. */
    Yr = matrix[0] * X + matrix[1] * Y + matrix[2] * Z;
    Yg = matrix[3] * X + matrix[4] * Y + matrix[5] * Z;
    Yb = matrix[6] * X + matrix[7] * Y + matrix[8] * Z;

    /* Clip input */
    Yr = TIFFmax(Yr, cielab->display.d_Y0R);
    Yg = TIFFmax(Yg, cielab->display.d_Y0G);
    Yb = TIFFmax(Yb, cielab->display.d_Y0B);

    /* Avoid overflow in case of wrong input values */
    Yr = TIFFmin(Yr, cielab->display.d_YCR);
    Yg = TIFFmin(Yg, cielab->display.d_YCG);
    Yb = TIFFmin(Yb, cielab->display.d_YCB);

    /* Turn luminosity to colour value. */
    i = (size_t)((Yr - cielab->display.d_Y0R) / cielab->rstep);
    i = TIFFmin((size_t)cielab->range, i);
    *r = RINT(cielab->Yr2r[i]);

    i = (size_t)((Yg - cielab->display.d_Y0G) / cielab->gstep);
    i = TIFFmin((size_t)cielab->range, i);
    *g = RINT(cielab->Yg2g[i]);

    i = (size_t)((Yb - cielab->display.d_Y0B) / cielab->bstep);
    i = TIFFmin((size_t)cielab->range, i);
    *b = RINT(cielab->Yb2b[i]);

    /* Clip output. */
    *r = TIFFmin(*r, cielab->display.d_Vrwr);
    *g = TIFFmin(*g, cielab->display.d_Vrwg);
    *b = TIFFmin(*b, cielab->display.d_Vrwb);
}
#undef RINT

/*
 * Allocate conversion state structures and make look_up tables for
 * the Yr,Yb,Yg <=> r,g,b conversions.
 */
int TIFFCIELabToRGBInit(TIFFCIELabToRGB *cielab, const TIFFDisplay *display,
                        float *refWhite)
{
    size_t i;
    double dfGamma;

    cielab->range = CIELABTORGB_TABLE_RANGE;

    _TIFFmemcpy(&cielab->display, display, sizeof(TIFFDisplay));

    /* Red */
    dfGamma = 1.0 / cielab->display.d_gammaR;
    cielab->rstep =
        (cielab->display.d_YCR - cielab->display.d_Y0R) / cielab->range;
    for (i = 0; i <= (size_t)cielab->range; i++)
    {
        cielab->Yr2r[i] = cielab->display.d_Vrwr *
                          ((float)pow((double)i / cielab->range, dfGamma));
    }

    /* Green */
    dfGamma = 1.0 / cielab->display.d_gammaG;
    cielab->gstep =
        (cielab->display.d_YCR - cielab->display.d_Y0R) / cielab->range;
    for (i = 0; i <= (size_t)cielab->range; i++)
    {
        cielab->Yg2g[i] = cielab->display.d_Vrwg *
                          ((float)pow((double)i / cielab->range, dfGamma));
    }

    /* Blue */
    dfGamma = 1.0 / cielab->display.d_gammaB;
    cielab->bstep =
        (cielab->display.d_YCR - cielab->display.d_Y0R) / cielab->range;
    for (i = 0; i <= (size_t)cielab->range; i++)
    {
        cielab->Yb2b[i] = cielab->display.d_Vrwb *
                          ((float)pow((double)i / cielab->range, dfGamma));
    }

    /* Init reference white point */
    cielab->X0 = refWhite[0];
    cielab->Y0 = refWhite[1];
    cielab->Z0 = refWhite[2];

    return 0;
}

/*
 * Convert color value from the YCbCr space to RGB.
 * The colorspace conversion algorithm comes from the IJG v5a code;
 * see below for more information on how it works.
 */
#define SHIFT 16
#define FIX(x) ((int32_t)((x) * (1L << SHIFT) + 0.5))
#define ONE_HALF ((int32_t)(1 << (SHIFT - 1)))
#define Code2V(c, RB, RW, CR)                                                  \
    ((((c) - (int32_t)(RB)) * (float)(CR)) /                                   \
     (float)(((RW) - (RB) != 0) ? ((RW) - (RB)) : 1))
/* !((f)>=(min)) written that way to deal with NaN */
#define CLAMP(f, min, max)                                                     \
    ((!((f) >= (min))) ? (min) : (f) > (max) ? (max) : (f))
#define HICLAMP(f, max) ((f) > (max) ? (max) : (f))

void TIFFYCbCrtoRGB(TIFFYCbCrToRGB *ycbcr, uint32_t Y, int32_t Cb, int32_t Cr,
                    uint32_t *r, uint32_t *g, uint32_t *b)
{
    int32_t i;

    /* XXX: Only 8-bit YCbCr input supported for now */
    Y = HICLAMP(Y, 255);
    Cb = CLAMP(Cb, 0, 255);
    Cr = CLAMP(Cr, 0, 255);

    i = ycbcr->Y_tab[Y] + ycbcr->Cr_r_tab[Cr];
    *r = CLAMP(i, 0, 255);
    i = ycbcr->Y_tab[Y] +
        (int)((ycbcr->Cb_g_tab[Cb] + ycbcr->Cr_g_tab[Cr]) >> SHIFT);
    *g = CLAMP(i, 0, 255);
    i = ycbcr->Y_tab[Y] + ycbcr->Cb_b_tab[Cb];
    *b = CLAMP(i, 0, 255);
}

/* Clamp function for sanitization purposes. Normally clamping should not */
/* occur for well behaved chroma and refBlackWhite coefficients */
static float CLAMPw(float v, float vmin, float vmax)
{
    if (v < vmin)
    {
        /* printf("%f clamped to %f\n", v, vmin); */
        return vmin;
    }
    if (v > vmax)
    {
        /* printf("%f clamped to %f\n", v, vmax); */
        return vmax;
    }
    return v;
}

/*
 * Initialize the YCbCr->RGB conversion tables.  The conversion
 * is done according to the 6.0 spec:
 *
 *    R = Y + Cr*(2 - 2*LumaRed)
 *    B = Y + Cb*(2 - 2*LumaBlue)
 *    G =   Y
 *        - LumaBlue*Cb*(2-2*LumaBlue)/LumaGreen
 *        - LumaRed*Cr*(2-2*LumaRed)/LumaGreen
 *
 * To avoid floating point arithmetic the fractional constants that
 * come out of the equations are represented as fixed point values
 * in the range 0...2^16.  We also eliminate multiplications by
 * pre-calculating possible values indexed by Cb and Cr (this code
 * assumes conversion is being done for 8-bit samples).
 */
int TIFFYCbCrToRGBInit(TIFFYCbCrToRGB *ycbcr, float *luma, float *refBlackWhite)
{
    TIFFRGBValue *clamptab;
    int i;

#define LumaRed luma[0]
#define LumaGreen luma[1]
#define LumaBlue luma[2]

    clamptab =
        (TIFFRGBValue *)((uint8_t *)ycbcr +
                         TIFFroundup_32(sizeof(TIFFYCbCrToRGB), sizeof(long)));
    _TIFFmemset(clamptab, 0, 256); /* v < 0 => 0 */
    ycbcr->clamptab = (clamptab += 256);
    for (i = 0; i < 256; i++)
        clamptab[i] = (TIFFRGBValue)i;
    _TIFFmemset(clamptab + 256, 255, 2 * 256); /* v > 255 => 255 */
    ycbcr->Cr_r_tab = (int *)(clamptab + 3 * 256);
    ycbcr->Cb_b_tab = ycbcr->Cr_r_tab + 256;
    ycbcr->Cr_g_tab = (int32_t *)(ycbcr->Cb_b_tab + 256);
    ycbcr->Cb_g_tab = ycbcr->Cr_g_tab + 256;
    ycbcr->Y_tab = ycbcr->Cb_g_tab + 256;

    {
        float f1 = 2 - 2 * LumaRed;
        int32_t D1 = FIX(CLAMP(f1, 0.0F, 2.0F));
        float f2 = LumaRed * f1 / LumaGreen;
        int32_t D2 = -FIX(CLAMP(f2, 0.0F, 2.0F));
        float f3 = 2 - 2 * LumaBlue;
        int32_t D3 = FIX(CLAMP(f3, 0.0F, 2.0F));
        float f4 = LumaBlue * f3 / LumaGreen;
        int32_t D4 = -FIX(CLAMP(f4, 0.0F, 2.0F));
        int x;

#undef LumaBlue
#undef LumaGreen
#undef LumaRed

        /*
         * i is the actual input pixel value in the range 0..255
         * Cb and Cr values are in the range -128..127 (actually
         * they are in a range defined by the ReferenceBlackWhite
         * tag) so there is some range shifting to do here when
         * constructing tables indexed by the raw pixel data.
         */
        for (i = 0, x = -128; i < 256; i++, x++)
        {
            int32_t Cr = (int32_t)CLAMPw(Code2V(x, refBlackWhite[4] - 128.0F,
                                                refBlackWhite[5] - 128.0F, 127),
                                         -128.0F * 32, 128.0F * 32);
            int32_t Cb = (int32_t)CLAMPw(Code2V(x, refBlackWhite[2] - 128.0F,
                                                refBlackWhite[3] - 128.0F, 127),
                                         -128.0F * 32, 128.0F * 32);

            ycbcr->Cr_r_tab[i] = (int32_t)((D1 * Cr + ONE_HALF) >> SHIFT);
            ycbcr->Cb_b_tab[i] = (int32_t)((D3 * Cb + ONE_HALF) >> SHIFT);
            ycbcr->Cr_g_tab[i] = D2 * Cr;
            ycbcr->Cb_g_tab[i] = D4 * Cb + ONE_HALF;
            ycbcr->Y_tab[i] = (int32_t)CLAMPw(
                Code2V(x + 128, refBlackWhite[0], refBlackWhite[1], 255),
                -128.0F * 32, 128.0F * 32);
        }
    }

    return 0;
}
#undef HICLAMP
#undef CLAMP
#undef Code2V
#undef SHIFT
#undef ONE_HALF
#undef FIX
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

- **for()**: A function/method defined in this file
- **RINT()**: A function/method defined in this file
- **LumaBlue()**: A function/method defined in this file
- **LumaRed()**: A function/method defined in this file
- **Code2V()**: A function/method defined in this file
- **HICLAMP()**: A function/method defined in this file
- **ONE_HALF()**: A function/method defined in this file
- **LumaGreen()**: A function/method defined in this file
- **SHIFT()**: A function/method defined in this file
- **FIX()**: A function/method defined in this file
- **CLAMP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `tiffiop.h`
- `math.h`

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

