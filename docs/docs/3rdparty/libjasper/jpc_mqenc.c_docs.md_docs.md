# Documentation for `docs/3rdparty/libjasper/jpc_mqenc.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjasper/jpc_mqenc.c_docs.md`
- **File Name**: `jpc_mqenc.c_docs.md`
- **File Size**: 15,413 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjasper/jpc_mqenc.c_docs.md](../../../docs/3rdparty/libjasper/jpc_mqenc.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjasper` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjasper/jpc_mqenc.c`

## File Metadata

- **Full Path**: `3rdparty/libjasper/jpc_mqenc.c`
- **File Name**: `jpc_mqenc.c`
- **File Size**: 12,240 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libjasper/jpc_mqenc.c](../../3rdparty/libjasper/jpc_mqenc.c)

## Purpose and Role

This file is located in the `3rdparty/libjasper` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Copyright (c) 1999-2000 Image Power, Inc. and the University of
 *   British Columbia.
 * Copyright (c) 2001-2003 Michael David Adams.
 * All rights reserved.
 */

/* __START_OF_JASPER_LICENSE__
 *
 * JasPer License Version 2.0
 *
 * Copyright (c) 2001-2006 Michael David Adams
 * Copyright (c) 1999-2000 Image Power, Inc.
 * Copyright (c) 1999-2000 The University of British Columbia
 *
 * All rights reserved.
 *
 * Permission is hereby granted, free of charge, to any person (the
 * "User") obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, and/or sell copies of the Software, and to permit
 * persons to whom the Software is furnished to do so, subject to the
 * following conditions:
 *
 * 1.  The above copyright notices and this permission notice (which
 * includes the disclaimer below) shall be included in all copies or
 * substantial portions of the Software.
 *
 * 2.  The name of a copyright holder shall not be used to endorse or
 * promote products derived from the Software without specific prior
 * written permission.
 *
 * THIS DISCLAIMER OF WARRANTY CONSTITUTES AN ESSENTIAL PART OF THIS
 * LICENSE.  NO USE OF THE SOFTWARE IS AUTHORIZED HEREUNDER EXCEPT UNDER
 * THIS DISCLAIMER.  THE SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS
 * "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING
 * BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
 * PARTICULAR PURPOSE AND NONINFRINGEMENT OF THIRD PARTY RIGHTS.  IN NO
 * EVENT SHALL THE COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, OR ANY SPECIAL
 * INDIRECT OR CONSEQUENTIAL DAMAGES, OR ANY DAMAGES WHATSOEVER RESULTING
 * FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT,
 * NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION
 * WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.  NO ASSURANCES ARE
 * PROVIDED BY THE COPYRIGHT HOLDERS THAT THE SOFTWARE DOES NOT INFRINGE
 * THE PATENT OR OTHER INTELLECTUAL PROPERTY RIGHTS OF ANY OTHER ENTITY.
 * EACH COPYRIGHT HOLDER DISCLAIMS ANY LIABILITY TO THE USER FOR CLAIMS
 * BROUGHT BY ANY OTHER ENTITY BASED ON INFRINGEMENT OF INTELLECTUAL
 * PROPERTY RIGHTS OR OTHERWISE.  AS A CONDITION TO EXERCISING THE RIGHTS
 * GRANTED HEREUNDER, EACH USER HEREBY ASSUMES SOLE RESPONSIBILITY TO SECURE
 * ANY OTHER INTELLECTUAL PROPERTY RIGHTS NEEDED, IF ANY.  THE SOFTWARE
 * IS NOT FAULT-TOLERANT AND IS NOT INTENDED FOR USE IN MISSION-CRITICAL
 * SYSTEMS, SUCH AS THOSE USED IN THE OPERATION OF NUCLEAR FACILITIES,
 * AIRCRAFT NAVIGATION OR COMMUNICATION SYSTEMS, AIR TRAFFIC CONTROL
 * SYSTEMS, DIRECT LIFE SUPPORT MACHINES, OR WEAPONS SYSTEMS, IN WHICH
 * THE FAILURE OF THE SOFTWARE OR SYSTEM COULD LEAD DIRECTLY TO DEATH,
 * PERSONAL INJURY, OR SEVERE PHYSICAL OR ENVIRONMENTAL DAMAGE ("HIGH
 * RISK ACTIVITIES").  THE COPYRIGHT HOLDERS SPECIFICALLY DISCLAIM ANY
 * EXPRESS OR IMPLIED WARRANTY OF FITNESS FOR HIGH RISK ACTIVITIES.
 *
 * __END_OF_JASPER_LICENSE__
 */

