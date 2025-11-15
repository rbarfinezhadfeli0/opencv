# Documentation for `docs/3rdparty/libjpeg-turbo/jconfigint.h.in_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/jconfigint.h.in_docs.md`
- **File Name**: `jconfigint.h.in_docs.md`
- **File Size**: 2,385 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/jconfigint.h.in_docs.md](../../../docs/3rdparty/libjpeg-turbo/jconfigint.h.in_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/jconfigint.h.in`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/jconfigint.h.in`
- **File Name**: `jconfigint.h.in`
- **File Size**: 1,799 bytes
- **File Type**: .in
- **Link to Source**: [3rdparty/libjpeg-turbo/jconfigint.h.in](../../3rdparty/libjpeg-turbo/jconfigint.h.in)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
/* libjpeg-turbo build number */
#define BUILD  "@BUILD@"

/* How to hide global symbols. */
#define HIDDEN  @HIDDEN@

/* Compiler's inline keyword */
#undef inline

/* How to obtain function inlining. */
#define INLINE  @INLINE@

/* How to obtain thread-local storage */
#define THREAD_LOCAL  @THREAD_LOCAL@

/* Define to the full name of this package. */
#define PACKAGE_NAME  "@CMAKE_PROJECT_NAME@"

/* Version number of package */
#define VERSION  "@VERSION@"

/* The size of `size_t', as computed by sizeof. */
#define SIZEOF_SIZE_T  @SIZE_T@

/* Define if your compiler has __builtin_ctzl() and sizeof(unsigned long) == sizeof(size_t). */
#cmakedefine HAVE_BUILTIN_CTZL

/* Define to 1 if you have the <intrin.h> header file. */
#cmakedefine HAVE_INTRIN_H

#if defined(_MSC_VER) && defined(HAVE_INTRIN_H)
#if (SIZEOF_SIZE_T == 8)
#define HAVE_BITSCANFORWARD64
#elif (SIZEOF_SIZE_T == 4)
#define HAVE_BITSCANFORWARD
#endif
#endif

#if defined(__has_attribute)
#if __has_attribute(fallthrough)
#define FALLTHROUGH  __attribute__((fallthrough));
#else
#define FALLTHROUGH
#endif
#else
#define FALLTHROUGH
#endif

/*
 * Define BITS_IN_JSAMPLE as either
 *   8   for 8-bit sample values (the usual setting)
 *   12  for 12-bit sample values
 * Only 8 and 12 are legal data precisions for lossy JPEG according to the
 * JPEG standard, and the IJG code does not support anything else!
 */

#ifndef BITS_IN_JSAMPLE
#define BITS_IN_JSAMPLE  8      /* use 8 or 12 */
#endif

#undef C_ARITH_CODING_SUPPORTED
#undef D_ARITH_CODING_SUPPORTED
#undef WITH_SIMD

#if BITS_IN_JSAMPLE == 8

/* Support arithmetic encoding */
#cmakedefine C_ARITH_CODING_SUPPORTED 1

/* Support arithmetic decoding */
#cmakedefine D_ARITH_CODING_SUPPORTED 1

/* Use accelerated SIMD routines. */
#cmakedefine WITH_SIMD 1

#endif

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

