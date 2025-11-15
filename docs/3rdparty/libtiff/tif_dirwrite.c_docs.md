# Documentation for `3rdparty/libtiff/tif_dirwrite.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_dirwrite.c`
- **File Name**: `tif_dirwrite.c`
- **File Size**: 149,688 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_dirwrite.c](../../3rdparty/libtiff/tif_dirwrite.c)

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
 * Directory Write Support Routines.
 */
#include "tiffiop.h"
#include <float.h> /*--: for Rational2Double */
#include <math.h>  /*--: for Rational2Double */

#ifdef HAVE_IEEEFP
#define TIFFCvtNativeToIEEEFloat(tif, n, fp)
#define TIFFCvtNativeToIEEEDouble(tif, n, dp)
#else
/* If your machine does not support IEEE floating point then you will need to
 * add support to tif_machdep.c to convert between the native format and
 * IEEE format. */
extern void TIFFCvtNativeToIEEEFloat(TIFF *tif, uint32_t n, float *fp);
extern void TIFFCvtNativeToIEEEDouble(TIFF *tif, uint32_t n, double *dp);
#endif

static int TIFFWriteDirectorySec(TIFF *tif, int isimage, int imagedone,
                                 uint64_t *pdiroff);

static int TIFFWriteDirectoryTagSampleformatArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  double *value);

static int TIFFWriteDirectoryTagAscii(TIFF *tif, uint32_t *ndir,
                                      TIFFDirEntry *dir, uint16_t tag,
                                      uint32_t count, char *value);
static int TIFFWriteDirectoryTagUndefinedArray(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, uint8_t *value);
static int TIFFWriteDirectoryTagByteArray(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t count, uint8_t *value);
static int TIFFWriteDirectoryTagSbyteArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, int8_t *value);
static int TIFFWriteDirectoryTagShort(TIFF *tif, uint32_t *ndir,
                                      TIFFDirEntry *dir, uint16_t tag,
                                      uint16_t value);
static int TIFFWriteDirectoryTagShortArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, uint16_t *value);
static int TIFFWriteDirectoryTagShortPerSample(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint16_t value);
static int TIFFWriteDirectoryTagSshortArray(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, int16_t *value);
static int TIFFWriteDirectoryTagLong(TIFF *tif, uint32_t *ndir,
                                     TIFFDirEntry *dir, uint16_t tag,
                                     uint32_t value);
static int TIFFWriteDirectoryTagLongArray(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t count, uint32_t *value);
static int TIFFWriteDirectoryTagSlongArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, int32_t *value);
static int TIFFWriteDirectoryTagLong8Array(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, uint64_t *value);
static int TIFFWriteDirectoryTagSlong8Array(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, int64_t *value);
static int TIFFWriteDirectoryTagRational(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir, uint16_t tag,
                                         double value);
static int TIFFWriteDirectoryTagRationalArray(TIFF *tif, uint32_t *ndir,
                                              TIFFDirEntry *dir, uint16_t tag,
                                              uint32_t count, float *value);
static int TIFFWriteDirectoryTagSrationalArray(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, float *value);
static int TIFFWriteDirectoryTagFloatArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, float *value);
static int TIFFWriteDirectoryTagDoubleArray(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, double *value);
static int TIFFWriteDirectoryTagIfdArray(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir, uint16_t tag,
                                         uint32_t count, uint32_t *value);
static int TIFFWriteDirectoryTagShortLong(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t value);
static int TIFFWriteDirectoryTagLongLong8Array(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, uint64_t *value);
static int TIFFWriteDirectoryTagIfdIfd8Array(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint32_t count, uint64_t *value);
static int TIFFWriteDirectoryTagColormap(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir);
static int TIFFWriteDirectoryTagTransferfunction(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir);
static int TIFFWriteDirectoryTagSubifd(TIFF *tif, uint32_t *ndir,
                                       TIFFDirEntry *dir);

static int TIFFWriteDirectoryTagCheckedAscii(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint32_t count, char *value);
static int TIFFWriteDirectoryTagCheckedUndefinedArray(TIFF *tif, uint32_t *ndir,
                                                      TIFFDirEntry *dir,
                                                      uint16_t tag,
                                                      uint32_t count,
                                                      uint8_t *value);
static int TIFFWriteDirectoryTagCheckedByteArray(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir,
                                                 uint16_t tag, uint32_t count,
                                                 uint8_t *value);
static int TIFFWriteDirectoryTagCheckedSbyteArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  int8_t *value);
static int TIFFWriteDirectoryTagCheckedShort(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint16_t value);
static int TIFFWriteDirectoryTagCheckedShortArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  uint16_t *value);
static int TIFFWriteDirectoryTagCheckedSshortArray(TIFF *tif, uint32_t *ndir,
                                                   TIFFDirEntry *dir,
                                                   uint16_t tag, uint32_t count,
                                                   int16_t *value);
static int TIFFWriteDirectoryTagCheckedLong(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t value);
static int TIFFWriteDirectoryTagCheckedLongArray(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir,
                                                 uint16_t tag, uint32_t count,
                                                 uint32_t *value);
static int TIFFWriteDirectoryTagCheckedSlongArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  int32_t *value);
static int TIFFWriteDirectoryTagCheckedLong8Array(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  uint64_t *value);
static int TIFFWriteDirectoryTagCheckedSlong8Array(TIFF *tif, uint32_t *ndir,
                                                   TIFFDirEntry *dir,
                                                   uint16_t tag, uint32_t count,
                                                   int64_t *value);
static int TIFFWriteDirectoryTagCheckedRational(TIFF *tif, uint32_t *ndir,
                                                TIFFDirEntry *dir, uint16_t tag,
                                                double value);
static int TIFFWriteDirectoryTagCheckedRationalArray(TIFF *tif, uint32_t *ndir,
                                                     TIFFDirEntry *dir,
                                                     uint16_t tag,
                                                     uint32_t count,
                                                     float *value);
static int TIFFWriteDirectoryTagCheckedSrationalArray(TIFF *tif, uint32_t *ndir,
                                                      TIFFDirEntry *dir,
                                                      uint16_t tag,
                                                      uint32_t count,
                                                      float *value);

/*--: Rational2Double: New functions to support true double-precision for custom
 * rational tag types. */
static int TIFFWriteDirectoryTagRationalDoubleArray(TIFF *tif, uint32_t *ndir,
                                                    TIFFDirEntry *dir,
                                                    uint16_t tag,
                                                    uint32_t count,
                                                    double *value);
static int TIFFWriteDirectoryTagSrationalDoubleArray(TIFF *tif, uint32_t *ndir,
                                                     TIFFDirEntry *dir,
                                                     uint16_t tag,
                                                     uint32_t count,
                                                     double *value);
static int
TIFFWriteDirectoryTagCheckedRationalDoubleArray(TIFF *tif, uint32_t *ndir,
                                                TIFFDirEntry *dir, uint16_t tag,
                                                uint32_t count, double *value);
static int TIFFWriteDirectoryTagCheckedSrationalDoubleArray(
    TIFF *tif, uint32_t *ndir, TIFFDirEntry *dir, uint16_t tag, uint32_t count,
    double *value);
static void DoubleToRational(double value, uint32_t *num, uint32_t *denom);
static void DoubleToSrational(double value, int32_t *num, int32_t *denom);

static int TIFFWriteDirectoryTagCheckedFloatArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  float *value);
static int TIFFWriteDirectoryTagCheckedDoubleArray(TIFF *tif, uint32_t *ndir,
                                                   TIFFDirEntry *dir,
                                                   uint16_t tag, uint32_t count,
                                                   double *value);
static int TIFFWriteDirectoryTagCheckedIfdArray(TIFF *tif, uint32_t *ndir,
                                                TIFFDirEntry *dir, uint16_t tag,
                                                uint32_t count,
                                                uint32_t *value);
static int TIFFWriteDirectoryTagCheckedIfd8Array(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir,
                                                 uint16_t tag, uint32_t count,
                                                 uint64_t *value);

static int TIFFWriteDirectoryTagData(TIFF *tif, uint32_t *ndir,
                                     TIFFDirEntry *dir, uint16_t tag,
                                     uint16_t datatype, uint32_t count,
                                     uint32_t datalength, void *data);

static int TIFFLinkDirectory(TIFF *);

/*
 * Write the contents of the current directory
 * to the specified file.  This routine doesn't
 * handle overwriting a directory with auxiliary
 * storage that's been changed.
 */
int TIFFWriteDirectory(TIFF *tif)
{
    return TIFFWriteDirectorySec(tif, TRUE, TRUE, NULL);
}

/*
 * This is an advanced writing function that must be used in a particular
 * sequence, and generally together with TIFFForceStrileArrayWriting(),
 * to make its intended effect. Its aim is to modify the location
 * where the [Strip/Tile][Offsets/ByteCounts] arrays are located in the file.
 * More precisely, when TIFFWriteCheck() will be called, the tag entries for
 * those arrays will be written with type = count = offset = 0 as a temporary
 * value.
 *
 * Its effect is only valid for the current directory, and before
 * TIFFWriteDirectory() is first called, and  will be reset when
 * changing directory.
 *
 * The typical sequence of calls is:
 * TIFFOpen()
 * [ TIFFCreateDirectory(tif) ]
 * Set fields with calls to TIFFSetField(tif, ...)
 * TIFFDeferStrileArrayWriting(tif)
 * TIFFWriteCheck(tif, ...)
 * TIFFWriteDirectory(tif)
 * ... potentially create other directories and come back to the above directory
 * TIFFForceStrileArrayWriting(tif): emit the arrays at the end of file
 *
 * Returns 1 in case of success, 0 otherwise.
 */
int TIFFDeferStrileArrayWriting(TIFF *tif)
{
    static const char module[] = "TIFFDeferStrileArrayWriting";
    if (tif->tif_mode == O_RDONLY)
    {
        TIFFErrorExtR(tif, tif->tif_name, "File opened in read-only mode");
        return 0;
    }
    if (tif->tif_diroff != 0)
    {
        TIFFErrorExtR(tif, module, "Directory has already been written");
        return 0;
    }

    tif->tif_dir.td_deferstrilearraywriting = TRUE;
    return 1;
}

