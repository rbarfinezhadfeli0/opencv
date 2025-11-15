# Documentation for `docs/3rdparty/openjpeg/openjp2/test_sparse_array.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/test_sparse_array.c_docs.md`
- **File Name**: `test_sparse_array.c_docs.md`
- **File Size**: 9,054 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/test_sparse_array.c_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/test_sparse_array.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/test_sparse_array.c`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/test_sparse_array.c`
- **File Name**: `test_sparse_array.c`
- **File Size**: 6,005 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/openjpeg/openjp2/test_sparse_array.c](../../../3rdparty/openjpeg/openjp2/test_sparse_array.c)

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

#undef NDEBUG

#include "opj_includes.h"

int main()
{
    OPJ_UINT32 i, j, w, h;
    OPJ_INT32 buffer[ 99 * 101 ];
    OPJ_BOOL ret;
    opj_sparse_array_int32_t* sa;

    sa = opj_sparse_array_int32_create(0, 1, 1, 1);
    assert(sa == NULL);
    opj_sparse_array_int32_free(sa);

    sa = opj_sparse_array_int32_create(1, 0, 1, 1);
    assert(sa == NULL);

    sa = opj_sparse_array_int32_create(1, 1, 0, 1);
    assert(sa == NULL);

    sa = opj_sparse_array_int32_create(1, 1, 1, 0);
    assert(sa == NULL);

    sa = opj_sparse_array_int32_create(99, 101, ~0U, ~0U);
    assert(sa == NULL);

    sa = opj_sparse_array_int32_create(99, 101, 15, 17);
    opj_sparse_array_int32_free(sa);

    sa = opj_sparse_array_int32_create(99, 101, 15, 17);
    ret = opj_sparse_array_int32_read(sa, 0, 0, 0, 1, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 0, 0, 1, 0, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 0, 0, 100, 1, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 0, 0, 1, 102, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 1, 0, 0, 1, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 0, 1, 1, 0, buffer, 1, 1, OPJ_FALSE);
    assert(!ret);
    ret = opj_sparse_array_int32_read(sa, 99, 101, 99, 101, buffer, 1, 1,
                                      OPJ_FALSE);
    assert(!ret);

    buffer[0] = 1;
    ret = opj_sparse_array_int32_read(sa, 0, 0, 1, 1, buffer, 1, 1, OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 0);

    memset(buffer, 0xFF, sizeof(buffer));
    ret = opj_sparse_array_int32_read(sa, 0, 0, 99, 101, buffer, 1, 99, OPJ_FALSE);
    assert(ret);
    for (i = 0; i < 99 * 101; i++) {
        assert(buffer[i] == 0);
    }

    buffer[0] = 1;
    ret = opj_sparse_array_int32_write(sa, 4, 5, 4 + 1, 5 + 1, buffer, 1, 1,
                                       OPJ_FALSE);
    assert(ret);

    buffer[0] = 2;
    ret = opj_sparse_array_int32_write(sa, 4, 5, 4 + 1, 5 + 1, buffer, 1, 1,
                                       OPJ_FALSE);
    assert(ret);

    buffer[0] = 0;
    buffer[1] = 0xFF;
    ret = opj_sparse_array_int32_read(sa, 4, 5, 4 + 1, 5 + 1, buffer, 1, 1,
                                      OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 2);
    assert(buffer[1] == 0xFF);

    buffer[0] = 0xFF;
    buffer[1] = 0xFF;
    buffer[2] = 0xFF;
    ret = opj_sparse_array_int32_read(sa, 4, 5, 4 + 1, 5 + 2, buffer, 0, 1,
                                      OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 2);
    assert(buffer[1] == 0);
    assert(buffer[2] == 0xFF);

    buffer[0] = 3;
    ret = opj_sparse_array_int32_write(sa, 4, 5, 4 + 1, 5 + 1, buffer, 0, 1,
                                       OPJ_FALSE);
    assert(ret);

    buffer[0] = 0;
    buffer[1] = 0xFF;
    ret = opj_sparse_array_int32_read(sa, 4, 5, 4 + 1, 5 + 1, buffer, 1, 1,
                                      OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 3);
    assert(buffer[1] == 0xFF);

    w = 15 + 1;
    h = 17 + 1;
    memset(buffer, 0xFF, sizeof(buffer));
    ret = opj_sparse_array_int32_read(sa, 2, 1, 2 + w, 1 + h, buffer, 1, w,
                                      OPJ_FALSE);
    assert(ret);
    for (j = 0; j < h; j++) {
        for (i = 0; i < w; i++) {
            if (i == 4 - 2 && j == 5 - 1) {
                assert(buffer[ j * w + i ] == 3);
            } else {
                assert(buffer[ j * w + i ] == 0);
            }
        }
    }

    opj_sparse_array_int32_free(sa);


    sa = opj_sparse_array_int32_create(99, 101, 15, 17);
    memset(buffer, 0xFF, sizeof(buffer));
    ret = opj_sparse_array_int32_read(sa, 0, 0, 2, 1, buffer, 2, 4, OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 0);
    assert(buffer[1] == -1);
    assert(buffer[2] == 0);

    buffer[0] = 1;
    buffer[2] = 3;
    ret = opj_sparse_array_int32_write(sa, 0, 0, 2, 1, buffer, 2, 4, OPJ_FALSE);
    assert(ret);

    memset(buffer, 0xFF, sizeof(buffer));
    ret = opj_sparse_array_int32_read(sa, 0, 0, 2, 1, buffer, 2, 4, OPJ_FALSE);
    assert(ret);
    assert(buffer[0] == 1);
    assert(buffer[1] == -1);
    assert(buffer[2] == 3);

    opj_sparse_array_int32_free(sa);

    return 0;
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

- **NDEBUG()**: A function/method defined in this file


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

