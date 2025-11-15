# Documentation for `3rdparty/zlib-ng/compress.c`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/compress.c`
- **File Name**: `compress.c`
- **File Size**: 3,891 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/zlib-ng/compress.c](../../3rdparty/zlib-ng/compress.c)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* compress.c -- compress a memory buffer
 * Copyright (C) 1995-2005, 2014, 2016 Jean-loup Gailly, Mark Adler
 * For conditions of distribution and use, see copyright notice in zlib.h
 */

#include "zbuild.h"
#include "zutil.h"

/* ===========================================================================
 *  Architecture-specific hooks.
 */
#ifdef S390_DFLTCC_DEFLATE
#  include "arch/s390/dfltcc_common.h"
#else
/* Returns the upper bound on compressed data length based on uncompressed data length, assuming default settings.
 * Zero means that arch-specific deflation code behaves identically to the regular zlib-ng algorithms. */
#  define DEFLATE_BOUND_COMPLEN(source_len) 0
#endif

/* ===========================================================================
     Compresses the source buffer into the destination buffer. The level
   parameter has the same meaning as in deflateInit.  sourceLen is the byte
   length of the source buffer. Upon entry, destLen is the total size of the
   destination buffer, which must be at least 0.1% larger than sourceLen plus
   12 bytes. Upon exit, destLen is the actual size of the compressed buffer.

     compress2 returns Z_OK if success, Z_MEM_ERROR if there was not enough
   memory, Z_BUF_ERROR if there was not enough room in the output buffer,
   Z_STREAM_ERROR if the level parameter is invalid.
*/
int Z_EXPORT PREFIX(compress2)(unsigned char *dest, z_uintmax_t *destLen, const unsigned char *source,
                        z_uintmax_t sourceLen, int level) {
    PREFIX3(stream) stream;
    int err;
    const unsigned int max = (unsigned int)-1;
    z_size_t left;

    left = *destLen;
    *destLen = 0;

    stream.zalloc = NULL;
    stream.zfree = NULL;
    stream.opaque = NULL;

    err = PREFIX(deflateInit)(&stream, level);
    if (err != Z_OK)
        return err;

    stream.next_out = dest;
    stream.avail_out = 0;
    stream.next_in = (z_const unsigned char *)source;
    stream.avail_in = 0;

    do {
        if (stream.avail_out == 0) {
            stream.avail_out = left > (unsigned long)max ? max : (unsigned int)left;
            left -= stream.avail_out;
        }
        if (stream.avail_in == 0) {
            stream.avail_in = sourceLen > (unsigned long)max ? max : (unsigned int)sourceLen;
            sourceLen -= stream.avail_in;
        }
        err = PREFIX(deflate)(&stream, sourceLen ? Z_NO_FLUSH : Z_FINISH);
    } while (err == Z_OK);

    *destLen = stream.total_out;
    PREFIX(deflateEnd)(&stream);
    return err == Z_STREAM_END ? Z_OK : err;
}

/* ===========================================================================
 */
int Z_EXPORT PREFIX(compress)(unsigned char *dest, z_uintmax_t *destLen, const unsigned char *source, z_uintmax_t sourceLen) {
    return PREFIX(compress2)(dest, destLen, source, sourceLen, Z_DEFAULT_COMPRESSION);
}

/* ===========================================================================
   If the default memLevel or windowBits for deflateInit() is changed, then
   this function needs to be updated.
 */
z_uintmax_t Z_EXPORT PREFIX(compressBound)(z_uintmax_t sourceLen) {
    z_uintmax_t complen = DEFLATE_BOUND_COMPLEN(sourceLen);

    if (complen > 0)
        /* Architecture-specific code provided an upper bound. */
        return complen + ZLIB_WRAPLEN;

#ifndef NO_QUICK_STRATEGY
    return sourceLen                       /* The source size itself */
      + (sourceLen == 0 ? 1 : 0)           /* Always at least one byte for any input */
      + (sourceLen < 9 ? 1 : 0)            /* One extra byte for lengths less than 9 */
      + DEFLATE_QUICK_OVERHEAD(sourceLen)  /* Source encoding overhead, padded to next full byte */
      + DEFLATE_BLOCK_OVERHEAD             /* Deflate block overhead bytes */
      + ZLIB_WRAPLEN;                      /* zlib wrapper */
#else
    return sourceLen + (sourceLen >> 4) + 7 + ZLIB_WRAPLEN;
#endif
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

- **needs()**: A function/method defined in this file
- **NO_QUICK_STRATEGY()**: A function/method defined in this file
- **S390_DFLTCC_DEFLATE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `zbuild.h`
- `zutil.h`


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

