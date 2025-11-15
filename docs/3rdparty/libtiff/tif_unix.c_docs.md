# Documentation for `3rdparty/libtiff/tif_unix.c`

## File Metadata

- **Full Path**: `3rdparty/libtiff/tif_unix.c`
- **File Name**: `tif_unix.c`
- **File Size**: 9,748 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libtiff/tif_unix.c](../../3rdparty/libtiff/tif_unix.c)

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
 * TIFF Library UNIX-specific Routines. These are should also work with the
 * Windows Common RunTime Library.
 */

#ifdef TIFF_DO_NOT_USE_NON_EXT_ALLOC_FUNCTIONS
#undef TIFF_DO_NOT_USE_NON_EXT_ALLOC_FUNCTIONS
#endif

#include "tif_config.h"

#ifdef HAVE_SYS_TYPES_H
#include <sys/types.h>
#endif

#include <errno.h>

#include <stdarg.h>
#include <stdlib.h>
#include <sys/stat.h>

#ifdef HAVE_UNISTD_H
#include <unistd.h>
#endif

#ifdef HAVE_FCNTL_H
#include <fcntl.h>
#endif

#ifdef HAVE_IO_H
#include <io.h>
#endif

#include "tiffiop.h"

#define TIFF_IO_MAX 2147483647U

typedef union fd_as_handle_union
{
    int fd;
    thandle_t h;
} fd_as_handle_union_t;

static tmsize_t _tiffReadProc(thandle_t fd, void *buf, tmsize_t size)
{
    fd_as_handle_union_t fdh;
    const size_t bytes_total = (size_t)size;
    size_t bytes_read;
    tmsize_t count = -1;
    if ((tmsize_t)bytes_total != size)
    {
        errno = EINVAL;
        return (tmsize_t)-1;
    }
    fdh.h = fd;
    for (bytes_read = 0; bytes_read < bytes_total; bytes_read += count)
    {
        char *buf_offset = (char *)buf + bytes_read;
        size_t io_size = bytes_total - bytes_read;
        if (io_size > TIFF_IO_MAX)
            io_size = TIFF_IO_MAX;
        /* Below is an obvious false positive of Coverity Scan */
        /* coverity[overflow_sink] */
        count = read(fdh.fd, buf_offset, (TIFFIOSize_t)io_size);
        if (count <= 0)
            break;
    }
    if (count < 0)
        return (tmsize_t)-1;
    /* Silence Coverity Scan warning about unsigned to signed underflow. */
    /* coverity[return_overflow:SUPPRESS] */
    return (tmsize_t)bytes_read;
}

static tmsize_t _tiffWriteProc(thandle_t fd, void *buf, tmsize_t size)
{
    fd_as_handle_union_t fdh;
    const size_t bytes_total = (size_t)size;
    size_t bytes_written;
    tmsize_t count = -1;
    if ((tmsize_t)bytes_total != size)
    {
        errno = EINVAL;
        return (tmsize_t)-1;
    }
    fdh.h = fd;
    for (bytes_written = 0; bytes_written < bytes_total; bytes_written += count)
    {
        const char *buf_offset = (char *)buf + bytes_written;
        size_t io_size = bytes_total - bytes_written;
        if (io_size > TIFF_IO_MAX)
            io_size = TIFF_IO_MAX;
        /* Below is an obvious false positive of Coverity Scan */
        /* coverity[overflow_sink] */
        count = write(fdh.fd, buf_offset, (TIFFIOSize_t)io_size);
        if (count <= 0)
            break;
    }
    if (count < 0)
        return (tmsize_t)-1;
    /* Silence Coverity Scan warning about unsigned to signed underflow. */
    /* coverity[return_overflow:SUPPRESS] */
    return (tmsize_t)bytes_written;
    /* return ((tmsize_t) write(fdh.fd, buf, bytes_total)); */
}

static uint64_t _tiffSeekProc(thandle_t fd, uint64_t off, int whence)
{
    fd_as_handle_union_t fdh;
    _TIFF_off_t off_io = (_TIFF_off_t)off;
    if ((uint64_t)off_io != off)
    {
        errno = EINVAL;
        return (uint64_t)-1; /* this is really gross */
    }
    fdh.h = fd;
    return ((uint64_t)_TIFF_lseek_f(fdh.fd, off_io, whence));
}

