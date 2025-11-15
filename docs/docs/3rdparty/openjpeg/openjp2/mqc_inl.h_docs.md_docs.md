# Documentation for `docs/3rdparty/openjpeg/openjp2/mqc_inl.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/mqc_inl.h_docs.md`
- **File Name**: `mqc_inl.h_docs.md`
- **File Size**: 11,315 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/mqc_inl.h_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/mqc_inl.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/mqc_inl.h`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/mqc_inl.h`
- **File Name**: `mqc_inl.h`
- **File Size**: 8,343 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openjpeg/openjp2/mqc_inl.h](../../../3rdparty/openjpeg/openjp2/mqc_inl.h)

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
 * Copyright (c) 2002-2014, Universite catholique de Louvain (UCL), Belgium
 * Copyright (c) 2002-2014, Professor Benoit Macq
 * Copyright (c) 2001-2003, David Janssens
 * Copyright (c) 2002-2003, Yannick Verschueren
 * Copyright (c) 2003-2007, Francois-Olivier Devaux
 * Copyright (c) 2003-2014, Antonin Descampe
 * Copyright (c) 2005, Herve Drolon, FreeImage Team
 * Copyright (c) 2008, Jerome Fimes, Communications & Systemes <jerome.fimes@c-s.fr>
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

#ifndef OPJ_MQC_INL_H
#define OPJ_MQC_INL_H

/* For internal use of opj_mqc_decode_macro() */
#define opj_mqc_mpsexchange_macro(d, curctx, a) \
{ \
    if (a < (*curctx)->qeval) { \
        d = !((*curctx)->mps); \
        *curctx = (*curctx)->nlps; \
    } else { \
        d = (*curctx)->mps; \
        *curctx = (*curctx)->nmps; \
    } \
}

/* For internal use of opj_mqc_decode_macro() */
#define opj_mqc_lpsexchange_macro(d, curctx, a) \
{ \
    if (a < (*curctx)->qeval) { \
        a = (*curctx)->qeval; \
        d = (*curctx)->mps; \
        *curctx = (*curctx)->nmps; \
    } else { \
        a = (*curctx)->qeval; \
        d = !((*curctx)->mps); \
        *curctx = (*curctx)->nlps; \
    } \
}


/**
Decode a symbol using raw-decoder. Cfr p.506 TAUBMAN
@param mqc MQC handle
@return Returns the decoded symbol (0 or 1)
*/
static INLINE OPJ_UINT32 opj_mqc_raw_decode(opj_mqc_t *mqc)
{
    OPJ_UINT32 d;
    if (mqc->ct == 0) {
        /* Given opj_mqc_raw_init_dec() we know that at some point we will */
        /* have a 0xFF 0xFF artificial marker */
        if (mqc->c == 0xff) {
            if (*mqc->bp  > 0x8f) {
                mqc->c = 0xff;
                mqc->ct = 8;
            } else {
                mqc->c = *mqc->bp;
                mqc->bp ++;
                mqc->ct = 7;
            }
        } else {
            mqc->c = *mqc->bp;
            mqc->bp ++;
            mqc->ct = 8;
        }
    }
    mqc->ct--;
    d = ((OPJ_UINT32)mqc->c >> mqc->ct) & 0x01U;

    return d;
}


#define opj_mqc_bytein_macro(mqc, c, ct) \
{ \
        OPJ_UINT32 l_c;  \
        /* Given opj_mqc_init_dec() we know that at some point we will */ \
        /* have a 0xFF 0xFF artificial marker */ \
        l_c = *(mqc->bp + 1); \
        if (*mqc->bp == 0xff) { \
            if (l_c > 0x8f) { \
                c += 0xff00; \
                ct = 8; \
                mqc->end_of_byte_stream_counter ++; \
            } else { \
                mqc->bp++; \
                c += l_c << 9; \
                ct = 7; \
            } \
        } else { \
            mqc->bp++; \
            c += l_c << 8; \
            ct = 8; \
        } \
}

/* For internal use of opj_mqc_decode_macro() */
#define opj_mqc_renormd_macro(mqc, a, c, ct) \
{ \
    do { \
        if (ct == 0) { \
            opj_mqc_bytein_macro(mqc, c, ct); \
        } \
        a <<= 1; \
        c <<= 1; \
        ct--; \
    } while (a < 0x8000); \
}