/*
 * MQ Arithmetic Encoder
 *
 * $Id: jpc_mqenc.c,v 1.2 2008-05-26 09:40:52 vp153 Exp $
 */

/******************************************************************************\
* Includes.
\******************************************************************************/

#include <assert.h>
#include <stdlib.h>

#include "jasper/jas_stream.h"
#include "jasper/jas_malloc.h"
#include "jasper/jas_math.h"
#include "jasper/jas_debug.h"

#include "jpc_mqenc.h"

/******************************************************************************\
* Macros
\******************************************************************************/

#if defined(DEBUG)
#define	JPC_MQENC_CALL(n, x) \
    ((jas_getdbglevel() >= (n)) ? ((void)(x)) : ((void)0))
#else
#define	JPC_MQENC_CALL(n, x)
#endif

#define	jpc_mqenc_codemps9(areg, creg, ctreg, curctx, enc) \
{ \
    jpc_mqstate_t *state = *(curctx); \
    (areg) -= state->qeval; \
    if (!((areg) & 0x8000)) { \
        if ((areg) < state->qeval) { \
            (areg) = state->qeval; \
        } else { \
            (creg) += state->qeval; \
        } \
        *(curctx) = state->nmps; \
        jpc_mqenc_renorme((areg), (creg), (ctreg), (enc)); \
    } else { \
        (creg) += state->qeval; \
    } \
}

#define	jpc_mqenc_codelps2(areg, creg, ctreg, curctx, enc) \
{ \
    jpc_mqstate_t *state = *(curctx); \
    (areg) -= state->qeval; \
    if ((areg) < state->qeval) { \
        (creg) += state->qeval; \
    } else { \
        (areg) = state->qeval; \
    } \
    *(curctx) = state->nlps; \
    jpc_mqenc_renorme((areg), (creg), (ctreg), (enc)); \
}

#define	jpc_mqenc_renorme(areg, creg, ctreg, enc) \
{ \
    do { \
        (areg) <<= 1; \
        (creg) <<= 1; \
        if (!--(ctreg)) { \
            jpc_mqenc_byteout((areg), (creg), (ctreg), (enc)); \
        } \
    } while (!((areg) & 0x8000)); \
}

#define	jpc_mqenc_byteout(areg, creg, ctreg, enc) \
{ \
    if ((enc)->outbuf != 0xff) { \
        if ((creg) & 0x8000000) { \
            if (++((enc)->outbuf) == 0xff) { \
                (creg) &= 0x7ffffff; \
                jpc_mqenc_byteout2(enc); \
                enc->outbuf = ((creg) >> 20) & 0xff; \
                (creg) &= 0xfffff; \
                (ctreg) = 7; \
            } else { \
                jpc_mqenc_byteout2(enc); \
                enc->outbuf = ((creg) >> 19) & 0xff; \
                (creg) &= 0x7ffff; \
                (ctreg) = 8; \
            } \
        } else { \
            jpc_mqenc_byteout2(enc); \
            (enc)->outbuf = ((creg) >> 19) & 0xff; \
            (creg) &= 0x7ffff; \
            (ctreg) = 8; \
        } \
    } else { \
        jpc_mqenc_byteout2(enc); \
        (enc)->outbuf = ((creg) >> 20) & 0xff; \
        (creg) &= 0xfffff; \
        (ctreg) = 7; \
    } \
}