static int _tiffCloseProc(thandle_t fd)
{
    fd_as_handle_union_t fdh;
    fdh.h = fd;
    return (close(fdh.fd));
}

static uint64_t _tiffSizeProc(thandle_t fd)
{
    _TIFF_stat_s sb;
    fd_as_handle_union_t fdh;
    fdh.h = fd;
    if (_TIFF_fstat_f(fdh.fd, &sb) < 0)
        return (0);
    else
        return ((uint64_t)sb.st_size);
}

#ifdef HAVE_MMAP
#include <sys/mman.h>

static int _tiffMapProc(thandle_t fd, void **pbase, toff_t *psize)
{
    uint64_t size64 = _tiffSizeProc(fd);
    tmsize_t sizem = (tmsize_t)size64;
    if (size64 && (uint64_t)sizem == size64)
    {
        fd_as_handle_union_t fdh;
        fdh.h = fd;
        *pbase =
            (void *)mmap(0, (size_t)sizem, PROT_READ, MAP_SHARED, fdh.fd, 0);
        if (*pbase != (void *)-1)
        {
            *psize = (tmsize_t)sizem;
            return (1);
        }
    }
    return (0);
}

static void _tiffUnmapProc(thandle_t fd, void *base, toff_t size)
{
    (void)fd;
    (void)munmap(base, (off_t)size);
}
#else  /* !HAVE_MMAP */
static int _tiffMapProc(thandle_t fd, void **pbase, toff_t *psize)
{
    (void)fd;
    (void)pbase;
    (void)psize;
    return (0);
}

static void _tiffUnmapProc(thandle_t fd, void *base, toff_t size)
{
    (void)fd;
    (void)base;
    (void)size;
}
#endif /* !HAVE_MMAP */

/*
 * Open a TIFF file descriptor for read/writing.
 */
TIFF *TIFFFdOpen(int fd, const char *name, const char *mode)
{
    return TIFFFdOpenExt(fd, name, mode, NULL);
}

TIFF *TIFFFdOpenExt(int fd, const char *name, const char *mode,
                    TIFFOpenOptions *opts)
{
    TIFF *tif;

    fd_as_handle_union_t fdh;
    fdh.fd = fd;
    tif = TIFFClientOpenExt(name, mode, fdh.h, _tiffReadProc, _tiffWriteProc,
                            _tiffSeekProc, _tiffCloseProc, _tiffSizeProc,
                            _tiffMapProc, _tiffUnmapProc, opts);
    if (tif)
        tif->tif_fd = fd;
    return (tif);
}

/*
 * Open a TIFF file for read/writing.
 */
TIFF *TIFFOpen(const char *name, const char *mode)
{
    return TIFFOpenExt(name, mode, NULL);
}

TIFF *TIFFOpenExt(const char *name, const char *mode, TIFFOpenOptions *opts)
{
    static const char module[] = "TIFFOpen";
    int m, fd;
    TIFF *tif;

    m = _TIFFgetMode(opts, NULL, mode, module);
    if (m == -1)
        return ((TIFF *)0);

/* for cygwin and mingw */
#ifdef O_BINARY
    m |= O_BINARY;
#endif

    fd = open(name, m, 0666);
    if (fd < 0)
    {
        if (errno > 0 && strerror(errno) != NULL)
        {
            _TIFFErrorEarly(opts, NULL, module, "%s: %s", name,
                            strerror(errno));
        }
        else
        {
            _TIFFErrorEarly(opts, NULL, module, "%s: Cannot open", name);
        }
        return ((TIFF *)0);
    }

    tif = TIFFFdOpenExt((int)fd, name, mode, opts);
    if (!tif)
        close(fd);
    return tif;
}

#ifdef _WIN32
#include <windows.h>
/*
 * Open a TIFF file with a Unicode filename, for read/writing.
 */