#define opj_mqc_decode_macro(d, mqc, curctx, a, c, ct) \
{ \
    /* Implements ISO 15444-1 C.3.2 Decoding a decision (DECODE) */ \
    /* Note: alternate "J.2 - Decoding an MPS or an LPS in the */ \
    /* software-conventions decoder" has been tried, but does not bring any */ \
    /* improvement. See https://github.com/uclouvain/openjpeg/issues/921 */ \
    a -= (*curctx)->qeval;  \
    if ((c >> 16) < (*curctx)->qeval) {  \
        opj_mqc_lpsexchange_macro(d, curctx, a);  \
        opj_mqc_renormd_macro(mqc, a, c, ct);  \
    } else {  \
        c -= (*curctx)->qeval << 16;  \
        if ((a & 0x8000) == 0) { \
            opj_mqc_mpsexchange_macro(d, curctx, a); \
            opj_mqc_renormd_macro(mqc, a, c, ct); \
        } else { \
            d = (*curctx)->mps; \
        } \
    } \
}

#define DOWNLOAD_MQC_VARIABLES(mqc, curctx, a, c, ct) \
        register const opj_mqc_state_t **curctx = mqc->curctx; \
        register OPJ_UINT32 c = mqc->c; \
        register OPJ_UINT32 a = mqc->a; \
        register OPJ_UINT32 ct = mqc->ct

#define UPLOAD_MQC_VARIABLES(mqc, curctx, a, c, ct) \
        mqc->curctx = curctx; \
        mqc->c = c; \
        mqc->a = a; \
        mqc->ct = ct;

/**
Input a byte
@param mqc MQC handle
*/
static INLINE void opj_mqc_bytein(opj_mqc_t *const mqc)
{
    opj_mqc_bytein_macro(mqc, mqc->c, mqc->ct);
}

/**
Renormalize mqc->a and mqc->c while decoding
@param mqc MQC handle
*/
#define opj_mqc_renormd(mqc) \
    opj_mqc_renormd_macro(mqc, mqc->a, mqc->c, mqc->ct)

/**
Decode a symbol
@param d OPJ_UINT32 value where to store the decoded symbol
@param mqc MQC handle
@return Returns the decoded symbol (0 or 1) in d
*/
#define opj_mqc_decode(d, mqc) \
    opj_mqc_decode_macro(d, mqc, mqc->curctx, mqc->a, mqc->c, mqc->ct)

/**
Output a byte, doing bit-stuffing if necessary.
After a 0xff byte, the next byte must be smaller than 0x90.
@param mqc MQC handle
*/
void opj_mqc_byteout(opj_mqc_t *mqc);

/**
Renormalize mqc->a and mqc->c while encoding, so that mqc->a stays between 0x8000 and 0x10000
@param mqc MQC handle
@param a_ value of mqc->a
@param c_ value of mqc->c_
@param ct_ value of mqc->ct_
*/
#define opj_mqc_renorme_macro(mqc, a_, c_, ct_) \
{ \
    do { \
        a_ <<= 1; \
        c_ <<= 1; \
        ct_--; \
        if (ct_ == 0) { \
            mqc->c = c_; \
            opj_mqc_byteout(mqc); \
            c_ = mqc->c; \
            ct_ = mqc->ct; \
        } \
    } while( (a_ & 0x8000) == 0); \
}

#define opj_mqc_codemps_macro(mqc, curctx, a, c, ct) \
{ \
    a -= (*curctx)->qeval; \
    if ((a & 0x8000) == 0) { \
        if (a < (*curctx)->qeval) { \
            a = (*curctx)->qeval; \
        } else { \
            c += (*curctx)->qeval; \
        } \
        *curctx = (*curctx)->nmps; \
        opj_mqc_renorme_macro(mqc, a, c, ct); \
    } else { \
        c += (*curctx)->qeval; \
    } \
}

#define opj_mqc_codelps_macro(mqc, curctx, a, c, ct) \
{ \
    a -= (*curctx)->qeval; \
    if (a < (*curctx)->qeval) { \
        c += (*curctx)->qeval; \
    } else { \
        a = (*curctx)->qeval; \
    } \
    *curctx = (*curctx)->nlps; \
    opj_mqc_renorme_macro(mqc, a, c, ct); \
}

#define opj_mqc_encode_macro(mqc, curctx, a, c, ct, d) \
{ \
    if ((*curctx)->mps == (d)) { \
        opj_mqc_codemps_macro(mqc, curctx, a, c, ct); \
    } else { \
        opj_mqc_codelps_macro(mqc, curctx, a, c, ct); \
    } \
}


#define opj_mqc_bypass_enc_macro(mqc, c, ct, d) \
{\
    if (ct == BYPASS_CT_INIT) {\
        ct = 8;\
    }\
    ct--;\
    c = c + ((d) << ct);\
    if (ct == 0) {\
        *mqc->bp = (OPJ_BYTE)c;\
        ct = 8;\
        /* If the previous byte was 0xff, make sure that the next msb is 0 */ \
        if (*mqc->bp == 0xff) {\
            ct = 7;\
        }\
        mqc->bp++;\
        c = 0;\
    }\
}

#endif /* OPJ_MQC_INL_H */
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

### Functions and Methods

- **OPJ_MQC_INL_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

