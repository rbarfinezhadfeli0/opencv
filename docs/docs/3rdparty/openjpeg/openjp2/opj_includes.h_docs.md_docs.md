# Documentation for `docs/3rdparty/openjpeg/openjp2/opj_includes.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/opj_includes.h_docs.md`
- **File Name**: `opj_includes.h_docs.md`
- **File Size**: 11,447 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/opj_includes.h_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/opj_includes.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/opj_includes.h`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/opj_includes.h`
- **File Name**: `opj_includes.h`
- **File Size**: 7,268 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openjpeg/openjp2/opj_includes.h](../../../3rdparty/openjpeg/openjp2/opj_includes.h)

## Purpose and Role

This file is located in the `3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * The copyright in this software is being made available under the 2-clauses
 * BSD License, included below. This software may be subject to other third
 * party and contributor rights, including patent rights, and no such rights
 * are granted under this license.
 *
 * Copyright (c) 2005, Herve Drolon, FreeImage Team
 * Copyright (c) 2008, 2011-2012, Centre National d'Etudes Spatiales (CNES), FR
 * Copyright (c) 2012, CS Systemes d'Information, France
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS `AS IS'
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
 * POSSIBILITY OF SUCH DAMAGE.
 */
#ifndef OPJ_INCLUDES_H
#define OPJ_INCLUDES_H

/*
 * This must be included before any system headers,
 * since they can react to macro defined there
 */
#include "opj_config_private.h"

/*
 ==========================================================
   Standard includes used by the library
 ==========================================================
*/
#include <memory.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <float.h>
#include <time.h>
#include <stdio.h>
#include <stdarg.h>
#include <ctype.h>
#include <assert.h>
#include <limits.h>
#include <stdint.h>
#include <inttypes.h>

/*
  Use fseeko() and ftello() if they are available since they use
  'off_t' rather than 'long'.  It is wrong to use fseeko() and
  ftello() only on systems with special LFS support since some systems
  (e.g. FreeBSD) support a 64-bit off_t by default.
*/
#if defined(OPJ_HAVE_FSEEKO) && !defined(fseek)
#  define fseek  fseeko
#  define ftell  ftello
#endif


#if defined(WIN32) && !defined(Windows95) && !defined(__BORLANDC__) && \
  !(defined(_MSC_VER) && _MSC_VER < 1400) && \
  !(defined(__MINGW32__) && __MSVCRT_VERSION__ < 0x800)
/*
  Windows '95 and Borland C do not support _lseeki64
  Visual Studio does not support _fseeki64 and _ftelli64 until the 2005 release.
  Without these interfaces, files over 2GB in size are not supported for Windows.
*/
#  define OPJ_FSEEK(stream,offset,whence) _fseeki64(stream,/* __int64 */ offset,whence)
#  define OPJ_FSTAT(fildes,stat_buff) _fstati64(fildes,/* struct _stati64 */ stat_buff)
#  define OPJ_FTELL(stream) /* __int64 */ _ftelli64(stream)
#  define OPJ_STAT_STRUCT_T struct _stati64
#  define OPJ_STAT(path,stat_buff) _stati64(path,/* struct _stati64 */ stat_buff)
#else
#  define OPJ_FSEEK(stream,offset,whence) fseek(stream,offset,whence)
#  define OPJ_FSTAT(fildes,stat_buff) fstat(fildes,stat_buff)
#  define OPJ_FTELL(stream) ftell(stream)
#  define OPJ_STAT_STRUCT_T struct stat
#  define OPJ_STAT(path,stat_buff) stat(path,stat_buff)
#endif


/*
 ==========================================================
   OpenJPEG interface
 ==========================================================
 */
#include "openjpeg.h"

/*
 ==========================================================
   OpenJPEG modules
 ==========================================================
*/

/* Are restricted pointers available? (C99) */
#if (__STDC_VERSION__ >= 199901L)
#define OPJ_RESTRICT restrict
#else
/* Not a C99 compiler */
#if defined(__GNUC__)
#define OPJ_RESTRICT __restrict__

/*
  vc14 (2015) outputs wrong results.
  Need to check OPJ_RESTRICT usage (or a bug in vc14)
    #elif defined(_MSC_VER) && (_MSC_VER >= 1400)
        #define OPJ_RESTRICT __restrict
*/
#else
#define OPJ_RESTRICT /* restrict */
#endif
#endif

#ifdef __has_attribute
#if __has_attribute(no_sanitize)
#define OPJ_NOSANITIZE(kind) __attribute__((no_sanitize(kind)))
#endif
#endif
#ifndef OPJ_NOSANITIZE
#define OPJ_NOSANITIZE(kind)
#endif


