# Documentation for `3rdparty/openjpeg/openjp2/t1_generate_luts.c`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/t1_generate_luts.c`
- **File Name**: `t1_generate_luts.c`
- **File Size**: 10,564 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/openjpeg/openjp2/t1_generate_luts.c](../../../3rdparty/openjpeg/openjp2/t1_generate_luts.c)

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
 * Copyright (c) 2007, Callum Lerwick <seg@haxxed.com>
 * Copyright (c) 2012, Carl Hetherington
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

// defined elsewhere
extern OPJ_BOOL vlc_init_tables();
extern OPJ_BOOL vlc_tables_initialized;
extern int vlc_tbl0[1024];
extern int vlc_tbl1[1024];

static int t1_init_ctxno_zc(OPJ_UINT32 f, OPJ_UINT32 orient)
{
    int h, v, d, n, t, hv;
    n = 0;
    h = ((f & T1_SIGMA_3) != 0) + ((f & T1_SIGMA_5) != 0);
    v = ((f & T1_SIGMA_1) != 0) + ((f & T1_SIGMA_7) != 0);
    d = ((f & T1_SIGMA_0) != 0) + ((f & T1_SIGMA_2) != 0) + ((
                f & T1_SIGMA_8) != 0) + ((f & T1_SIGMA_6) != 0);

    switch (orient) {
    case 2:
        t = h;
        h = v;
        v = t;
    case 0:
    case 1:
        if (!h) {
            if (!v) {
                if (!d) {
                    n = 0;
                } else if (d == 1) {
                    n = 1;
                } else {
                    n = 2;
                }
            } else if (v == 1) {
                n = 3;
            } else {
                n = 4;
            }
        } else if (h == 1) {
            if (!v) {
                if (!d) {
                    n = 5;
                } else {
                    n = 6;
                }
            } else {
                n = 7;
            }
        } else {
            n = 8;
        }
        break;
    case 3:
        hv = h + v;
        if (!d) {
            if (!hv) {
                n = 0;
            } else if (hv == 1) {
                n = 1;
            } else {
                n = 2;
            }
        } else if (d == 1) {
            if (!hv) {
                n = 3;
            } else if (hv == 1) {
                n = 4;
            } else {
                n = 5;
            }
        } else if (d == 2) {
            if (!hv) {
                n = 6;
            } else {
                n = 7;
            }
        } else {
            n = 8;
        }
        break;
    }

    return (T1_CTXNO_ZC + n);
}

static int t1_init_ctxno_sc(OPJ_UINT32 f)
{
    int hc, vc, n;
    n = 0;

    hc = opj_int_min(((f & (T1_LUT_SIG_E | T1_LUT_SGN_E)) ==
                      T1_LUT_SIG_E) + ((f & (T1_LUT_SIG_W | T1_LUT_SGN_W)) == T1_LUT_SIG_W),
                     1) - opj_int_min(((f & (T1_LUT_SIG_E | T1_LUT_SGN_E)) ==
                                       (T1_LUT_SIG_E | T1_LUT_SGN_E)) +
                                      ((f & (T1_LUT_SIG_W | T1_LUT_SGN_W)) ==
                                       (T1_LUT_SIG_W | T1_LUT_SGN_W)), 1);

    vc = opj_int_min(((f & (T1_LUT_SIG_N | T1_LUT_SGN_N)) ==
                      T1_LUT_SIG_N) + ((f & (T1_LUT_SIG_S | T1_LUT_SGN_S)) == T1_LUT_SIG_S),
                     1) - opj_int_min(((f & (T1_LUT_SIG_N | T1_LUT_SGN_N)) ==
                                       (T1_LUT_SIG_N | T1_LUT_SGN_N)) +
                                      ((f & (T1_LUT_SIG_S | T1_LUT_SGN_S)) ==
                                       (T1_LUT_SIG_S | T1_LUT_SGN_S)), 1);

    if (hc < 0) {
        hc = -hc;
        vc = -vc;
    }
    if (!hc) {
        if (vc == -1) {
            n = 1;
        } else if (!vc) {
            n = 0;
        } else {
            n = 1;
        }
    } else if (hc == 1) {
        if (vc == -1) {
            n = 2;
        } else if (!vc) {
            n = 3;
        } else {
            n = 4;
        }
    }

    return (T1_CTXNO_SC + n);
}

static int t1_init_spb(OPJ_UINT32 f)
{
    int hc, vc, n;

    hc = opj_int_min(((f & (T1_LUT_SIG_E | T1_LUT_SGN_E)) ==
                      T1_LUT_SIG_E) + ((f & (T1_LUT_SIG_W | T1_LUT_SGN_W)) == T1_LUT_SIG_W),
                     1) - opj_int_min(((f & (T1_LUT_SIG_E | T1_LUT_SGN_E)) ==
                                       (T1_LUT_SIG_E | T1_LUT_SGN_E)) +
                                      ((f & (T1_LUT_SIG_W | T1_LUT_SGN_W)) ==
                                       (T1_LUT_SIG_W | T1_LUT_SGN_W)), 1);

    vc = opj_int_min(((f & (T1_LUT_SIG_N | T1_LUT_SGN_N)) ==
                      T1_LUT_SIG_N) + ((f & (T1_LUT_SIG_S | T1_LUT_SGN_S)) == T1_LUT_SIG_S),
                     1) - opj_int_min(((f & (T1_LUT_SIG_N | T1_LUT_SGN_N)) ==
                                       (T1_LUT_SIG_N | T1_LUT_SGN_N)) +
                                      ((f & (T1_LUT_SIG_S | T1_LUT_SGN_S)) ==
                                       (T1_LUT_SIG_S | T1_LUT_SGN_S)), 1);

    if (!hc && !vc) {
        n = 0;
    } else {
        n = (!(hc > 0 || (!hc && vc > 0)));
    }

    return n;
}