/*
 * Similar to TIFFWriteDirectory(), writes the directory out
 * but leaves all data structures in memory so that it can be
 * written again.  This will make a partially written TIFF file
 * readable before it is successfully completed/closed.
 */
int TIFFCheckpointDirectory(TIFF *tif)
{
    int rc;
    /* Setup the strips arrays, if they haven't already been. */
    if (tif->tif_dir.td_stripoffset_p == NULL)
        (void)TIFFSetupStrips(tif);
    rc = TIFFWriteDirectorySec(tif, TRUE, FALSE, NULL);
    (void)TIFFSetWriteOffset(tif, TIFFSeekFile(tif, 0, SEEK_END));
    return rc;
}

int TIFFWriteCustomDirectory(TIFF *tif, uint64_t *pdiroff)
{
    return TIFFWriteDirectorySec(tif, FALSE, FALSE, pdiroff);
}

/*
 * Similar to TIFFWriteDirectorySec(), but if the directory has already
 * been written once, it is relocated to the end of the file, in case it
 * has changed in size.  Note that this will result in the loss of the
 * previously used directory space.
 */

static int TIFFRewriteDirectorySec(TIFF *tif, int isimage, int imagedone,
                                   uint64_t *pdiroff)
{
    static const char module[] = "TIFFRewriteDirectory";

    /* We don't need to do anything special if it hasn't been written. */
    if (tif->tif_diroff == 0)
        return TIFFWriteDirectory(tif);

    /*
     * Find and zero the pointer to this directory, so that TIFFLinkDirectory
     * will cause it to be added after this directories current pre-link.
     */
    uint64_t torewritediroff = tif->tif_diroff;

    if (!(tif->tif_flags & TIFF_BIGTIFF))
    {
        if (tif->tif_header.classic.tiff_diroff == tif->tif_diroff)
        {
            tif->tif_header.classic.tiff_diroff = 0;
            tif->tif_diroff = 0;

            TIFFSeekFile(tif, 4, SEEK_SET);
            if (!WriteOK(tif, &(tif->tif_header.classic.tiff_diroff), 4))
            {
                TIFFErrorExtR(tif, tif->tif_name, "Error updating TIFF header");
                return (0);
            }
        }
        else if (tif->tif_diroff > 0xFFFFFFFFU)
        {
            TIFFErrorExtR(tif, module,
                          "tif->tif_diroff exceeds 32 bit range allowed for "
                          "Classic TIFF");
            return (0);
        }
        else
        {
            uint32_t nextdir;
            nextdir = tif->tif_header.classic.tiff_diroff;
            while (1)
            {
                uint16_t dircount;
                uint32_t nextnextdir;

                if (!SeekOK(tif, nextdir) || !ReadOK(tif, &dircount, 2))
                {
                    TIFFErrorExtR(tif, module,
                                  "Error fetching directory count");
                    return (0);
                }
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabShort(&dircount);
                (void)TIFFSeekFile(tif, nextdir + 2 + dircount * 12, SEEK_SET);
                if (!ReadOK(tif, &nextnextdir, 4))
                {
                    TIFFErrorExtR(tif, module, "Error fetching directory link");
                    return (0);
                }
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong(&nextnextdir);
                if (nextnextdir == tif->tif_diroff)
                {
                    uint32_t m;
                    m = 0;
                    (void)TIFFSeekFile(tif, nextdir + 2 + dircount * 12,
                                       SEEK_SET);
                    if (!WriteOK(tif, &m, 4))
                    {
                        TIFFErrorExtR(tif, module,
                                      "Error writing directory link");
                        return (0);
                    }
                    tif->tif_diroff = 0;
                    /* Force a full-traversal to reach the zeroed pointer */
                    tif->tif_lastdiroff = 0;
                    break;
                }
                nextdir = nextnextdir;
            }
        }
        /* Remove skipped offset from IFD loop directory list. */
        _TIFFRemoveEntryFromDirectoryListByOffset(tif, torewritediroff);
    }
    else
    {
        if (tif->tif_header.big.tiff_diroff == tif->tif_diroff)
        {
            tif->tif_header.big.tiff_diroff = 0;
            tif->tif_diroff = 0;

            TIFFSeekFile(tif, 8, SEEK_SET);
            if (!WriteOK(tif, &(tif->tif_header.big.tiff_diroff), 8))
            {
                TIFFErrorExtR(tif, tif->tif_name, "Error updating TIFF header");
                return (0);
            }
        }
        else
        {
            uint64_t nextdir;
            nextdir = tif->tif_header.big.tiff_diroff;
            while (1)
            {
                uint64_t dircount64;
                uint16_t dircount;
                uint64_t nextnextdir;

                if (!SeekOK(tif, nextdir) || !ReadOK(tif, &dircount64, 8))
                {
                    TIFFErrorExtR(tif, module,
                                  "Error fetching directory count");
                    return (0);
                }
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(&dircount64);
                if (dircount64 > 0xFFFF)
                {
                    TIFFErrorExtR(tif, module,
                                  "Sanity check on tag count failed, likely "
                                  "corrupt TIFF");
                    return (0);
                }
                dircount = (uint16_t)dircount64;
                (void)TIFFSeekFile(tif, nextdir + 8 + dircount * 20, SEEK_SET);
                if (!ReadOK(tif, &nextnextdir, 8))
                {
                    TIFFErrorExtR(tif, module, "Error fetching directory link");
                    return (0);
                }
                if (tif->tif_flags & TIFF_SWAB)
                    TIFFSwabLong8(&nextnextdir);
                if (nextnextdir == tif->tif_diroff)
                {
                    uint64_t m;
                    m = 0;
                    (void)TIFFSeekFile(tif, nextdir + 8 + dircount * 20,
                                       SEEK_SET);
                    if (!WriteOK(tif, &m, 8))
                    {
                        TIFFErrorExtR(tif, module,
                                      "Error writing directory link");
                        return (0);
                    }
                    tif->tif_diroff = 0;
                    /* Force a full-traversal to reach the zeroed pointer */
                    tif->tif_lastdiroff = 0;
                    break;
                }
                nextdir = nextnextdir;
            }
        }
        /* Remove skipped offset from IFD loop directory list. */
        _TIFFRemoveEntryFromDirectoryListByOffset(tif, torewritediroff);
    }

    /*
     * Now use TIFFWriteDirectorySec() normally.
     */
    return TIFFWriteDirectorySec(tif, isimage, imagedone, pdiroff);
} /*-- TIFFRewriteDirectorySec() --*/

/*
 * Similar to TIFFWriteDirectory(), but if the directory has already
 * been written once, it is relocated to the end of the file, in case it
 * has changed in size.  Note that this will result in the loss of the
 * previously used directory space.
 */
int TIFFRewriteDirectory(TIFF *tif)
{
    return TIFFRewriteDirectorySec(tif, TRUE, TRUE, NULL);
}

