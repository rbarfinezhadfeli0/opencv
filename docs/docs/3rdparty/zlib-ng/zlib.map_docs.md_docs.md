# Documentation for `docs/3rdparty/zlib-ng/zlib.map_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/zlib.map_docs.md`
- **File Name**: `zlib.map_docs.md`
- **File Size**: 1,947 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/zlib.map_docs.md](../../../docs/3rdparty/zlib-ng/zlib.map_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/zlib.map`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/zlib.map`
- **File Name**: `zlib.map`
- **File Size**: 1,425 bytes
- **File Type**: .map
- **Link to Source**: [3rdparty/zlib-ng/zlib.map](../../3rdparty/zlib-ng/zlib.map)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
ZLIB_1.2.0 {
  global:
    compressBound;
    deflateBound;
    inflateBack;
    inflateBackEnd;
    inflateBackInit_;
    inflateCopy;
  local:
    deflate_copyright;
    inflate_copyright;
    zcalloc;
    zcfree;
    z_errmsg;
    gz_error;
    gz_intmax;
    _*;
};

ZLIB_1.2.0.2 {
    gzclearerr;
    gzungetc;
    zlibCompileFlags;
} ZLIB_1.2.0;

ZLIB_1.2.0.8 {
    deflatePrime;
} ZLIB_1.2.0.2;

ZLIB_1.2.2 {
    adler32_combine;
    crc32_combine;
    deflateSetHeader;
    inflateGetHeader;
} ZLIB_1.2.0.8;

ZLIB_1.2.2.3 {
    deflateTune;
    gzdirect;
} ZLIB_1.2.2;

ZLIB_1.2.2.4 {
    inflatePrime;
} ZLIB_1.2.2.3;

ZLIB_1.2.3.3 {
    adler32_combine64;
    crc32_combine64;
    gzopen64;
    gzseek64;
    gztell64;
    inflateUndermine;
} ZLIB_1.2.2.4;

ZLIB_1.2.3.4 {
    inflateReset2;
    inflateMark;
} ZLIB_1.2.3.3;

ZLIB_1.2.3.5 {
    gzbuffer;
    gzoffset;
    gzoffset64;
    gzclose_r;
    gzclose_w;
} ZLIB_1.2.3.4;

ZLIB_1.2.5.1 {
    deflatePending;
} ZLIB_1.2.3.5;

ZLIB_1.2.5.2 {
    deflateResetKeep;
    gzgetc_;
    inflateResetKeep;
} ZLIB_1.2.5.1;

ZLIB_1.2.7.1 {
    inflateGetDictionary;
    gzvprintf;
} ZLIB_1.2.5.2;

ZLIB_1.2.9 {
    inflateCodesUsed;
    inflateValidate;
    uncompress2;
    gzfread;
    gzfwrite;
    deflateGetDictionary;
    adler32_z;
    crc32_z;
} ZLIB_1.2.7.1;

ZLIB_1.2.12 {
    crc32_combine_gen;
    crc32_combine_gen64;
    crc32_combine_op;
} ZLIB_1.2.9;

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