static void dump_array16(int array[], int size)
{
    int i;
    --size;
    for (i = 0; i < size; ++i) {
        printf("0x%04x,", array[i]);
        if (!((i + 1) & 0x7)) {
            printf("\n    ");
        } else {
            printf(" ");
        }
    }
    printf("0x%04x\n};\n\n", array[size]);
}

int main(int argc, char **argv)
{
    unsigned int i, j;
    double u, v, t;

    int lut_ctxno_zc[2048];
    int lut_nmsedec_sig[1 << T1_NMSEDEC_BITS];
    int lut_nmsedec_sig0[1 << T1_NMSEDEC_BITS];
    int lut_nmsedec_ref[1 << T1_NMSEDEC_BITS];
    int lut_nmsedec_ref0[1 << T1_NMSEDEC_BITS];
    (void)argc;
    (void)argv;

    printf("/* This file was automatically generated by t1_generate_luts.c */\n\n");

    /* lut_ctxno_zc */
    for (j = 0; j < 4; ++j) {
        for (i = 0; i < 512; ++i) {
            OPJ_UINT32 orient = j;
            if (orient == 2) {
                orient = 1;
            } else if (orient == 1) {
                orient = 2;
            }
            lut_ctxno_zc[(orient << 9) | i] = t1_init_ctxno_zc(i, j);
        }
    }

    printf("static const OPJ_BYTE lut_ctxno_zc[2048] = {\n    ");
    for (i = 0; i < 2047; ++i) {
        printf("%i,", lut_ctxno_zc[i]);
        if (!((i + 1) & 0x1f)) {
            printf("\n    ");
        } else {
            printf(" ");
        }
    }
    printf("%i\n};\n\n", lut_ctxno_zc[2047]);

    /* lut_ctxno_sc */
    printf("static const OPJ_BYTE lut_ctxno_sc[256] = {\n    ");
    for (i = 0; i < 255; ++i) {
        printf("0x%x,", t1_init_ctxno_sc(i));
        if (!((i + 1) & 0xf)) {
            printf("\n    ");
        } else {
            printf(" ");
        }
    }
    printf("0x%x\n};\n\n", t1_init_ctxno_sc(255));

    /* lut_spb */
    printf("static const OPJ_BYTE lut_spb[256] = {\n    ");
    for (i = 0; i < 255; ++i) {
        printf("%i,", t1_init_spb(i));
        if (!((i + 1) & 0x1f)) {
            printf("\n    ");
        } else {
            printf(" ");
        }
    }
    printf("%i\n};\n\n", t1_init_spb(255));

    /* FIXME FIXME FIXME */
    /* fprintf(stdout,"nmsedec luts:\n"); */
    for (i = 0U; i < (1U << T1_NMSEDEC_BITS); ++i) {
        t = i / pow(2, T1_NMSEDEC_FRACBITS);
        u = t;
        v = t - 1.5;
        lut_nmsedec_sig[i] =
            opj_int_max(0,
                        (int)(floor((u * u - v * v) * pow(2, T1_NMSEDEC_FRACBITS) + 0.5) / pow(2,
                                T1_NMSEDEC_FRACBITS) * 8192.0));
        lut_nmsedec_sig0[i] =
            opj_int_max(0,
                        (int)(floor((u * u) * pow(2, T1_NMSEDEC_FRACBITS) + 0.5) / pow(2,
                                T1_NMSEDEC_FRACBITS) * 8192.0));
        u = t - 1.0;
        if (i & (1 << (T1_NMSEDEC_BITS - 1))) {
            v = t - 1.5;
        } else {
            v = t - 0.5;
        }
        lut_nmsedec_ref[i] =
            opj_int_max(0,
                        (int)(floor((u * u - v * v) * pow(2, T1_NMSEDEC_FRACBITS) + 0.5) / pow(2,
                                T1_NMSEDEC_FRACBITS) * 8192.0));
        lut_nmsedec_ref0[i] =
            opj_int_max(0,
                        (int)(floor((u * u) * pow(2, T1_NMSEDEC_FRACBITS) + 0.5) / pow(2,
                                T1_NMSEDEC_FRACBITS) * 8192.0));
    }

    printf("static const OPJ_INT16 lut_nmsedec_sig[1U << T1_NMSEDEC_BITS] = {\n    ");
    dump_array16(lut_nmsedec_sig, 1U << T1_NMSEDEC_BITS);

    printf("static const OPJ_INT16 lut_nmsedec_sig0[1U << T1_NMSEDEC_BITS] = {\n    ");
    dump_array16(lut_nmsedec_sig0, 1U << T1_NMSEDEC_BITS);

    printf("static const OPJ_INT16 lut_nmsedec_ref[1U << T1_NMSEDEC_BITS] = {\n    ");
    dump_array16(lut_nmsedec_ref, 1U << T1_NMSEDEC_BITS);

    printf("static const OPJ_INT16 lut_nmsedec_ref0[1U << T1_NMSEDEC_BITS] = {\n    ");
    dump_array16(lut_nmsedec_ref0, 1U << T1_NMSEDEC_BITS);

    vlc_tables_initialized = vlc_init_tables();
    printf("static const OPJ_UINT16 vlc_tbl0[1024] = {\n    ");
    dump_array16(vlc_tbl0, 1024);
    printf("static const OPJ_UINT16 vlc_tbl1[1024] = {\n    ");
    dump_array16(vlc_tbl1, 1024);

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