static int TIFFWriteDirectorySec(TIFF *tif, int isimage, int imagedone,
                                 uint64_t *pdiroff)
{
    static const char module[] = "TIFFWriteDirectorySec";
    uint32_t ndir;
    TIFFDirEntry *dir;
    uint32_t dirsize;
    void *dirmem;
    uint32_t m;
    if (tif->tif_mode == O_RDONLY)
        return (1);

    _TIFFFillStriles(tif);

    /*
     * Clear write state so that subsequent images with
     * different characteristics get the right buffers
     * setup for them.
     */
    if (imagedone)
    {
        if (tif->tif_flags & TIFF_POSTENCODE)
        {
            tif->tif_flags &= ~TIFF_POSTENCODE;
            if (!(*tif->tif_postencode)(tif))
            {
                TIFFErrorExtR(tif, module,
                              "Error post-encoding before directory write");
                return (0);
            }
        }
        (*tif->tif_close)(tif); /* shutdown encoder */
        /*
         * Flush any data that might have been written
         * by the compression close+cleanup routines.  But
         * be careful not to write stuff if we didn't add data
         * in the previous steps as the "rawcc" data may well be
         * a previously read tile/strip in mixed read/write mode.
         */
        if (tif->tif_rawcc > 0 && (tif->tif_flags & TIFF_BEENWRITING) != 0)
        {
            if (!TIFFFlushData1(tif))
            {
                TIFFErrorExtR(tif, module,
                              "Error flushing data before directory write");
                return (0);
            }
        }
        if ((tif->tif_flags & TIFF_MYBUFFER) && tif->tif_rawdata)
        {
            _TIFFfreeExt(tif, tif->tif_rawdata);
            tif->tif_rawdata = NULL;
            tif->tif_rawcp = NULL;
            tif->tif_rawcc = 0;
            tif->tif_rawdatasize = 0;
            tif->tif_rawdataoff = 0;
            tif->tif_rawdataloaded = 0;
        }
        tif->tif_flags &= ~(TIFF_BEENWRITING | TIFF_BUFFERSETUP);
    }

    if (TIFFFieldSet(tif, FIELD_COMPRESSION) &&
        (tif->tif_dir.td_compression == COMPRESSION_DEFLATE))
    {
        TIFFWarningExtR(tif, module,
                        "Creating TIFF with legacy Deflate codec identifier, "
                        "COMPRESSION_ADOBE_DEFLATE is more widely supported");
    }
    dir = NULL;
    dirmem = NULL;
    dirsize = 0;
    while (1)
    {
        /* The first loop only determines "ndir" and uses TIFFLinkDirectory() to
         * set the offset at which the IFD is to be written to the file.
         * The second loop writes IFD entries to the file. */
        ndir = 0;
        if (dir == NULL)
            tif->tif_dir.td_dirdatasize_write = 0;
        if (isimage)
        {
            /*-- Step 1: Process named tags for an image with FIELD bits
             *           associated. --*/
            if (TIFFFieldSet(tif, FIELD_IMAGEDIMENSIONS))
            {
                if (!TIFFWriteDirectoryTagShortLong(tif, &ndir, dir,
                                                    TIFFTAG_IMAGEWIDTH,
                                                    tif->tif_dir.td_imagewidth))
                    goto bad;
                if (!TIFFWriteDirectoryTagShortLong(
                        tif, &ndir, dir, TIFFTAG_IMAGELENGTH,
                        tif->tif_dir.td_imagelength))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_TILEDIMENSIONS))
            {
                if (!TIFFWriteDirectoryTagShortLong(tif, &ndir, dir,
                                                    TIFFTAG_TILEWIDTH,
                                                    tif->tif_dir.td_tilewidth))
                    goto bad;
                if (!TIFFWriteDirectoryTagShortLong(tif, &ndir, dir,
                                                    TIFFTAG_TILELENGTH,
                                                    tif->tif_dir.td_tilelength))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_RESOLUTION))
            {
                if (!TIFFWriteDirectoryTagRational(tif, &ndir, dir,
                                                   TIFFTAG_XRESOLUTION,
                                                   tif->tif_dir.td_xresolution))
                    goto bad;
                if (!TIFFWriteDirectoryTagRational(tif, &ndir, dir,
                                                   TIFFTAG_YRESOLUTION,
                                                   tif->tif_dir.td_yresolution))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_POSITION))
            {
                if (!TIFFWriteDirectoryTagRational(tif, &ndir, dir,
                                                   TIFFTAG_XPOSITION,
                                                   tif->tif_dir.td_xposition))
                    goto bad;
                if (!TIFFWriteDirectoryTagRational(tif, &ndir, dir,
                                                   TIFFTAG_YPOSITION,
                                                   tif->tif_dir.td_yposition))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_SUBFILETYPE))
            {
                if (!TIFFWriteDirectoryTagLong(tif, &ndir, dir,
                                               TIFFTAG_SUBFILETYPE,
                                               tif->tif_dir.td_subfiletype))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_BITSPERSAMPLE))
            {
                if (!TIFFWriteDirectoryTagShortPerSample(
                        tif, &ndir, dir, TIFFTAG_BITSPERSAMPLE,
                        tif->tif_dir.td_bitspersample))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_COMPRESSION))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_COMPRESSION,
                                                tif->tif_dir.td_compression))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_PHOTOMETRIC))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_PHOTOMETRIC,
                                                tif->tif_dir.td_photometric))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_THRESHHOLDING))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_THRESHHOLDING,
                                                tif->tif_dir.td_threshholding))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_FILLORDER))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_FILLORDER,
                                                tif->tif_dir.td_fillorder))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_ORIENTATION))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_ORIENTATION,
                                                tif->tif_dir.td_orientation))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_SAMPLESPERPIXEL))
            {
                if (!TIFFWriteDirectoryTagShort(
                        tif, &ndir, dir, TIFFTAG_SAMPLESPERPIXEL,
                        tif->tif_dir.td_samplesperpixel))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_ROWSPERSTRIP))
            {
                if (!TIFFWriteDirectoryTagShortLong(
                        tif, &ndir, dir, TIFFTAG_ROWSPERSTRIP,
                        tif->tif_dir.td_rowsperstrip))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_MINSAMPLEVALUE))
            {
                if (!TIFFWriteDirectoryTagShortPerSample(
                        tif, &ndir, dir, TIFFTAG_MINSAMPLEVALUE,
                        tif->tif_dir.td_minsamplevalue))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_MAXSAMPLEVALUE))
            {
                if (!TIFFWriteDirectoryTagShortPerSample(
                        tif, &ndir, dir, TIFFTAG_MAXSAMPLEVALUE,
                        tif->tif_dir.td_maxsamplevalue))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_PLANARCONFIG))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_PLANARCONFIG,
                                                tif->tif_dir.td_planarconfig))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_RESOLUTIONUNIT))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_RESOLUTIONUNIT,
                                                tif->tif_dir.td_resolutionunit))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_PAGENUMBER))
            {
                if (!TIFFWriteDirectoryTagShortArray(
                        tif, &ndir, dir, TIFFTAG_PAGENUMBER, 2,
                        &tif->tif_dir.td_pagenumber[0]))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_STRIPBYTECOUNTS))
            {
                if (!isTiled(tif))
                {
                    if (!TIFFWriteDirectoryTagLongLong8Array(
                            tif, &ndir, dir, TIFFTAG_STRIPBYTECOUNTS,
                            tif->tif_dir.td_nstrips,
                            tif->tif_dir.td_stripbytecount_p))
                        goto bad;
                }
                else
                {
                    if (!TIFFWriteDirectoryTagLongLong8Array(
                            tif, &ndir, dir, TIFFTAG_TILEBYTECOUNTS,
                            tif->tif_dir.td_nstrips,
                            tif->tif_dir.td_stripbytecount_p))
                        goto bad;
                }
            }
            if (TIFFFieldSet(tif, FIELD_STRIPOFFSETS))
            {
                if (!isTiled(tif))
                {
                    /* td_stripoffset_p might be NULL in an odd OJPEG case. See
                     *  tif_dirread.c around line 3634.
                     * XXX: OJPEG hack.
                     * If a) compression is OJPEG, b) it's not a tiled TIFF,
                     * and c) the number of strips is 1,
                     * then we tolerate the absence of stripoffsets tag,
                     * because, presumably, all required data is in the
                     * JpegInterchangeFormat stream.
                     * We can get here when using tiffset on such a file.
                     * See http://bugzilla.maptools.org/show_bug.cgi?id=2500
                     */
                    if (tif->tif_dir.td_stripoffset_p != NULL &&
                        !TIFFWriteDirectoryTagLongLong8Array(
                            tif, &ndir, dir, TIFFTAG_STRIPOFFSETS,
                            tif->tif_dir.td_nstrips,
                            tif->tif_dir.td_stripoffset_p))
                        goto bad;
                }
                else
                {
                    if (!TIFFWriteDirectoryTagLongLong8Array(
                            tif, &ndir, dir, TIFFTAG_TILEOFFSETS,
                            tif->tif_dir.td_nstrips,
                            tif->tif_dir.td_stripoffset_p))
                        goto bad;
                }
            }
            if (TIFFFieldSet(tif, FIELD_COLORMAP))
            {
                if (!TIFFWriteDirectoryTagColormap(tif, &ndir, dir))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_EXTRASAMPLES))
            {
                if (tif->tif_dir.td_extrasamples)
                {
                    uint16_t na;
                    uint16_t *nb;
                    TIFFGetFieldDefaulted(tif, TIFFTAG_EXTRASAMPLES, &na, &nb);
                    if (!TIFFWriteDirectoryTagShortArray(
                            tif, &ndir, dir, TIFFTAG_EXTRASAMPLES, na, nb))
                        goto bad;
                }
            }
            if (TIFFFieldSet(tif, FIELD_SAMPLEFORMAT))
            {
                if (!TIFFWriteDirectoryTagShortPerSample(
                        tif, &ndir, dir, TIFFTAG_SAMPLEFORMAT,
                        tif->tif_dir.td_sampleformat))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_SMINSAMPLEVALUE))
            {
                if (!TIFFWriteDirectoryTagSampleformatArray(
                        tif, &ndir, dir, TIFFTAG_SMINSAMPLEVALUE,
                        tif->tif_dir.td_samplesperpixel,
                        tif->tif_dir.td_sminsamplevalue))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_SMAXSAMPLEVALUE))
            {
                if (!TIFFWriteDirectoryTagSampleformatArray(
                        tif, &ndir, dir, TIFFTAG_SMAXSAMPLEVALUE,
                        tif->tif_dir.td_samplesperpixel,
                        tif->tif_dir.td_smaxsamplevalue))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_IMAGEDEPTH))
            {
                if (!TIFFWriteDirectoryTagLong(tif, &ndir, dir,
                                               TIFFTAG_IMAGEDEPTH,
                                               tif->tif_dir.td_imagedepth))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_TILEDEPTH))
            {
                if (!TIFFWriteDirectoryTagLong(tif, &ndir, dir,
                                               TIFFTAG_TILEDEPTH,
                                               tif->tif_dir.td_tiledepth))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_HALFTONEHINTS))
            {
                if (!TIFFWriteDirectoryTagShortArray(
                        tif, &ndir, dir, TIFFTAG_HALFTONEHINTS, 2,
                        &tif->tif_dir.td_halftonehints[0]))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_YCBCRSUBSAMPLING))
            {
                if (!TIFFWriteDirectoryTagShortArray(
                        tif, &ndir, dir, TIFFTAG_YCBCRSUBSAMPLING, 2,
                        &tif->tif_dir.td_ycbcrsubsampling[0]))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_YCBCRPOSITIONING))
            {
                if (!TIFFWriteDirectoryTagShort(
                        tif, &ndir, dir, TIFFTAG_YCBCRPOSITIONING,
                        tif->tif_dir.td_ycbcrpositioning))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_REFBLACKWHITE))
            {
                if (!TIFFWriteDirectoryTagRationalArray(
                        tif, &ndir, dir, TIFFTAG_REFERENCEBLACKWHITE, 6,
                        tif->tif_dir.td_refblackwhite))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_TRANSFERFUNCTION))
            {
                if (!TIFFWriteDirectoryTagTransferfunction(tif, &ndir, dir))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_INKNAMES))
            {
                if (!TIFFWriteDirectoryTagAscii(
                        tif, &ndir, dir, TIFFTAG_INKNAMES,
                        tif->tif_dir.td_inknameslen, tif->tif_dir.td_inknames))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_NUMBEROFINKS))
            {
                if (!TIFFWriteDirectoryTagShort(tif, &ndir, dir,
                                                TIFFTAG_NUMBEROFINKS,
                                                tif->tif_dir.td_numberofinks))
                    goto bad;
            }
            if (TIFFFieldSet(tif, FIELD_SUBIFD))
            {
                if (!TIFFWriteDirectoryTagSubifd(tif, &ndir, dir))
                    goto bad;
            }
            /*-- Step 2: Process named tags for an image with FIELD bits
                         added by a codec.
                         Attention: There is only code for some field_types,
                         which are actually used by current codecs. --*/
            {
                uint32_t n;
                for (n = 0; n < tif->tif_nfields; n++)
                {
                    const TIFFField *o;
                    o = tif->tif_fields[n];
                    if ((o->field_bit >= FIELD_CODEC) &&
                        (TIFFFieldSet(tif, o->field_bit)))
                    {
                        switch (o->set_get_field_type)
                        {
                            case TIFF_SETGET_ASCII:
                            {
                                uint32_t pa;
                                char *pb;
                                assert(o->field_type == TIFF_ASCII);
                                assert(o->field_readcount == TIFF_VARIABLE);
                                assert(o->field_passcount == 0);
                                TIFFGetField(tif, o->field_tag, &pb);
                                pa = (uint32_t)(strlen(pb) + 1);
                                if (!TIFFWriteDirectoryTagAscii(
                                        tif, &ndir, dir, (uint16_t)o->field_tag,
                                        pa, pb))
                                    goto bad;
                            }
                            break;
                            case TIFF_SETGET_UINT16:
                            {
                                uint16_t p;
                                assert(o->field_type == TIFF_SHORT);
                                assert(o->field_readcount == 1);
                                assert(o->field_passcount == 0);
                                TIFFGetField(tif, o->field_tag, &p);
                                if (!TIFFWriteDirectoryTagShort(
                                        tif, &ndir, dir, (uint16_t)o->field_tag,
                                        p))
                                    goto bad;
                            }
                            break;
                            case TIFF_SETGET_UINT32:
                            {
                                uint32_t p;
                                assert(o->field_type == TIFF_LONG);
                                assert(o->field_readcount == 1);
                                assert(o->field_passcount == 0);
                                TIFFGetField(tif, o->field_tag, &p);
                                if (!TIFFWriteDirectoryTagLong(
                                        tif, &ndir, dir, (uint16_t)o->field_tag,
                                        p))
                                    goto bad;
                            }
                            break;
                            case TIFF_SETGET_C32_UINT8:
                            {
                                uint32_t pa;
                                void *pb;
                                assert(o->field_type == TIFF_UNDEFINED);
                                assert(o->field_readcount == TIFF_VARIABLE2);
                                assert(o->field_passcount == 1);
                                TIFFGetField(tif, o->field_tag, &pa, &pb);
                                if (!TIFFWriteDirectoryTagUndefinedArray(
                                        tif, &ndir, dir, (uint16_t)o->field_tag,
                                        pa, pb))
                                    goto bad;
                            }
                            break;
                            default:
                                TIFFErrorExtR(
                                    tif, module,
                                    "Cannot write tag %" PRIu32 " (%s)",
                                    TIFFFieldTag(o),
                                    o->field_name ? o->field_name : "unknown");
                                goto bad;
                        }
                    }
                }
            }
        }
        /*-- Step 3: Process custom tags without FIELD bit for an image
         *           or for custom IFDs (e.g. EXIF) with !isimage. --*/
        for (m = 0; m < (uint32_t)(tif->tif_dir.td_customValueCount); m++)
        {
            uint16_t tag =
                (uint16_t)tif->tif_dir.td_customValues[m].info->field_tag;
            uint32_t count = tif->tif_dir.td_customValues[m].count;
            switch (tif->tif_dir.td_customValues[m].info->field_type)
            {
                case TIFF_ASCII:
                    if (!TIFFWriteDirectoryTagAscii(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_UNDEFINED:
                    if (!TIFFWriteDirectoryTagUndefinedArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_BYTE:
                    if (!TIFFWriteDirectoryTagByteArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_SBYTE:
                    if (!TIFFWriteDirectoryTagSbyteArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_SHORT:
                    if (!TIFFWriteDirectoryTagShortArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_SSHORT:
                    if (!TIFFWriteDirectoryTagSshortArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_LONG:
                    if (!TIFFWriteDirectoryTagLongArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_SLONG:
                    if (!TIFFWriteDirectoryTagSlongArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_LONG8:
                    if (!TIFFWriteDirectoryTagLong8Array(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_SLONG8:
                    if (!TIFFWriteDirectoryTagSlong8Array(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_RATIONAL:
                {
                    /*-- Rational2Double: For Rationals evaluate
                     * "set_get_field_type" to determine internal storage size.
                     */
                    int tv_size;
                    tv_size = TIFFFieldSetGetSize(
                        tif->tif_dir.td_customValues[m].info);
                    if (tv_size == 8)
                    {
                        if (!TIFFWriteDirectoryTagRationalDoubleArray(
                                tif, &ndir, dir, tag, count,
                                tif->tif_dir.td_customValues[m].value))
                            goto bad;
                    }
                    else
                    {
                        /*-- default should be tv_size == 4 */
                        if (!TIFFWriteDirectoryTagRationalArray(
                                tif, &ndir, dir, tag, count,
                                tif->tif_dir.td_customValues[m].value))
                            goto bad;
                        /*-- ToDo: After Testing, this should be removed and
                         * tv_size==4 should be set as default. */
                        if (tv_size != 4)
                        {
                            TIFFErrorExtR(
                                tif, "TIFFLib: _TIFFWriteDirectorySec()",
                                "Rational2Double: .set_get_field_type is "
                                "not 4 but %d",
                                tv_size);
                        }
                    }
                }
                break;
                case TIFF_SRATIONAL:
                {
                    /*-- Rational2Double: For Rationals evaluate
                     * "set_get_field_type" to determine internal storage size.
                     */
                    int tv_size;
                    tv_size = TIFFFieldSetGetSize(
                        tif->tif_dir.td_customValues[m].info);
                    if (tv_size == 8)
                    {
                        if (!TIFFWriteDirectoryTagSrationalDoubleArray(
                                tif, &ndir, dir, tag, count,
                                tif->tif_dir.td_customValues[m].value))
                            goto bad;
                    }
                    else
                    {
                        /*-- default should be tv_size == 4 */
                        if (!TIFFWriteDirectoryTagSrationalArray(
                                tif, &ndir, dir, tag, count,
                                tif->tif_dir.td_customValues[m].value))
                            goto bad;
                        /*-- ToDo: After Testing, this should be removed and
                         * tv_size==4 should be set as default. */
                        if (tv_size != 4)
                        {
                            TIFFErrorExtR(
                                tif, "TIFFLib: _TIFFWriteDirectorySec()",
                                "Rational2Double: .set_get_field_type is "
                                "not 4 but %d",
                                tv_size);
                        }
                    }
                }
                break;
                case TIFF_FLOAT:
                    if (!TIFFWriteDirectoryTagFloatArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_DOUBLE:
                    if (!TIFFWriteDirectoryTagDoubleArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_IFD:
                    if (!TIFFWriteDirectoryTagIfdArray(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                case TIFF_IFD8:
                    if (!TIFFWriteDirectoryTagIfdIfd8Array(
                            tif, &ndir, dir, tag, count,
                            tif->tif_dir.td_customValues[m].value))
                        goto bad;
                    break;
                default:
                    assert(0); /* we should never get here */
                    break;
            }
        }
        /* "break" if IFD has been written above in second pass.*/
        if (dir != NULL)
            break;

        /* Evaluate IFD data size: Finally, add the size of the IFD tag entries
         * themselves. */
        if (!(tif->tif_flags & TIFF_BIGTIFF))
            tif->tif_dir.td_dirdatasize_write += 2 + ndir * 12 + 4;
        else
            tif->tif_dir.td_dirdatasize_write += 8 + ndir * 20 + 8;

        /* Setup a new directory within first pass. */
        dir = _TIFFmallocExt(tif, ndir * sizeof(TIFFDirEntry));
        if (dir == NULL)
        {
            TIFFErrorExtR(tif, module, "Out of memory");
            goto bad;
        }
        if (isimage)
        {
            /* Check, weather the IFD to be written is new or an already written
             * IFD can be overwritten or needs to be re-written to a different
             * location in the file because the IFD is extended with additional
             * tags or the IFD data size is increased.
             * - tif_diroff == 0, if a new directory has to be linked.
             * - tif_diroff != 0, IFD has been re-read from file and will be
             *   overwritten or re-written.
             */
            if (tif->tif_diroff == 0)
            {
                if (!TIFFLinkDirectory(tif))
                    goto bad;
            }
            else if (tif->tif_dir.td_dirdatasize_write >
                     tif->tif_dir.td_dirdatasize_read)
            {
                if (dir != NULL)
                {
                    _TIFFfreeExt(tif, dir);
                    dir = NULL;
                }
                if (!TIFFRewriteDirectorySec(tif, isimage, imagedone, pdiroff))
                    goto bad;
                return (1);
            }
        }
        else
        {
            /* For !isimage, which means custom-IFD like EXIFIFD or
             * checkpointing an IFD, determine whether to overwrite or append at
             * the end of the file.
             */
            if (!((tif->tif_dir.td_dirdatasize_read > 0) &&
                  (tif->tif_dir.td_dirdatasize_write <=
                   tif->tif_dir.td_dirdatasize_read)))
            {
                /* Append at end of file and increment to an even offset. */
                tif->tif_diroff =
                    (TIFFSeekFile(tif, 0, SEEK_END) + 1) & (~((toff_t)1));
            }
        }
        /* Return IFD offset */
        if (pdiroff != NULL)
            *pdiroff = tif->tif_diroff;
        if (!(tif->tif_flags & TIFF_BIGTIFF))
            dirsize = 2 + ndir * 12 + 4;
        else
            dirsize = 8 + ndir * 20 + 8;
        /* Append IFD data stright after the IFD tag entries.
         * Data that does not fit into an IFD tag entry is written to the file
         * in the second pass of the while loop. That offset is stored in "dir".
         */
        tif->tif_dataoff = tif->tif_diroff + dirsize;
        if (!(tif->tif_flags & TIFF_BIGTIFF))
            tif->tif_dataoff = (uint32_t)tif->tif_dataoff;
        if ((tif->tif_dataoff < tif->tif_diroff) ||
            (tif->tif_dataoff < (uint64_t)dirsize))
        {
            TIFFErrorExtR(tif, module, "Maximum TIFF file size exceeded");
            goto bad;
        }
        if (tif->tif_dataoff & 1)
            tif->tif_dataoff++;
    } /* while() */
    if (isimage)
    {
        /* For SubIFDs remember offset of SubIFD tag within main IFD.
         * However, might be already done in TIFFWriteDirectoryTagSubifd() if
         * there are more than one SubIFD. */
        if (TIFFFieldSet(tif, FIELD_SUBIFD) && (tif->tif_subifdoff == 0))
        {
            uint32_t na;
            TIFFDirEntry *nb;
            for (na = 0, nb = dir;; na++, nb++)
            {
                if (na == ndir)
                {
                    TIFFErrorExtR(tif, module, "Cannot find SubIFD tag");
                    goto bad;
                }
                if (nb->tdir_tag == TIFFTAG_SUBIFD)
                    break;
            }
            if (!(tif->tif_flags & TIFF_BIGTIFF))
                tif->tif_subifdoff = tif->tif_diroff + 2 + na * 12 + 8;
            else
                tif->tif_subifdoff = tif->tif_diroff + 8 + na * 20 + 12;
        }
    }
    /* Copy/swab IFD entries from "dir" into "dirmem",
     * which is then written to file. */
    dirmem = _TIFFmallocExt(tif, dirsize);
    if (dirmem == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        goto bad;
    }
    if (!(tif->tif_flags & TIFF_BIGTIFF))
    {
        uint8_t *n;
        uint32_t nTmp;
        TIFFDirEntry *o;
        n = dirmem;
        *(uint16_t *)n = (uint16_t)ndir;
        if (tif->tif_flags & TIFF_SWAB)
            TIFFSwabShort((uint16_t *)n);
        n += 2;
        o = dir;
        for (m = 0; m < ndir; m++)
        {
            *(uint16_t *)n = o->tdir_tag;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabShort((uint16_t *)n);
            n += 2;
            *(uint16_t *)n = o->tdir_type;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabShort((uint16_t *)n);
            n += 2;
            nTmp = (uint32_t)o->tdir_count;
            _TIFFmemcpy(n, &nTmp, 4);
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabLong((uint32_t *)n);
            n += 4;
            /* This is correct. The data has been */
            /* swabbed previously in TIFFWriteDirectoryTagData */
            _TIFFmemcpy(n, &o->tdir_offset, 4);
            n += 4;
            o++;
        }
        nTmp = (uint32_t)tif->tif_nextdiroff;
        if (tif->tif_flags & TIFF_SWAB)
            TIFFSwabLong(&nTmp);
        _TIFFmemcpy(n, &nTmp, 4);
    }
    else
    {
        uint8_t *n;
        TIFFDirEntry *o;
        n = dirmem;
        *(uint64_t *)n = ndir;
        if (tif->tif_flags & TIFF_SWAB)
            TIFFSwabLong8((uint64_t *)n);
        n += 8;
        o = dir;
        for (m = 0; m < ndir; m++)
        {
            *(uint16_t *)n = o->tdir_tag;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabShort((uint16_t *)n);
            n += 2;
            *(uint16_t *)n = o->tdir_type;
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabShort((uint16_t *)n);
            n += 2;
            _TIFFmemcpy(n, &o->tdir_count, 8);
            if (tif->tif_flags & TIFF_SWAB)
                TIFFSwabLong8((uint64_t *)n);
            n += 8;
            _TIFFmemcpy(n, &o->tdir_offset, 8);
            n += 8;
            o++;
        }
        _TIFFmemcpy(n, &tif->tif_nextdiroff, 8);
        if (tif->tif_flags & TIFF_SWAB)
            TIFFSwabLong8((uint64_t *)n);
    }
    _TIFFfreeExt(tif, dir);
    dir = NULL;
    if (!SeekOK(tif, tif->tif_diroff))
    {
        TIFFErrorExtR(tif, module,
                      "IO error writing directory at seek to offset");
        goto bad;
    }
    if (!WriteOK(tif, dirmem, (tmsize_t)dirsize))
    {
        TIFFErrorExtR(tif, module, "IO error writing directory");
        goto bad;
    }
    _TIFFfreeExt(tif, dirmem);

    /* Increment tif_curdir if IFD wasn't already written to file and no error
     * occurred during IFD writing above. */
    if (isimage && !tif->tif_dir.td_iswrittentofile)
    {
        if (!((tif->tif_flags & TIFF_INSUBIFD) &&
              !(TIFFFieldSet(tif, FIELD_SUBIFD))))
        {
            /*-- Normal main-IFD case --*/
            if (tif->tif_curdircount != TIFF_NON_EXISTENT_DIR_NUMBER)
            {
                tif->tif_curdir = tif->tif_curdircount;
            }
            else
            {
                /*ToDo SU: NEW_IFD_CURDIR_INCREMENTING:  Delete this
                 * unexpected case after some testing time. */
                /* Attention: tif->tif_curdircount is already set within
                 * TIFFNumberOfDirectories() */
                tif->tif_curdircount = TIFFNumberOfDirectories(tif);
                tif->tif_curdir = tif->tif_curdircount;
                TIFFErrorExtR(
                    tif, module,
                    "tif_curdircount is TIFF_NON_EXISTENT_DIR_NUMBER, "
                    "not expected !! Line %d",
                    __LINE__);
                goto bad;
            }
        }
        else
        {
            /*-- SubIFD case -- */
            /* tif_curdir is always set to 0 for all SubIFDs. */
            tif->tif_curdir = 0;
        }
    }
    /* Increment tif_curdircount only if main-IFD of an image was not already
     * present on file. */
    /* Check in combination with (... && !(TIFFFieldSet(tif, FIELD_SUBIFD)))
     * is necessary here because TIFF_INSUBIFD was already set above for the
     * next SubIFD when this main-IFD (with FIELD_SUBIFD) is currently being
     * written. */
    if (isimage && !tif->tif_dir.td_iswrittentofile &&
        !((tif->tif_flags & TIFF_INSUBIFD) &&
          !(TIFFFieldSet(tif, FIELD_SUBIFD))))
        tif->tif_curdircount++;

    tif->tif_dir.td_iswrittentofile = TRUE;

    /* Reset SubIFD writing stage after last SubIFD has been written. */
    if (imagedone && (tif->tif_flags & TIFF_INSUBIFD) && tif->tif_nsubifd == 0)
        tif->tif_flags &= ~TIFF_INSUBIFD;

    /* Add or update this directory to the IFD list. */
    if (!_TIFFCheckDirNumberAndOffset(tif, tif->tif_curdir, tif->tif_diroff))
    {
        TIFFErrorExtR(tif, module,
                      "Starting directory %u at offset 0x%" PRIx64 " (%" PRIu64
                      ") might cause an IFD loop",
                      tif->tif_curdir, tif->tif_diroff, tif->tif_diroff);
    }

    if (imagedone)
    {
        TIFFFreeDirectory(tif);
        tif->tif_flags &= ~TIFF_DIRTYDIRECT;
        tif->tif_flags &= ~TIFF_DIRTYSTRIP;
        /* Reset directory-related state for subsequent directories. */
        TIFFCreateDirectory(tif);
    }
    else
    {
        /* IFD is only checkpointed to file (or a custom IFD like EXIF is
         * written), thus set IFD data size written to file. */
        tif->tif_dir.td_dirdatasize_read = tif->tif_dir.td_dirdatasize_write;
    }
    return (1);
bad:
    if (dir != NULL)
        _TIFFfreeExt(tif, dir);
    if (dirmem != NULL)
        _TIFFfreeExt(tif, dirmem);
    return (0);
}

static int8_t TIFFClampDoubleToInt8(double val)
{
    if (val > 127)
        return 127;
    if (val < -128 || val != val)
        return -128;
    return (int8_t)val;
}

static int16_t TIFFClampDoubleToInt16(double val)
{
    if (val > 32767)
        return 32767;
    if (val < -32768 || val != val)
        return -32768;
    return (int16_t)val;
}

static int32_t TIFFClampDoubleToInt32(double val)
{
    if (val > 0x7FFFFFFF)
        return 0x7FFFFFFF;
    if (val < -0x7FFFFFFF - 1 || val != val)
        return -0x7FFFFFFF - 1;
    return (int32_t)val;
}

static uint8_t TIFFClampDoubleToUInt8(double val)
{
    if (val < 0)
        return 0;
    if (val > 255 || val != val)
        return 255;
    return (uint8_t)val;
}

static uint16_t TIFFClampDoubleToUInt16(double val)
{
    if (val < 0)
        return 0;
    if (val > 65535 || val != val)
        return 65535;
    return (uint16_t)val;
}

static uint32_t TIFFClampDoubleToUInt32(double val)
{
    if (val < 0)
        return 0;
    if (val > 0xFFFFFFFFU || val != val)
        return 0xFFFFFFFFU;
    return (uint32_t)val;
}

static int TIFFWriteDirectoryTagSampleformatArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  double *value)
{
    static const char module[] = "TIFFWriteDirectoryTagSampleformatArray";
    void *conv;
    uint32_t i;
    int ok;
    conv = _TIFFmallocExt(tif, count * sizeof(double));
    if (conv == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }

    switch (tif->tif_dir.td_sampleformat)
    {
        case SAMPLEFORMAT_IEEEFP:
            if (tif->tif_dir.td_bitspersample <= 32)
            {
                for (i = 0; i < count; ++i)
                    ((float *)conv)[i] = _TIFFClampDoubleToFloat(value[i]);
                ok = TIFFWriteDirectoryTagFloatArray(tif, ndir, dir, tag, count,
                                                     (float *)conv);
            }
            else
            {
                ok = TIFFWriteDirectoryTagDoubleArray(tif, ndir, dir, tag,
                                                      count, value);
            }
            break;
        case SAMPLEFORMAT_INT:
            if (tif->tif_dir.td_bitspersample <= 8)
            {
                for (i = 0; i < count; ++i)
                    ((int8_t *)conv)[i] = TIFFClampDoubleToInt8(value[i]);
                ok = TIFFWriteDirectoryTagSbyteArray(tif, ndir, dir, tag, count,
                                                     (int8_t *)conv);
            }
            else if (tif->tif_dir.td_bitspersample <= 16)
            {
                for (i = 0; i < count; ++i)
                    ((int16_t *)conv)[i] = TIFFClampDoubleToInt16(value[i]);
                ok = TIFFWriteDirectoryTagSshortArray(tif, ndir, dir, tag,
                                                      count, (int16_t *)conv);
            }
            else
            {
                for (i = 0; i < count; ++i)
                    ((int32_t *)conv)[i] = TIFFClampDoubleToInt32(value[i]);
                ok = TIFFWriteDirectoryTagSlongArray(tif, ndir, dir, tag, count,
                                                     (int32_t *)conv);
            }
            break;
        case SAMPLEFORMAT_UINT:
            if (tif->tif_dir.td_bitspersample <= 8)
            {
                for (i = 0; i < count; ++i)
                    ((uint8_t *)conv)[i] = TIFFClampDoubleToUInt8(value[i]);
                ok = TIFFWriteDirectoryTagByteArray(tif, ndir, dir, tag, count,
                                                    (uint8_t *)conv);
            }
            else if (tif->tif_dir.td_bitspersample <= 16)
            {
                for (i = 0; i < count; ++i)
                    ((uint16_t *)conv)[i] = TIFFClampDoubleToUInt16(value[i]);
                ok = TIFFWriteDirectoryTagShortArray(tif, ndir, dir, tag, count,
                                                     (uint16_t *)conv);
            }
            else
            {
                for (i = 0; i < count; ++i)
                    ((uint32_t *)conv)[i] = TIFFClampDoubleToUInt32(value[i]);
                ok = TIFFWriteDirectoryTagLongArray(tif, ndir, dir, tag, count,
                                                    (uint32_t *)conv);
            }
            break;
        default:
            ok = 0;
    }

    _TIFFfreeExt(tif, conv);
    return (ok);
}

static int TIFFWriteDirectoryTagAscii(TIFF *tif, uint32_t *ndir,
                                      TIFFDirEntry *dir, uint16_t tag,
                                      uint32_t count, char *value)
{
    return (
        TIFFWriteDirectoryTagCheckedAscii(tif, ndir, dir, tag, count, value));
}

static int TIFFWriteDirectoryTagUndefinedArray(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, uint8_t *value)
{
    return (TIFFWriteDirectoryTagCheckedUndefinedArray(tif, ndir, dir, tag,
                                                       count, value));
}

static int TIFFWriteDirectoryTagByteArray(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t count, uint8_t *value)
{
    return (TIFFWriteDirectoryTagCheckedByteArray(tif, ndir, dir, tag, count,
                                                  value));
}

static int TIFFWriteDirectoryTagSbyteArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, int8_t *value)
{
    return (TIFFWriteDirectoryTagCheckedSbyteArray(tif, ndir, dir, tag, count,
                                                   value));
}

static int TIFFWriteDirectoryTagShort(TIFF *tif, uint32_t *ndir,
                                      TIFFDirEntry *dir, uint16_t tag,
                                      uint16_t value)
{
    return (TIFFWriteDirectoryTagCheckedShort(tif, ndir, dir, tag, value));
}

static int TIFFWriteDirectoryTagShortArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, uint16_t *value)
{
    return (TIFFWriteDirectoryTagCheckedShortArray(tif, ndir, dir, tag, count,
                                                   value));
}

static int TIFFWriteDirectoryTagShortPerSample(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint16_t value)
{
    static const char module[] = "TIFFWriteDirectoryTagShortPerSample";
    uint16_t *m;
    uint16_t *na;
    uint16_t nb;
    int o;
    if (dir == NULL)
    {
        /* only evaluate IFD data size and inc. ndir */
        return (TIFFWriteDirectoryTagCheckedShortArray(
            tif, ndir, dir, tag, tif->tif_dir.td_samplesperpixel, NULL));
    }
    m = _TIFFmallocExt(tif, tif->tif_dir.td_samplesperpixel * sizeof(uint16_t));
    if (m == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }
    for (na = m, nb = 0; nb < tif->tif_dir.td_samplesperpixel; na++, nb++)
        *na = value;
    o = TIFFWriteDirectoryTagCheckedShortArray(
        tif, ndir, dir, tag, tif->tif_dir.td_samplesperpixel, m);
    _TIFFfreeExt(tif, m);
    return (o);
}

static int TIFFWriteDirectoryTagSshortArray(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, int16_t *value)
{
    return (TIFFWriteDirectoryTagCheckedSshortArray(tif, ndir, dir, tag, count,
                                                    value));
}

static int TIFFWriteDirectoryTagLong(TIFF *tif, uint32_t *ndir,
                                     TIFFDirEntry *dir, uint16_t tag,
                                     uint32_t value)
{
    return (TIFFWriteDirectoryTagCheckedLong(tif, ndir, dir, tag, value));
}

static int TIFFWriteDirectoryTagLongArray(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t count, uint32_t *value)
{
    return (TIFFWriteDirectoryTagCheckedLongArray(tif, ndir, dir, tag, count,
                                                  value));
}

static int TIFFWriteDirectoryTagSlongArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, int32_t *value)
{
    return (TIFFWriteDirectoryTagCheckedSlongArray(tif, ndir, dir, tag, count,
                                                   value));
}

/************************************************************************/
/*                 TIFFWriteDirectoryTagLong8Array()                    */
/*                                                                      */
/*      Write either Long8 or Long array depending on file type.        */
/************************************************************************/
static int TIFFWriteDirectoryTagLong8Array(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, uint64_t *value)
{
    static const char module[] = "TIFFWriteDirectoryTagLong8Array";
    uint64_t *ma;
    uint32_t mb;
    uint32_t *p;
    uint32_t *q;
    int o;

    /* is this just a counting pass? */
    if (dir == NULL)
    {
        /* only evaluate IFD data size and inc. ndir */
        return (TIFFWriteDirectoryTagCheckedLong8Array(tif, ndir, dir, tag,
                                                       count, value));
    }

    /* We always write Long8 for BigTIFF, no checking needed. */
    if (tif->tif_flags & TIFF_BIGTIFF)
        return (TIFFWriteDirectoryTagCheckedLong8Array(tif, ndir, dir, tag,
                                                       count, value));

    /*
    ** For classic tiff we want to verify everything is in range for long
    ** and convert to long format.
    */
    p = _TIFFmallocExt(tif, count * sizeof(uint32_t));
    if (p == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }

    for (q = p, ma = value, mb = 0; mb < count; ma++, mb++, q++)
    {
        if (*ma > 0xFFFFFFFF)
        {
            TIFFErrorExtR(tif, module,
                          "Attempt to write unsigned long value %" PRIu64
                          " larger than 0xFFFFFFFF for tag %d in Classic TIFF "
                          "file. TIFF file writing aborted",
                          *ma, tag);
            _TIFFfreeExt(tif, p);
            return (0);
        }
        *q = (uint32_t)(*ma);
    }

    o = TIFFWriteDirectoryTagCheckedLongArray(tif, ndir, dir, tag, count, p);
    _TIFFfreeExt(tif, p);

    return (o);
}

/************************************************************************/
/*                 TIFFWriteDirectoryTagSlong8Array()                   */
/*                                                                      */
/*      Write either SLong8 or SLong array depending on file type.      */
/************************************************************************/
static int TIFFWriteDirectoryTagSlong8Array(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, int64_t *value)
{
    static const char module[] = "TIFFWriteDirectoryTagSlong8Array";
    int64_t *ma;
    uint32_t mb;
    int32_t *p;
    int32_t *q;
    int o;

    /* is this just a counting pass? */
    if (dir == NULL)
    {
        /* only evaluate IFD data size and inc. ndir */
        return (TIFFWriteDirectoryTagCheckedSlong8Array(tif, ndir, dir, tag,
                                                        count, value));
    }
    /* We always write SLong8 for BigTIFF, no checking needed. */
    if (tif->tif_flags & TIFF_BIGTIFF)
        return (TIFFWriteDirectoryTagCheckedSlong8Array(tif, ndir, dir, tag,
                                                        count, value));

    /*
    ** For classic tiff we want to verify everything is in range for signed-long
    ** and convert to signed-long format.
    */
    p = _TIFFmallocExt(tif, count * sizeof(uint32_t));
    if (p == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }

    for (q = p, ma = value, mb = 0; mb < count; ma++, mb++, q++)
    {
        if (*ma > (2147483647))
        {
            TIFFErrorExtR(tif, module,
                          "Attempt to write signed long value %" PRIi64
                          " larger than 0x7FFFFFFF (2147483647) for tag %d in "
                          "Classic TIFF file. TIFF writing to file aborted",
                          *ma, tag);
            _TIFFfreeExt(tif, p);
            return (0);
        }
        else if (*ma < (-2147483647 - 1))
        {
            TIFFErrorExtR(tif, module,
                          "Attempt to write signed long value %" PRIi64
                          " smaller than 0x80000000 (-2147483648) for tag %d "
                          "in Classic TIFF file. TIFF writing to file aborted",
                          *ma, tag);
            _TIFFfreeExt(tif, p);
            return (0);
        }
        *q = (int32_t)(*ma);
    }

    o = TIFFWriteDirectoryTagCheckedSlongArray(tif, ndir, dir, tag, count, p);
    _TIFFfreeExt(tif, p);

    return (o);
}

static int TIFFWriteDirectoryTagRational(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir, uint16_t tag,
                                         double value)
{
    return (TIFFWriteDirectoryTagCheckedRational(tif, ndir, dir, tag, value));
}

static int TIFFWriteDirectoryTagRationalArray(TIFF *tif, uint32_t *ndir,
                                              TIFFDirEntry *dir, uint16_t tag,
                                              uint32_t count, float *value)
{
    return (TIFFWriteDirectoryTagCheckedRationalArray(tif, ndir, dir, tag,
                                                      count, value));
}

static int TIFFWriteDirectoryTagSrationalArray(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, float *value)
{
    return (TIFFWriteDirectoryTagCheckedSrationalArray(tif, ndir, dir, tag,
                                                       count, value));
}

/*-- Rational2Double: additional write functions */
static int TIFFWriteDirectoryTagRationalDoubleArray(TIFF *tif, uint32_t *ndir,
                                                    TIFFDirEntry *dir,
                                                    uint16_t tag,
                                                    uint32_t count,
                                                    double *value)
{
    return (TIFFWriteDirectoryTagCheckedRationalDoubleArray(tif, ndir, dir, tag,
                                                            count, value));
}

static int TIFFWriteDirectoryTagSrationalDoubleArray(TIFF *tif, uint32_t *ndir,
                                                     TIFFDirEntry *dir,
                                                     uint16_t tag,
                                                     uint32_t count,
                                                     double *value)
{
    return (TIFFWriteDirectoryTagCheckedSrationalDoubleArray(
        tif, ndir, dir, tag, count, value));
}

static int TIFFWriteDirectoryTagFloatArray(TIFF *tif, uint32_t *ndir,
                                           TIFFDirEntry *dir, uint16_t tag,
                                           uint32_t count, float *value)
{
    return (TIFFWriteDirectoryTagCheckedFloatArray(tif, ndir, dir, tag, count,
                                                   value));
}

static int TIFFWriteDirectoryTagDoubleArray(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t count, double *value)
{
    return (TIFFWriteDirectoryTagCheckedDoubleArray(tif, ndir, dir, tag, count,
                                                    value));
}

static int TIFFWriteDirectoryTagIfdArray(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir, uint16_t tag,
                                         uint32_t count, uint32_t *value)
{
    return (TIFFWriteDirectoryTagCheckedIfdArray(tif, ndir, dir, tag, count,
                                                 value));
}

static int TIFFWriteDirectoryTagShortLong(TIFF *tif, uint32_t *ndir,
                                          TIFFDirEntry *dir, uint16_t tag,
                                          uint32_t value)
{
    if (value <= 0xFFFF)
        return (TIFFWriteDirectoryTagCheckedShort(tif, ndir, dir, tag,
                                                  (uint16_t)value));
    else
        return (TIFFWriteDirectoryTagCheckedLong(tif, ndir, dir, tag, value));
}

static int _WriteAsType(TIFF *tif, uint64_t strile_size,
                        uint64_t uncompressed_threshold)
{
    const uint16_t compression = tif->tif_dir.td_compression;
    if (compression == COMPRESSION_NONE)
    {
        return strile_size > uncompressed_threshold;
    }
    else if (compression == COMPRESSION_JPEG ||
             compression == COMPRESSION_LZW ||
             compression == COMPRESSION_ADOBE_DEFLATE ||
             compression == COMPRESSION_DEFLATE ||
             compression == COMPRESSION_LZMA ||
             compression == COMPRESSION_LERC ||
             compression == COMPRESSION_ZSTD ||
             compression == COMPRESSION_WEBP || compression == COMPRESSION_JXL)
    {
        /* For a few select compression types, we assume that in the worst */
        /* case the compressed size will be 10 times the uncompressed size. */
        /* This is overly pessismistic ! */
        return strile_size >= uncompressed_threshold / 10;
    }
    return 1;
}

static int WriteAsLong8(TIFF *tif, uint64_t strile_size)
{
    return _WriteAsType(tif, strile_size, 0xFFFFFFFFU);
}

static int WriteAsLong4(TIFF *tif, uint64_t strile_size)
{
    return _WriteAsType(tif, strile_size, 0xFFFFU);
}

/************************************************************************/
/*                TIFFWriteDirectoryTagLongLong8Array()                 */
/*                                                                      */
/*      Write out LONG8 array and write a SHORT/LONG/LONG8 depending    */
/*      on strile size and Classic/BigTIFF mode.                        */
/************************************************************************/

static int TIFFWriteDirectoryTagLongLong8Array(TIFF *tif, uint32_t *ndir,
                                               TIFFDirEntry *dir, uint16_t tag,
                                               uint32_t count, uint64_t *value)
{
    static const char module[] = "TIFFWriteDirectoryTagLongLong8Array";
    int o;
    int write_aslong4;

    if (tif->tif_dir.td_deferstrilearraywriting)
    {
        if (dir == NULL)
        {
            /* This is just a counting pass to count IFD entries.
             * For deferstrilearraywriting no extra bytes will be written
             * into IFD space. */
            (*ndir)++;
            return 1;
        }
        return TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_NOTYPE, 0, 0,
                                         NULL);
    }

    if (tif->tif_flags & TIFF_BIGTIFF)
    {
        int write_aslong8 = 1;
        /* In the case of ByteCounts array, we may be able to write them on LONG
         * if the strip/tilesize is not too big. Also do that for count > 1 in
         * the case someone would want to create a single-strip file with a
         * growing height, in which case using LONG8 will be safer. */
        if (count > 1 && tag == TIFFTAG_STRIPBYTECOUNTS)
        {
            write_aslong8 = WriteAsLong8(tif, TIFFStripSize64(tif));
        }
        else if (count > 1 && tag == TIFFTAG_TILEBYTECOUNTS)
        {
            write_aslong8 = WriteAsLong8(tif, TIFFTileSize64(tif));
        }
        if (write_aslong8)
        {
            return TIFFWriteDirectoryTagCheckedLong8Array(tif, ndir, dir, tag,
                                                          count, value);
        }
    }

    write_aslong4 = 1;
    if (count > 1 && tag == TIFFTAG_STRIPBYTECOUNTS)
    {
        write_aslong4 = WriteAsLong4(tif, TIFFStripSize64(tif));
    }
    else if (count > 1 && tag == TIFFTAG_TILEBYTECOUNTS)
    {
        write_aslong4 = WriteAsLong4(tif, TIFFTileSize64(tif));
    }
    if (write_aslong4)
    {
        /*
        ** For classic tiff we want to verify everything is in range for LONG
        ** and convert to long format.
        */

        uint32_t *p = _TIFFmallocExt(tif, count * sizeof(uint32_t));
        uint32_t *q;
        uint64_t *ma;
        uint32_t mb;

        if (p == NULL)
        {
            TIFFErrorExtR(tif, module, "Out of memory");
            return (0);
        }

        for (q = p, ma = value, mb = 0; mb < count; ma++, mb++, q++)
        {
            if (*ma > 0xFFFFFFFF)
            {
                TIFFErrorExtR(tif, module,
                              "Attempt to write value larger than 0xFFFFFFFF "
                              "in LONG array.");
                _TIFFfreeExt(tif, p);
                return (0);
            }
            *q = (uint32_t)(*ma);
        }

        o = TIFFWriteDirectoryTagCheckedLongArray(tif, ndir, dir, tag, count,
                                                  p);
        _TIFFfreeExt(tif, p);
    }
    else
    {
        uint16_t *p = _TIFFmallocExt(tif, count * sizeof(uint16_t));
        uint16_t *q;
        uint64_t *ma;
        uint32_t mb;

        if (p == NULL)
        {
            TIFFErrorExtR(tif, module, "Out of memory");
            return (0);
        }

        for (q = p, ma = value, mb = 0; mb < count; ma++, mb++, q++)
        {
            if (*ma > 0xFFFF)
            {
                /* Should not happen normally given the check we did before */
                TIFFErrorExtR(tif, module,
                              "Attempt to write value larger than 0xFFFF in "
                              "SHORT array.");
                _TIFFfreeExt(tif, p);
                return (0);
            }
            *q = (uint16_t)(*ma);
        }

        o = TIFFWriteDirectoryTagCheckedShortArray(tif, ndir, dir, tag, count,
                                                   p);
        _TIFFfreeExt(tif, p);
    }

    return (o);
}

/************************************************************************/
/*                 TIFFWriteDirectoryTagIfdIfd8Array()                  */
/*                                                                      */
/*      Write either IFD8 or IFD array depending on file type.          */
/************************************************************************/

static int TIFFWriteDirectoryTagIfdIfd8Array(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint32_t count, uint64_t *value)
{
    static const char module[] = "TIFFWriteDirectoryTagIfdIfd8Array";
    uint64_t *ma;
    uint32_t mb;
    uint32_t *p;
    uint32_t *q;
    int o;

    /* We always write IFD8 for BigTIFF, no checking needed. */
    if (tif->tif_flags & TIFF_BIGTIFF)
        return TIFFWriteDirectoryTagCheckedIfd8Array(tif, ndir, dir, tag, count,
                                                     value);

    /*
    ** For classic tiff we want to verify everything is in range for IFD
    ** and convert to long format.
    */

    p = _TIFFmallocExt(tif, count * sizeof(uint32_t));
    if (p == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }

    for (q = p, ma = value, mb = 0; mb < count; ma++, mb++, q++)
    {
        if (*ma > 0xFFFFFFFF)
        {
            TIFFErrorExtR(tif, module,
                          "Attempt to write value larger than 0xFFFFFFFF in "
                          "Classic TIFF file.");
            _TIFFfreeExt(tif, p);
            return (0);
        }
        *q = (uint32_t)(*ma);
    }

    o = TIFFWriteDirectoryTagCheckedIfdArray(tif, ndir, dir, tag, count, p);
    _TIFFfreeExt(tif, p);

    return (o);
}

/*
 * Auxiliary function to determine the IFD data size to be written to the file.
 * The IFD data size is finally the size of the IFD tag entries plus the IFD
 * data that is written directly after the IFD tag entries.
 */
static void EvaluateIFDdatasizeWrite(TIFF *tif, uint32_t count,
                                     uint32_t typesize, uint32_t *ndir)
{
    uint64_t datalength = (uint64_t)count * typesize;
    if (datalength > ((tif->tif_flags & TIFF_BIGTIFF) ? 0x8U : 0x4U))
    {
        /* LibTIFF increments write address to an even offset, thus datalength
         * written is also incremented. */
        if (datalength & 1)
            datalength++;
        tif->tif_dir.td_dirdatasize_write += datalength;
    }
    (*ndir)++;
}

static int TIFFWriteDirectoryTagColormap(TIFF *tif, uint32_t *ndir,
                                         TIFFDirEntry *dir)
{
    static const char module[] = "TIFFWriteDirectoryTagColormap";
    uint32_t m;
    uint16_t *n;
    int o;
    m = (1 << tif->tif_dir.td_bitspersample);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, 3 * m, sizeof(uint16_t), ndir);
        return 1;
    }

    n = _TIFFmallocExt(tif, 3 * m * sizeof(uint16_t));
    if (n == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }
    _TIFFmemcpy(&n[0], tif->tif_dir.td_colormap[0], m * sizeof(uint16_t));
    _TIFFmemcpy(&n[m], tif->tif_dir.td_colormap[1], m * sizeof(uint16_t));
    _TIFFmemcpy(&n[2 * m], tif->tif_dir.td_colormap[2], m * sizeof(uint16_t));
    o = TIFFWriteDirectoryTagCheckedShortArray(tif, ndir, dir, TIFFTAG_COLORMAP,
                                               3 * m, n);
    _TIFFfreeExt(tif, n);
    return (o);
}

static int TIFFWriteDirectoryTagTransferfunction(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir)
{
    static const char module[] = "TIFFWriteDirectoryTagTransferfunction";
    uint32_t m;
    uint16_t n;
    uint16_t *o;
    int p;
    /* TIFFTAG_TRANSFERFUNCTION expects (1 or 3) pointer to arrays with
     *  (1 << BitsPerSample) * uint16_t values.
     */
    m = (1 << tif->tif_dir.td_bitspersample);
    /* clang-format off */
    n = (tif->tif_dir.td_samplesperpixel - tif->tif_dir.td_extrasamples) > 1 ? 3 : 1;
    /* clang-format on */

    /* Check for proper number of transferfunctions */
    for (int i = 0; i < n; i++)
    {
        if (tif->tif_dir.td_transferfunction[i] == NULL)
        {
            TIFFWarningExtR(tif, module,
                            "Too few TransferFunctions provided. Tag "
                            "not written to file");
            return (1); /* Not an error; only tag is not written. */
        }
    }
    /*
     * Check if the table can be written as a single column,
     * or if it must be written as 3 columns.  Note that we
     * write a 3-column tag if there are 2 samples/pixel and
     * a single column of data won't suffice--hmm.
     */
    if (n == 3)
    {
        if (!_TIFFmemcmp(tif->tif_dir.td_transferfunction[0],
                         tif->tif_dir.td_transferfunction[2],
                         m * sizeof(uint16_t)) &&
            !_TIFFmemcmp(tif->tif_dir.td_transferfunction[0],
                         tif->tif_dir.td_transferfunction[1],
                         m * sizeof(uint16_t)))
            n = 1;
    }
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, n * m, 2, ndir);
        return 1;
    }

    o = _TIFFmallocExt(tif, n * m * sizeof(uint16_t));
    if (o == NULL)
    {
        TIFFErrorExtR(tif, module, "Out of memory");
        return (0);
    }
    _TIFFmemcpy(&o[0], tif->tif_dir.td_transferfunction[0],
                m * sizeof(uint16_t));
    if (n > 1)
        _TIFFmemcpy(&o[m], tif->tif_dir.td_transferfunction[1],
                    m * sizeof(uint16_t));
    if (n > 2)
        _TIFFmemcpy(&o[2 * m], tif->tif_dir.td_transferfunction[2],
                    m * sizeof(uint16_t));
    p = TIFFWriteDirectoryTagCheckedShortArray(
        tif, ndir, dir, TIFFTAG_TRANSFERFUNCTION, n * m, o);
    _TIFFfreeExt(tif, o);
    return (p);
}

static int TIFFWriteDirectoryTagSubifd(TIFF *tif, uint32_t *ndir,
                                       TIFFDirEntry *dir)
{
    static const char module[] = "TIFFWriteDirectoryTagSubifd";
    uint64_t m;
    int n;
    if (tif->tif_dir.td_nsubifd == 0)
        return (1);
    m = tif->tif_dataoff;
    if (!(tif->tif_flags & TIFF_BIGTIFF))
    {
        uint32_t *o;
        uint64_t *pa;
        uint32_t *pb;
        uint16_t p;
        o = _TIFFmallocExt(tif, tif->tif_dir.td_nsubifd * sizeof(uint32_t));
        if (o == NULL)
        {
            TIFFErrorExtR(tif, module, "Out of memory");
            return (0);
        }
        pa = tif->tif_dir.td_subifd;
        pb = o;
        for (p = 0; p < tif->tif_dir.td_nsubifd; p++)
        {
            assert(pa != 0);

            /* Could happen if an classicTIFF has a SubIFD of type LONG8 (which
             * is illegal) */
            if (*pa > 0xFFFFFFFFUL)
            {
                TIFFErrorExtR(tif, module, "Illegal value for SubIFD tag");
                _TIFFfreeExt(tif, o);
                return (0);
            }
            *pb++ = (uint32_t)(*pa++);
        }
        n = TIFFWriteDirectoryTagCheckedIfdArray(tif, ndir, dir, TIFFTAG_SUBIFD,
                                                 tif->tif_dir.td_nsubifd, o);
        _TIFFfreeExt(tif, o);
    }
    else
        n = TIFFWriteDirectoryTagCheckedIfd8Array(
            tif, ndir, dir, TIFFTAG_SUBIFD, tif->tif_dir.td_nsubifd,
            tif->tif_dir.td_subifd);

    if (dir == NULL)
        /* Just have evaluated IFD data size and incremented ndir
         * above in sub-functions. */
        return (n);

    if (!n)
        return (0);
    /*
     * Total hack: if this directory includes a SubIFD
     * tag then force the next <n> directories to be
     * written as ``sub directories'' of this one.  This
     * is used to write things like thumbnails and
     * image masks that one wants to keep out of the
     * normal directory linkage access mechanism.
     */
    tif->tif_flags |= TIFF_INSUBIFD;
    tif->tif_nsubifd = tif->tif_dir.td_nsubifd;
    if (tif->tif_dir.td_nsubifd == 1)
        tif->tif_subifdoff = 0;
    else
        tif->tif_subifdoff = m;
    return (1);
}

static int TIFFWriteDirectoryTagCheckedAscii(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint32_t count, char *value)
{
    assert(sizeof(char) == 1);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 1, ndir);
        return 1;
    }
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_ASCII, count,
                                      count, value));
}

static int TIFFWriteDirectoryTagCheckedUndefinedArray(TIFF *tif, uint32_t *ndir,
                                                      TIFFDirEntry *dir,
                                                      uint16_t tag,
                                                      uint32_t count,
                                                      uint8_t *value)
{
    assert(sizeof(uint8_t) == 1);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 1, ndir);
        return 1;
    }
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_UNDEFINED,
                                      count, count, value));
}

