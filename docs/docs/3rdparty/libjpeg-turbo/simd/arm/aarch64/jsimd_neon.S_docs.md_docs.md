# Documentation for `docs/3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S_docs.md`
- **File Name**: `jsimd_neon.S_docs.md`
- **File Size**: 50,681 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S_docs.md](../../../../../../docs/3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/arm/aarch64` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S`
- **File Name**: `jsimd_neon.S`
- **File Size**: 98,482 bytes
- **File Type**: .s
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S](../../../../../3rdparty/libjpeg-turbo/simd/arm/aarch64/jsimd_neon.S)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/arm/aarch64` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
/*
 * Armv8 Neon optimizations for libjpeg-turbo
 *
 * Copyright (C) 2009-2011, Nokia Corporation and/or its subsidiary(-ies).
 *                          All Rights Reserved.
 * Author:  Siarhei Siamashka <siarhei.siamashka@nokia.com>
 * Copyright (C) 2013-2014, Linaro Limited.  All Rights Reserved.
 * Author:  Ragesh Radhakrishnan <ragesh.r@linaro.org>
 * Copyright (C) 2014-2016, 2020, D. R. Commander.  All Rights Reserved.
 * Copyright (C) 2015-2016, 2018, Matthieu Darbois.  All Rights Reserved.
 * Copyright (C) 2016, Siarhei Siamashka.  All Rights Reserved.
 *
 * This software is provided 'as-is', without any express or implied
 * warranty.  In no event will the authors be held liable for any damages
 * arising from the use of this software.
 *
 * Permission is granted to anyone to use this software for any purpose,
 * including commercial applications, and to alter it and redistribute it
 * freely, subject to the following restrictions:
 *
 * 1. The origin of this software must not be misrepresented; you must not
 *    claim that you wrote the original software. If you use this software
 *    in a product, an acknowledgment in the product documentation would be
 *    appreciated but is not required.
 * 2. Altered source versions must be plainly marked as such, and must not be
 *    misrepresented as being the original software.
 * 3. This notice may not be removed or altered from any source distribution.
 */

#if defined(__linux__) && defined(__ELF__)
.section .note.GNU-stack, "", %progbits  /* mark stack as non-executable */
#endif

#if defined(__APPLE__)
.section __DATA, __const
#elif defined(_WIN32)
.section .rdata
#else
.section .rodata, "a", %progbits
#endif

/* Constants for jsimd_idct_islow_neon() */

#define F_0_298   2446  /* FIX(0.298631336) */
#define F_0_390   3196  /* FIX(0.390180644) */
#define F_0_541   4433  /* FIX(0.541196100) */
#define F_0_765   6270  /* FIX(0.765366865) */
#define F_0_899   7373  /* FIX(0.899976223) */
#define F_1_175   9633  /* FIX(1.175875602) */
#define F_1_501  12299  /* FIX(1.501321110) */
#define F_1_847  15137  /* FIX(1.847759065) */
#define F_1_961  16069  /* FIX(1.961570560) */
#define F_2_053  16819  /* FIX(2.053119869) */
#define F_2_562  20995  /* FIX(2.562915447) */
#define F_3_072  25172  /* FIX(3.072711026) */

.balign 16
Ljsimd_idct_islow_neon_consts:
  .short F_0_298
  .short -F_0_390
  .short F_0_541
  .short F_0_765
  .short - F_0_899
  .short F_1_175
  .short F_1_501
  .short - F_1_847
  .short - F_1_961
  .short F_2_053
  .short - F_2_562
  .short F_3_072
  .short 0          /* padding */
  .short 0
  .short 0
  .short 0

#undef F_0_298
#undef F_0_390
#undef F_0_541
#undef F_0_765
#undef F_0_899
#undef F_1_175
#undef F_1_501
#undef F_1_847
#undef F_1_961
#undef F_2_053
#undef F_2_562
#undef F_3_072

/* Constants for jsimd_ycc_*_neon() */

.balign 16
Ljsimd_ycc_rgb_neon_consts:
  .short 0,      0,     0,      0
  .short 22971, -11277, -23401, 29033
  .short -128,  -128,   -128,   -128
  .short -128,  -128,   -128,   -128

/* Constants for jsimd_*_ycc_neon() */

.balign 16
Ljsimd_rgb_ycc_neon_consts:
  .short 19595, 38470, 7471, 11059
  .short 21709, 32768, 27439, 5329
  .short 32767, 128, 32767, 128
  .short 32767, 128, 32767, 128

/* Constants for jsimd_fdct_islow_neon() */

#define F_0_298   2446  /* FIX(0.298631336) */
#define F_0_390   3196  /* FIX(0.390180644) */
#define F_0_541   4433  /* FIX(0.541196100) */
#define F_0_765   6270  /* FIX(0.765366865) */
#define F_0_899   7373  /* FIX(0.899976223) */
#define F_1_175   9633  /* FIX(1.175875602) */
#define F_1_501  12299  /* FIX(1.501321110) */
#define F_1_847  15137  /* FIX(1.847759065) */
#define F_1_961  16069  /* FIX(1.961570560) */
#define F_2_053  16819  /* FIX(2.053119869) */
#define F_2_562  20995  /* FIX(2.562915447) */
#define F_3_072  25172  /* FIX(3.072711026) */

.balign 16
Ljsimd_fdct_islow_neon_consts:
  .short F_0_298
  .short -F_0_390
  .short F_0_541
  .short F_0_765
  .short - F_0_899
  .short F_1_175
  .short F_1_501
  .short - F_1_847
  .short - F_1_961
  .short F_2_053
  .short - F_2_562
  .short F_3_072
  .short 0          /* padding */
  .short 0
  .short 0
  .short 0

#undef F_0_298
#undef F_0_390
#undef F_0_541
#undef F_0_765
#undef F_0_899
#undef F_1_175
#undef F_1_501
#undef F_1_847
#undef F_1_961
#undef F_2_053
#undef F_2_562
#undef F_3_072

/* Constants for jsimd_huff_encode_one_block_neon() */

.balign 16
Ljsimd_huff_encode_one_block_neon_consts:
    .byte 0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, \
          0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80
    .byte    0,   1,   2,   3,  16,  17,  32,  33, \
            18,  19,   4,   5,   6,   7,  20,  21  /* L0 => L3 : 4 lines OK */
    .byte   34,  35,  48,  49, 255, 255,  50,  51, \
            36,  37,  22,  23,   8,   9,  10,  11  /* L0 => L3 : 4 lines OK */
    .byte    8,   9,  22,  23,  36,  37,  50,  51, \
           255, 255, 255, 255, 255, 255,  52,  53  /* L1 => L4 : 4 lines OK */
    .byte   54,  55,  40,  41,  26,  27,  12,  13, \
            14,  15,  28,  29,  42,  43,  56,  57  /* L0 => L3 : 4 lines OK */
    .byte    6,   7,  20,  21,  34,  35,  48,  49, \
            50,  51,  36,  37,  22,  23,   8,   9  /* L4 => L7 : 4 lines OK */
    .byte   42,  43,  28,  29,  14,  15,  30,  31, \
            44,  45,  58,  59, 255, 255, 255, 255  /* L1 => L4 : 4 lines OK */
    .byte  255, 255, 255, 255,  56,  57,  42,  43, \
            28,  29,  14,  15,  30,  31,  44,  45  /* L3 => L6 : 4 lines OK */
    .byte   26,  27,  40,  41,  42,  43,  28,  29, \
            14,  15,  30,  31,  44,  45,  46,  47  /* L5 => L7 : 3 lines OK */
    .byte  255, 255, 255, 255,   0,   1, 255, 255, \
           255, 255, 255, 255, 255, 255, 255, 255  /* L4 : 1 lines OK */
    .byte  255, 255, 255, 255, 255, 255, 255, 255, \
             0,   1,  16,  17,   2,   3, 255, 255  /* L5 => L6 : 2 lines OK */
    .byte  255, 255, 255, 255, 255, 255, 255, 255, \
           255, 255, 255, 255,   8,   9,  22,  23  /* L5 => L6 : 2 lines OK */
    .byte    4,   5,   6,   7, 255, 255, 255, 255, \
           255, 255, 255, 255, 255, 255, 255, 255  /* L7 : 1 line OK */

.text


/*****************************************************************************/

/* Supplementary macro for setting function attributes */
.macro asm_function fname
#ifdef __APPLE__
    .private_extern _\fname
    .globl _\fname
_\fname:
#else
    .global \fname
#ifdef __ELF__
    .hidden \fname
    .type \fname, %function
#endif
\fname:
#endif
.endm

/* Get symbol location */
.macro get_symbol_loc reg, symbol
#ifdef __APPLE__
    adrp            \reg, \symbol@PAGE
    add             \reg, \reg, \symbol@PAGEOFF
#else
    adrp            \reg, \symbol
    add             \reg, \reg, :lo12:\symbol
#endif
.endm

.macro transpose_8x8 l0, l1, l2, l3, l4, l5, l6, l7, t0, t1, t2, t3
    trn1            \t0\().8h, \l0\().8h, \l1\().8h
    trn1            \t1\().8h, \l2\().8h, \l3\().8h
    trn1            \t2\().8h, \l4\().8h, \l5\().8h
    trn1            \t3\().8h, \l6\().8h, \l7\().8h
    trn2            \l1\().8h, \l0\().8h, \l1\().8h
    trn2            \l3\().8h, \l2\().8h, \l3\().8h
    trn2            \l5\().8h, \l4\().8h, \l5\().8h
    trn2            \l7\().8h, \l6\().8h, \l7\().8h

    trn1            \l4\().4s, \t2\().4s, \t3\().4s
    trn2            \t3\().4s, \t2\().4s, \t3\().4s
    trn1            \t2\().4s, \t0\().4s, \t1\().4s
    trn2            \l2\().4s, \t0\().4s, \t1\().4s
    trn1            \t0\().4s, \l1\().4s, \l3\().4s
    trn2            \l3\().4s, \l1\().4s, \l3\().4s
    trn2            \t1\().4s, \l5\().4s, \l7\().4s
    trn1            \l5\().4s, \l5\().4s, \l7\().4s

    trn2            \l6\().2d, \l2\().2d, \t3\().2d
    trn1            \l0\().2d, \t2\().2d, \l4\().2d
    trn1            \l1\().2d, \t0\().2d, \l5\().2d
    trn2            \l7\().2d, \l3\().2d, \t1\().2d
    trn1            \l2\().2d, \l2\().2d, \t3\().2d
    trn2            \l4\().2d, \t2\().2d, \l4\().2d
    trn1            \l3\().2d, \l3\().2d, \t1\().2d
    trn2            \l5\().2d, \t0\().2d, \l5\().2d
.endm


#define CENTERJSAMPLE  128

/*****************************************************************************/

/*
 * Perform dequantization and inverse DCT on one block of coefficients.
 *
 * GLOBAL(void)
 * jsimd_idct_islow_neon(void *dct_table, JCOEFPTR coef_block,
 *                       JSAMPARRAY output_buf, JDIMENSION output_col)
 */

#define CONST_BITS  13
#define PASS1_BITS  2

#define XFIX_P_0_298  v0.h[0]
#define XFIX_N_0_390  v0.h[1]
#define XFIX_P_0_541  v0.h[2]
#define XFIX_P_0_765  v0.h[3]
#define XFIX_N_0_899  v0.h[4]
#define XFIX_P_1_175  v0.h[5]
#define XFIX_P_1_501  v0.h[6]
#define XFIX_N_1_847  v0.h[7]
#define XFIX_N_1_961  v1.h[0]
#define XFIX_P_2_053  v1.h[1]
#define XFIX_N_2_562  v1.h[2]
#define XFIX_P_3_072  v1.h[3]

asm_function jsimd_idct_islow_neon
    DCT_TABLE       .req x0
    COEF_BLOCK      .req x1
    OUTPUT_BUF      .req x2
    OUTPUT_COL      .req x3
    TMP1            .req x0
    TMP2            .req x1
    TMP3            .req x9
    TMP4            .req x10
    TMP5            .req x11
    TMP6            .req x12
    TMP7            .req x13
    TMP8            .req x14

    /* OUTPUT_COL is a JDIMENSION (unsigned int) argument, so the ABI doesn't
       guarantee that the upper (unused) 32 bits of x3 are valid.  This
       instruction ensures that those bits are set to zero. */
    uxtw x3, w3

    sub             sp, sp, #64
    get_symbol_loc  x15, Ljsimd_idct_islow_neon_consts
    mov             x10, sp
    st1             {v8.8b, v9.8b, v10.8b, v11.8b}, [x10], #32
    st1             {v12.8b, v13.8b, v14.8b, v15.8b}, [x10], #32
    ld1             {v0.8h, v1.8h}, [x15]
    ld1             {v2.8h, v3.8h, v4.8h, v5.8h}, [COEF_BLOCK], #64
    ld1             {v18.8h, v19.8h, v20.8h, v21.8h}, [DCT_TABLE], #64
    ld1             {v6.8h, v7.8h, v8.8h, v9.8h}, [COEF_BLOCK], #64
    ld1             {v22.8h, v23.8h, v24.8h, v25.8h}, [DCT_TABLE], #64

    cmeq            v16.8h, v3.8h, #0
    cmeq            v26.8h, v4.8h, #0
    cmeq            v27.8h, v5.8h, #0
    cmeq            v28.8h, v6.8h, #0
    cmeq            v29.8h, v7.8h, #0
    cmeq            v30.8h, v8.8h, #0
    cmeq            v31.8h, v9.8h, #0

    and             v10.16b, v16.16b, v26.16b
    and             v11.16b, v27.16b, v28.16b
    and             v12.16b, v29.16b, v30.16b
    and             v13.16b, v31.16b, v10.16b
    and             v14.16b, v11.16b, v12.16b
    mul             v2.8h, v2.8h, v18.8h
    and             v15.16b, v13.16b, v14.16b
    shl             v10.8h, v2.8h, #(PASS1_BITS)
    sqxtn           v16.8b, v15.8h
    mov             TMP1, v16.d[0]
    mvn             TMP2, TMP1

    cbnz            TMP2, 2f
    /* case all AC coeffs are zeros */
    dup             v2.2d, v10.d[0]
    dup             v6.2d, v10.d[1]
    mov             v3.16b, v2.16b
    mov             v7.16b, v6.16b
    mov             v4.16b, v2.16b
    mov             v8.16b, v6.16b
    mov             v5.16b, v2.16b
    mov             v9.16b, v6.16b
1:
    /* for this transpose, we should organise data like this:
     * 00, 01, 02, 03, 40, 41, 42, 43
     * 10, 11, 12, 13, 50, 51, 52, 53
     * 20, 21, 22, 23, 60, 61, 62, 63
     * 30, 31, 32, 33, 70, 71, 72, 73
     * 04, 05, 06, 07, 44, 45, 46, 47
     * 14, 15, 16, 17, 54, 55, 56, 57
     * 24, 25, 26, 27, 64, 65, 66, 67
     * 34, 35, 36, 37, 74, 75, 76, 77
     */
    trn1            v28.8h, v2.8h, v3.8h
    trn1            v29.8h, v4.8h, v5.8h
    trn1            v30.8h, v6.8h, v7.8h
    trn1            v31.8h, v8.8h, v9.8h
    trn2            v16.8h, v2.8h, v3.8h
    trn2            v17.8h, v4.8h, v5.8h
    trn2            v18.8h, v6.8h, v7.8h
    trn2            v19.8h, v8.8h, v9.8h
    trn1            v2.4s, v28.4s, v29.4s
    trn1            v6.4s, v30.4s, v31.4s
    trn1            v3.4s, v16.4s, v17.4s
    trn1            v7.4s, v18.4s, v19.4s
    trn2            v4.4s, v28.4s, v29.4s
    trn2            v8.4s, v30.4s, v31.4s
    trn2            v5.4s, v16.4s, v17.4s
    trn2            v9.4s, v18.4s, v19.4s
    /* Even part: reverse the even part of the forward DCT. */
    add             v18.8h, v4.8h, v8.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*2], quantptr[DCTSIZE*2]) + DEQUANTIZE(inptr[DCTSIZE*6], quantptr[DCTSIZE*6]) */
    add             v22.8h, v2.8h, v6.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) + DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull2          v19.4s, v18.8h, XFIX_P_0_541   /* z1h z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sub             v26.8h, v2.8h, v6.8h           /* z2 - z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) - DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull           v18.4s, v18.4h, XFIX_P_0_541   /* z1l z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sshll2          v23.4s, v22.8h, #(CONST_BITS)  /* tmp0h tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    mov             v21.16b, v19.16b               /* tmp3 = z1 */
    mov             v20.16b, v18.16b               /* tmp3 = z1 */
    smlal2          v19.4s, v8.8h, XFIX_N_1_847    /* tmp2h tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    smlal           v18.4s, v8.4h, XFIX_N_1_847    /* tmp2l tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    sshll2          v27.4s, v26.8h, #(CONST_BITS)  /* tmp1h tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    smlal2          v21.4s, v4.8h, XFIX_P_0_765    /* tmp3h tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    smlal           v20.4s, v4.4h, XFIX_P_0_765    /* tmp3l tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    sshll           v22.4s, v22.4h, #(CONST_BITS)  /* tmp0l tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    sshll           v26.4s, v26.4h, #(CONST_BITS)  /* tmp1l tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    add             v2.4s, v22.4s, v20.4s          /* tmp10l tmp10 = tmp0 + tmp3; */
    sub             v6.4s, v22.4s, v20.4s          /* tmp13l tmp13 = tmp0 - tmp3; */
    add             v8.4s, v26.4s, v18.4s          /* tmp11l tmp11 = tmp1 + tmp2; */
    sub             v4.4s, v26.4s, v18.4s          /* tmp12l tmp12 = tmp1 - tmp2; */
    add             v28.4s, v23.4s, v21.4s         /* tmp10h tmp10 = tmp0 + tmp3; */
    sub             v31.4s, v23.4s, v21.4s         /* tmp13h tmp13 = tmp0 - tmp3; */
    add             v29.4s, v27.4s, v19.4s         /* tmp11h tmp11 = tmp1 + tmp2; */
    sub             v30.4s, v27.4s, v19.4s         /* tmp12h tmp12 = tmp1 - tmp2; */

    /* Odd part per figure 8; the matrix is unitary and hence its
     * transpose is its inverse.  i0..i3 are y7,y5,y3,y1 respectively.
     */

    add             v22.8h, v9.8h, v5.8h    /* z3 = tmp0 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v24.8h, v7.8h, v3.8h    /* z4 = tmp1 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v18.8h, v9.8h, v3.8h    /* z1 = tmp0 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v20.8h, v7.8h, v5.8h    /* z2 = tmp1 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v26.8h, v22.8h, v24.8h  /* z5 = z3 + z4 */

    smull2          v11.4s, v9.8h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull2          v13.4s, v7.8h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull2          v15.4s, v5.8h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull2          v17.4s, v3.8h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull2          v27.4s, v26.8h, XFIX_P_1_175  /* z5h z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull2          v23.4s, v22.8h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull2          v25.4s, v24.8h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull2          v19.4s, v18.8h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull2          v21.4s, v20.8h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    smull           v10.4s, v9.4h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull           v12.4s, v7.4h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull           v14.4s, v5.4h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull           v16.4s, v3.4h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull           v26.4s, v26.4h, XFIX_P_1_175  /* z5l z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull           v22.4s, v22.4h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull           v24.4s, v24.4h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull           v18.4s, v18.4h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull           v20.4s, v20.4h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    add             v23.4s, v23.4s, v27.4s  /* z3 += z5 */
    add             v22.4s, v22.4s, v26.4s  /* z3 += z5 */
    add             v25.4s, v25.4s, v27.4s  /* z4 += z5 */
    add             v24.4s, v24.4s, v26.4s  /* z4 += z5 */

    add             v11.4s, v11.4s, v19.4s  /* tmp0 += z1 */
    add             v10.4s, v10.4s, v18.4s  /* tmp0 += z1 */
    add             v13.4s, v13.4s, v21.4s  /* tmp1 += z2 */
    add             v12.4s, v12.4s, v20.4s  /* tmp1 += z2 */
    add             v15.4s, v15.4s, v21.4s  /* tmp2 += z2 */
    add             v14.4s, v14.4s, v20.4s  /* tmp2 += z2 */
    add             v17.4s, v17.4s, v19.4s  /* tmp3 += z1 */
    add             v16.4s, v16.4s, v18.4s  /* tmp3 += z1 */

    add             v11.4s, v11.4s, v23.4s  /* tmp0 += z3 */
    add             v10.4s, v10.4s, v22.4s  /* tmp0 += z3 */
    add             v13.4s, v13.4s, v25.4s  /* tmp1 += z4 */
    add             v12.4s, v12.4s, v24.4s  /* tmp1 += z4 */
    add             v17.4s, v17.4s, v25.4s  /* tmp3 += z4 */
    add             v16.4s, v16.4s, v24.4s  /* tmp3 += z4 */
    add             v15.4s, v15.4s, v23.4s  /* tmp2 += z3 */
    add             v14.4s, v14.4s, v22.4s  /* tmp2 += z3 */

    /* Final output stage: inputs are tmp10..tmp13, tmp0..tmp3 */

    add             v18.4s, v2.4s, v16.4s   /* tmp10 + tmp3 */
    add             v19.4s, v28.4s, v17.4s  /* tmp10 + tmp3 */
    sub             v20.4s, v2.4s, v16.4s   /* tmp10 - tmp3 */
    sub             v21.4s, v28.4s, v17.4s  /* tmp10 - tmp3 */
    add             v22.4s, v8.4s, v14.4s   /* tmp11 + tmp2 */
    add             v23.4s, v29.4s, v15.4s  /* tmp11 + tmp2 */
    sub             v24.4s, v8.4s, v14.4s   /* tmp11 - tmp2 */
    sub             v25.4s, v29.4s, v15.4s  /* tmp11 - tmp2 */
    add             v26.4s, v4.4s, v12.4s   /* tmp12 + tmp1 */
    add             v27.4s, v30.4s, v13.4s  /* tmp12 + tmp1 */
    sub             v28.4s, v4.4s, v12.4s   /* tmp12 - tmp1 */
    sub             v29.4s, v30.4s, v13.4s  /* tmp12 - tmp1 */
    add             v14.4s, v6.4s, v10.4s   /* tmp13 + tmp0 */
    add             v15.4s, v31.4s, v11.4s  /* tmp13 + tmp0 */
    sub             v16.4s, v6.4s, v10.4s   /* tmp13 - tmp0 */
    sub             v17.4s, v31.4s, v11.4s  /* tmp13 - tmp0 */

    shrn            v2.4h, v18.4s, #16  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS+PASS1_BITS+3) */
    shrn            v9.4h, v20.4s, #16  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS+PASS1_BITS+3) */
    shrn            v3.4h, v22.4s, #16  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS+PASS1_BITS+3) */
    shrn            v8.4h, v24.4s, #16  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS+PASS1_BITS+3) */
    shrn            v4.4h, v26.4s, #16  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS+PASS1_BITS+3) */
    shrn            v7.4h, v28.4s, #16  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS+PASS1_BITS+3) */
    shrn            v5.4h, v14.4s, #16  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS+PASS1_BITS+3) */
    shrn            v6.4h, v16.4s, #16  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS+PASS1_BITS+3) */
    shrn2           v2.8h, v19.4s, #16  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS+PASS1_BITS+3) */
    shrn2           v9.8h, v21.4s, #16  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS+PASS1_BITS+3) */
    shrn2           v3.8h, v23.4s, #16  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS+PASS1_BITS+3) */
    shrn2           v8.8h, v25.4s, #16  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS+PASS1_BITS+3) */
    shrn2           v4.8h, v27.4s, #16  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS+PASS1_BITS+3) */
    shrn2           v7.8h, v29.4s, #16  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS+PASS1_BITS+3) */
    shrn2           v5.8h, v15.4s, #16  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS+PASS1_BITS+3) */
    shrn2           v6.8h, v17.4s, #16  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS+PASS1_BITS+3) */
    movi            v0.16b, #(CENTERJSAMPLE)
    /* Prepare pointers (dual-issue with Neon instructions) */
      ldp             TMP1, TMP2, [OUTPUT_BUF], 16
    sqrshrn         v28.8b, v2.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      ldp             TMP3, TMP4, [OUTPUT_BUF], 16
    sqrshrn         v29.8b, v3.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      add             TMP1, TMP1, OUTPUT_COL
    sqrshrn         v30.8b, v4.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      add             TMP2, TMP2, OUTPUT_COL
    sqrshrn         v31.8b, v5.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      add             TMP3, TMP3, OUTPUT_COL
    sqrshrn2        v28.16b, v6.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      add             TMP4, TMP4, OUTPUT_COL
    sqrshrn2        v29.16b, v7.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      ldp             TMP5, TMP6, [OUTPUT_BUF], 16
    sqrshrn2        v30.16b, v8.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      ldp             TMP7, TMP8, [OUTPUT_BUF], 16
    sqrshrn2        v31.16b, v9.8h, #(CONST_BITS + PASS1_BITS + 3 - 16)
      add             TMP5, TMP5, OUTPUT_COL
    add             v16.16b, v28.16b, v0.16b
      add             TMP6, TMP6, OUTPUT_COL
    add             v18.16b, v29.16b, v0.16b
      add             TMP7, TMP7, OUTPUT_COL
    add             v20.16b, v30.16b, v0.16b
      add             TMP8, TMP8, OUTPUT_COL
    add             v22.16b, v31.16b, v0.16b

    /* Transpose the final 8-bit samples */
    trn1            v28.16b, v16.16b, v18.16b
    trn1            v30.16b, v20.16b, v22.16b
    trn2            v29.16b, v16.16b, v18.16b
    trn2            v31.16b, v20.16b, v22.16b

    trn1            v16.8h, v28.8h, v30.8h
    trn2            v18.8h, v28.8h, v30.8h
    trn1            v20.8h, v29.8h, v31.8h
    trn2            v22.8h, v29.8h, v31.8h

    uzp1            v28.4s, v16.4s, v18.4s
    uzp2            v30.4s, v16.4s, v18.4s
    uzp1            v29.4s, v20.4s, v22.4s
    uzp2            v31.4s, v20.4s, v22.4s

    /* Store results to the output buffer */
    st1             {v28.d}[0], [TMP1]
    st1             {v29.d}[0], [TMP2]
    st1             {v28.d}[1], [TMP3]
    st1             {v29.d}[1], [TMP4]
    st1             {v30.d}[0], [TMP5]
    st1             {v31.d}[0], [TMP6]
    st1             {v30.d}[1], [TMP7]
    st1             {v31.d}[1], [TMP8]
    ld1             {v8.8b, v9.8b, v10.8b, v11.8b}, [sp], #32
    ld1             {v12.8b, v13.8b, v14.8b, v15.8b}, [sp], #32
    blr             x30

.balign 16
2:
    mul             v3.8h, v3.8h, v19.8h
    mul             v4.8h, v4.8h, v20.8h
    mul             v5.8h, v5.8h, v21.8h
    add             TMP4, xzr, TMP2, LSL #32
    mul             v6.8h, v6.8h, v22.8h
    mul             v7.8h, v7.8h, v23.8h
    adds            TMP3, xzr, TMP2, LSR #32
    mul             v8.8h, v8.8h, v24.8h
    mul             v9.8h, v9.8h, v25.8h
    b.ne            3f
    /* Right AC coef is zero */
    dup             v15.2d, v10.d[1]
    /* Even part: reverse the even part of the forward DCT. */
    add             v18.4h, v4.4h, v8.4h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*2], quantptr[DCTSIZE*2]) + DEQUANTIZE(inptr[DCTSIZE*6], quantptr[DCTSIZE*6]) */
    add             v22.4h, v2.4h, v6.4h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) + DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    sub             v26.4h, v2.4h, v6.4h           /* z2 - z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) - DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull           v18.4s, v18.4h, XFIX_P_0_541   /* z1l z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sshll           v22.4s, v22.4h, #(CONST_BITS)  /* tmp0l tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    mov             v20.16b, v18.16b               /* tmp3 = z1 */
    sshll           v26.4s, v26.4h, #(CONST_BITS)  /* tmp1l tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    smlal           v18.4s, v8.4h, XFIX_N_1_847    /* tmp2l tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    smlal           v20.4s, v4.4h, XFIX_P_0_765    /* tmp3l tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    add             v2.4s, v22.4s, v20.4s          /* tmp10l tmp10 = tmp0 + tmp3; */
    sub             v6.4s, v22.4s, v20.4s          /* tmp13l tmp13 = tmp0 - tmp3; */
    add             v8.4s, v26.4s, v18.4s          /* tmp11l tmp11 = tmp1 + tmp2; */
    sub             v4.4s, v26.4s, v18.4s          /* tmp12l tmp12 = tmp1 - tmp2; */

    /* Odd part per figure 8; the matrix is unitary and hence its
     * transpose is its inverse.  i0..i3 are y7,y5,y3,y1 respectively.
     */

    add             v22.4h, v9.4h, v5.4h    /* z3 = tmp0 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v24.4h, v7.4h, v3.4h    /* z4 = tmp1 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v18.4h, v9.4h, v3.4h    /* z1 = tmp0 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v20.4h, v7.4h, v5.4h    /* z2 = tmp1 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v26.4h, v22.4h, v24.4h  /* z5 = z3 + z4 */

    smull           v10.4s, v9.4h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull           v12.4s, v7.4h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull           v14.4s, v5.4h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull           v16.4s, v3.4h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull           v26.4s, v26.4h, XFIX_P_1_175  /* z5l z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull           v22.4s, v22.4h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull           v24.4s, v24.4h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull           v18.4s, v18.4h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull           v20.4s, v20.4h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    add             v22.4s, v22.4s, v26.4s  /* z3 += z5 */
    add             v24.4s, v24.4s, v26.4s  /* z4 += z5 */

    add             v10.4s, v10.4s, v18.4s  /* tmp0 += z1 */
    add             v12.4s, v12.4s, v20.4s  /* tmp1 += z2 */
    add             v14.4s, v14.4s, v20.4s  /* tmp2 += z2 */
    add             v16.4s, v16.4s, v18.4s  /* tmp3 += z1 */

    add             v10.4s, v10.4s, v22.4s  /* tmp0 += z3 */
    add             v12.4s, v12.4s, v24.4s  /* tmp1 += z4 */
    add             v16.4s, v16.4s, v24.4s  /* tmp3 += z4 */
    add             v14.4s, v14.4s, v22.4s  /* tmp2 += z3 */

    /* Final output stage: inputs are tmp10..tmp13, tmp0..tmp3 */

    add             v18.4s, v2.4s, v16.4s  /* tmp10 + tmp3 */
    sub             v20.4s, v2.4s, v16.4s  /* tmp10 - tmp3 */
    add             v22.4s, v8.4s, v14.4s  /* tmp11 + tmp2 */
    sub             v24.4s, v8.4s, v14.4s  /* tmp11 - tmp2 */
    add             v26.4s, v4.4s, v12.4s  /* tmp12 + tmp1 */
    sub             v28.4s, v4.4s, v12.4s  /* tmp12 - tmp1 */
    add             v14.4s, v6.4s, v10.4s  /* tmp13 + tmp0 */
    sub             v16.4s, v6.4s, v10.4s  /* tmp13 - tmp0 */

    rshrn           v2.4h, v18.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS-PASS1_BITS) */
    rshrn           v3.4h, v22.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS-PASS1_BITS) */
    rshrn           v4.4h, v26.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS-PASS1_BITS) */
    rshrn           v5.4h, v14.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v2.8h, v16.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v3.8h, v28.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS-PASS1_BITS) */
    rshrn2          v4.8h, v24.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS-PASS1_BITS) */
    rshrn2          v5.8h, v20.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS-PASS1_BITS) */
    mov             v6.16b, v15.16b
    mov             v7.16b, v15.16b
    mov             v8.16b, v15.16b
    mov             v9.16b, v15.16b
    b               1b

.balign 16
3:
    cbnz            TMP4, 4f
    /* Left AC coef is zero */
    dup             v14.2d, v10.d[0]
    /* Even part: reverse the even part of the forward DCT. */
    add             v18.8h, v4.8h, v8.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*2], quantptr[DCTSIZE*2]) + DEQUANTIZE(inptr[DCTSIZE*6], quantptr[DCTSIZE*6]) */
    add             v22.8h, v2.8h, v6.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) + DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull2          v19.4s, v18.8h, XFIX_P_0_541   /* z1h z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sub             v26.8h, v2.8h, v6.8h           /* z2 - z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) - DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    sshll2          v23.4s, v22.8h, #(CONST_BITS)  /* tmp0h tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    mov             v21.16b, v19.16b               /* tmp3 = z1 */
    smlal2          v19.4s, v8.8h, XFIX_N_1_847    /* tmp2h tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    sshll2          v27.4s, v26.8h, #(CONST_BITS)  /* tmp1h tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    smlal2          v21.4s, v4.8h, XFIX_P_0_765    /* tmp3h tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    add             v28.4s, v23.4s, v21.4s         /* tmp10h tmp10 = tmp0 + tmp3; */
    sub             v31.4s, v23.4s, v21.4s         /* tmp13h tmp13 = tmp0 - tmp3; */
    add             v29.4s, v27.4s, v19.4s         /* tmp11h tmp11 = tmp1 + tmp2; */
    sub             v30.4s, v27.4s, v19.4s         /* tmp12h tmp12 = tmp1 - tmp2; */

    /* Odd part per figure 8; the matrix is unitary and hence its
     * transpose is its inverse.  i0..i3 are y7,y5,y3,y1 respectively.
     */

    add             v22.8h, v9.8h, v5.8h    /* z3 = tmp0 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v24.8h, v7.8h, v3.8h    /* z4 = tmp1 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v18.8h, v9.8h, v3.8h    /* z1 = tmp0 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v20.8h, v7.8h, v5.8h    /* z2 = tmp1 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v26.8h, v22.8h, v24.8h  /* z5 = z3 + z4 */

    smull2          v11.4s, v9.8h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull2          v13.4s, v7.8h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull2          v15.4s, v5.8h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull2          v17.4s, v3.8h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull2          v27.4s, v26.8h, XFIX_P_1_175  /* z5h z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull2          v23.4s, v22.8h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull2          v25.4s, v24.8h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull2          v19.4s, v18.8h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull2          v21.4s, v20.8h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    add             v23.4s, v23.4s, v27.4s  /* z3 += z5 */
    add             v22.4s, v22.4s, v26.4s  /* z3 += z5 */
    add             v25.4s, v25.4s, v27.4s  /* z4 += z5 */
    add             v24.4s, v24.4s, v26.4s  /* z4 += z5 */

    add             v11.4s, v11.4s, v19.4s  /* tmp0 += z1 */
    add             v13.4s, v13.4s, v21.4s  /* tmp1 += z2 */
    add             v15.4s, v15.4s, v21.4s  /* tmp2 += z2 */
    add             v17.4s, v17.4s, v19.4s  /* tmp3 += z1 */

    add             v11.4s, v11.4s, v23.4s  /* tmp0 += z3 */
    add             v13.4s, v13.4s, v25.4s  /* tmp1 += z4 */
    add             v17.4s, v17.4s, v25.4s  /* tmp3 += z4 */
    add             v15.4s, v15.4s, v23.4s  /* tmp2 += z3 */

    /* Final output stage: inputs are tmp10..tmp13, tmp0..tmp3 */

    add             v19.4s, v28.4s, v17.4s  /* tmp10 + tmp3 */
    sub             v21.4s, v28.4s, v17.4s  /* tmp10 - tmp3 */
    add             v23.4s, v29.4s, v15.4s  /* tmp11 + tmp2 */
    sub             v25.4s, v29.4s, v15.4s  /* tmp11 - tmp2 */
    add             v27.4s, v30.4s, v13.4s  /* tmp12 + tmp1 */
    sub             v29.4s, v30.4s, v13.4s  /* tmp12 - tmp1 */
    add             v15.4s, v31.4s, v11.4s  /* tmp13 + tmp0 */
    sub             v17.4s, v31.4s, v11.4s  /* tmp13 - tmp0 */

    mov             v2.16b, v14.16b
    mov             v3.16b, v14.16b
    mov             v4.16b, v14.16b
    mov             v5.16b, v14.16b
    rshrn           v6.4h, v19.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS-PASS1_BITS) */
    rshrn           v7.4h, v23.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS-PASS1_BITS) */
    rshrn           v8.4h, v27.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS-PASS1_BITS) */
    rshrn           v9.4h, v15.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v6.8h, v17.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v7.8h, v29.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS-PASS1_BITS) */
    rshrn2          v8.8h, v25.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS-PASS1_BITS) */
    rshrn2          v9.8h, v21.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS-PASS1_BITS) */
    b               1b

.balign 16
4:
    /* "No" AC coef is zero */
    /* Even part: reverse the even part of the forward DCT. */
    add             v18.8h, v4.8h, v8.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*2], quantptr[DCTSIZE*2]) + DEQUANTIZE(inptr[DCTSIZE*6], quantptr[DCTSIZE*6]) */
    add             v22.8h, v2.8h, v6.8h           /* z2 + z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) + DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull2          v19.4s, v18.8h, XFIX_P_0_541   /* z1h z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sub             v26.8h, v2.8h, v6.8h           /* z2 - z3 = DEQUANTIZE(inptr[DCTSIZE*0], quantptr[DCTSIZE*0]) - DEQUANTIZE(inptr[DCTSIZE*4], quantptr[DCTSIZE*4]) */
    smull           v18.4s, v18.4h, XFIX_P_0_541   /* z1l z1 = MULTIPLY(z2 + z3, FIX_0_541196100); */
    sshll2          v23.4s, v22.8h, #(CONST_BITS)  /* tmp0h tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    mov             v21.16b, v19.16b               /* tmp3 = z1 */
    mov             v20.16b, v18.16b               /* tmp3 = z1 */
    smlal2          v19.4s, v8.8h, XFIX_N_1_847    /* tmp2h tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    smlal           v18.4s, v8.4h, XFIX_N_1_847    /* tmp2l tmp2 = z1 + MULTIPLY(z3, -FIX_1_847759065); */
    sshll2          v27.4s, v26.8h, #(CONST_BITS)  /* tmp1h tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    smlal2          v21.4s, v4.8h, XFIX_P_0_765    /* tmp3h tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    smlal           v20.4s, v4.4h, XFIX_P_0_765    /* tmp3l tmp3 = z1 + MULTIPLY(z2, FIX_0_765366865); */
    sshll           v22.4s, v22.4h, #(CONST_BITS)  /* tmp0l tmp0 = LEFT_SHIFT(z2 + z3, CONST_BITS); */
    sshll           v26.4s, v26.4h, #(CONST_BITS)  /* tmp1l tmp1 = LEFT_SHIFT(z2 - z3, CONST_BITS); */
    add             v2.4s, v22.4s, v20.4s          /* tmp10l tmp10 = tmp0 + tmp3; */
    sub             v6.4s, v22.4s, v20.4s          /* tmp13l tmp13 = tmp0 - tmp3; */
    add             v8.4s, v26.4s, v18.4s          /* tmp11l tmp11 = tmp1 + tmp2; */
    sub             v4.4s, v26.4s, v18.4s          /* tmp12l tmp12 = tmp1 - tmp2; */
    add             v28.4s, v23.4s, v21.4s         /* tmp10h tmp10 = tmp0 + tmp3; */
    sub             v31.4s, v23.4s, v21.4s         /* tmp13h tmp13 = tmp0 - tmp3; */
    add             v29.4s, v27.4s, v19.4s         /* tmp11h tmp11 = tmp1 + tmp2; */
    sub             v30.4s, v27.4s, v19.4s         /* tmp12h tmp12 = tmp1 - tmp2; */

    /* Odd part per figure 8; the matrix is unitary and hence its
     * transpose is its inverse.  i0..i3 are y7,y5,y3,y1 respectively.
     */

    add             v22.8h, v9.8h, v5.8h    /* z3 = tmp0 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v24.8h, v7.8h, v3.8h    /* z4 = tmp1 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v18.8h, v9.8h, v3.8h    /* z1 = tmp0 + tmp3 = DEQUANTIZE(inptr[DCTSIZE*7], quantptr[DCTSIZE*7]) + DEQUANTIZE(inptr[DCTSIZE*1], quantptr[DCTSIZE*1]) */
    add             v20.8h, v7.8h, v5.8h    /* z2 = tmp1 + tmp2 = DEQUANTIZE(inptr[DCTSIZE*5], quantptr[DCTSIZE*5]) + DEQUANTIZE(inptr[DCTSIZE*3], quantptr[DCTSIZE*3]) */
    add             v26.8h, v22.8h, v24.8h  /* z5 = z3 + z4 */

    smull2          v11.4s, v9.8h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull2          v13.4s, v7.8h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull2          v15.4s, v5.8h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull2          v17.4s, v3.8h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull2          v27.4s, v26.8h, XFIX_P_1_175  /* z5h z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull2          v23.4s, v22.8h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull2          v25.4s, v24.8h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull2          v19.4s, v18.8h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull2          v21.4s, v20.8h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    smull           v10.4s, v9.4h, XFIX_P_0_298   /* tmp0 = MULTIPLY(tmp0, FIX_0_298631336) */
    smull           v12.4s, v7.4h, XFIX_P_2_053   /* tmp1 = MULTIPLY(tmp1, FIX_2_053119869) */
    smull           v14.4s, v5.4h, XFIX_P_3_072   /* tmp2 = MULTIPLY(tmp2, FIX_3_072711026) */
    smull           v16.4s, v3.4h, XFIX_P_1_501   /* tmp3 = MULTIPLY(tmp3, FIX_1_501321110) */
    smull           v26.4s, v26.4h, XFIX_P_1_175  /* z5l z5 = MULTIPLY(z3 + z4, FIX_1_175875602) */
    smull           v22.4s, v22.4h, XFIX_N_1_961  /* z3 = MULTIPLY(z3, -FIX_1_961570560) */
    smull           v24.4s, v24.4h, XFIX_N_0_390  /* z4 = MULTIPLY(z4, -FIX_0_390180644) */
    smull           v18.4s, v18.4h, XFIX_N_0_899  /* z1 = MULTIPLY(z1, -FIX_0_899976223) */
    smull           v20.4s, v20.4h, XFIX_N_2_562  /* z2 = MULTIPLY(z2, -FIX_2_562915447) */

    add             v23.4s, v23.4s, v27.4s  /* z3 += z5 */
    add             v22.4s, v22.4s, v26.4s  /* z3 += z5 */
    add             v25.4s, v25.4s, v27.4s  /* z4 += z5 */
    add             v24.4s, v24.4s, v26.4s  /* z4 += z5 */

    add             v11.4s, v11.4s, v19.4s  /* tmp0 += z1 */
    add             v10.4s, v10.4s, v18.4s  /* tmp0 += z1 */
    add             v13.4s, v13.4s, v21.4s  /* tmp1 += z2 */
    add             v12.4s, v12.4s, v20.4s  /* tmp1 += z2 */
    add             v15.4s, v15.4s, v21.4s  /* tmp2 += z2 */
    add             v14.4s, v14.4s, v20.4s  /* tmp2 += z2 */
    add             v17.4s, v17.4s, v19.4s  /* tmp3 += z1 */
    add             v16.4s, v16.4s, v18.4s  /* tmp3 += z1 */

    add             v11.4s, v11.4s, v23.4s  /* tmp0 += z3 */
    add             v10.4s, v10.4s, v22.4s  /* tmp0 += z3 */
    add             v13.4s, v13.4s, v25.4s  /* tmp1 += z4 */
    add             v12.4s, v12.4s, v24.4s  /* tmp1 += z4 */
    add             v17.4s, v17.4s, v25.4s  /* tmp3 += z4 */
    add             v16.4s, v16.4s, v24.4s  /* tmp3 += z4 */
    add             v15.4s, v15.4s, v23.4s  /* tmp2 += z3 */
    add             v14.4s, v14.4s, v22.4s  /* tmp2 += z3 */

    /* Final output stage: inputs are tmp10..tmp13, tmp0..tmp3 */

    add             v18.4s, v2.4s, v16.4s   /* tmp10 + tmp3 */
    add             v19.4s, v28.4s, v17.4s  /* tmp10 + tmp3 */
    sub             v20.4s, v2.4s, v16.4s   /* tmp10 - tmp3 */
    sub             v21.4s, v28.4s, v17.4s  /* tmp10 - tmp3 */
    add             v22.4s, v8.4s, v14.4s   /* tmp11 + tmp2 */
    add             v23.4s, v29.4s, v15.4s  /* tmp11 + tmp2 */
    sub             v24.4s, v8.4s, v14.4s   /* tmp11 - tmp2 */
    sub             v25.4s, v29.4s, v15.4s  /* tmp11 - tmp2 */
    add             v26.4s, v4.4s, v12.4s   /* tmp12 + tmp1 */
    add             v27.4s, v30.4s, v13.4s  /* tmp12 + tmp1 */
    sub             v28.4s, v4.4s, v12.4s   /* tmp12 - tmp1 */
    sub             v29.4s, v30.4s, v13.4s  /* tmp12 - tmp1 */
    add             v14.4s, v6.4s, v10.4s   /* tmp13 + tmp0 */
    add             v15.4s, v31.4s, v11.4s  /* tmp13 + tmp0 */
    sub             v16.4s, v6.4s, v10.4s   /* tmp13 - tmp0 */
    sub             v17.4s, v31.4s, v11.4s  /* tmp13 - tmp0 */

    rshrn           v2.4h, v18.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS-PASS1_BITS) */
    rshrn           v3.4h, v22.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS-PASS1_BITS) */
    rshrn           v4.4h, v26.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS-PASS1_BITS) */
    rshrn           v5.4h, v14.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS-PASS1_BITS) */
    rshrn           v6.4h, v19.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*0] = (int)DESCALE(tmp10 + tmp3, CONST_BITS-PASS1_BITS) */
    rshrn           v7.4h, v23.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*1] = (int)DESCALE(tmp11 + tmp2, CONST_BITS-PASS1_BITS) */
    rshrn           v8.4h, v27.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*2] = (int)DESCALE(tmp12 + tmp1, CONST_BITS-PASS1_BITS) */
    rshrn           v9.4h, v15.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*3] = (int)DESCALE(tmp13 + tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v2.8h, v16.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v3.8h, v28.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS-PASS1_BITS) */
    rshrn2          v4.8h, v24.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS-PASS1_BITS) */
    rshrn2          v5.8h, v20.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS-PASS1_BITS) */
    rshrn2          v6.8h, v17.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*4] = (int)DESCALE(tmp13 - tmp0, CONST_BITS-PASS1_BITS) */
    rshrn2          v7.8h, v29.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*5] = (int)DESCALE(tmp12 - tmp1, CONST_BITS-PASS1_BITS) */
    rshrn2          v8.8h, v25.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*6] = (int)DESCALE(tmp11 - tmp2, CONST_BITS-PASS1_BITS) */
    rshrn2          v9.8h, v21.4s, #(CONST_BITS - PASS1_BITS)  /* wsptr[DCTSIZE*7] = (int)DESCALE(tmp10 - tmp3, CONST_BITS-PASS1_BITS) */
    b               1b

    .unreq          DCT_TABLE
    .unreq          COEF_BLOCK
    .unreq          OUTPUT_BUF
    .unreq          OUTPUT_COL
    .unreq          TMP1
    .unreq          TMP2
    .unreq          TMP3
    .unreq          TMP4
    .unreq          TMP5
    .unreq          TMP6
    .unreq          TMP7
    .unreq          TMP8

#undef CENTERJSAMPLE
#undef CONST_BITS
#undef PASS1_BITS
#undef XFIX_P_0_298
#undef XFIX_N_0_390
#undef XFIX_P_0_541
#undef XFIX_P_0_765
#undef XFIX_N_0_899
#undef XFIX_P_1_175
#undef XFIX_P_1_501
#undef XFIX_N_1_847
#undef XFIX_N_1_961
#undef XFIX_P_2_053
#undef XFIX_N_2_562
#undef XFIX_P_3_072


/*****************************************************************************/

/*
 * jsimd_ycc_extrgb_convert_neon
 * jsimd_ycc_extbgr_convert_neon
 * jsimd_ycc_extrgbx_convert_neon
 * jsimd_ycc_extbgrx_convert_neon
 * jsimd_ycc_extxbgr_convert_neon
 * jsimd_ycc_extxrgb_convert_neon
 *
 * Colorspace conversion YCbCr -> RGB
 */

.macro do_load size
  .if \size == 8
    ld1             {v4.8b}, [U], 8
    ld1             {v5.8b}, [V], 8
    ld1             {v0.8b}, [Y], 8
    prfm            pldl1keep, [U, #64]
    prfm            pldl1keep, [V, #64]
    prfm            pldl1keep, [Y, #64]
  .elseif \size == 4
    ld1             {v4.b}[0], [U], 1
    ld1             {v4.b}[1], [U], 1
    ld1             {v4.b}[2], [U], 1
    ld1             {v4.b}[3], [U], 1
    ld1             {v5.b}[0], [V], 1
    ld1             {v5.b}[1], [V], 1
    ld1             {v5.b}[2], [V], 1
    ld1             {v5.b}[3], [V], 1
    ld1             {v0.b}[0], [Y], 1
    ld1             {v0.b}[1], [Y], 1
    ld1             {v0.b}[2], [Y], 1
    ld1             {v0.b}[3], [Y], 1
  .elseif \size == 2
    ld1             {v4.b}[4], [U], 1
    ld1             {v4.b}[5], [U], 1
    ld1             {v5.b}[4], [V], 1
    ld1             {v5.b}[5], [V], 1
    ld1             {v0.b}[4], [Y], 1
    ld1             {v0.b}[5], [Y], 1
  .elseif \size == 1
    ld1             {v4.b}[6], [U], 1
    ld1             {v5.b}[6], [V], 1
    ld1             {v0.b}[6], [Y], 1
  .else
    .error unsupported macroblock size
  .endif
.endm

.macro do_store bpp, size, fast_st3
  .if \bpp == 24
    .if \size == 8
      .if \fast_st3 == 1
        st3         {v10.8b, v11.8b, v12.8b}, [RGB], 24
      .else
        st1         {v10.b}[0], [RGB], #1
        st1         {v11.b}[0], [RGB], #1
        st1         {v12.b}[0], [RGB], #1

        st1         {v10.b}[1], [RGB], #1
        st1         {v11.b}[1], [RGB], #1
        st1         {v12.b}[1], [RGB], #1

        st1         {v10.b}[2], [RGB], #1
        st1         {v11.b}[2], [RGB], #1
        st1         {v12.b}[2], [RGB], #1

        st1         {v10.b}[3], [RGB], #1
        st1         {v11.b}[3], [RGB], #1
        st1         {v12.b}[3], [RGB], #1

        st1         {v10.b}[4], [RGB], #1
        st1         {v11.b}[4], [RGB], #1
        st1         {v12.b}[4], [RGB], #1

        st1         {v10.b}[5], [RGB], #1
        st1         {v11.b}[5], [RGB], #1
        st1         {v12.b}[5], [RGB], #1

        st1         {v10.b}[6], [RGB], #1
        st1         {v11.b}[6], [RGB], #1
        st1         {v12.b}[6], [RGB], #1

        st1         {v10.b}[7], [RGB], #1
        st1         {v11.b}[7], [RGB], #1
        st1         {v12.b}[7], [RGB], #1
      .endif
    .elseif \size == 4
      st3           {v10.b, v11.b, v12.b}[0], [RGB], 3
      st3           {v10.b, v11.b, v12.b}[1], [RGB], 3
      st3           {v10.b, v11.b, v12.b}[2], [RGB], 3
      st3           {v10.b, v11.b, v12.b}[3], [RGB], 3
    .elseif \size == 2
      st3           {v10.b, v11.b, v12.b}[4], [RGB], 3
      st3           {v10.b, v11.b, v12.b}[5], [RGB], 3
    .elseif \size == 1
      st3           {v10.b, v11.b, v12.b}[6], [RGB], 3
    .else
     .error unsupported macroblock size
    .endif
  .elseif \bpp == 32
    .if \size == 8
      st4           {v10.8b, v11.8b, v12.8b, v13.8b}, [RGB], 32
    .elseif \size == 4
      st4           {v10.b, v11.b, v12.b, v13.b}[0], [RGB], 4
      st4           {v10.b, v11.b, v12.b, v13.b}[1], [RGB], 4
      st4           {v10.b, v11.b, v12.b, v13.b}[2], [RGB], 4
      st4           {v10.b, v11.b, v12.b, v13.b}[3], [RGB], 4
    .elseif \size == 2
      st4           {v10.b, v11.b, v12.b, v13.b}[4], [RGB], 4
      st4           {v10.b, v11.b, v12.b, v13.b}[5], [RGB], 4
    .elseif \size == 1
      st4           {v10.b, v11.b, v12.b, v13.b}[6], [RGB], 4
    .els
... [truncated]
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

