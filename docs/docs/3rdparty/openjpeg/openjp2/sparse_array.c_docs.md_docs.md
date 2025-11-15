# Documentation for `docs/3rdparty/openjpeg/openjp2/sparse_array.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/sparse_array.c_docs.md`
- **File Name**: `sparse_array.c_docs.md`
- **File Size**: 18,793 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/sparse_array.c_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/sparse_array.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/sparse_array.c`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/sparse_array.c`
- **File Name**: `sparse_array.c`
- **File Size**: 15,756 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/openjpeg/openjp2/sparse_array.c](../../../3rdparty/openjpeg/openjp2/sparse_array.c)

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
 * Copyright (c) 2017, IntoPix SA <contact@intopix.com>
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

#include "opj_includes.h"


struct opj_sparse_array_int32 {
    OPJ_UINT32 width;
    OPJ_UINT32 height;
    OPJ_UINT32 block_width;
    OPJ_UINT32 block_height;
    OPJ_UINT32 block_count_hor;
    OPJ_UINT32 block_count_ver;
    OPJ_INT32** data_blocks;
};

opj_sparse_array_int32_t* opj_sparse_array_int32_create(OPJ_UINT32 width,
        OPJ_UINT32 height,
        OPJ_UINT32 block_width,
        OPJ_UINT32 block_height)
{
    opj_sparse_array_int32_t* sa;

    if (width == 0 || height == 0 || block_width == 0 || block_height == 0) {
        return NULL;
    }
    if (block_width > ((OPJ_UINT32)~0U) / block_height / sizeof(OPJ_INT32)) {
        return NULL;
    }

    sa = (opj_sparse_array_int32_t*) opj_calloc(1,
            sizeof(opj_sparse_array_int32_t));
    sa->width = width;
    sa->height = height;
    sa->block_width = block_width;
    sa->block_height = block_height;
    sa->block_count_hor = opj_uint_ceildiv(width, block_width);
    sa->block_count_ver = opj_uint_ceildiv(height, block_height);
    if (sa->block_count_hor > ((OPJ_UINT32)~0U) / sa->block_count_ver) {
        opj_free(sa);
        return NULL;
    }
    sa->data_blocks = (OPJ_INT32**) opj_calloc(sizeof(OPJ_INT32*),
                      (size_t) sa->block_count_hor * sa->block_count_ver);
    if (sa->data_blocks == NULL) {
        opj_free(sa);
        return NULL;
    }

    return sa;
}

void opj_sparse_array_int32_free(opj_sparse_array_int32_t* sa)
{
    if (sa) {
        OPJ_UINT32 i;
        for (i = 0; i < sa->block_count_hor * sa->block_count_ver; i++) {
            if (sa->data_blocks[i]) {
                opj_free(sa->data_blocks[i]);
            }
        }
        opj_free(sa->data_blocks);
        opj_free(sa);
    }
}

OPJ_BOOL opj_sparse_array_is_region_valid(const opj_sparse_array_int32_t* sa,
        OPJ_UINT32 x0,
        OPJ_UINT32 y0,
        OPJ_UINT32 x1,
        OPJ_UINT32 y1)
{
    return !(x0 >= sa->width || x1 <= x0 || x1 > sa->width ||
             y0 >= sa->height || y1 <= y0 || y1 > sa->height);
}