static int TIFFWriteDirectoryTagCheckedByteArray(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir,
                                                 uint16_t tag, uint32_t count,
                                                 uint8_t *value)
{
    assert(sizeof(uint8_t) == 1);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 1, ndir);
        return 1;
    }
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_BYTE, count,
                                      count, value));
}

static int TIFFWriteDirectoryTagCheckedSbyteArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  int8_t *value)
{
    assert(sizeof(int8_t) == 1);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 1, ndir);
        return 1;
    }
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_SBYTE, count,
                                      count, value));
}

static int TIFFWriteDirectoryTagCheckedShort(TIFF *tif, uint32_t *ndir,
                                             TIFFDirEntry *dir, uint16_t tag,
                                             uint16_t value)
{
    uint16_t m;
    assert(sizeof(uint16_t) == 2);
    if (dir == NULL)
    {
        /* No additional data to IFD data size just increment ndir. */
        (*ndir)++;
        return 1;
    }
    m = value;
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabShort(&m);
    return (
        TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_SHORT, 1, 2, &m));
}

static int TIFFWriteDirectoryTagCheckedShortArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  uint16_t *value)
{
    assert(count < 0x80000000);
    assert(sizeof(uint16_t) == 2);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 2, ndir);
        return 1;
    }
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabArrayOfShort(value, count);
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_SHORT, count,
                                      count * 2, value));
}

