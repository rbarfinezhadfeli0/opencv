# Documentation for `docs/3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S_docs.md`
- **File Name**: `jsimd_dspr2.S_docs.md`
- **File Size**: 50,649 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S_docs.md](../../../../../docs/3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libjpeg-turbo/simd/mips` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S`

## File Metadata

- **Full Path**: `3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S`
- **File Name**: `jsimd_dspr2.S`
- **File Size**: 148,685 bytes
- **File Type**: .s
- **Link to Source**: [3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S](../../../../3rdparty/libjpeg-turbo/simd/mips/jsimd_dspr2.S)

## Purpose and Role

This file is located in the `3rdparty/libjpeg-turbo/simd/mips` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
/*
 * MIPS DSPr2 optimizations for libjpeg-turbo
 *
 * Copyright (C) 2013-2014, MIPS Technologies, Inc., California.
 *                          All Rights Reserved.
 * Authors:  Teodora Novkovic <teodora.novkovic@imgtec.com>
 *           Darko Laus       <darko.laus@imgtec.com>
 * Copyright (C) 2015, D. R. Commander.  All Rights Reserved.
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

#include "jsimd_dspr2_asm.h"


/*****************************************************************************/
LEAF_DSPR2(jsimd_c_null_convert_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = input_buf
 * a2     = output_buf
 * a3     = output_row
 * 16(sp) = num_rows
 * 20(sp) = cinfo->num_components
 *
 * Null conversion for compression
 */
    SAVE_REGS_ON_STACK 8, s0, s1

    lw          t9, 24(sp)      /* t9 = num_rows */
    lw          s0, 28(sp)      /* s0 = cinfo->num_components */
    andi        t0, a0, 3       /* t0 = cinfo->image_width & 3 */
    beqz        t0, 4f          /* no residual */
     nop
0:
    addiu       t9, t9, -1
    bltz        t9, 7f
     li         t1, 0
1:
    sll         t3, t1, 2
    lwx         t5, t3(a2)      /* t5 = outptr = output_buf[ci] */
    lw          t2, 0(a1)       /* t2 = inptr = *input_buf */
    sll         t4, a3, 2
    lwx         t5, t4(t5)      /* t5 = outptr = output_buf[ci][output_row] */
    addu        t2, t2, t1
    addu        s1, t5, a0
    addu        t6, t5, t0
2:
    lbu         t3, 0(t2)
    addiu       t5, t5, 1
    sb          t3, -1(t5)
    bne         t6, t5, 2b
     addu       t2, t2, s0
3:
    lbu         t3, 0(t2)
    addu        t4, t2, s0
    addu        t7, t4, s0
    addu        t8, t7, s0
    addu        t2, t8, s0
    lbu         t4, 0(t4)
    lbu         t7, 0(t7)
    lbu         t8, 0(t8)
    addiu       t5, t5, 4
    sb          t3, -4(t5)
    sb          t4, -3(t5)
    sb          t7, -2(t5)
    bne         s1, t5, 3b
     sb         t8, -1(t5)
    addiu       t1, t1, 1
    bne         t1, s0, 1b
     nop
    addiu       a1, a1, 4
    bgez        t9, 0b
     addiu      a3, a3, 1
    b           7f
     nop
4:
    addiu       t9, t9, -1
    bltz        t9, 7f
     li         t1, 0
5:
    sll         t3, t1, 2
    lwx         t5, t3(a2)      /* t5 = outptr = output_buf[ci] */
    lw          t2, 0(a1)       /* t2 = inptr = *input_buf */
    sll         t4, a3, 2
    lwx         t5, t4(t5)      /* t5 = outptr = output_buf[ci][output_row] */
    addu        t2, t2, t1
    addu        s1, t5, a0
    addu        t6, t5, t0
6:
    lbu         t3, 0(t2)
    addu        t4, t2, s0
    addu        t7, t4, s0
    addu        t8, t7, s0
    addu        t2, t8, s0
    lbu         t4, 0(t4)
    lbu         t7, 0(t7)
    lbu         t8, 0(t8)
    addiu       t5, t5, 4
    sb          t3, -4(t5)
    sb          t4, -3(t5)
    sb          t7, -2(t5)
    bne         s1, t5, 6b
     sb         t8, -1(t5)
    addiu       t1, t1, 1
    bne         t1, s0, 5b
     nop
    addiu       a1, a1, 4
    bgez        t9, 4b
     addiu      a3, a3, 1
7:
    RESTORE_REGS_FROM_STACK 8, s0, s1

    j           ra
     nop

END(jsimd_c_null_convert_dspr2)


/*****************************************************************************/
/*
 * jsimd_extrgb_ycc_convert_dspr2
 * jsimd_extbgr_ycc_convert_dspr2
 * jsimd_extrgbx_ycc_convert_dspr2
 * jsimd_extbgrx_ycc_convert_dspr2
 * jsimd_extxbgr_ycc_convert_dspr2
 * jsimd_extxrgb_ycc_convert_dspr2
 *
 * Colorspace conversion RGB -> YCbCr
 */

.macro GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2  colorid, pixel_size, \
                                             r_offs, g_offs, b_offs

.macro DO_RGB_TO_YCC  r, g, b, inptr
    lbu         \r, \r_offs(\inptr)
    lbu         \g, \g_offs(\inptr)
    lbu         \b, \b_offs(\inptr)
    addiu       \inptr, \pixel_size
.endm

LEAF_DSPR2(jsimd_\colorid\()_ycc_convert_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = input_buf
 * a2     = output_buf
 * a3     = output_row
 * 16(sp) = num_rows
 */
    SAVE_REGS_ON_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    lw          t7, 48(sp)      /* t7 = num_rows */
    li          s0, 0x4c8b      /* FIX(0.29900) */
    li          s1, 0x9646      /* FIX(0.58700) */
    li          s2, 0x1d2f      /* FIX(0.11400) */
    li          s3, 0xffffd4cd  /* -FIX(0.16874) */
    li          s4, 0xffffab33  /* -FIX(0.33126) */
    li          s5, 0x8000      /* FIX(0.50000) */
    li          s6, 0xffff94d1  /* -FIX(0.41869) */
    li          s7, 0xffffeb2f  /* -FIX(0.08131) */
    li          t8, 0x807fff    /* CBCR_OFFSET + ONE_HALF-1 */

0:
    addiu       t7, -1          /* --num_rows */
    lw          t6, 0(a1)       /* t6 = input_buf[0] */
    lw          t0, 0(a2)
    lw          t1, 4(a2)
    lw          t2, 8(a2)
    sll         t3, a3, 2
    lwx         t0, t3(t0)      /* t0 = output_buf[0][output_row] */
    lwx         t1, t3(t1)      /* t1 = output_buf[1][output_row] */
    lwx         t2, t3(t2)      /* t2 = output_buf[2][output_row] */

    addu        t9, t2, a0      /* t9 = end address */
    addiu       a3, 1

1:
    DO_RGB_TO_YCC t3, t4, t5, t6

    mtlo        s5, $ac0
    mtlo        t8, $ac1
    mtlo        t8, $ac2
    maddu       $ac0, s2, t5
    maddu       $ac1, s5, t5
    maddu       $ac2, s5, t3
    maddu       $ac0, s0, t3
    maddu       $ac1, s3, t3
    maddu       $ac2, s6, t4
    maddu       $ac0, s1, t4
    maddu       $ac1, s4, t4
    maddu       $ac2, s7, t5
    extr.w      t3, $ac0, 16
    extr.w      t4, $ac1, 16
    extr.w      t5, $ac2, 16
    sb          t3, 0(t0)
    sb          t4, 0(t1)
    sb          t5, 0(t2)
    addiu       t0, 1
    addiu       t2, 1
    bne         t2, t9, 1b
     addiu      t1, 1
    bgtz        t7, 0b
     addiu      a1, 4

    RESTORE_REGS_FROM_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    j           ra
     nop
END(jsimd_\colorid\()_ycc_convert_dspr2)

.purgem DO_RGB_TO_YCC

.endm

/*-------------------------------------id -- pix R  G  B */
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extrgb,  3, 0, 1, 2
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extbgr,  3, 2, 1, 0
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extrgbx, 4, 0, 1, 2
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extbgrx, 4, 2, 1, 0
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extxbgr, 4, 3, 2, 1
GENERATE_JSIMD_RGB_YCC_CONVERT_DSPR2 extxrgb, 4, 1, 2, 3


/*****************************************************************************/
/*
 * jsimd_ycc_extrgb_convert_dspr2
 * jsimd_ycc_extbgr_convert_dspr2
 * jsimd_ycc_extrgbx_convert_dspr2
 * jsimd_ycc_extbgrx_convert_dspr2
 * jsimd_ycc_extxbgr_convert_dspr2
 * jsimd_ycc_extxrgb_convert_dspr2
 *
 * Colorspace conversion YCbCr -> RGB
 */

.macro GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2  colorid, pixel_size, \
                                             r_offs, g_offs, b_offs, a_offs

.macro STORE_YCC_TO_RGB  scratch0 scratch1 scratch2 outptr
    sb          \scratch0, \r_offs(\outptr)
    sb          \scratch1, \g_offs(\outptr)
    sb          \scratch2, \b_offs(\outptr)
.if (\pixel_size == 4)
    li          t0, 0xFF
    sb          t0, \a_offs(\outptr)
.endif
    addiu       \outptr, \pixel_size
.endm

LEAF_DSPR2(jsimd_ycc_\colorid\()_convert_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = input_buf
 * a2     = input_row
 * a3     = output_buf
 * 16(sp) = num_rows
 */
    SAVE_REGS_ON_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    lw          s1, 48(sp)
    li          t3, 0x8000
    li          t4, 0x166e9     /* FIX(1.40200) */
    li          t5, 0x1c5a2     /* FIX(1.77200) */
    li          t6, 0xffff492e  /* -FIX(0.71414) */
    li          t7, 0xffffa7e6  /* -FIX(0.34414) */
    repl.ph     t8, 128

0:
    lw          s0, 0(a3)
    lw          t0, 0(a1)
    lw          t1, 4(a1)
    lw          t2, 8(a1)
    sll         s5, a2, 2
    addiu       s1, -1
    lwx         s2, s5(t0)
    lwx         s3, s5(t1)
    lwx         s4, s5(t2)
    addu        t9, s2, a0
    addiu       a2, 1

1:
    lbu         s7, 0(s4)       /* cr */
    lbu         s6, 0(s3)       /* cb */
    lbu         s5, 0(s2)       /* y */
    addiu       s2, 1
    addiu       s4, 1
    addiu       s7, -128
    addiu       s6, -128
    mul         t2, t7, s6
    mul         t0, t6, s7      /* Crgtab[cr] */
    sll         s7, 15
    mulq_rs.w   t1, t4, s7      /* Crrtab[cr] */
    sll         s6, 15
    addu        t2, t3          /* Cbgtab[cb] */
    addu        t2, t0

    mulq_rs.w   t0, t5, s6      /* Cbbtab[cb] */
    sra         t2, 16
    addu        t1, s5
    addu        t2, s5          /* add y */
    ins         t2, t1, 16, 16
    subu.ph     t2, t2, t8
    addu        t0, s5
    shll_s.ph   t2, t2, 8
    subu        t0, 128
    shra.ph     t2, t2, 8
    shll_s.w    t0, t0, 24
    addu.ph     t2, t2, t8      /* clip & store */
    sra         t0, t0, 24
    sra         t1, t2, 16
    addiu       t0, 128

    STORE_YCC_TO_RGB t1, t2, t0, s0

    bne         s2, t9, 1b
     addiu      s3, 1
    bgtz        s1, 0b
     addiu      a3, 4

    RESTORE_REGS_FROM_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    j           ra
     nop
END(jsimd_ycc_\colorid\()_convert_dspr2)

.purgem STORE_YCC_TO_RGB

.endm

/*-------------------------------------id -- pix R  G  B  A */
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extrgb,  3, 0, 1, 2, 3
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extbgr,  3, 2, 1, 0, 3
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extrgbx, 4, 0, 1, 2, 3
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extbgrx, 4, 2, 1, 0, 3
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extxbgr, 4, 3, 2, 1, 0
GENERATE_JSIMD_YCC_RGB_CONVERT_DSPR2 extxrgb, 4, 1, 2, 3, 0


/*****************************************************************************/
/*
 * jsimd_extrgb_gray_convert_dspr2
 * jsimd_extbgr_gray_convert_dspr2
 * jsimd_extrgbx_gray_convert_dspr2
 * jsimd_extbgrx_gray_convert_dspr2
 * jsimd_extxbgr_gray_convert_dspr2
 * jsimd_extxrgb_gray_convert_dspr2
 *
 * Colorspace conversion RGB -> GRAY
 */

.macro GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2  colorid, pixel_size, \
                                              r_offs, g_offs, b_offs

.macro DO_RGB_TO_GRAY  r, g, b, inptr
    lbu         \r, \r_offs(\inptr)
    lbu         \g, \g_offs(\inptr)
    lbu         \b, \b_offs(\inptr)
    addiu       \inptr, \pixel_size
.endm

LEAF_DSPR2(jsimd_\colorid\()_gray_convert_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = input_buf
 * a2     = output_buf
 * a3     = output_row
 * 16(sp) = num_rows
 */
    SAVE_REGS_ON_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    li          s0, 0x4c8b      /* s0 = FIX(0.29900) */
    li          s1, 0x9646      /* s1 = FIX(0.58700) */
    li          s2, 0x1d2f      /* s2 = FIX(0.11400) */
    li          s7, 0x8000      /* s7 = FIX(0.50000) */
    lw          s6, 48(sp)
    andi        t7, a0, 3

0:
    addiu       s6, -1          /* s6 = num_rows */
    lw          t0, 0(a1)
    lw          t1, 0(a2)
    sll         t3, a3, 2
    lwx         t1, t3(t1)
    addiu       a3, 1
    addu        t9, t1, a0
    subu        t8, t9, t7
    beq         t1, t8, 2f
     nop

1:
    DO_RGB_TO_GRAY t3, t4, t5, t0
    DO_RGB_TO_GRAY s3, s4, s5, t0

    mtlo        s7, $ac0
    maddu       $ac0, s2, t5
    maddu       $ac0, s1, t4
    maddu       $ac0, s0, t3
    mtlo        s7, $ac1
    maddu       $ac1, s2, s5
    maddu       $ac1, s1, s4
    maddu       $ac1, s0, s3
    extr.w      t6, $ac0, 16

    DO_RGB_TO_GRAY t3, t4, t5, t0
    DO_RGB_TO_GRAY s3, s4, s5, t0

    mtlo        s7, $ac0
    maddu       $ac0, s2, t5
    maddu       $ac0, s1, t4
    extr.w      t2, $ac1, 16
    maddu       $ac0, s0, t3
    mtlo        s7, $ac1
    maddu       $ac1, s2, s5
    maddu       $ac1, s1, s4
    maddu       $ac1, s0, s3
    extr.w      t5, $ac0, 16
    sb          t6, 0(t1)
    sb          t2, 1(t1)
    extr.w      t3, $ac1, 16
    addiu       t1, 4
    sb          t5, -2(t1)
    sb          t3, -1(t1)
    bne         t1, t8, 1b
     nop

2:
    beqz        t7, 4f
     nop

3:
    DO_RGB_TO_GRAY t3, t4, t5, t0

    mtlo        s7, $ac0
    maddu       $ac0, s2, t5
    maddu       $ac0, s1, t4
    maddu       $ac0, s0, t3
    extr.w      t6, $ac0, 16
    sb          t6, 0(t1)
    addiu       t1, 1
    bne         t1, t9, 3b
     nop

4:
    bgtz        s6, 0b
     addiu      a1, 4

    RESTORE_REGS_FROM_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    j           ra
     nop
END(jsimd_\colorid\()_gray_convert_dspr2)

.purgem DO_RGB_TO_GRAY

.endm

/*-------------------------------------id --  pix R  G  B */
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extrgb,  3, 0, 1, 2
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extbgr,  3, 2, 1, 0
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extrgbx, 4, 0, 1, 2
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extbgrx, 4, 2, 1, 0
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extxbgr, 4, 3, 2, 1
GENERATE_JSIMD_RGB_GRAY_CONVERT_DSPR2 extxrgb, 4, 1, 2, 3


/*****************************************************************************/
/*
 * jsimd_h2v2_merged_upsample_dspr2
 * jsimd_h2v2_extrgb_merged_upsample_dspr2
 * jsimd_h2v2_extrgbx_merged_upsample_dspr2
 * jsimd_h2v2_extbgr_merged_upsample_dspr2
 * jsimd_h2v2_extbgrx_merged_upsample_dspr2
 * jsimd_h2v2_extxbgr_merged_upsample_dspr2
 * jsimd_h2v2_extxrgb_merged_upsample_dspr2
 *
 * Merged h2v2 upsample routines
 */
.macro GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2  colorid, pixel_size, \
                                            r1_offs, g1_offs, \
                                            b1_offs, a1_offs, \
                                            r2_offs, g2_offs, \
                                            b2_offs, a2_offs

.macro STORE_H2V2_2_PIXELS  scratch0 scratch1 scratch2 scratch3 scratch4 \
                            scratch5 outptr
    sb          \scratch0, \r1_offs(\outptr)
    sb          \scratch1, \g1_offs(\outptr)
    sb          \scratch2, \b1_offs(\outptr)
    sb          \scratch3, \r2_offs(\outptr)
    sb          \scratch4, \g2_offs(\outptr)
    sb          \scratch5, \b2_offs(\outptr)
.if (\pixel_size == 8)
    li          \scratch0, 0xFF
    sb          \scratch0, \a1_offs(\outptr)
    sb          \scratch0, \a2_offs(\outptr)
.endif
    addiu       \outptr, \pixel_size
.endm

.macro STORE_H2V2_1_PIXEL  scratch0 scratch1 scratch2 outptr
    sb          \scratch0, \r1_offs(\outptr)
    sb          \scratch1, \g1_offs(\outptr)
    sb          \scratch2, \b1_offs(\outptr)

.if (\pixel_size == 8)
    li          t0, 0xFF
    sb          t0, \a1_offs(\outptr)
.endif
.endm

LEAF_DSPR2(jsimd_h2v2_\colorid\()_merged_upsample_dspr2)
/*
 * a0     = cinfo->output_width
 * a1     = input_buf
 * a2     = in_row_group_ctr
 * a3     = output_buf
 * 16(sp) = cinfo->sample_range_limit
 */
    SAVE_REGS_ON_STACK 40, s0, s1, s2, s3, s4, s5, s6, s7, ra

    lw          t9, 56(sp)      /* cinfo->sample_range_limit */
    lw          v0, 0(a1)
    lw          v1, 4(a1)
    lw          t0, 8(a1)
    sll         t1, a2, 3
    addiu       t2, t1, 4
    sll         t3, a2, 2
    lw          t4, 0(a3)       /* t4 = output_buf[0] */
    lwx         t1, t1(v0)      /* t1 = input_buf[0][in_row_group_ctr*2] */
    lwx         t2, t2(v0)      /* t2 = input_buf[0][in_row_group_ctr*2 + 1] */
    lwx         t5, t3(v1)      /* t5 = input_buf[1][in_row_group_ctr] */
    lwx         t6, t3(t0)      /* t6 = input_buf[2][in_row_group_ctr] */
    lw          t7, 4(a3)       /* t7 = output_buf[1] */
    li          s1, 0xe6ea
    addiu       t8, s1, 0x7fff    /* t8 = 0x166e9 [FIX(1.40200)] */
    addiu       s0, t8, 0x5eb9    /* s0 = 0x1c5a2 [FIX(1.77200)] */
    addiu       s1, zero, 0xa7e6  /* s4 = 0xffffa7e6 [-FIX(0.34414)] */
    xori        s2, s1, 0xeec8    /* s3 = 0xffff492e [-FIX(0.71414)] */
    srl         t3, a0, 1
    blez        t3, 2f
     addu       t0, t5, t3      /* t0 = end address */
 1:
    lbu         t3, 0(t5)
    lbu         s3, 0(t6)
    addiu       t5, t5, 1
    addiu       t3, t3, -128    /* (cb - 128) */
    addiu       s3, s3, -128    /* (cr - 128) */
    mult        $ac1, s1, t3
    madd        $ac1, s2, s3
    sll         s3, s3, 15
    sll         t3, t3, 15
    mulq_rs.w   s4, t8, s3      /* s4 = (C1 * cr + ONE_HALF)>> SCALEBITS */
    extr_r.w    s5, $ac1, 16
    mulq_rs.w   s6, s0, t3      /* s6 = (C2 * cb + ONE_HALF)>> SCALEBITS */
    lbu         v0, 0(t1)
    addiu       t6, t6, 1
    addiu       t1, t1, 2
    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         AT, 0(t3)
    lbu         s7, 0(s3)
    lbu         ra, 0(v1)
    lbu         v0, -1(t1)
    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         t3, 0(t3)
    lbu         s3, 0(s3)
    lbu         v1, 0(v1)
    lbu         v0, 0(t2)

    STORE_H2V2_2_PIXELS AT, s7, ra, t3, s3, v1, t4

    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         AT, 0(t3)
    lbu         s7, 0(s3)
    lbu         ra, 0(v1)
    lbu         v0, 1(t2)
    addiu       t2, t2, 2
    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         t3, 0(t3)
    lbu         s3, 0(s3)
    lbu         v1, 0(v1)

    STORE_H2V2_2_PIXELS AT, s7, ra, t3, s3, v1, t7

    bne         t0, t5, 1b
     nop
2:
    andi        t0, a0, 1
    beqz        t0, 4f
     lbu        t3, 0(t5)
    lbu         s3, 0(t6)
    addiu       t3, t3, -128    /* (cb - 128) */
    addiu       s3, s3, -128    /* (cr - 128) */
    mult        $ac1, s1, t3
    madd        $ac1, s2, s3
    sll         s3, s3, 15
    sll         t3, t3, 15
    lbu         v0, 0(t1)
    extr_r.w    s5, $ac1, 16
    mulq_rs.w   s4, t8, s3      /* s4 = (C1 * cr + ONE_HALF)>> SCALEBITS */
    mulq_rs.w   s6, s0, t3      /* s6 = (C2 * cb + ONE_HALF)>> SCALEBITS */
    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         t3, 0(t3)
    lbu         s3, 0(s3)
    lbu         v1, 0(v1)
    lbu         v0, 0(t2)

    STORE_H2V2_1_PIXEL t3, s3, v1, t4

    addu        t3, v0, s4      /* y+cred */
    addu        s3, v0, s5      /* y+cgreen */
    addu        v1, v0, s6      /* y+cblue */
    addu        t3, t9, t3      /* y+cred */
    addu        s3, t9, s3      /* y+cgreen */
    addu        v1, t9, v1      /* y+cblue */
    lbu         t3, 0(t3)
    lbu         s3, 0(s3)
    lbu         v1, 0(v1)

    STORE_H2V2_1_PIXEL t3, s3, v1, t7
4:
    RESTORE_REGS_FROM_STACK 40, s0, s1, s2, s3, s4, s5, s6, s7, ra

    j           ra
     nop

END(jsimd_h2v2_\colorid\()_merged_upsample_dspr2)

.purgem STORE_H2V2_1_PIXEL
.purgem STORE_H2V2_2_PIXELS
.endm

/*------------------------------------id -- pix R1 G1 B1 A1 R2 G2 B2 A2 */
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extrgb,  6, 0, 1, 2, 6, 3, 4, 5, 6
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extbgr,  6, 2, 1, 0, 3, 5, 4, 3, 6
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extrgbx, 8, 0, 1, 2, 3, 4, 5, 6, 7
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extbgrx, 8, 2, 1, 0, 3, 6, 5, 4, 7
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extxbgr, 8, 3, 2, 1, 0, 7, 6, 5, 4
GENERATE_H2V2_MERGED_UPSAMPLE_DSPR2 extxrgb, 8, 1, 2, 3, 0, 5, 6, 7, 4


/*****************************************************************************/
/*
 * jsimd_h2v1_merged_upsample_dspr2
 * jsimd_h2v1_extrgb_merged_upsample_dspr2
 * jsimd_h2v1_extrgbx_merged_upsample_dspr2
 * jsimd_h2v1_extbgr_merged_upsample_dspr2
 * jsimd_h2v1_extbgrx_merged_upsample_dspr2
 * jsimd_h2v1_extxbgr_merged_upsample_dspr2
 * jsimd_h2v1_extxrgb_merged_upsample_dspr2
 *
 * Merged h2v1 upsample routines
 */

.macro GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2  colorid, pixel_size, \
                                            r1_offs, g1_offs, \
                                            b1_offs, a1_offs, \
                                            r2_offs, g2_offs, \
                                            b2_offs, a2_offs

.macro STORE_H2V1_2_PIXELS  scratch0 scratch1 scratch2 scratch3 scratch4 \
                            scratch5 outptr
    sb          \scratch0, \r1_offs(\outptr)
    sb          \scratch1, \g1_offs(\outptr)
    sb          \scratch2, \b1_offs(\outptr)
    sb          \scratch3, \r2_offs(\outptr)
    sb          \scratch4, \g2_offs(\outptr)
    sb          \scratch5, \b2_offs(\outptr)
.if (\pixel_size == 8)
    li          t0, 0xFF
    sb          t0, \a1_offs(\outptr)
    sb          t0, \a2_offs(\outptr)
.endif
    addiu       \outptr, \pixel_size
.endm

.macro STORE_H2V1_1_PIXEL  scratch0 scratch1 scratch2 outptr
    sb          \scratch0, \r1_offs(\outptr)
    sb          \scratch1, \g1_offs(\outptr)
    sb          \scratch2, \b1_offs(\outptr)
.if (\pixel_size == 8)
    li          t0, 0xFF
    sb          t0, \a1_offs(\outptr)
.endif
.endm

LEAF_DSPR2(jsimd_h2v1_\colorid\()_merged_upsample_dspr2)
/*
 * a0     = cinfo->output_width
 * a1     = input_buf
 * a2     = in_row_group_ctr
 * a3     = output_buf
 * 16(sp) = range_limit
 */
    SAVE_REGS_ON_STACK 40, s0, s1, s2, s3, s4, s5, s6, s7, ra

    li          t0, 0xe6ea
    lw          t1, 0(a1)         /* t1 = input_buf[0] */
    lw          t2, 4(a1)         /* t2 = input_buf[1] */
    lw          t3, 8(a1)         /* t3 = input_buf[2] */
    lw          t8, 56(sp)        /* t8 = range_limit */
    addiu       s1, t0, 0x7fff    /* s1 = 0x166e9 [FIX(1.40200)] */
    addiu       s2, s1, 0x5eb9    /* s2 = 0x1c5a2 [FIX(1.77200)] */
    addiu       s0, t0, 0x9916    /* s0 = 0x8000 */
    addiu       s4, zero, 0xa7e6  /* s4 = 0xffffa7e6 [-FIX(0.34414)] */
    xori        s3, s4, 0xeec8    /* s3 = 0xffff492e [-FIX(0.71414)] */
    srl         t0, a0, 1
    sll         t4, a2, 2
    lwx         s5, t4(t1)      /* s5 = inptr0 */
    lwx         s6, t4(t2)      /* s6 = inptr1 */
    lwx         s7, t4(t3)      /* s7 = inptr2 */
    lw          t7, 0(a3)       /* t7 = outptr */
    blez        t0, 2f
     addu       t9, s6, t0      /* t9 = end address */
1:
    lbu         t2, 0(s6)       /* t2 = cb */
    lbu         t0, 0(s7)       /* t0 = cr */
    lbu         t1, 0(s5)       /* t1 = y */
    addiu       t2, t2, -128    /* t2 = cb - 128 */
    addiu       t0, t0, -128    /* t0 = cr - 128 */
    mult        $ac1, s4, t2
    madd        $ac1, s3, t0
    sll         t0, t0, 15
    sll         t2, t2, 15
    mulq_rs.w   t0, s1, t0      /* t0 = (C1*cr + ONE_HALF)>> SCALEBITS */
    extr_r.w    t5, $ac1, 16
    mulq_rs.w   t6, s2, t2      /* t6 = (C2*cb + ONE_HALF)>> SCALEBITS */
    addiu       s7, s7, 1
    addiu       s6, s6, 1
    addu        t2, t1, t0      /* t2 = y + cred */
    addu        t3, t1, t5      /* t3 = y + cgreen */
    addu        t4, t1, t6      /* t4 = y + cblue */
    addu        t2, t8, t2
    addu        t3, t8, t3
    addu        t4, t8, t4
    lbu         t1, 1(s5)
    lbu         v0, 0(t2)
    lbu         v1, 0(t3)
    lbu         ra, 0(t4)
    addu        t2, t1, t0
    addu        t3, t1, t5
    addu        t4, t1, t6
    addu        t2, t8, t2
    addu        t3, t8, t3
    addu        t4, t8, t4
    lbu         t2, 0(t2)
    lbu         t3, 0(t3)
    lbu         t4, 0(t4)

    STORE_H2V1_2_PIXELS v0, v1, ra, t2, t3, t4, t7

    bne         t9, s6, 1b
     addiu      s5, s5, 2
2:
    andi        t0, a0, 1
    beqz        t0, 4f
     nop
3:
    lbu         t2, 0(s6)
    lbu         t0, 0(s7)
    lbu         t1, 0(s5)
    addiu       t2, t2, -128    /* (cb - 128) */
    addiu       t0, t0, -128    /* (cr - 128) */
    mul         t3, s4, t2
    mul         t4, s3, t0
    sll         t0, t0, 15
    sll         t2, t2, 15
    mulq_rs.w   t0, s1, t0      /* (C1*cr + ONE_HALF)>> SCALEBITS */
    mulq_rs.w   t6, s2, t2      /* (C2*cb + ONE_HALF)>> SCALEBITS */
    addu        t3, t3, s0
    addu        t3, t4, t3
    sra         t5, t3, 16      /* (C4*cb + ONE_HALF + C3*cr)>> SCALEBITS */
    addu        t2, t1, t0      /* y + cred */
    addu        t3, t1, t5      /* y + cgreen */
    addu        t4, t1, t6      /* y + cblue */
    addu        t2, t8, t2
    addu        t3, t8, t3
    addu        t4, t8, t4
    lbu         t2, 0(t2)
    lbu         t3, 0(t3)
    lbu         t4, 0(t4)

    STORE_H2V1_1_PIXEL t2, t3, t4, t7
4:
    RESTORE_REGS_FROM_STACK 40, s0, s1, s2, s3, s4, s5, s6, s7, ra

    j           ra
     nop

END(jsimd_h2v1_\colorid\()_merged_upsample_dspr2)

.purgem STORE_H2V1_1_PIXEL
.purgem STORE_H2V1_2_PIXELS
.endm

/*------------------------------------id -- pix R1 G1 B1 A1 R2 G2 B2 A2 */
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extrgb,  6, 0, 1, 2, 6, 3, 4, 5, 6
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extbgr,  6, 2, 1, 0, 3, 5, 4, 3, 6
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extrgbx, 8, 0, 1, 2, 3, 4, 5, 6, 7
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extbgrx, 8, 2, 1, 0, 3, 6, 5, 4, 7
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extxbgr, 8, 3, 2, 1, 0, 7, 6, 5, 4
GENERATE_H2V1_MERGED_UPSAMPLE_DSPR2 extxrgb, 8, 1, 2, 3, 0, 5, 6, 7, 4


/*****************************************************************************/
/*
 * jsimd_h2v2_fancy_upsample_dspr2
 *
 * Fancy processing for the common case of 2:1 horizontal and 2:1 vertical.
 */
LEAF_DSPR2(jsimd_h2v2_fancy_upsample_dspr2)
/*
 * a0 = cinfo->max_v_samp_factor
 * a1 = downsampled_width
 * a2 = input_data
 * a3 = output_data_ptr
 */
    SAVE_REGS_ON_STACK 24, s0, s1, s2, s3, s4, s5

    li            s4, 0
    lw            s2, 0(a3)       /* s2 = *output_data_ptr */
0:
    li            t9, 2
    lw            s1, -4(a2)      /* s1 = inptr1 */

1:
    lw            s0, 0(a2)       /* s0 = inptr0 */
    lwx           s3, s4(s2)
    addiu         s5, a1, -2      /* s5 = downsampled_width - 2 */
    srl           t4, s5, 1
    sll           t4, t4, 1
    lbu           t0, 0(s0)
    lbu           t1, 1(s0)
    lbu           t2, 0(s1)
    lbu           t3, 1(s1)
    addiu         s0, 2
    addiu         s1, 2
    addu          t8, s0, t4      /* t8 = end address */
    andi          s5, s5, 1       /* s5 = residual */
    sll           t4, t0, 1
    sll           t6, t1, 1
    addu          t0, t0, t4      /* t0 = (*inptr0++) * 3 */
    addu          t1, t1, t6      /* t1 = (*inptr0++) * 3 */
    addu          t7, t0, t2      /* t7 = thiscolsum */
    addu          t6, t1, t3      /* t5 = nextcolsum */
    sll           t0, t7, 2       /* t0 = thiscolsum * 4 */
    subu          t1, t0, t7      /* t1 = thiscolsum * 3 */
    shra_r.w      t0, t0, 4
    addiu         t1, 7
    addu          t1, t1, t6
    srl           t1, t1, 4
    sb            t0, 0(s3)
    sb            t1, 1(s3)
    beq           t8, s0, 22f     /* skip to final iteration if width == 3 */
     addiu        s3, 2
2:
    lh            t0, 0(s0)       /* t0 = A3|A2 */
    lh            t2, 0(s1)       /* t2 = B3|B2 */
    addiu         s0, 2
    addiu         s1, 2
    preceu.ph.qbr t0, t0          /* t0 = 0|A3|0|A2 */
    preceu.ph.qbr t2, t2          /* t2 = 0|B3|0|B2 */
    shll.ph       t1, t0, 1
    sll           t3, t6, 1
    addu.ph       t0, t1, t0      /* t0 = A3*3|A2*3 */
    addu          t3, t3, t6      /* t3 = this * 3 */
    addu.ph       t0, t0, t2      /* t0 = next2|next1 */
    addu          t1, t3, t7
    andi          t7, t0, 0xFFFF  /* t7 = next1 */
    sll           t2, t7, 1
    addu          t2, t7, t2      /* t2 = next1*3 */
    addu          t4, t2, t6
    srl           t6, t0, 16      /* t6 = next2 */
    shra_r.w      t1, t1, 4       /* t1 = (this*3 + last + 8) >> 4 */
    addu          t0, t3, t7
    addiu         t0, 7
    srl           t0, t0, 4       /* t0 = (this*3 + next1 + 7) >> 4 */
    shra_r.w      t4, t4, 4       /* t3 = (next1*3 + this + 8) >> 4 */
    addu          t2, t2, t6
    addiu         t2, 7
    srl           t2, t2, 4       /* t2 = (next1*3 + next2 + 7) >> 4 */
    sb            t1, 0(s3)
    sb            t0, 1(s3)
    sb            t4, 2(s3)
    sb            t2, 3(s3)
    bne           t8, s0, 2b
     addiu        s3, 4
22:
    beqz          s5, 4f
     addu         t8, s0, s5
3:
    lbu           t0, 0(s0)
    lbu           t2, 0(s1)
    addiu         s0, 1
    addiu         s1, 1
    sll           t3, t6, 1
    sll           t1, t0, 1
    addu          t1, t0, t1      /* t1 = inptr0 * 3 */
    addu          t3, t3, t6      /* t3 = thiscolsum * 3 */
    addu          t5, t1, t2
    addu          t1, t3, t7
    shra_r.w      t1, t1, 4
    addu          t0, t3, t5
    addiu         t0, 7
    srl           t0, t0, 4
    sb            t1, 0(s3)
    sb            t0, 1(s3)
    addiu         s3, 2
    move          t7, t6
    bne           t8, s0, 3b
     move         t6, t5
4:
    sll           t0, t6, 2       /* t0 = thiscolsum * 4 */
    subu          t1, t0, t6      /* t1 = thiscolsum * 3 */
    addu          t1, t1, t7
    addiu         s4, 4
    shra_r.w      t1, t1, 4
    addiu         t0, 7
    srl           t0, t0, 4
    sb            t1, 0(s3)
    sb            t0, 1(s3)
    addiu         t9, -1
    addiu         s3, 2
    bnez          t9, 1b
     lw           s1, 4(a2)
    srl           t0, s4, 2
    subu          t0, a0, t0
    bgtz          t0, 0b
     addiu        a2, 4

    RESTORE_REGS_FROM_STACK 24, s0, s1, s2, s3, s4, s5

    j             ra
     nop
END(jsimd_h2v2_fancy_upsample_dspr2)


/*****************************************************************************/
LEAF_DSPR2(jsimd_h2v1_fancy_upsample_dspr2)
/*
 * a0 = cinfo->max_v_samp_factor
 * a1 = downsampled_width
 * a2 = input_data
 * a3 = output_data_ptr
 */
    SAVE_REGS_ON_STACK 16, s0, s1, s2, s3

    .set at

    beqz          a0, 3f
     sll          t0, a0, 2
    lw            s1, 0(a3)
    li            s3, 0x10001
    addu          s0, s1, t0
0:
    addiu         t8, a1, -2
    srl           t9, t8, 2
    lw            t7, 0(a2)
    lw            s2, 0(s1)
    lbu           t0, 0(t7)
    lbu           t1, 1(t7)       /* t1 = inptr[1] */
    sll           t2, t0, 1
    addu          t2, t2, t0      /* t2 = invalue*3 */
    addu          t2, t2, t1
    shra_r.w      t2, t2, 2
    sb            t0, 0(s2)
    sb            t2, 1(s2)
    beqz          t9, 11f
     addiu        s2, 2
1:
    ulw           t0, 0(t7)       /* t0 = |P3|P2|P1|P0| */
    ulw           t1, 1(t7)
    ulh           t2, 4(t7)       /* t2 = |0|0|P5|P4| */
    preceu.ph.qbl t3, t0          /* t3 = |0|P3|0|P2| */
    preceu.ph.qbr t0, t0          /* t0 = |0|P1|0|P0| */
    preceu.ph.qbr t2, t2          /* t2 = |0|P5|0|P4| */
    preceu.ph.qbl t4, t1          /* t4 = |0|P4|0|P3| */
    preceu.ph.qbr t1, t1          /* t1 = |0|P2|0|P1| */
    shll.ph       t5, t4, 1
    shll.ph       t6, t1, 1
    addu.ph       t5, t5, t4      /* t5 = |P4*3|P3*3| */
    addu.ph       t6, t6, t1      /* t6 = |P2*3|P1*3| */
    addu.ph       t4, t3, s3
    addu.ph       t0, t0, s3
    addu.ph       t4, t4, t5
    addu.ph       t0, t0, t6
    shrl.ph       t4, t4, 2       /* t4 = |0|P3|0|P2| */
    shrl.ph       t0, t0, 2       /* t0 = |0|P1|0|P0| */
    addu.ph       t2, t2, t5
    addu.ph       t3, t3, t6
    shra_r.ph     t2, t2, 2       /* t2 = |0|P5|0|P4| */
    shra_r.ph     t3, t3, 2       /* t3 = |0|P3|0|P2| */
    shll.ph       t2, t2, 8
    shll.ph       t3, t3, 8
    or            t2, t4, t2
    or            t3, t3, t0
    addiu         t9, -1
    usw           t3, 0(s2)
    usw           t2, 4(s2)
    addiu         s2, 8
    bgtz          t9, 1b
     addiu        t7, 4
11:
    andi          t8, 3
    beqz          t8, 22f
     addiu        t7, 1

2:
    lbu           t0, 0(t7)
    addiu         t7, 1
    sll           t1, t0, 1
    addu          t2, t0, t1      /* t2 = invalue */
    lbu           t3, -2(t7)
    lbu           t4, 0(t7)
    addiu         t3, 1
    addiu         t4, 2
    addu          t3, t3, t2
    addu          t4, t4, t2
    srl           t3, 2
    srl           t4, 2
    sb            t3, 0(s2)
    sb            t4, 1(s2)
    addiu         t8, -1
    bgtz          t8, 2b
     addiu        s2, 2

22:
    lbu           t0, 0(t7)
    lbu           t2, -1(t7)
    sll           t1, t0, 1
    addu          t1, t1, t0      /* t1 = invalue * 3 */
    addu          t1, t1, t2
    addiu         t1, 1
    srl           t1, t1, 2
    sb            t1, 0(s2)
    sb            t0, 1(s2)
    addiu         s1, 4
    bne           s1, s0, 0b
     addiu        a2, 4
3:
    RESTORE_REGS_FROM_STACK 16, s0, s1, s2, s3

    j             ra
     nop
END(jsimd_h2v1_fancy_upsample_dspr2)


/*****************************************************************************/
LEAF_DSPR2(jsimd_h2v1_downsample_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = cinfo->max_v_samp_factor
 * a2     = compptr->v_samp_factor
 * a3     = compptr->width_in_blocks
 * 16(sp) = input_data
 * 20(sp) = output_data
 */
    .set at

    SAVE_REGS_ON_STACK 24, s0, s1, s2, s3, s4

    beqz        a2, 7f
     lw         s1, 44(sp)      /* s1 = output_data */
    lw          s0, 40(sp)      /* s0 = input_data */
    srl         s2, a0, 2
    andi        t9, a0, 2
    srl         t7, t9, 1
    addu        s2, t7, s2
    sll         t0, a3, 3       /* t0 = width_in_blocks*DCT */
    srl         t7, t0, 1
    subu        s2, t7, s2
0:
    andi        t6, a0, 1       /* t6 = temp_index */
    addiu       t6, -1
    lw          t4, 0(s1)       /* t4 = outptr */
    lw          t5, 0(s0)       /* t5 = inptr0 */
    li          s3, 0           /* s3 = bias */
    srl         t7, a0, 1       /* t7 = image_width1 */
    srl         s4, t7, 2
    andi        t8, t7, 3
1:
    ulhu        t0, 0(t5)
    ulhu        t1, 2(t5)
    ulhu        t2, 4(t5)
    ulhu        t3, 6(t5)
    raddu.w.qb  t0, t0
    raddu.w.qb  t1, t1
    raddu.w.qb  t2, t2
    raddu.w.qb  t3, t3
    shra.ph     t0, t0, 1
    shra_r.ph   t1, t1, 1
    shra.ph     t2, t2, 1
    shra_r.ph   t3, t3, 1
    sb          t0, 0(t4)
    sb          t1, 1(t4)
    sb          t2, 2(t4)
    sb          t3, 3(t4)
    addiu       s4, -1
    addiu       t4, 4
    bgtz        s4, 1b
     addiu      t5, 8
    beqz        t8, 3f
     addu       s4, t4, t8
2:
    ulhu        t0, 0(t5)
    raddu.w.qb  t0, t0
    addqh.w     t0, t0, s3
    xori        s3, s3, 1
    sb          t0, 0(t4)
    addiu       t4, 1
    bne         t4, s4, 2b
     addiu      t5, 2
3:
    lbux        t1, t6(t5)
    sll         t1, 1
    addqh.w     t2, t1, s3      /* t2 = pixval1 */
    xori        s3, s3, 1
    addqh.w     t3, t1, s3      /* t3 = pixval2 */
    blez        s2, 5f
     append     t3, t2,  8
    addu        t5, t4, s2      /* t5 = loop_end2 */
4:
    ush         t3, 0(t4)
    addiu       s2, -1
    bgtz        s2, 4b
     addiu      t4,  2
5:
    beqz        t9, 6f
     nop
    sb          t2, 0(t4)
6:
    addiu       s1, 4
    addiu       a2, -1
    bnez        a2, 0b
     addiu      s0, 4
7:
    RESTORE_REGS_FROM_STACK 24, s0, s1, s2, s3, s4

    j           ra
    nop
END(jsimd_h2v1_downsample_dspr2)


/*****************************************************************************/
LEAF_DSPR2(jsimd_h2v2_downsample_dspr2)
/*
 * a0     = cinfo->image_width
 * a1     = cinfo->max_v_samp_factor
 * a2     = compptr->v_samp_factor
 * a3     = compptr->width_in_blocks
 * 16(sp) = input_data
 * 20(sp) = output_data
 */
    .set at

    SAVE_REGS_ON_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    beqz        a2, 8f
     lw         s1, 52(sp)      /* s1 = output_data */
    lw          s0, 48(sp)      /* s0 = input_data */

    andi        t6, a0, 1       /* t6 = temp_index */
    addiu       t6, -1
    srl         t7, a0, 1       /* t7 = image_width1 */
    srl         s4, t7, 2
    andi        t8, t7, 3
    andi        t9, a0, 2
    srl         s2, a0, 2
    srl         t7, t9, 1
    addu        s2, t7, s2
    sll         t0, a3, 3       /* s2 = width_in_blocks*DCT */
    srl         t7, t0, 1
    subu        s2, t7, s2
0:
    lw          t4, 0(s1)       /* t4 = outptr */
    lw          t5, 0(s0)       /* t5 = inptr0 */
    lw          s7, 4(s0)       /* s7 = inptr1 */
    li          s6, 1           /* s6 = bias */
2:
    ulw         t0, 0(t5)       /* t0 = |P3|P2|P1|P0| */
    ulw         t1, 0(s7)       /* t1 = |Q3|Q2|Q1|Q0| */
    ulw         t2, 4(t5)
    ulw         t3, 4(s7)
    precrq.ph.w t7, t0, t1      /* t2 = |P3|P2|Q3|Q2| */
    ins         t0, t1, 16, 16  /* t0 = |Q1|Q0|P1|P0| */
    raddu.w.qb  t1, t7
    raddu.w.qb  t0, t0
    shra_r.w    t1, t1, 2
    addiu       t0, 1
    srl         t0, 2
    precrq.ph.w t7, t2, t3
    ins         t2, t3, 16, 16
    raddu.w.qb  t7, t7
    raddu.w.qb  t2, t2
    shra_r.w    t7, t7, 2
    addiu       t2, 1
    srl         t2, 2
    sb          t0, 0(t4)
    sb          t1, 1(t4)
    sb          t2, 2(t4)
    sb          t7, 3(t4)
    addiu       t4, 4
    addiu       t5, 8
    addiu       s4, s4, -1
    bgtz        s4, 2b
     addiu      s7, 8
    beqz        t8, 4f
     addu       t8, t4, t8
3:
    ulhu        t0, 0(t5)
    ulhu        t1, 0(s7)
    ins         t0, t1, 16, 16
    raddu.w.qb  t0, t0
    addu        t0, t0, s6
    srl         t0, 2
    xori        s6, s6, 3
    sb          t0, 0(t4)
    addiu       t5, 2
    addiu       t4, 1
    bne         t8, t4, 3b
     addiu      s7, 2
4:
    lbux        t1, t6(t5)
    sll         t1, 1
    lbux        t0, t6(s7)
    sll         t0, 1
    addu        t1, t1, t0
    addu        t3, t1, s6
    srl         t0, t3, 2       /* t2 = pixval1 */
    xori        s6, s6, 3
    addu        t2, t1, s6
    srl         t1, t2, 2       /* t3 = pixval2 */
    blez        s2, 6f
     append     t1, t0, 8
5:
    ush         t1, 0(t4)
    addiu       s2, -1
    bgtz        s2, 5b
     addiu      t4, 2
6:
    beqz        t9, 7f
     nop
    sb          t0, 0(t4)
7:
    addiu       s1, 4
    addiu       a2, -1
    bnez        a2, 0b
     addiu      s0, 8
8:
    RESTORE_REGS_FROM_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    j           ra
     nop
END(jsimd_h2v2_downsample_dspr2)


/*****************************************************************************/
LEAF_DSPR2(jsimd_h2v2_smooth_downsample_dspr2)
/*
 * a0     = input_data
 * a1     = output_data
 * a2     = compptr->v_samp_factor
 * a3     = cinfo->max_v_samp_factor
 * 16(sp) = cinfo->smoothing_factor
 * 20(sp) = compptr->width_in_blocks
 * 24(sp) = cinfo->image_width
 */
    .set at

    SAVE_REGS_ON_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    lw          s7, 52(sp)      /* compptr->width_in_blocks */
    lw          s0, 56(sp)      /* cinfo->image_width */
    lw          s6, 48(sp)      /* cinfo->smoothing_factor */
    sll         s7, 3           /* output_cols = width_in_blocks * DCTSIZE */
    sll         v0, s7, 1
    subu        v0, v0, s0
    blez        v0, 2f
    move        v1, zero
    addiu       t0, a3, 2       /* t0 = cinfo->max_v_samp_factor + 2 */
0:
    addiu       t1, a0, -4
    sll         t2, v1, 2
    lwx         t1, t2(t1)
    move        t3, v0
    addu        t1, t1, s0
    lbu         t2, -1(t1)
1:
    addiu       t3, t3, -1
    sb          t2, 0(t1)
    bgtz        t3, 1b
    addiu       t1, t1, 1
    addiu       v1, v1, 1
    bne         v1, t0, 0b
    nop
2:
    li          v0, 80
    mul         v0, s6, v0
    li          v1, 16384
    move        t4, zero
    move        t5, zero
    subu        t6, v1, v0      /* t6 = 16384 - tmp_smoot_f * 80 */
    sll         t7, s6, 4       /* t7 = tmp_smoot_f * 16 */
3:
/* Special case for first column: pretend column -1 is same as column 0 */
    sll         v0, t4, 2
    lwx         t8, v0(a1)      /*  outptr = output_data[outrow] */
    sll         v1, t5, 2
    addiu       t9, v1, 4
    addiu       s0, v1, -4
    addiu       s1, v1, 8
    lwx         s2, v1(a0)      /* inptr0 = input_data[inrow] */
    lwx         t9, t9(a0)      /* inptr1 = input_data[inrow+1] */
    lwx         s0, s0(a0)      /* above_ptr = input_data[inrow-1] */
    lwx         s1, s1(a0)      /* below_ptr = input_data[inrow+2] */
    lh          v0, 0(s2)
    lh          v1, 0(t9)
    lh          t0, 0(s0)
    lh          t1, 0(s1)
    ins         v0, v1, 16, 16
    ins         t0, t1, 16, 16
    raddu.w.qb  t2, v0
    raddu.w.qb  s3, t0
    lbu         v0, 0(s2)
    lbu         v1, 2(s2)
    lbu         t0, 0(t9)
    lbu         t1, 2(t9)
    addu        v0, v0, v1
    mult        $ac1, t2, t6
    addu        t0, t0, t1
    lbu         t2, 2(s0)
    addu        t0, t0, v0
    lbu         t3, 2(s1)
    addu        s3, t0, s3
    lbu         v0, 0(s0)
    lbu         t0, 0(s1)
    sll         s3, s3, 1
    addu        v0, v0, t2
    addu        t0, t0, t3
    addu        t0, t0, v0
    addu        s3, t0, s3
    madd        $ac1, s3, t7
    extr_r.w    v0, $ac1, 16
    addiu       t8, t8, 1
    addiu       s2, s2, 2
    addiu       t9, t9, 2
    addiu       s0, s0, 2
    addiu       s1, s1, 2
    sb          v0, -1(t8)
    addiu       s4, s7, -2
    and         s4, s4, 3
    addu        s5, s4, t8      /* end address */
4:
    lh          v0, 0(s2)
    lh          v1, 0(t9)
    lh          t0, 0(s0)
    lh          t1, 0(s1)
    ins         v0, v1, 16, 16
    ins         t0, t1, 16, 16
    raddu.w.qb  t2, v0
    raddu.w.qb  s3, t0
    lbu         v0, -1(s2)
    lbu         v1, 2(s2)
    lbu         t0, -1(t9)
    lbu         t1, 2(t9)
    addu        v0, v0, v1
    mult        $ac1, t2, t6
    addu        t0, t0, t1
    lbu         t2, 2(s0)
    addu        t0, t0, v0
    lbu         t3, 2(s1)
    addu        s3, t0, s3
    lbu         v0, -1(s0)
    lbu         t0, -1(s1)
    sll         s3, s3, 1
    addu        v0, v0, t2
    addu        t0, t0, t3
    addu        t0, t0, v0
    addu        s3, t0, s3
    madd        $ac1, s3, t7
    extr_r.w    t2, $ac1, 16
    addiu       t8, t8, 1
    addiu       s2, s2, 2
    addiu       t9, t9, 2
    addiu       s0, s0, 2
    sb          t2, -1(t8)
    bne         s5, t8, 4b
    addiu       s1, s1, 2
    addiu       s5, s7, -2
    subu        s5, s5, s4
    addu        s5, s5, t8      /* end address */
5:
    lh          v0, 0(s2)
    lh          v1, 0(t9)
    lh          t0, 0(s0)
    lh          t1, 0(s1)
    ins         v0, v1, 16, 16
    ins         t0, t1, 16, 16
    raddu.w.qb  t2, v0
    raddu.w.qb  s3, t0
    lbu         v0, -1(s2)
    lbu         v1, 2(s2)
    lbu         t0, -1(t9)
    lbu         t1, 2(t9)
    addu        v0, v0, v1
    mult        $ac1, t2, t6
    addu        t0, t0, t1
    lbu         t2, 2(s0)
    addu        t0, t0, v0
    lbu         t3, 2(s1)
    addu        s3, t0, s3
    lbu         v0, -1(s0)
    lbu         t0, -1(s1)
    sll         s3, s3, 1
    addu        v0, v0, t2
    addu        t0, t0, t3
    lh          v1, 2(t9)
    addu        t0, t0, v0
    lh          v0, 2(s2)
    addu        s3, t0, s3
    lh          t0, 2(s0)
    lh          t1, 2(s1)
    madd        $ac1, s3, t7
    extr_r.w    t2, $ac1, 16
    ins         t0, t1, 16, 16
    ins         v0, v1, 16, 16
    raddu.w.qb  s3, t0
    lbu         v1, 4(s2)
    lbu         t0, 1(t9)
    lbu         t1, 4(t9)
    sb          t2, 0(t8)
    raddu.w.qb  t3, v0
    lbu         v0, 1(s2)
    addu        t0, t0, t1
    mult        $ac1, t3, t6
    addu        v0, v0, v1
    lbu         t2, 4(s0)
    addu        t0, t0, v0
    lbu         v0, 1(s0)
    addu        s3, t0, s3
    lbu         t0, 1(s1)
    lbu         t3, 4(s1)
    addu        v0, v0, t2
    sll         s3, s3, 1
    addu        t0, t0, t3
    lh          v1, 4(t9)
    addu        t0, t0, v0
    lh          v0, 4(s2)
    addu        s3, t0, s3
    lh          t0, 4(s0)
    lh          t1, 4(s1)
    madd        $ac1, s3, t7
    extr_r.w    t2, $ac1, 16
    ins         t0, t1, 16, 16
    ins         v0, v1, 16, 16
    raddu.w.qb  s3, t0
    lbu         v1, 6(s2)
    lbu         t0, 3(t9)
    lbu         t1, 6(t9)
    sb          t2, 1(t8)
    raddu.w.qb  t3, v0
    lbu         v0, 3(s2)
    addu        t0, t0, t1
    mult        $ac1, t3, t6
    addu        v0, v0, v1
    lbu         t2, 6(s0)
    addu        t0, t0, v0
    lbu         v0, 3(s0)
    addu        s3, t0, s3
    lbu         t0, 3(s1)
    lbu         t3, 6(s1)
    addu        v0, v0, t2
    sll         s3, s3, 1
    addu        t0, t0, t3
    lh          v1, 6(t9)
    addu        t0, t0, v0
    lh          v0, 6(s2)
    addu        s3, t0, s3
    lh          t0, 6(s0)
    lh          t1, 6(s1)
    madd        $ac1, s3, t7
    extr_r.w    t3, $ac1, 16
    ins         t0, t1, 16, 16
    ins         v0, v1, 16, 16
    raddu.w.qb  s3, t0
    lbu         v1, 8(s2)
    lbu         t0, 5(t9)
    lbu         t1, 8(t9)
    sb          t3, 2(t8)
    raddu.w.qb  t2, v0
    lbu         v0, 5(s2)
    addu        t0, t0, t1
    mult        $ac1, t2, t6
    addu        v0, v0, v1
    lbu         t2, 8(s0)
    addu        t0, t0, v0
    lbu         v0, 5(s0)
    addu        s3, t0, s3
    lbu         t0, 5(s1)
    lbu         t3, 8(s1)
    addu        v0, v0, t2
    sll         s3, s3, 1
    addu        t0, t0, t3
    addiu       t8, t8, 4
    addu        t0, t0, v0
    addiu       s2, s2, 8
    addu        s3, t0, s3
    addiu       t9, t9, 8
    madd        $ac1, s3, t7
    extr_r.w    t1, $ac1, 16
    addiu       s0, s0, 8
    addiu       s1, s1, 8
    bne         s5, t8, 5b
    sb          t1, -1(t8)
/* Special case for last column */
    lh          v0, 0(s2)
    lh          v1, 0(t9)
    lh          t0, 0(s0)
    lh          t1, 0(s1)
    ins         v0, v1, 16, 16
    ins         t0, t1, 16, 16
    raddu.w.qb  t2, v0
    raddu.w.qb  s3, t0
    lbu         v0, -1(s2)
    lbu         v1, 1(s2)
    lbu         t0, -1(t9)
    lbu         t1, 1(t9)
    addu        v0, v0, v1
    mult        $ac1, t2, t6
    addu        t0, t0, t1
    lbu         t2, 1(s0)
    addu        t0, t0, v0
    lbu         t3, 1(s1)
    addu        s3, t0, s3
    lbu         v0, -1(s0)
    lbu         t0, -1(s1)
    sll         s3, s3, 1
    addu        v0, v0, t2
    addu        t0, t0, t3
    addu        t0, t0, v0
    addu        s3, t0, s3
    madd        $ac1, s3, t7
    extr_r.w    t0, $ac1, 16
    addiu       t5, t5, 2
    sb          t0, 0(t8)
    addiu       t4, t4, 1
    bne         t4, a2, 3b
    addiu       t5, t5, 2

    RESTORE_REGS_FROM_STACK 32, s0, s1, s2, s3, s4, s5, s6, s7

    j           ra
     nop

END(jsimd_h2v2_smooth_downsample_dspr2)


/*****************************************************************************/
LEAF_DSPR2(jsimd_int_upsample_dspr2)
/*
 * a0     = upsample->h_expand[compptr->component_index]
 * a1     = upsample->v_expand[compptr->component_index]
 * a2     = input_data
 * a3     = output_data_ptr
 * 16(sp) = cinfo->output_width
 * 20(sp) = cinfo->max_v_samp_factor
 */
    .set at

    SAVE_REGS_ON_STACK 16, s0, s1, s2, s3

    lw          s0, 0(a3)       /* s0 = output_data */
    lw          s1, 32(sp)      /* s1 = cinfo->output_width */
    lw          s2, 36(sp)      /* s2 = cinfo->max_v_samp_factor */
    li          t6, 0           /* t6 = inrow */
    beqz        s2, 10f
     li         s3, 0           /* s3 = outrow */
0:
    addu        t0, a2, t6
    addu        t7, s0, s3
    lw          t3, 0(t0)       /* t3 = inptr */
    lw          t8, 0(t7)       /* t8 = outptr */
    beqz        s1, 4f
     addu       t5, t8, s1      /* t5 = outend */
1:
    lb          t2, 0(t3)       /* t2 = invalue = *inptr++ */
    addiu       t3, 1
    beqz        a0, 3f
     move       t0, a0          /* t0 = h_expand */
2:
    sb          t2, 0(t8)
    addiu       t0, -1
    bgtz        t0, 2b
     addiu      t8, 1
3:
    bgt         t5, t8, 1b
     nop
4:
    addiu       t9, a1, -1      /* t9 = v_expand - 1 */
    blez        t9, 9f
     nop
5:
    lw          t3, 0(s0)
    lw          t4, 4(s0)
    subu        t0, s1, 0xF
    blez        t0, 7f
     addu       t5, t3, s1      /* t5 = end address */
    andi        t7, s1, 0xF     /* t7 = residual */
    subu        t8, t5, t7
6:
    ulw         t0, 0(t3)
    ulw         t1, 4(t3)
    ulw         t2, 8(t3)
    usw         t0, 0(t4)
    ulw         t0, 12(t3)
    usw         t1, 4(t4)
    usw         t2, 8(t4)
    usw         t0, 12(t4)
    addiu       t3, 16
    bne         t3, t8, 6b
     addiu      t4, 16
    beqz        t7, 8f
     nop
7:
    lbu         t0, 0(t3)
    sb          t0, 0(t4)
    addiu       t3, 1
    bne         t3, t5, 7b
     addiu      t4, 1
8:
    addiu       t9, -1
    bgtz        t9, 5b
   
... [truncated]
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

