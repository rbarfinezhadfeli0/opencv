# Documentation for `docs/3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c_docs.md`
- **File Name**: `dfltcc_inflate.c_docs.md`
- **File Size**: 9,592 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c_docs.md](../../../../../docs/3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/arch/s390` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c`
- **File Name**: `dfltcc_inflate.c`
- **File Size**: 6,327 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c](../../../../3rdparty/zlib-ng/arch/s390/dfltcc_inflate.c)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/arch/s390` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* dfltcc_inflate.c - IBM Z DEFLATE CONVERSION CALL decompression support. */

/*
   Use the following commands to build zlib-ng with DFLTCC decompression support:

        $ ./configure --with-dfltcc-inflate
   or

        $ cmake -DWITH_DFLTCC_INFLATE=1 .

   and then

        $ make
*/

#include "zbuild.h"
#include "zutil.h"
#include "inftrees.h"
#include "inflate.h"
#include "dfltcc_inflate.h"
#include "dfltcc_detail.h"

void Z_INTERNAL PREFIX(dfltcc_reset_inflate_state)(PREFIX3(streamp) strm) {
    struct inflate_state *state = (struct inflate_state *)strm->state;

    dfltcc_reset_state(&state->arch.common);
}

int Z_INTERNAL PREFIX(dfltcc_can_inflate)(PREFIX3(streamp) strm) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_state *dfltcc_state = &state->arch.common;

    /* Unsupported hardware */
    return is_bit_set(dfltcc_state->af.fns, DFLTCC_XPND) && is_bit_set(dfltcc_state->af.fmts, DFLTCC_FMT0);
}

static inline dfltcc_cc dfltcc_xpnd(PREFIX3(streamp) strm) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_param_v0 *param = &state->arch.common.param;
    size_t avail_in = strm->avail_in;
    size_t avail_out = strm->avail_out;
    dfltcc_cc cc;

    cc = dfltcc(DFLTCC_XPND | HBT_CIRCULAR,
                param, &strm->next_out, &avail_out,
                &strm->next_in, &avail_in, state->window);
    strm->avail_in = avail_in;
    strm->avail_out = avail_out;
    return cc;
}

dfltcc_inflate_action Z_INTERNAL PREFIX(dfltcc_inflate)(PREFIX3(streamp) strm, int flush, int *ret) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_state *dfltcc_state = &state->arch.common;
    struct dfltcc_param_v0 *param = &dfltcc_state->param;
    dfltcc_cc cc;

    if (flush == Z_BLOCK || flush == Z_TREES) {
        /* DFLTCC does not support stopping on block boundaries */
        if (PREFIX(dfltcc_inflate_disable)(strm)) {
            *ret = Z_STREAM_ERROR;
            return DFLTCC_INFLATE_BREAK;
        } else
            return DFLTCC_INFLATE_SOFTWARE;
    }

    if (state->last) {
        if (state->bits != 0) {
            strm->next_in++;
            strm->avail_in--;
            state->bits = 0;
        }
        state->mode = CHECK;
        return DFLTCC_INFLATE_CONTINUE;
    }

    if (strm->avail_in == 0 && !param->cf)
        return DFLTCC_INFLATE_BREAK;

    /* if window not in use yet, initialize */
    if (state->wsize == 0)
        state->wsize = 1U << state->wbits;

    /* Translate stream to parameter block */
    param->cvt = ((state->wrap & 4) && state->flags) ? CVT_CRC32 : CVT_ADLER32;
    param->sbb = state->bits;
    if (param->hl)
        param->nt = 0; /* Honor history for the first block */
    if (state->wrap & 4)
        param->cv = state->flags ? ZSWAP32(state->check) : state->check;

    /* Inflate */
    do {
        cc = dfltcc_xpnd(strm);
    } while (cc == DFLTCC_CC_AGAIN);

    /* Translate parameter block to stream */
    strm->msg = oesc_msg(dfltcc_state->msg, param->oesc);
    state->last = cc == DFLTCC_CC_OK;
    state->bits = param->sbb;
    if (state->wrap & 4)
        strm->adler = state->check = state->flags ? ZSWAP32(param->cv) : param->cv;
    if (cc == DFLTCC_CC_OP2_CORRUPT && param->oesc != 0) {
        /* Report an error if stream is corrupted */
        state->mode = BAD;
        return DFLTCC_INFLATE_CONTINUE;
    }
    state->mode = TYPEDO;
    /* Break if operands are exhausted, otherwise continue looping */
    return (cc == DFLTCC_CC_OP1_TOO_SHORT || cc == DFLTCC_CC_OP2_TOO_SHORT) ?
        DFLTCC_INFLATE_BREAK : DFLTCC_INFLATE_CONTINUE;
}

int Z_INTERNAL PREFIX(dfltcc_was_inflate_used)(PREFIX3(streamp) strm) {
    struct inflate_state *state = (struct inflate_state *)strm->state;

    return !state->arch.common.param.nt;
}

/*
   Rotates a circular buffer.
   The implementation is based on https://cplusplus.com/reference/algorithm/rotate/
 */
static void rotate(unsigned char *start, unsigned char *pivot, unsigned char *end) {
    unsigned char *p = pivot;
    unsigned char tmp;

    while (p != start) {
        tmp = *start;
        *start = *p;
        *p = tmp;

        start++;
        p++;

        if (p == end)
            p = pivot;
        else if (start == pivot)
            pivot = p;
    }
}

int Z_INTERNAL PREFIX(dfltcc_inflate_disable)(PREFIX3(streamp) strm) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_state *dfltcc_state = &state->arch.common;
    struct dfltcc_param_v0 *param = &dfltcc_state->param;

    if (!PREFIX(dfltcc_can_inflate)(strm))
        return 0;
    if (PREFIX(dfltcc_was_inflate_used)(strm))
        /* DFLTCC has already decompressed some data. Since there is not
         * enough information to resume decompression in software, the call
         * must fail.
         */
        return 1;
    /* DFLTCC was not used yet - decompress in software */
    memset(&dfltcc_state->af, 0, sizeof(dfltcc_state->af));
    /* Convert the window from the hardware to the software format */
    rotate(state->window, state->window + param->ho, state->window + HB_SIZE);
    state->whave = state->wnext = MIN(param->hl, state->wsize);
    return 0;
}

/*
   Preloading history.
*/
int Z_INTERNAL PREFIX(dfltcc_inflate_set_dictionary)(PREFIX3(streamp) strm,
                                                     const unsigned char *dictionary, uInt dict_length) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_param_v0 *param = &state->arch.common.param;

    /* if window not in use yet, initialize */
    if (state->wsize == 0)
        state->wsize = 1U << state->wbits;

    append_history(param, state->window, dictionary, dict_length);
    state->havedict = 1;
    return Z_OK;
}

int Z_INTERNAL PREFIX(dfltcc_inflate_get_dictionary)(PREFIX3(streamp) strm,
                                                     unsigned char *dictionary, uInt *dict_length) {
    struct inflate_state *state = (struct inflate_state *)strm->state;
    struct dfltcc_param_v0 *param = &state->arch.common.param;

    if (dictionary && state->window)
        get_history(param, state->window, dictionary);
    if (dict_length)
        *dict_length = param->hl;
    return Z_OK;
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

- **dfltcc_param_v0**: A class/struct defined in this file
- **inflate_state**: A class/struct defined in this file
- **dfltcc_state**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `zbuild.h`
- `inflate.h`
- `dfltcc_inflate.h`
- `zutil.h`
- `dfltcc_detail.h`
- `inftrees.h`

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