static int TIFFWriteDirectoryTagCheckedSshortArray(TIFF *tif, uint32_t *ndir,
                                                   TIFFDirEntry *dir,
                                                   uint16_t tag, uint32_t count,
                                                   int16_t *value)
{
    assert(count < 0x80000000);
    assert(sizeof(int16_t) == 2);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 2, ndir);
        return 1;
    }
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabArrayOfShort((uint16_t *)value, count);
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_SSHORT, count,
                                      count * 2, value));
}

static int TIFFWriteDirectoryTagCheckedLong(TIFF *tif, uint32_t *ndir,
                                            TIFFDirEntry *dir, uint16_t tag,
                                            uint32_t value)
{
    uint32_t m;
    assert(sizeof(uint32_t) == 4);
    if (dir == NULL)
    {
        /* No additional data to IFD data size just increment ndir. */
        (*ndir)++;
        return 1;
    }
    m = value;
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabLong(&m);
    return (
        TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_LONG, 1, 4, &m));
}

static int TIFFWriteDirectoryTagCheckedLongArray(TIFF *tif, uint32_t *ndir,
                                                 TIFFDirEntry *dir,
                                                 uint16_t tag, uint32_t count,
                                                 uint32_t *value)
{
    assert(count < 0x40000000);
    assert(sizeof(uint32_t) == 4);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 4, ndir);
        return 1;
    }
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabArrayOfLong(value, count);
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_LONG, count,
                                      count * 4, value));
}

static int TIFFWriteDirectoryTagCheckedSlongArray(TIFF *tif, uint32_t *ndir,
                                                  TIFFDirEntry *dir,
                                                  uint16_t tag, uint32_t count,
                                                  int32_t *value)
{
    assert(count < 0x40000000);
    assert(sizeof(int32_t) == 4);
    if (dir == NULL) /* Just evaluate IFD data size and increment ndir. */
    {
        EvaluateIFDdatasizeWrite(tif, count, 4, ndir);
        return 1;
    }
    if (tif->tif_flags & TIFF_SWAB)
        TIFFSwabArrayOfLong((uint32_t *)value, count);
    return (TIFFWriteDirectoryTagData(tif, ndir, dir, tag, TIFF_SLONG, count,
                                      count * 4, value)
... [Content truncated - file is very large]
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
- **to()**: A function/method defined in this file
- **that()**: A function/method defined in this file
- **which()**: A function/method defined in this file
- **HAVE_IEEEFP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `tiffiop.h`
- `float.h`
- `math.h`

**Python Imports:**
- `IFD`
- `file`
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