static OPJ_BOOL opj_sparse_array_int32_read_or_write(
    const opj_sparse_array_int32_t* sa,
    OPJ_UINT32 x0,
    OPJ_UINT32 y0,
    OPJ_UINT32 x1,
    OPJ_UINT32 y1,
    OPJ_INT32* buf,
    OPJ_UINT32 buf_col_stride,
    OPJ_UINT32 buf_line_stride,
    OPJ_BOOL forgiving,
    OPJ_BOOL is_read_op)
{
    OPJ_UINT32 y, block_y;
    OPJ_UINT32 y_incr = 0;
    const OPJ_UINT32 block_width = sa->block_width;

    if (!opj_sparse_array_is_region_valid(sa, x0, y0, x1, y1)) {
        return forgiving;
    }

    block_y = y0 / sa->block_height;
    for (y = y0; y < y1; block_y ++, y += y_incr) {
        OPJ_UINT32 x, block_x;
        OPJ_UINT32 x_incr = 0;
        OPJ_UINT32 block_y_offset;
        y_incr = (y == y0) ? sa->block_height - (y0 % sa->block_height) :
                 sa->block_height;
        block_y_offset = sa->block_height - y_incr;
        y_incr = opj_uint_min(y_incr, y1 - y);
        block_x = x0 / block_width;
        for (x = x0; x < x1; block_x ++, x += x_incr) {
            OPJ_UINT32 j;
            OPJ_UINT32 block_x_offset;
            OPJ_INT32* src_block;
            x_incr = (x == x0) ? block_width - (x0 % block_width) : block_width;
            block_x_offset = block_width - x_incr;
            x_incr = opj_uint_min(x_incr, x1 - x);
            src_block = sa->data_blocks[block_y * sa->block_count_hor + block_x];
            if (is_read_op) {
                if (src_block == NULL) {
                    if (buf_col_stride == 1) {
                        OPJ_INT32* dest_ptr = buf + (y - y0) * (OPJ_SIZE_T)buf_line_stride +
                                              (x - x0) * buf_col_stride;
                        for (j = 0; j < y_incr; j++) {
                            memset(dest_ptr, 0, sizeof(OPJ_INT32) * x_incr);
                            dest_ptr += buf_line_stride;
                        }
                    } else {
                        OPJ_INT32* dest_ptr = buf + (y - y0) * (OPJ_SIZE_T)buf_line_stride +
                                              (x - x0) * buf_col_stride;
                        for (j = 0; j < y_incr; j++) {
                            OPJ_UINT32 k;
                            for (k = 0; k < x_incr; k++) {
                                dest_ptr[k * buf_col_stride] = 0;
                            }
                            dest_ptr += buf_line_stride;
                        }
                    }
                } else {
                    const OPJ_INT32* OPJ_RESTRICT src_ptr = src_block + block_y_offset *
                                                            (OPJ_SIZE_T)block_width + block_x_offset;
                    if (buf_col_stride == 1) {
                        OPJ_INT32* OPJ_RESTRICT dest_ptr = buf + (y - y0) * (OPJ_SIZE_T)buf_line_stride
                                                           +
                                                           (x - x0) * buf_col_stride;
                        if (x_incr == 4) {
                            /* Same code as general branch, but the compiler */
                            /* can have an efficient memcpy() */
                            (void)(x_incr); /* trick to silent cppcheck duplicateBranch warning */
                            for (j = 0; j < y_incr; j++) {
                                memcpy(dest_ptr, src_ptr, sizeof(OPJ_INT32) * x_incr);
                                dest_ptr += buf_line_stride;
                                src_ptr += block_width;
                            }
                        } else {
                            for (j = 0; j < y_incr; j++) {
                                memcpy(dest_ptr, src_ptr, sizeof(OPJ_INT32) * x_incr);
                                dest_ptr += buf_line_stride;
                                src_ptr += block_width;
                            }
                        }
                    } else {
                        OPJ_INT32* OPJ_RESTRICT dest_ptr = buf + (y - y0) * (OPJ_SIZE_T)buf_line_stride
                                                           +
                                                           (x - x0) * buf_col_stride;
                        if (x_incr == 1) {
                            for (j = 0; j < y_incr; j++) {
                                *dest_ptr = *src_ptr;
                                dest_ptr += buf_line_stride;
                                src_ptr += block_width;
                            }
                        } else if (y_incr == 1 && buf_col_stride == 2) {
                            OPJ_UINT32 k;
                            for (k = 0; k < (x_incr & ~3U); k += 4) {
                                dest_ptr[k * buf_col_stride] = src_ptr[k];
                                dest_ptr[(k + 1) * buf_col_stride] = src_ptr[k + 1];
                                dest_ptr[(k + 2) * buf_col_stride] = src_ptr[k + 2];
                                dest_ptr[(k + 3) * buf_col_stride] = src_ptr[k + 3];
                            }
                            for (; k < x_incr; k++) {
                                dest_ptr[k * buf_col_stride] = src_ptr[k];
                            }
                        } else if (x_incr >= 8 && buf_col_stride == 8) {
                            for (j = 0; j < y_incr; j++) {
                                OPJ_UINT32 k;
                                for (k = 0; k < (x_incr & ~3U); k += 4) {
                                    dest_ptr[k * buf_col_stride] = src_ptr[k];
                                    dest_ptr[(k + 1) * buf_col_stride] = src_ptr[k + 1];
                                    dest_ptr[(k + 2) * buf_col_stride] = src_ptr[k + 2];
                                    dest_ptr[(k + 3) * buf_col_stride] = src_ptr[k + 3];
                                }
                                for (; k < x_incr; k++) {
                                    dest_ptr[k * buf_col_stride] = src_ptr[k];
                                }
                                dest_ptr += buf_line_stride;
                                src_ptr += block_width;
                            }
                        } else {
                            /* General case */
                            for (j = 0; j < y_incr; j++) {
                                OPJ_UINT32 k;
                                for (k = 0; k < x_incr; k++) {
                                    dest_ptr[k * buf_col_stride] = src_ptr[k];
                                }
                                dest_ptr += buf_line_stride;
                                src_ptr += block_width;
                            }
                        }
                    }
                }
            } else {
                if (src_block == NULL) {
                    src_block = (OPJ_INT32*) opj_calloc(1,
                                                        (size_t) sa->block_width * sa->block_height * sizeof(OPJ_INT32));
                    if (src_block == NULL) {
                        return OPJ_FALSE;
                    }
                    sa->data_blocks[block_y * sa->block_count_hor + block_x] = src_block;
                }

                if (buf_col_stride == 1) {
                    OPJ_INT32* OPJ_RESTRICT dest_ptr = src_block + block_y_offset *
                                                       (OPJ_SIZE_T)block_width + block_x_offset;
                    const OPJ_INT32* OPJ_RESTRICT src_ptr = buf + (y - y0) *
                                                            (OPJ_SIZE_T)buf_line_stride + (x - x0) * buf_col_stride;
                    if (x_incr == 4) {
                        /* Same code as general branch, but the compiler */
                        /* can have an efficient memcpy() */
                        (void)(x_incr); /* trick to silent cppcheck duplicateBranch warning */
                        for (j = 0; j < y_incr; j++) {
                            memcpy(dest_ptr, src_ptr, sizeof(OPJ_INT32) * x_incr);
                            dest_ptr += block_width;
                            src_ptr += buf_line_stride;
                        }
                    } else {
                        for (j = 0; j < y_incr; j++) {
                            memcpy(dest_ptr, src_ptr, sizeof(OPJ_INT32) * x_incr);
                            dest_ptr += block_width;
                            src_ptr += buf_line_stride;
                        }
                    }
                } else {
                    OPJ_INT32* OPJ_RESTRICT dest_ptr = src_block + block_y_offset *
                                                       (OPJ_SIZE_T)block_width + block_x_offset;
                    const OPJ_INT32* OPJ_RESTRICT src_ptr = buf + (y - y0) *
                                                            (OPJ_SIZE_T)buf_line_stride + (x - x0) * buf_col_stride;
                    if (x_incr == 1) {
                        for (j = 0; j < y_incr; j++) {
                            *dest_ptr = *src_ptr;
                            src_ptr += buf_line_stride;
                            dest_ptr += block_width;
                        }
                    } else if (x_incr >= 8 && buf_col_stride == 8) {
                        for (j = 0; j < y_incr; j++) {
                            OPJ_UINT32 k;
                            for (k = 0; k < (x_incr & ~3U); k += 4) {
                                dest_ptr[k] = src_ptr[k * buf_col_stride];
                                dest_ptr[k + 1] = src_ptr[(k + 1) * buf_col_stride];
                                dest_ptr[k + 2] = src_ptr[(k + 2) * buf_col_stride];
                                dest_ptr[k + 3] = src_ptr[(k + 3) * buf_col_stride];
                            }
                            for (; k < x_incr; k++) {
                                dest_ptr[k] = src_ptr[k * buf_col_stride];
                            }
                            src_ptr += buf_line_stride;
                            dest_ptr += block_width;
                        }
                    } else {
                        /* General case */
                        for (j = 0; j < y_incr; j++) {
                            OPJ_UINT32 k;
                            for (k = 0; k < x_incr; k++) {
                                dest_ptr[k] = src_ptr[k * buf_col_stride];
                            }
                            src_ptr += buf_line_stride;
                            dest_ptr += block_width;
                        }
                    }
                }
            }
        }
    }

    return OPJ_TRUE;
}

