# Documentation for `docs/3rdparty/libtiff/tif_dirread.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libtiff/tif_dirread.c_docs.md`
- **File Name**: `tif_dirread.c_docs.md`
- **File Size**: 103,621 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libtiff/tif_dirread.c_docs.md](../../../docs/3rdparty/libtiff/tif_dirread.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libtiff` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libtiff/tif_dirread.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_dirread.c`
- **File Name**: `tif_dirread.c`
- **File Size**: 281,847 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_dirread.c](../../3rdparty/libtiff/tif_dirread.c)

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
 * TIFF Library.
 *
 * Directory Read Support Routines.
 */

/* Suggested pending improvements:
 * - add a field 'field_info' to the TIFFDirEntry structure, and set that with
 *   the pointer to the appropriate TIFFField structure early on in
 *   TIFFReadDirectory, so as to eliminate current possibly repetitive lookup.
 */

#include "tiffconf.h"
#include "tiffiop.h"
#include <float.h>
#include <limits.h>
#include <stdlib.h>
#include <string.h>

#define FAILED_FII ((uint32_t)-1)

#ifdef HAVE_IEEEFP
#define TIFFCvtIEEEFloatToNative(tif, n, fp)
#define TIFFCvtIEEEDoubleToNative(tif, n, dp)
#else
/* If your machine does not support IEEE floating point then you will need to
 * add support to tif_machdep.c to convert between the native format and
 * IEEE format. */
extern void TIFFCvtIEEEFloatToNative(TIFF *, uint32_t, float *);
extern void TIFFCvtIEEEDoubleToNative(TIFF *, uint32_t, double *);
#endif

enum TIFFReadDirEntryErr
{
    TIFFReadDirEntryErrOk = 0,
    TIFFReadDirEntryErrCount = 1,
    TIFFReadDirEntryErrType = 2,
    TIFFReadDirEntryErrIo = 3,
    TIFFReadDirEntryErrRange = 4,
    TIFFReadDirEntryErrPsdif = 5,
    TIFFReadDirEntryErrSizesan = 6,
    TIFFReadDirEntryErrAlloc = 7,
};