TIFF *TIFFOpenW(const wchar_t *name, const char *mode)
{
    return TIFFOpenWExt(name, mode, NULL);
}
TIFF *TIFFOpenWExt(const wchar_t *name, const char *mode, TIFFOpenOptions *opts)
{
    static const char module[] = "TIFFOpenW";
    int m, fd;
    int mbsize;
    char *mbname;
    TIFF *tif;

    m = _TIFFgetMode(opts, NULL, mode, module);
    if (m == -1)
        return ((TIFF *)0);

/* for cygwin and mingw */
#ifdef O_BINARY
    m |= O_BINARY;
#endif

    fd = _wopen(name, m, 0666);
    if (fd < 0)
    {
        _TIFFErrorEarly(opts, NULL, module, "%ls: Cannot open", name);
        return ((TIFF *)0);
    }

    mbname = NULL;
    mbsize = WideCharToMultiByte(CP_ACP, 0, name, -1, NULL, 0, NULL, NULL);
    if (mbsize > 0)
    {
        mbname = _TIFFmalloc(mbsize);
        if (!mbname)
        {
            _TIFFErrorEarly(
                opts, NULL, module,
                "Can't allocate space for filename conversion buffer");
            return ((TIFF *)0);
        }

        WideCharToMultiByte(CP_ACP, 0, name, -1, mbname, mbsize, NULL, NULL);
    }

    tif = TIFFFdOpenExt((int)fd, (mbname != NULL) ? mbname : "<unknown>", mode,
                        opts);

    _TIFFfree(mbname);

    if (!tif)
        close(fd);
    return tif;
}
#endif

void *_TIFFmalloc(tmsize_t s)
{
    if (s == 0)
        return ((void *)NULL);

    return (malloc((size_t)s));
}

void *_TIFFcalloc(tmsize_t nmemb, tmsize_t siz)
{
    if (nmemb == 0 || siz == 0)
        return ((void *)NULL);

    return calloc((size_t)nmemb, (size_t)siz);
}

void _TIFFfree(void *p) { free(p); }

void *_TIFFrealloc(void *p, tmsize_t s) { return (realloc(p, (size_t)s)); }

void _TIFFmemset(void *p, int v, tmsize_t c) { memset(p, v, (size_t)c); }

void _TIFFmemcpy(void *d, const void *s, tmsize_t c)
{
    memcpy(d, s, (size_t)c);
}

int _TIFFmemcmp(const void *p1, const void *p2, tmsize_t c)
{
    return (memcmp(p1, p2, (size_t)c));
}

static void unixWarningHandler(const char *module, const char *fmt, va_list ap)
{
    if (module != NULL)
        fprintf(stderr, "%s: ", module);
    fprintf(stderr, "Warning, ");
    vfprintf(stderr, fmt, ap);
    fprintf(stderr, ".\n");
}
TIFFErrorHandler _TIFFwarningHandler = unixWarningHandler;

static void unixErrorHandler(const char *module, const char *fmt, va_list ap)
{
    if (module != NULL)
        fprintf(stderr, "%s: ", module);
    vfprintf(stderr, fmt, ap);
    fprintf(stderr, ".\n");
}
TIFFErrorHandler _TIFFerrorHandler = unixErrorHandler;
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

- **HAVE_MMAP()**: A function/method defined in this file
- **union()**: A function/method defined in this file
- **O_BINARY()**: A function/method defined in this file
- **HAVE_UNISTD_H()**: A function/method defined in this file
- **_WIN32()**: A function/method defined in this file
- **HAVE_FCNTL_H()**: A function/method defined in this file
- **HAVE_IO_H()**: A function/method defined in this file
- **TIFF_DO_NOT_USE_NON_EXT_ALLOC_FUNCTIONS()**: A function/method defined in this file
- **HAVE_SYS_TYPES_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `windows.h`
- `sys/stat.h`
- `tiffiop.h`
- `unistd.h`
- `stdlib.h`
- `fcntl.h`
- `tif_config.h`
- `stdarg.h`
- `io.h`
- `sys/mman.h`
- `sys/types.h`
- `errno.h`


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