OPJ_BOOL opj_sparse_array_int32_read(const opj_sparse_array_int32_t* sa,
                                     OPJ_UINT32 x0,
                                     OPJ_UINT32 y0,
                                     OPJ_UINT32 x1,
                                     OPJ_UINT32 y1,
                                     OPJ_INT32* dest,
                                     OPJ_UINT32 dest_col_stride,
                                     OPJ_UINT32 dest_line_stride,
                                     OPJ_BOOL forgiving)
{
    return opj_sparse_array_int32_read_or_write(
               (opj_sparse_array_int32_t*)sa, x0, y0, x1, y1,
               dest,
               dest_col_stride,
               dest_line_stride,
               forgiving,
               OPJ_TRUE);
}

OPJ_BOOL opj_sparse_array_int32_write(opj_sparse_array_int32_t* sa,
                                      OPJ_UINT32 x0,
                                      OPJ_UINT32 y0,
                                      OPJ_UINT32 x1,
                                      OPJ_UINT32 y1,
                                      const OPJ_INT32* src,
                                      OPJ_UINT32 src_col_stride,
                                      OPJ_UINT32 src_line_stride,
                                      OPJ_BOOL forgiving)
{
    return opj_sparse_array_int32_read_or_write(sa, x0, y0, x1, y1,
            (OPJ_INT32*)src,
            src_col_stride,
            src_line_stride,
            forgiving,
            OPJ_FALSE);
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

### Classes and Structures

- **opj_sparse_array_int32**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opj_includes.h`


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