static enum TIFFReadDirEntryErr
TIFFReadDirEntryByte(TIFF *tif, TIFFDirEntry *direntry, uint8_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySbyte(TIFF *tif, TIFFDirEntry *direntry, int8_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryShort(TIFF *tif, TIFFDirEntry *direntry, uint16_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySshort(TIFF *tif, TIFFDirEntry *direntry, int16_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong(TIFF *tif, TIFFDirEntry *direntry, uint32_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong(TIFF *tif, TIFFDirEntry *direntry, int32_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong8(TIFF *tif, TIFFDirEntry *direntry, uint64_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong8(TIFF *tif, TIFFDirEntry *direntry, int64_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryFloat(TIFF *tif, TIFFDirEntry *direntry, float *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryDouble(TIFF *tif, TIFFDirEntry *direntry, double *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryIfd8(TIFF *tif, TIFFDirEntry *direntry, uint64_t *value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryArray(TIFF *tif, TIFFDirEntry *direntry, uint32_t *count,
                      uint32_t desttypesize, void **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryByteArray(TIFF *tif, TIFFDirEntry *direntry, uint8_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySbyteArray(TIFF *tif, TIFFDirEntry *direntry, int8_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryShortArray(TIFF *tif, TIFFDirEntry *direntry, uint16_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySshortArray(TIFF *tif, TIFFDirEntry *direntry, int16_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryLongArray(TIFF *tif, TIFFDirEntry *direntry, uint32_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlongArray(TIFF *tif, TIFFDirEntry *direntry, int32_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong8Array(TIFF *tif, TIFFDirEntry *direntry, uint64_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong8Array(TIFF *tif, TIFFDirEntry *direntry, int64_t **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryFloatArray(TIFF *tif, TIFFDirEntry *direntry, float **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryDoubleArray(TIFF *tif, TIFFDirEntry *direntry, double **value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryIfd8Array(TIFF *tif, TIFFDirEntry *direntry, uint64_t **value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryPersampleShort(TIFF *tif, TIFFDirEntry *direntry,
                               uint16_t *value);

static void TIFFReadDirEntryCheckedByte(TIFF *tif, TIFFDirEntry *direntry,
                                        uint8_t *value);
static void TIFFReadDirEntryCheckedSbyte(TIFF *tif, TIFFDirEntry *direntry,
                                         int8_t *value);
static void TIFFReadDirEntryCheckedShort(TIFF *tif, TIFFDirEntry *direntry,
                                         uint16_t *value);
static void TIFFReadDirEntryCheckedSshort(TIFF *tif, TIFFDirEntry *direntry,
                                          int16_t *value);
static void TIFFReadDirEntryCheckedLong(TIFF *tif, TIFFDirEntry *direntry,
                                        uint32_t *value);
static void TIFFReadDirEntryCheckedSlong(TIFF *tif, TIFFDirEntry *direntry,
                                         int32_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedLong8(TIFF *tif, TIFFDirEntry *direntry,
                             uint64_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedSlong8(TIFF *tif, TIFFDirEntry *direntry,
                              int64_t *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedRational(TIFF *tif, TIFFDirEntry *direntry,
                                double *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedSrational(TIFF *tif, TIFFDirEntry *direntry,
                                 double *value);
static void TIFFReadDirEntryCheckedFloat(TIFF *tif, TIFFDirEntry *direntry,
                                         float *value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedDouble(TIFF *tif, TIFFDirEntry *direntry, double *value);
#if 0
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckedRationalDirect(TIFF *tif, TIFFDirEntry *direntry,
                                      TIFFRational_t *value);
#endif
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteSbyte(int8_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteShort(uint16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteSshort(int16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteLong(uint32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteSlong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeByteSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteByte(uint8_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteShort(uint16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteSshort(int16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteLong(uint32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteSlong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSbyteSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortSbyte(int8_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortSshort(int16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortLong(uint32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortSlong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeShortSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSshortShort(uint16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSshortLong(uint32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSshortSlong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSshortLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSshortSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLongSbyte(int8_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLongSshort(int16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLongSlong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLongLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLongSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSlongLong(uint32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSlongLong8(uint64_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSlongSlong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLong8Sbyte(int8_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLong8Sshort(int16_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLong8Slong(int32_t value);
static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeLong8Slong8(int64_t value);

static enum TIFFReadDirEntryErr
TIFFReadDirEntryCheckRangeSlong8Long8(uint64_t value);

static enum TIFFReadDirEntryErr TIFFReadDirEntryData(TIFF *tif, uint64_t offset,
                                                     tmsize_t size, void *dest);
static void TIFFReadDirEntryOutputErr(TIFF *tif, enum TIFFReadDirEntryErr err,
                                      const char *module, const char *tagname,
                                      int recover);

static void TIFFReadDirectoryCheckOrder(TIFF *tif, TIFFDirEntry *dir,
                                        uint16_t dircount);
static TIFFDirEntry *TIFFReadDirectoryFindEntry(TIFF *tif, TIFFDirEntry *dir,
                                                uint16_t dircount,
                                                uint16_t tagid);
static void TIFFReadDirectoryFindFieldInfo(TIFF *tif, uint16_t tagid,
                                           uint32_t *fii);

static int EstimateStripByteCounts(TIFF *tif, TIFFDirEntry *dir,
                                   uint16_t dircount);
static void MissingRequired(TIFF *, const char *);
static int CheckDirCount(TIFF *, TIFFDirEntry *, uint32_t);
static uint16_t TIFFFetchDirectory(TIFF *tif, uint64_t diroff,
                                   TIFFDirEntry **pdir, uint64_t *nextdiroff);
static int TIFFFetchNormalTag(TIFF *, TIFFDirEntry *, int recover);
static int TIFFFetchStripThing(TIFF *tif, TIFFDirEntry *dir, uint32_t nstrips,
                               uint64_t **lpp);
static int TIFFFetchSubjectDistance(TIFF *, TIFFDirEntry *);
static void ChopUpSingleUncompressedStrip(TIFF *);
static void TryChopUpUncompressedBigTiff(TIFF *);
static uint64_t TIFFReadUInt64(const uint8_t *value);
static int _TIFFGetMaxColorChannels(uint16_t photometric);

static int _TIFFFillStrilesInternal(TIFF *tif, int loadStripByteCount);

typedef union _UInt64Aligned_t
{
    double d;
    uint64_t l;
    uint32_t i[2];
    uint16_t s[4];
    uint8_t c[8];
} UInt64Aligned_t;

/*
  Unaligned safe copy of a uint64_t value from an octet array.
*/
static uint64_t TIFFReadUInt64(const uint8_t *value)
{
    UInt64Aligned_t result;

    result.c[0] = value[0];
    result.c[1] = value[1];
    result.c[2] = value[2];
    result.c[3] = value[3];
    result.c[4] = value[4];
    result.c[5] = value[5];
    result.c[6] = value[6];
    result.c[7] = value[7];

    return result.l;
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryByte(TIFF *tif, TIFFDirEntry *direntry, uint8_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_UNDEFINED: /* Support to read TIFF_UNDEFINED with
                                field_readcount==1 */
            TIFFReadDirEntryCheckedByte(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeByteSbyte(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeByteShort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeByteSshort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeByteLong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeByteSlong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeByteLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeByteSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySbyte(TIFF *tif, TIFFDirEntry *direntry, int8_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_UNDEFINED: /* Support to read TIFF_UNDEFINED with
                                field_readcount==1 */
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSbyteByte(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            TIFFReadDirEntryCheckedSbyte(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSbyteShort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSbyteSshort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSbyteLong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSbyteSlong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSbyteLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSbyteSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int8_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntrySbyte() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntryShort(TIFF *tif, TIFFDirEntry *direntry, uint16_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeShortSbyte(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
            TIFFReadDirEntryCheckedShort(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeShortSshort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeShortLong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeShortSlong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeShortLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeShortSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntryShort() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySshort(TIFF *tif, TIFFDirEntry *direntry, int16_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSshortShort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
            TIFFReadDirEntryCheckedSshort(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSshortLong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSshortSlong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSshortLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSshortSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int16_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntrySshort() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong(TIFF *tif, TIFFDirEntry *direntry, uint32_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLongSbyte(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLongSshort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
            TIFFReadDirEntryCheckedLong(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLongSlong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeLongLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeLongSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntryLong() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong(TIFF *tif, TIFFDirEntry *direntry, int32_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeSlongLong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
            TIFFReadDirEntryCheckedSlong(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSlongLong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSlongSlong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int32_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntrySlong() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong8(TIFF *tif, TIFFDirEntry *direntry, uint64_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLong8Sbyte(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLong8Sshort(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            err = TIFFReadDirEntryCheckRangeLong8Slong(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, value);
            return (err);
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeLong8Slong8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntryLong8() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong8(TIFF *tif, TIFFDirEntry *direntry, int64_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            err = TIFFReadDirEntryCheckRangeSlong8Long8(m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (int64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, value);
            return (err);
        default:
            return (TIFFReadDirEntryErrType);
    }
} /*-- TIFFReadDirEntrySlong8() --*/

static enum TIFFReadDirEntryErr
TIFFReadDirEntryFloat(TIFF *tif, TIFFDirEntry *direntry, float *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_RATIONAL:
        {
            double m;
            err = TIFFReadDirEntryCheckedRational(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SRATIONAL:
        {
            double m;
            err = TIFFReadDirEntryCheckedSrational(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_FLOAT:
            TIFFReadDirEntryCheckedFloat(tif, direntry, value);
            return (TIFFReadDirEntryErrOk);
        case TIFF_DOUBLE:
        {
            double m;
            err = TIFFReadDirEntryCheckedDouble(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            if ((m > FLT_MAX) || (m < -FLT_MAX))
                return (TIFFReadDirEntryErrRange);
            *value = (float)m;
            return (TIFFReadDirEntryErrOk);
        }
        default:
            return (TIFFReadDirEntryErrType);
    }
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryDouble(TIFF *tif, TIFFDirEntry *direntry, double *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t m;
            TIFFReadDirEntryCheckedByte(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
        {
            int8_t m;
            TIFFReadDirEntryCheckedSbyte(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SHORT:
        {
            uint16_t m;
            TIFFReadDirEntryCheckedShort(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
        {
            int16_t m;
            TIFFReadDirEntryCheckedSshort(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
        {
            int32_t m;
            TIFFReadDirEntryCheckedSlong(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        {
            uint64_t m;
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
        {
            int64_t m;
            err = TIFFReadDirEntryCheckedSlong8(tif, direntry, &m);
            if (err != TIFFReadDirEntryErrOk)
                return (err);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_RATIONAL:
            err = TIFFReadDirEntryCheckedRational(tif, direntry, value);
            return (err);
        case TIFF_SRATIONAL:
            err = TIFFReadDirEntryCheckedSrational(tif, direntry, value);
            return (err);
        case TIFF_FLOAT:
        {
            float m;
            TIFFReadDirEntryCheckedFloat(tif, direntry, &m);
            *value = (double)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_DOUBLE:
            err = TIFFReadDirEntryCheckedDouble(tif, direntry, value);
            return (err);
        default:
            return (TIFFReadDirEntryErrType);
    }
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryIfd8(TIFF *tif, TIFFDirEntry *direntry, uint64_t *value)
{
    enum TIFFReadDirEntryErr err;
    if (direntry->tdir_count != 1)
        return (TIFFReadDirEntryErrCount);
    switch (direntry->tdir_type)
    {
        case TIFF_LONG:
        case TIFF_IFD:
        {
            uint32_t m;
            TIFFReadDirEntryCheckedLong(tif, direntry, &m);
            *value = (uint64_t)m;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_LONG8:
        case TIFF_IFD8:
            err = TIFFReadDirEntryCheckedLong8(tif, direntry, value);
            return (err);
        default:
            return (TIFFReadDirEntryErrType);
    }
}

#define INITIAL_THRESHOLD (1024 * 1024)
#define THRESHOLD_MULTIPLIER 10
#define MAX_THRESHOLD                                                          \
    (THRESHOLD_MULTIPLIER * THRESHOLD_MULTIPLIER * THRESHOLD_MULTIPLIER *      \
     INITIAL_THRESHOLD)

static enum TIFFReadDirEntryErr TIFFReadDirEntryDataAndRealloc(TIFF *tif,
                                                               uint64_t offset,
                                                               tmsize_t size,
                                                               void **pdest)
{
#if SIZEOF_SIZE_T == 8
    tmsize_t threshold = INITIAL_THRESHOLD;
#endif
    tmsize_t already_read = 0;

    assert(!isMapped(tif));

    if (!SeekOK(tif, offset))
        return (TIFFReadDirEntryErrIo);

    /* On 64 bit processes, read first a maximum of 1 MB, then 10 MB, etc */
    /* so as to avoid allocating too much memory in case the file is too */
    /* short. We could ask for the file size, but this might be */
    /* expensive with some I/O layers (think of reading a gzipped file) */
    /* Restrict to 64 bit processes, so as to avoid reallocs() */
    /* on 32 bit processes where virtual memory is scarce.  */
    while (already_read < size)
    {
        void *new_dest;
        tmsize_t bytes_read;
        tmsize_t to_read = size - already_read;
#if SIZEOF_SIZE_T == 8
        if (to_read >= threshold && threshold < MAX_THRESHOLD)
        {
            to_read = threshold;
            threshold *= THRESHOLD_MULTIPLIER;
        }
#endif

        new_dest =
            (uint8_t *)_TIFFreallocExt(tif, *pdest, already_read + to_read);
        if (new_dest == NULL)
        {
            TIFFErrorExtR(tif, tif->tif_name,
                          "Failed to allocate memory for %s "
                          "(%" TIFF_SSIZE_FORMAT
                          " elements of %" TIFF_SSIZE_FORMAT " bytes each)",
                          "TIFFReadDirEntryArray", (tmsize_t)1,
                          already_read + to_read);
            return TIFFReadDirEntryErrAlloc;
        }
        *pdest = new_dest;

        bytes_read = TIFFReadFile(tif, (char *)*pdest + already_read, to_read);
        already_read += bytes_read;
        if (bytes_read != to_read)
        {
            return TIFFReadDirEntryErrIo;
        }
    }
    return TIFFReadDirEntryErrOk;
}

/* Caution: if raising that value, make sure int32 / uint32 overflows can't
 * occur elsewhere */
#define MAX_SIZE_TAG_DATA 2147483647U

static enum TIFFReadDirEntryErr
TIFFReadDirEntryArrayWithLimit(TIFF *tif, TIFFDirEntry *direntry,
                               uint32_t *count, uint32_t desttypesize,
                               void **value, uint64_t maxcount)
{
    int typesize;
    uint32_t datasize;
    void *data;
    uint64_t target_count64;
    int original_datasize_clamped;
    typesize = TIFFDataWidth(direntry->tdir_type);

    target_count64 =
        (direntry->tdir_count > maxcount) ? maxcount : direntry->tdir_count;

    if ((target_count64 == 0) || (typesize == 0))
    {
        *value = 0;
        return (TIFFReadDirEntryErrOk);
    }
    (void)desttypesize;

    /* We just want to know if the original tag size is more than 4 bytes
     * (classic TIFF) or 8 bytes (BigTIFF)
     */
    original_datasize_clamped =
        ((direntry->tdir_count > 10) ? 10 : (int)direntry->tdir_count) *
        typesize;

    /*
     * As a sanity check, make sure we have no more than a 2GB tag array
     * in either the current data type or the dest data type.  This also
     * avoids problems with overflow of tmsize_t on 32bit systems.
     */
    if ((uint64_t)(MAX_SIZE_TAG_DATA / typesize) < target_count64)
        return (TIFFReadDirEntryErrSizesan);
    if ((uint64_t)(MAX_SIZE_TAG_DATA / desttypesize) < target_count64)
        return (TIFFReadDirEntryErrSizesan);

    *count = (uint32_t)target_count64;
    datasize = (*count) * typesize;
    assert((tmsize_t)datasize > 0);

    if (datasize > 100 * 1024 * 1024)
    {
        /* Before allocating a huge amount of memory for corrupted files, check
         * if size of requested memory is not greater than file size.
         */
        const uint64_t filesize = TIFFGetFileSize(tif);
        if (datasize > filesize)
        {
            TIFFWarningExtR(tif, "ReadDirEntryArray",
                            "Requested memory size for tag %d (0x%x) %" PRIu32
                            " is greater than filesize %" PRIu64
                            ". Memory not allocated, tag not read",
                            direntry->tdir_tag, direntry->tdir_tag, datasize,
                            filesize);
            return (TIFFReadDirEntryErrAlloc);
        }
    }

    if (isMapped(tif) && datasize > (uint64_t)tif->tif_size)
        return TIFFReadDirEntryErrIo;

    if (!isMapped(tif) && (((tif->tif_flags & TIFF_BIGTIFF) && datasize > 8) ||
                           (!(tif->tif_flags & TIFF_BIGTIFF) && datasize > 4)))
    {
        data = NULL;
    }
    else
    {
        data = _TIFFCheckMalloc(tif, *count, typesize, "ReadDirEntryArray");
        if (data == 0)
            return (TIFFReadDirEntryErrAlloc);
    }
    if (!(tif->tif_flags & TIFF_BIGTIFF))
    {
        /* Only the condition on original_datasize_clamped. The second
         * one is implied, but Coverity Scan cannot see it. */
        if (original_datasize_clamped <= 4 && datasize <= 4)
            _TIFFmemcpy(data, &direntry->tdir_offset, datasize);
        else
        {
            enum TIFFReadDirEntryErr err;
            uint32_t offset = direntry->tdir_offset.toff_long;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabLong(&offset);
            if (isMapped(tif))
                err = TIFFReadDirEntryData(tif, (uint64_t)offset,
                                           (tmsize_t)datasize, data);
            else
                err = TIFFReadDirEntryDataAndRealloc(tif, (uint64_t)offset,
                                                     (tmsize_t)datasize, &data);
            if (err != TIFFReadDirEntryErrOk)
            {
                _TIFFfreeExt(tif, data);
                return (err);
            }
        }
    }
    else
    {
        /* See above comment for the Classic TIFF case */
        if (original_datasize_clamped <= 8 && datasize <= 8)
            _TIFFmemcpy(data, &direntry->tdir_offset, datasize);
        else
        {
            enum TIFFReadDirEntryErr err;
            uint64_t offset = direntry->tdir_offset.toff_long8;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabLong8(&offset);
            if (isMapped(tif))
                err = TIFFReadDirEntryData(tif, (uint64_t)offset,
                                           (tmsize_t)datasize, data);
            else
                err = TIFFReadDirEntryDataAndRealloc(tif, (uint64_t)offset,
                                                     (tmsize_t)datasize, &data);
            if (err != TIFFReadDirEntryErrOk)
            {
                _TIFFfreeExt(tif, data);
                return (err);
            }
        }
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryArray(TIFF *tif, TIFFDirEntry *direntry, uint32_t *count,
                      uint32_t desttypesize, void **value)
{
    return TIFFReadDirEntryArrayWithLimit(tif, direntry, count, desttypesize,
                                          value, ~((uint64_t)0));
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryByteArray(TIFF *tif, TIFFDirEntry *direntry, uint8_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    uint8_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_ASCII:
        case TIFF_UNDEFINED:
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 1, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_ASCII:
        case TIFF_UNDEFINED:
        case TIFF_BYTE:
            *value = (uint8_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        case TIFF_SBYTE:
        {
            int8_t *m;
            uint32_t n;
            m = (int8_t *)origdata;
            for (n = 0; n < count; n++)
            {
                err = TIFFReadDirEntryCheckRangeByteSbyte(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (uint8_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
    }
    data = (uint8_t *)_TIFFmallocExt(tif, count);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_SHORT:
        {
            uint16_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                err = TIFFReadDirEntryCheckRangeByteShort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                err = TIFFReadDirEntryCheckRangeByteSshort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                err = TIFFReadDirEntryCheckRangeByteLong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                err = TIFFReadDirEntryCheckRangeByteSlong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeByteLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            uint8_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeByteSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint8_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySbyteArray(TIFF *tif, TIFFDirEntry *direntry, int8_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    int8_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_UNDEFINED:
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 1, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_UNDEFINED:
        case TIFF_BYTE:
        {
            uint8_t *m;
            uint32_t n;
            m = (uint8_t *)origdata;
            for (n = 0; n < count; n++)
            {
                err = TIFFReadDirEntryCheckRangeSbyteByte(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (int8_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SBYTE:
            *value = (int8_t *)origdata;
            return (TIFFReadDirEntryErrOk);
    }
    data = (int8_t *)_TIFFmallocExt(tif, count);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_SHORT:
        {
            uint16_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                err = TIFFReadDirEntryCheckRangeSbyteShort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                err = TIFFReadDirEntryCheckRangeSbyteSshort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                err = TIFFReadDirEntryCheckRangeSbyteLong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                err = TIFFReadDirEntryCheckRangeSbyteSlong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeSbyteLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            int8_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeSbyteSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int8_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryShortArray(TIFF *tif, TIFFDirEntry *direntry, uint16_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    uint16_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 2, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_SHORT:
            *value = (uint16_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfShort(*value, count);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SSHORT:
        {
            int16_t *m;
            uint32_t n;
            m = (int16_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)m);
                err = TIFFReadDirEntryCheckRangeShortSshort(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (uint16_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
    }
    data = (uint16_t *)_TIFFmallocExt(tif, count * 2);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (uint16_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                err = TIFFReadDirEntryCheckRangeShortSbyte(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint16_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                err = TIFFReadDirEntryCheckRangeShortLong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint16_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                err = TIFFReadDirEntryCheckRangeShortSlong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint16_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeShortLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint16_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            uint16_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeShortSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint16_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySshortArray(TIFF *tif, TIFFDirEntry *direntry, int16_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    int16_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 2, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_SHORT:
        {
            uint16_t *m;
            uint32_t n;
            m = (uint16_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(m);
                err = TIFFReadDirEntryCheckRangeSshortShort(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (int16_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SSHORT:
            *value = (int16_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfShort((uint16_t *)(*value), count);
            return (TIFFReadDirEntryErrOk);
    }
    data = (int16_t *)_TIFFmallocExt(tif, count * 2);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int16_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int16_t)(*ma++);
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                err = TIFFReadDirEntryCheckRangeSshortLong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int16_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                err = TIFFReadDirEntryCheckRangeSshortSlong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int16_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeSshortLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int16_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            int16_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeSshortSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int16_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryLongArray(TIFF *tif, TIFFDirEntry *direntry, uint32_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    uint32_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 4, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_LONG:
            *value = (uint32_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong(*value, count);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SLONG:
        {
            int32_t *m;
            uint32_t n;
            m = (int32_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)m);
                err = TIFFReadDirEntryCheckRangeLongSlong(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (uint32_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
    }
    data = (uint32_t *)_TIFFmallocExt(tif, count * 4);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (uint32_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                err = TIFFReadDirEntryCheckRangeLongSbyte(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint32_t)(*ma++);
            }
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (uint32_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                err = TIFFReadDirEntryCheckRangeLongSshort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint32_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeLongLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint32_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            uint32_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeLongSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint32_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlongArray(TIFF *tif, TIFFDirEntry *direntry, int32_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    int32_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 4, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_LONG:
        {
            uint32_t *m;
            uint32_t n;
            m = (uint32_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)m);
                err = TIFFReadDirEntryCheckRangeSlongLong(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (int32_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG:
            *value = (int32_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong((uint32_t *)(*value), count);
            return (TIFFReadDirEntryErrOk);
    }
    data = (int32_t *)_TIFFmallocExt(tif, count * 4);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int32_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int32_t)(*ma++);
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (int32_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                *mb++ = (int32_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                err = TIFFReadDirEntryCheckRangeSlongLong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int32_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            int32_t *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                err = TIFFReadDirEntryCheckRangeSlongSlong8(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (int32_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong8ArrayWithLimit(TIFF *tif, TIFFDirEntry *direntry,
                                    uint64_t **value, uint64_t maxcount)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    uint64_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArrayWithLimit(tif, direntry, &count, 8, &origdata,
                                         maxcount);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_LONG8:
            *value = (uint64_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong8(*value, count);
            return (TIFFReadDirEntryErrOk);
        case TIFF_SLONG8:
        {
            int64_t *m;
            uint32_t n;
            m = (int64_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)m);
                err = TIFFReadDirEntryCheckRangeLong8Slong8(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (uint64_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
    }
    data = (uint64_t *)_TIFFmallocExt(tif, count * 8);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (uint64_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                err = TIFFReadDirEntryCheckRangeLong8Sbyte(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint64_t)(*ma++);
            }
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (uint64_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                err = TIFFReadDirEntryCheckRangeLong8Sshort(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint64_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                *mb++ = (uint64_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            uint64_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                err = TIFFReadDirEntryCheckRangeLong8Slong(*ma);
                if (err != TIFFReadDirEntryErrOk)
                    break;
                *mb++ = (uint64_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    if (err != TIFFReadDirEntryErrOk)
    {
        _TIFFfreeExt(tif, data);
        return (err);
    }
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryLong8Array(TIFF *tif, TIFFDirEntry *direntry, uint64_t **value)
{
    return TIFFReadDirEntryLong8ArrayWithLimit(tif, direntry, value,
                                               ~((uint64_t)0));
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntrySlong8Array(TIFF *tif, TIFFDirEntry *direntry, int64_t **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    int64_t *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 8, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_LONG8:
        {
            uint64_t *m;
            uint32_t n;
            m = (uint64_t *)origdata;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(m);
                err = TIFFReadDirEntryCheckRangeSlong8Long8(*m);
                if (err != TIFFReadDirEntryErrOk)
                {
                    _TIFFfreeExt(tif, origdata);
                    return (err);
                }
                m++;
            }
            *value = (int64_t *)origdata;
            return (TIFFReadDirEntryErrOk);
        }
        case TIFF_SLONG8:
            *value = (int64_t *)origdata;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong8((uint64_t *)(*value), count);
            return (TIFFReadDirEntryErrOk);
    }
    data = (int64_t *)_TIFFmallocExt(tif, count * 8);
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int64_t)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (int64_t)(*ma++);
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (int64_t)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                *mb++ = (int64_t)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                *mb++ = (int64_t)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            int64_t *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                *mb++ = (int64_t)(*ma++);
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryFloatArray(TIFF *tif, TIFFDirEntry *direntry, float **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    float *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
        case TIFF_RATIONAL:
        case TIFF_SRATIONAL:
        case TIFF_FLOAT:
        case TIFF_DOUBLE:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 4, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_FLOAT:
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong((uint32_t *)origdata, count);
            TIFFCvtIEEEFloatToNative(tif, count, (float *)origdata);
            *value = (float *)origdata;
            return (TIFFReadDirEntryErrOk);
    }
    data = (float *)_TIFFmallocExt(tif, count * sizeof(float));
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            float *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (float)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            float *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (float)(*ma++);
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            float *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            float *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            float *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            float *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            float *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            float *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                *mb++ = (float)(*ma++);
            }
        }
        break;
        case TIFF_RATIONAL:
        {
            uint32_t *ma;
            uint32_t maa;
            uint32_t mab;
            float *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                maa = *ma++;
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                mab = *ma++;
                if (mab == 0)
                    *mb++ = 0.0;
                else
                    *mb++ = (float)maa / (float)mab;
            }
        }
        break;
        case TIFF_SRATIONAL:
        {
            uint32_t *ma;
            int32_t maa;
            uint32_t mab;
            float *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                maa = *(int32_t *)ma;
                ma++;
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                mab = *ma++;
                if (mab == 0)
                    *mb++ = 0.0;
                else
                    *mb++ = (float)maa / (float)mab;
            }
        }
        break;
        case TIFF_DOUBLE:
        {
            double *ma;
            float *mb;
            uint32_t n;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong8((uint64_t *)origdata, count);
            TIFFCvtIEEEDoubleToNative(tif, count, (double *)origdata);
            ma = (double *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                double val = *ma++;
                if (val > FLT_MAX)
                    val = FLT_MAX;
                else if (val < -FLT_MAX)
                    val = -FLT_MAX;
                *mb++ = (float)val;
            }
        }
        break;
    }
    _TIFFfreeExt(tif, origdata);
    *value = data;
    return (TIFFReadDirEntryErrOk);
}

static enum TIFFReadDirEntryErr
TIFFReadDirEntryDoubleArray(TIFF *tif, TIFFDirEntry *direntry, double **value)
{
    enum TIFFReadDirEntryErr err;
    uint32_t count;
    void *origdata;
    double *data;
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        case TIFF_SBYTE:
        case TIFF_SHORT:
        case TIFF_SSHORT:
        case TIFF_LONG:
        case TIFF_SLONG:
        case TIFF_LONG8:
        case TIFF_SLONG8:
        case TIFF_RATIONAL:
        case TIFF_SRATIONAL:
        case TIFF_FLOAT:
        case TIFF_DOUBLE:
            break;
        default:
            return (TIFFReadDirEntryErrType);
    }
    err = TIFFReadDirEntryArray(tif, direntry, &count, 8, &origdata);
    if ((err != TIFFReadDirEntryErrOk) || (origdata == 0))
    {
        *value = 0;
        return (err);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_DOUBLE:
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong8((uint64_t *)origdata, count);
            TIFFCvtIEEEDoubleToNative(tif, count, (double *)origdata);
            *value = (double *)origdata;
            return (TIFFReadDirEntryErrOk);
    }
    data = (double *)_TIFFmallocExt(tif, count * sizeof(double));
    if (data == 0)
    {
        _TIFFfreeExt(tif, origdata);
        return (TIFFReadDirEntryErrAlloc);
    }
    switch (direntry->tdir_type)
    {
        case TIFF_BYTE:
        {
            uint8_t *ma;
            double *mb;
            uint32_t n;
            ma = (uint8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (double)(*ma++);
        }
        break;
        case TIFF_SBYTE:
        {
            int8_t *ma;
            double *mb;
            uint32_t n;
            ma = (int8_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (double)(*ma++);
        }
        break;
        case TIFF_SHORT:
        {
            uint16_t *ma;
            double *mb;
            uint32_t n;
            ma = (uint16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_SSHORT:
        {
            int16_t *ma;
            double *mb;
            uint32_t n;
            ma = (int16_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort((uint16_t *)ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_LONG:
        {
            uint32_t *ma;
            double *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_SLONG:
        {
            int32_t *ma;
            double *mb;
            uint32_t n;
            ma = (int32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong((uint32_t *)ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_LONG8:
        {
            uint64_t *ma;
            double *mb;
            uint32_t n;
            ma = (uint64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_SLONG8:
        {
            int64_t *ma;
            double *mb;
            uint32_t n;
            ma = (int64_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8((uint64_t *)ma);
                *mb++ = (double)(*ma++);
            }
        }
        break;
        case TIFF_RATIONAL:
        {
            uint32_t *ma;
            uint32_t maa;
            uint32_t mab;
            double *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                maa = *ma++;
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                mab = *ma++;
                if (mab == 0)
                    *mb++ = 0.0;
                else
                    *mb++ = (double)maa / (double)mab;
            }
        }
        break;
        case TIFF_SRATIONAL:
        {
            uint32_t *ma;
            int32_t maa;
            uint32_t mab;
            double *mb;
            uint32_t n;
            ma = (uint32_t *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
            {
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                maa = *(int32_t *)ma;
                ma++;
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(ma);
                mab = *ma++;
                if (mab == 0)
                    *mb++ = 0.0;
                else
                    *mb++ = (double)maa / (double)mab;
            }
        }
        break;
        case TIFF_FLOAT:
        {
            float *ma;
            double *mb;
            uint32_t n;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabArrayOfLong((uint32_t *)origdata, count);
            TIFFCvtIEEEFloatToNative(tif, count, (float *)origdata);
            ma = (float *)origdata;
            mb = data;
            for (n = 0; n < count; n++)
                *mb++ = (double)(*ma++);
        }
        break;
    }
    _TIFFfreeExt(tif, origd
... [truncated]

## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