#define	jpc_mqenc_byteout2(enc) \
{ \
    if (enc->outbuf >= 0) { \
        if (jas_stream_putc(enc->out, (unsigned char)enc->outbuf) == EOF) { \
            enc->err |= 1; \
        } \
    } \
    enc->lastbyte = enc->outbuf; \
}

/******************************************************************************\
* Local function protoypes.
\******************************************************************************/

static void jpc_mqenc_setbits(jpc_mqenc_t *mqenc);

/******************************************************************************\
* Code for creation and destruction of encoder.
\******************************************************************************/

/* Create a MQ encoder. */

jpc_mqenc_t *jpc_mqenc_create(int maxctxs, jas_stream_t *out)
{
    jpc_mqenc_t *mqenc;

    /* Allocate memory for the MQ encoder. */
    if (!(mqenc = jas_malloc(sizeof(jpc_mqenc_t)))) {
        goto error;
    }
    mqenc->out = out;
    mqenc->maxctxs = maxctxs;

    /* Allocate memory for the per-context state information. */
    if (!(mqenc->ctxs = jas_alloc2(mqenc->maxctxs, sizeof(jpc_mqstate_t *)))) {
        goto error;
    }

    /* Set the current context to the first one. */
    mqenc->curctx = mqenc->ctxs;

    jpc_mqenc_init(mqenc);

    /* Initialize the per-context state information to something sane. */
    jpc_mqenc_setctxs(mqenc, 0, 0);

    return mqenc;

error:
    if (mqenc) {
        jpc_mqenc_destroy(mqenc);
    }
    return 0;
}

/* Destroy a MQ encoder. */

void jpc_mqenc_destroy(jpc_mqenc_t *mqenc)
{
    if (mqenc->ctxs) {
        jas_free(mqenc->ctxs);
    }
    jas_free(mqenc);
}

/******************************************************************************\
* State initialization code.
\******************************************************************************/

/* Initialize the coding state of a MQ encoder. */

void jpc_mqenc_init(jpc_mqenc_t *mqenc)
{
    mqenc->areg = 0x8000;
    mqenc->outbuf = -1;
    mqenc->creg = 0;
    mqenc->ctreg = 12;
    mqenc->lastbyte = -1;
    mqenc->err = 0;
}

/* Initialize one or more contexts. */

void jpc_mqenc_setctxs(jpc_mqenc_t *mqenc, int numctxs, jpc_mqctx_t *ctxs)
{
    jpc_mqstate_t **ctx;
    int n;

    ctx = mqenc->ctxs;
    n = JAS_MIN(mqenc->maxctxs, numctxs);
    while (--n >= 0) {
        *ctx = &jpc_mqstates[2 * ctxs->ind + ctxs->mps];
        ++ctx;
        ++ctxs;
    }
    n = mqenc->maxctxs - numctxs;
    while (--n >= 0) {
        *ctx = &jpc_mqstates[0];
        ++ctx;
    }

}

/* Get the coding state for a MQ encoder. */

void jpc_mqenc_getstate(jpc_mqenc_t *mqenc, jpc_mqencstate_t *state)
{
    state->areg = mqenc->areg;
    state->creg = mqenc->creg;
    state->ctreg = mqenc->ctreg;
    state->lastbyte = mqenc->lastbyte;
}

/******************************************************************************\
* Code for coding symbols.
\******************************************************************************/

/* Encode a bit. */