/* MSVC before 2013 and Borland C do not have lrintf */
#if defined(_MSC_VER)
#include <intrin.h>
static INLINE long opj_lrintf(float f)
{
#ifdef _M_X64
    return _mm_cvt_ss2si(_mm_load_ss(&f));

    /* commented out line breaks many tests */
    /* return (long)((f>0.0f) ? (f + 0.5f):(f -0.5f)); */
#elif defined(_M_IX86)
    int i;
    _asm{
        fld f
        fistp i
    };

    return i;
#else
    return (long)((f>0.0f) ? (f + 0.5f) : (f - 0.5f));
#endif
}
#elif defined(__BORLANDC__)
static INLINE long opj_lrintf(float f)
{
#ifdef _M_X64
    return (long)((f > 0.0f) ? (f + 0.5f) : (f - 0.5f));
#else
    int i;

    _asm {
        fld f
        fistp i
    };

    return i;
#endif
}
#else
static INLINE long opj_lrintf(float f)
{
    return lrintf(f);
}
#endif

#if defined(_MSC_VER) && (_MSC_VER < 1400)
#define vsnprintf _vsnprintf
#endif

/* MSVC x86 is really bad at doing int64 = int32 * int32 on its own. Use intrinsic. */
#if defined(_MSC_VER) && (_MSC_VER >= 1400) && !defined(__INTEL_COMPILER) && defined(_M_IX86)
#   include <intrin.h>
#   pragma intrinsic(__emul)
#endif

/* Apparently Visual Studio doesn't define __SSE__ / __SSE2__ macros */
#if defined(_M_X64)
/* Intel 64bit support SSE and SSE2 */
#   ifndef __SSE__
#       define __SSE__ 1
#   endif
#   ifndef __SSE2__
#       define __SSE2__ 1
#   endif
#endif

/* For x86, test the value of the _M_IX86_FP macro. */
/* See https://msdn.microsoft.com/en-us/library/b0084kay.aspx */
#if defined(_M_IX86_FP)
#   if _M_IX86_FP >= 1
#       ifndef __SSE__
#           define __SSE__ 1
#       endif
#   endif
#   if _M_IX86_FP >= 2
#       ifndef __SSE2__
#           define __SSE2__ 1
#       endif
#   endif
#endif

/* Type to use for bit-fields in internal headers */
typedef unsigned int OPJ_BITFIELD;

#define OPJ_UNUSED(x) (void)x

#include "opj_clock.h"
#include "opj_malloc.h"
#include "event.h"
#include "function_list.h"
#include "bio.h"
#include "cio.h"

#include "thread.h"
#include "tls_keys.h"

#include "image.h"
#include "invert.h"
#include "j2k.h"
#include "jp2.h"

#include "mqc.h"
#include "bio.h"

#include "pi.h"
#include "tgt.h"
#include "tcd.h"
#include "t1.h"
#include "dwt.h"
#include "t2.h"
#include "mct.h"
#include "opj_intmath.h"
#include "sparse_array.h"

#ifdef USE_JPIP
#include "cidx_manager.h"
#include "indexbox_manager.h"
#endif

/* JPWL>> */
#ifdef USE_JPWL
#include "openjpwl/jpwl.h"
#endif /* USE_JPWL */
/* <<JPWL */

/* V2 */
#include "opj_codec.h"


#endif /* OPJ_INCLUDES_H */
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

- **_stati64**: A class/struct defined in this file
- **stat**: A class/struct defined in this file

### Functions and Methods

- **__SSE__()**: A function/method defined in this file
- **OPJ_NOSANITIZE()**: A function/method defined in this file
- **unsigned()**: A function/method defined in this file
- **USE_JPIP()**: A function/method defined in this file
- **USE_JPWL()**: A function/method defined in this file
- **OPJ_INCLUDES_H()**: A function/method defined in this file
- **_M_X64()**: A function/method defined in this file
- **__has_attribute()**: A function/method defined in this file
- **__SSE2__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `j2k.h`
- `ctype.h`
- `openjpwl/jpwl.h`
- `intrin.h`
- `t2.h`
- `string.h`
- `sparse_array.h`
- `stdint.h`
- `time.h`
- `memory.h`
- `assert.h`
- `mct.h`
- `opj_codec.h`
- `tcd.h`
- `opj_malloc.h`
- `indexbox_manager.h`
- `pi.h`
- `limits.h`
- `event.h`
- `openjpeg.h`
- `thread.h`
- `bio.h`
- `opj_config_private.h`
- `t1.h`
- `cio.h`
- `function_list.h`
- `jp2.h`
- `cidx_manager.h`
- `stdio.h`
- `inttypes.h`
- `stdlib.h`
- `image.h`
- `invert.h`
- `stdarg.h`
- `dwt.h`
- `float.h`
- `opj_intmath.h`
- `tls_keys.h`
- `math.h`
- `opj_clock.h`
- `mqc.h`
- `tgt.h`


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