int jpc_mqenc_putbit_func(jpc_mqenc_t *mqenc, int bit)
{
    const jpc_mqstate_t *state;
    JAS_DBGLOG(100, ("jpc_mqenc_putbit(%p, %d)\n", mqenc, bit));
    JPC_MQENC_CALL(100, jpc_mqenc_dump(mqenc, stderr));

    state = *(mqenc->curctx);

    if (state->mps == bit) {
        /* Apply the CODEMPS algorithm as defined in the standard. */
        mqenc->areg -= state->qeval;
        if (!(mqenc->areg & 0x8000)) {
            jpc_mqenc_codemps2(mqenc);
        } else {
            mqenc->creg += state->qeval;
        }
    } else {
        /* Apply the CODELPS algorithm as defined in the standard. */
        jpc_mqenc_codelps2(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc->curctx, mqenc);
    }

    return jpc_mqenc_error(mqenc) ? (-1) : 0;
}

int jpc_mqenc_codemps2(jpc_mqenc_t *mqenc)
{
    /* Note: This function only performs part of the work associated with
    the CODEMPS algorithm from the standard.  Some of the work is also
    performed by the caller. */

    jpc_mqstate_t *state = *(mqenc->curctx);
    if (mqenc->areg < state->qeval) {
        mqenc->areg = state->qeval;
    } else {
        mqenc->creg += state->qeval;
    }
    *mqenc->curctx = state->nmps;
    jpc_mqenc_renorme(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc);
    return jpc_mqenc_error(mqenc) ? (-1) : 0;
}

int jpc_mqenc_codelps(jpc_mqenc_t *mqenc)
{
    jpc_mqenc_codelps2(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc->curctx, mqenc);
    return jpc_mqenc_error(mqenc) ? (-1) : 0;
}

/******************************************************************************\
* Miscellaneous code.
\******************************************************************************/

/* Terminate the code word. */

int jpc_mqenc_flush(jpc_mqenc_t *mqenc, int termmode)
{
    int_fast16_t k;

    switch (termmode) {
    case JPC_MQENC_PTERM:
        k = 11 - mqenc->ctreg + 1;
        while (k > 0) {
            mqenc->creg <<= mqenc->ctreg;
            mqenc->ctreg = 0;
            jpc_mqenc_byteout(mqenc->areg, mqenc->creg, mqenc->ctreg,
              mqenc);
            k -= mqenc->ctreg;
        }
        if (mqenc->outbuf != 0xff) {
            jpc_mqenc_byteout(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc);
        }
        break;
    case JPC_MQENC_DEFTERM:
        jpc_mqenc_setbits(mqenc);
        mqenc->creg <<= mqenc->ctreg;
        jpc_mqenc_byteout(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc);
        mqenc->creg <<= mqenc->ctreg;
        jpc_mqenc_byteout(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc);
        if (mqenc->outbuf != 0xff) {
            jpc_mqenc_byteout(mqenc->areg, mqenc->creg, mqenc->ctreg, mqenc);
        }
        break;
    default:
        abort();
        break;
    }
    return 0;
}

static void jpc_mqenc_setbits(jpc_mqenc_t *mqenc)
{
    uint_fast32_t tmp = mqenc->creg + mqenc->areg;
    mqenc->creg |= 0xffff;
    if (mqenc->creg >= tmp) {
        mqenc->creg -= 0x8000;
    }
}

/* Dump a MQ encoder to a stream for debugging. */

int jpc_mqenc_dump(jpc_mqenc_t *mqenc, FILE *out)
{
    fprintf(out, "AREG = %08x, CREG = %08x, CTREG = %d\n",
      (unsigned)mqenc->areg, (unsigned)mqenc->creg, (int)mqenc->ctreg);
    fprintf(out, "IND = %02d, MPS = %d, QEVAL = %04x\n",
      (int)(*mqenc->curctx - jpc_mqstates), (int)(*mqenc->curctx)->mps,
      (int)(*mqenc->curctx)->qeval);
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

- **only()**: A function/method defined in this file
- **protoypes()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `jasper/jas_malloc.h`
- `jasper/jas_debug.h`
- `jasper/jas_math.h`
- `stdlib.h`
- `assert.h`
- `jpc_mqenc.h`
- `jasper/jas_stream.h`

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

