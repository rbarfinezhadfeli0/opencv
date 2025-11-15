# Documentation for `3rdparty/libpng/powerpc/filter_vsx_intrinsics.c`

## File Metadata

- **Full Path**: `3rdparty/libpng/powerpc/filter_vsx_intrinsics.c`
- **File Name**: `filter_vsx_intrinsics.c`
- **File Size**: 25,418 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libpng/powerpc/filter_vsx_intrinsics.c](../../../3rdparty/libpng/powerpc/filter_vsx_intrinsics.c)

## Purpose and Role

This file is located in the `3rdparty/libpng/powerpc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* filter_vsx_intrinsics.c - PowerPC optimised filter functions
 *
 * Copyright (c) 2018 Cosmin Truta
 * Copyright (c) 2017 Glenn Randers-Pehrson
 * Written by Vadim Barkov, 2017.
 *
 * This code is released under the libpng license.
 * For conditions of distribution and use, see the disclaimer
 * and license in png.h
 */

#include <stdio.h>
#include <stdint.h>
#include "../pngpriv.h"

#ifdef PNG_READ_SUPPORTED

/* This code requires -maltivec and -mvsx on the command line: */
#if PNG_POWERPC_VSX_IMPLEMENTATION == 1 /* intrinsics code from pngpriv.h */

#include <altivec.h>

#if PNG_POWERPC_VSX_OPT > 0

#ifndef __VSX__
#  error "This code requires VSX support (POWER7 and later). Please provide -mvsx compiler flag."
#endif

#define vec_ld_unaligned(vec,data) vec = vec_vsx_ld(0,data)
#define vec_st_unaligned(vec,data) vec_vsx_st(vec,0,data)


/* Functions in this file look at most 3 pixels (a,b,c) to predict the 4th (d).
 * They're positioned like this:
 *    prev:  c b
 *    row:   a d
 * The Sub filter predicts d=a, Avg d=(a+b)/2, and Paeth predicts d to be
 * whichever of a, b, or c is closest to p=a+b-c.
 * ( this is taken from ../intel/filter_sse2_intrinsics.c )
 */

#define vsx_declare_common_vars(row_info,row,prev_row,offset) \
   png_byte i;\
   png_bytep rp = row + offset;\
   png_const_bytep pp = prev_row;\
   size_t unaligned_top = 16 - (((size_t)rp % 16));\
   size_t istop;\
   if(unaligned_top == 16)\
      unaligned_top = 0;\
   istop = row_info->rowbytes;\
   if((unaligned_top < istop))\
      istop -= unaligned_top;\
   else{\
      unaligned_top = istop;\
      istop = 0;\
   }

void png_read_filter_row_up_vsx(png_row_infop row_info, png_bytep row,
                                png_const_bytep prev_row)
{
   vector unsigned char rp_vec;
   vector unsigned char pp_vec;
   vsx_declare_common_vars(row_info,row,prev_row,0)

   /* Altivec operations require 16-byte aligned data
    * but input can be unaligned. So we calculate
    * unaligned part as usual.
    */
   for (i = 0; i < unaligned_top; i++)
   {
      *rp = (png_byte)(((int)(*rp) + (int)(*pp++)) & 0xff);
      rp++;
   }

   /* Using SIMD while we can */
   while( istop >= 16 )
   {
      rp_vec = vec_ld(0,rp);
      vec_ld_unaligned(pp_vec,pp);

      rp_vec = vec_add(rp_vec,pp_vec);

      vec_st(rp_vec,0,rp);

      pp += 16;
      rp += 16;
      istop -= 16;
   }

   if(istop > 0)
   {
      /* If byte count of row is not divisible by 16
       * we will process remaining part as usual
       */
      for (i = 0; i < istop; i++)
      {
         *rp = (png_byte)(((int)(*rp) + (int)(*pp++)) & 0xff);
         rp++;
      }
}

}

static const vector unsigned char VSX_LEFTSHIFTED1_4 = {16,16,16,16, 0, 1, 2, 3,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_LEFTSHIFTED2_4 = {16,16,16,16,16,16,16,16, 4, 5, 6, 7,16,16,16,16};
static const vector unsigned char VSX_LEFTSHIFTED3_4 = {16,16,16,16,16,16,16,16,16,16,16,16, 8, 9,10,11};

static const vector unsigned char VSX_LEFTSHIFTED1_3 = {16,16,16, 0, 1, 2,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_LEFTSHIFTED2_3 = {16,16,16,16,16,16, 3, 4, 5,16,16,16,16,16,16,16};
static const vector unsigned char VSX_LEFTSHIFTED3_3 = {16,16,16,16,16,16,16,16,16, 6, 7, 8,16,16,16,16};
static const vector unsigned char VSX_LEFTSHIFTED4_3 = {16,16,16,16,16,16,16,16,16,16,16,16, 9,10,11,16};

static const vector unsigned char VSX_NOT_SHIFTED1_4 = {16,16,16,16, 4, 5, 6, 7,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_NOT_SHIFTED2_4 = {16,16,16,16,16,16,16,16, 8, 9,10,11,16,16,16,16};
static const vector unsigned char VSX_NOT_SHIFTED3_4 = {16,16,16,16,16,16,16,16,16,16,16,16,12,13,14,15};

static const vector unsigned char VSX_NOT_SHIFTED1_3 = {16,16,16, 3, 4, 5,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_NOT_SHIFTED2_3 = {16,16,16,16,16,16, 6, 7, 8,16,16,16,16,16,16,16};
static const vector unsigned char VSX_NOT_SHIFTED3_3 = {16,16,16,16,16,16,16,16,16, 9,10,11,16,16,16,16};
static const vector unsigned char VSX_NOT_SHIFTED4_3 = {16,16,16,16,16,16,16,16,16,16,16,16,12,13,14,16};

static const vector unsigned char VSX_CHAR_ZERO = {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};
#ifdef __LITTLE_ENDIAN__

static const vector unsigned char VSX_CHAR_TO_SHORT1_4 = { 4,16, 5,16, 6,16, 7,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT2_4 = { 8,16, 9,16,10,16,11,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT3_4 = {12,16,13,16,14,16,15,16,16,16,16,16,16,16,16,16};

static const vector unsigned char VSX_SHORT_TO_CHAR1_4 = {16,16,16,16, 0, 2, 4, 6,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR2_4 = {16,16,16,16,16,16,16,16, 0, 2, 4, 6,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR3_4 = {16,16,16,16,16,16,16,16,16,16,16,16, 0, 2, 4, 6};

static const vector unsigned char VSX_CHAR_TO_SHORT1_3 = { 3,16, 4,16, 5,16,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT2_3 = { 6,16, 7,16, 8,16,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT3_3 = { 9,16,10,16,11,16,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT4_3 = {12,16,13,16,14,16,16,16,16,16,16,16,16,16,16,16};

static const vector unsigned char VSX_SHORT_TO_CHAR1_3 = {16,16,16, 0, 2, 4,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR2_3 = {16,16,16,16,16,16, 0, 2, 4,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR3_3 = {16,16,16,16,16,16,16,16,16, 0, 2, 4,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR4_3 = {16,16,16,16,16,16,16,16,16,16,16,16, 0, 2, 4,16};

#elif defined(__BIG_ENDIAN__)

static const vector unsigned char VSX_CHAR_TO_SHORT1_4 = {16, 4,16, 5,16, 6,16, 7,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT2_4 = {16, 8,16, 9,16,10,16,11,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT3_4 = {16,12,16,13,16,14,16,15,16,16,16,16,16,16,16,16};

static const vector unsigned char VSX_SHORT_TO_CHAR1_4 = {16,16,16,16, 1, 3, 5, 7,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR2_4 = {16,16,16,16,16,16,16,16, 1, 3, 5, 7,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR3_4 = {16,16,16,16,16,16,16,16,16,16,16,16, 1, 3, 5, 7};

static const vector unsigned char VSX_CHAR_TO_SHORT1_3 = {16, 3,16, 4,16, 5,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT2_3 = {16, 6,16, 7,16, 8,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT3_3 = {16, 9,16,10,16,11,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_CHAR_TO_SHORT4_3 = {16,12,16,13,16,14,16,16,16,16,16,16,16,16,16,16};

static const vector unsigned char VSX_SHORT_TO_CHAR1_3 = {16,16,16, 1, 3, 5,16,16,16,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR2_3 = {16,16,16,16,16,16, 1, 3, 5,16,16,16,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR3_3 = {16,16,16,16,16,16,16,16,16, 1, 3, 5,16,16,16,16};
static const vector unsigned char VSX_SHORT_TO_CHAR4_3 = {16,16,16,16,16,16,16,16,16,16,16,16, 1, 3, 5,16};

#endif

#define vsx_char_to_short(vec,offset,bpp) (vector unsigned short)vec_perm((vec),VSX_CHAR_ZERO,VSX_CHAR_TO_SHORT##offset##_##bpp)
#define vsx_short_to_char(vec,offset,bpp) vec_perm(((vector unsigned char)(vec)),VSX_CHAR_ZERO,VSX_SHORT_TO_CHAR##offset##_##bpp)

#ifdef PNG_USE_ABS
#  define vsx_abs(number) abs(number)
#else
#  define vsx_abs(number) (number > 0) ? (number) : -(number)
#endif

void png_read_filter_row_sub4_vsx(png_row_infop row_info, png_bytep row,
                                  png_const_bytep prev_row)
{
   png_byte bpp = 4;

   vector unsigned char rp_vec;
   vector unsigned char part_vec;

   vsx_declare_common_vars(row_info,row,prev_row,bpp)

   PNG_UNUSED(pp)

   /* Altivec operations require 16-byte aligned data
    * but input can be unaligned. So we calculate
    * unaligned part as usual.
    */
   for (i = 0; i < unaligned_top; i++)
   {
      *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
      rp++;
   }

   /* Using SIMD while we can */
   while( istop >= 16 )
   {
      for(i=0;i < bpp ; i++)
      {
         *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
         rp++;
      }
      rp -= bpp;

      rp_vec = vec_ld(0,rp);
      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED1_4);
      rp_vec = vec_add(rp_vec,part_vec);

      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED2_4);
      rp_vec = vec_add(rp_vec,part_vec);

      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED3_4);
      rp_vec = vec_add(rp_vec,part_vec);

      vec_st(rp_vec,0,rp);

      rp += 16;
      istop -= 16;
   }

   if(istop > 0)
      for (i = 0; i < istop % 16; i++)
      {
         *rp = (png_byte)(((int)(*rp) + (int)(*(rp - bpp))) & 0xff);
         rp++;
      }

}

void png_read_filter_row_sub3_vsx(png_row_infop row_info, png_bytep row,
                                  png_const_bytep prev_row)
{
   png_byte bpp = 3;

   vector unsigned char rp_vec;
   vector unsigned char part_vec;

   vsx_declare_common_vars(row_info,row,prev_row,bpp)

   PNG_UNUSED(pp)

   /* Altivec operations require 16-byte aligned data
    * but input can be unaligned. So we calculate
    * unaligned part as usual.
    */
   for (i = 0; i < unaligned_top; i++)
   {
      *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
      rp++;
   }

   /* Using SIMD while we can */
   while( istop >= 16 )
   {
      for(i=0;i < bpp ; i++)
      {
         *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
         rp++;
      }
      rp -= bpp;

      rp_vec = vec_ld(0,rp);
      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED1_3);
      rp_vec = vec_add(rp_vec,part_vec);

      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED2_3);
      rp_vec = vec_add(rp_vec,part_vec);

      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED3_3);
      rp_vec = vec_add(rp_vec,part_vec);

      part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED4_3);
      rp_vec = vec_add(rp_vec,part_vec);

      vec_st(rp_vec,0,rp);
      rp += 15;
      istop -= 16;

      /* Since 16 % bpp = 16 % 3 = 1, last element of array must
       * be proceeded manually
       */
      *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
      rp++;
   }

   if(istop > 0)
      for (i = 0; i < istop % 16; i++)
      {
         *rp = (png_byte)(((int)(*rp) + (int)(*(rp-bpp))) & 0xff);
         rp++;
      }
}

void png_read_filter_row_avg4_vsx(png_row_infop row_info, png_bytep row,
                                  png_const_bytep prev_row)
{
   png_byte bpp = 4;

   vector unsigned char rp_vec;
   vector unsigned char pp_vec;
   vector unsigned char pp_part_vec;
   vector unsigned char rp_part_vec;
   vector unsigned char avg_vec;

   vsx_declare_common_vars(row_info,row,prev_row,bpp)
   rp -= bpp;
   if(istop >= bpp)
      istop -= bpp;

   for (i = 0; i < bpp; i++)
   {
      *rp = (png_byte)(((int)(*rp) +
         ((int)(*pp++) / 2 )) & 0xff);

      rp++;
   }

   /* Altivec operations require 16-byte aligned data
    * but input can be unaligned. So we calculate
    * unaligned part as usual.
    */
   for (i = 0; i < unaligned_top; i++)
   {
      *rp = (png_byte)(((int)(*rp) +
         (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

      rp++;
   }

   /* Using SIMD while we can */
   while( istop >= 16 )
   {
      for(i=0;i < bpp ; i++)
      {
         *rp = (png_byte)(((int)(*rp) +
            (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

         rp++;
      }
      rp -= bpp;
      pp -= bpp;

      vec_ld_unaligned(pp_vec,pp);
      rp_vec = vec_ld(0,rp);

      rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED1_4);
      pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED1_4);
      avg_vec = vec_avg(rp_part_vec,pp_part_vec);
      avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
      rp_vec = vec_add(rp_vec,avg_vec);

      rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED2_4);
      pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED2_4);
      avg_vec = vec_avg(rp_part_vec,pp_part_vec);
      avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
      rp_vec = vec_add(rp_vec,avg_vec);

      rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED3_4);
      pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED3_4);
      avg_vec = vec_avg(rp_part_vec,pp_part_vec);
      avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
      rp_vec = vec_add(rp_vec,avg_vec);

      vec_st(rp_vec,0,rp);

      rp += 16;
      pp += 16;
      istop -= 16;
   }

   if(istop  > 0)
      for (i = 0; i < istop % 16; i++)
      {
         *rp = (png_byte)(((int)(*rp) +
            (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

         rp++;
      }
}

void png_read_filter_row_avg3_vsx(png_row_infop row_info, png_bytep row,
                                  png_const_bytep prev_row)
{
  png_byte bpp = 3;

  vector unsigned char rp_vec;
  vector unsigned char pp_vec;
  vector unsigned char pp_part_vec;
  vector unsigned char rp_part_vec;
  vector unsigned char avg_vec;

  vsx_declare_common_vars(row_info,row,prev_row,bpp)
  rp -= bpp;
  if(istop >= bpp)
     istop -= bpp;

  for (i = 0; i < bpp; i++)
  {
     *rp = (png_byte)(((int)(*rp) +
        ((int)(*pp++) / 2 )) & 0xff);

     rp++;
  }

  /* Altivec operations require 16-byte aligned data
   * but input can be unaligned. So we calculate
   * unaligned part as usual.
   */
  for (i = 0; i < unaligned_top; i++)
  {
     *rp = (png_byte)(((int)(*rp) +
        (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

     rp++;
  }

  /* Using SIMD while we can */
  while( istop >= 16 )
  {
     for(i=0;i < bpp ; i++)
     {
        *rp = (png_byte)(((int)(*rp) +
           (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

        rp++;
     }
     rp -= bpp;
     pp -= bpp;

     vec_ld_unaligned(pp_vec,pp);
     rp_vec = vec_ld(0,rp);

     rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED1_3);
     pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED1_3);
     avg_vec = vec_avg(rp_part_vec,pp_part_vec);
     avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
     rp_vec = vec_add(rp_vec,avg_vec);

     rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED2_3);
     pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED2_3);
     avg_vec = vec_avg(rp_part_vec,pp_part_vec);
     avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
     rp_vec = vec_add(rp_vec,avg_vec);

     rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED3_3);
     pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED3_3);
     avg_vec = vec_avg(rp_part_vec,pp_part_vec);
     avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
     rp_vec = vec_add(rp_vec,avg_vec);

     rp_part_vec = vec_perm(rp_vec,VSX_CHAR_ZERO,VSX_LEFTSHIFTED4_3);
     pp_part_vec = vec_perm(pp_vec,VSX_CHAR_ZERO,VSX_NOT_SHIFTED4_3);
     avg_vec = vec_avg(rp_part_vec,pp_part_vec);
     avg_vec = vec_sub(avg_vec, vec_and(vec_xor(rp_part_vec,pp_part_vec),vec_splat_u8(1)));
     rp_vec = vec_add(rp_vec,avg_vec);

     vec_st(rp_vec,0,rp);

     rp += 15;
     pp += 15;
     istop -= 16;

     /* Since 16 % bpp = 16 % 3 = 1, last element of array must
      * be proceeded manually
      */
     *rp = (png_byte)(((int)(*rp) +
        (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);
     rp++;
  }

  if(istop  > 0)
     for (i = 0; i < istop % 16; i++)
     {
        *rp = (png_byte)(((int)(*rp) +
           (int)(*pp++ + *(rp-bpp)) / 2 ) & 0xff);

        rp++;
     }
}

/* Bytewise c ? t : e. */
#define if_then_else(c,t,e) vec_sel(e,t,c)

#define vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp) {\
      c = *(pp - bpp);\
      a = *(rp - bpp);\
      b = *pp++;\
      p = b - c;\
      pc = a - c;\
      pa = vsx_abs(p);\
      pb = vsx_abs(pc);\
      pc = vsx_abs(p + pc);\
      if (pb < pa) pa = pb, a = b;\
      if (pc < pa) a = c;\
      a += *rp;\
      *rp++ = (png_byte)a;\
      }

void png_read_filter_row_paeth4_vsx(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row)
{
   png_byte bpp = 4;

   int a, b, c, pa, pb, pc, p;
   vector unsigned char rp_vec;
   vector unsigned char pp_vec;
   vector unsigned short a_vec,b_vec,c_vec,nearest_vec;
   vector signed short pa_vec,pb_vec,pc_vec,smallest_vec;

   vsx_declare_common_vars(row_info,row,prev_row,bpp)
   rp -= bpp;
   if(istop >= bpp)
      istop -= bpp;

   /* Process the first pixel in the row completely (this is the same as 'up'
    * because there is only one candidate predictor for the first row).
    */
   for(i = 0; i < bpp ; i++)
   {
      *rp = (png_byte)( *rp + *pp);
      rp++;
      pp++;
   }

   for(i = 0; i < unaligned_top ; i++)
   {
      vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
   }

   while( istop >= 16)
   {
      for(i = 0; i < bpp ; i++)
      {
         vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
      }

      rp -= bpp;
      pp -= bpp;
      rp_vec = vec_ld(0,rp);
      vec_ld_unaligned(pp_vec,pp);

      a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED1_4),1,4);
      b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED1_4),1,4);
      c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED1_4),1,4);
      pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
      pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
      pc_vec = vec_add(pa_vec,pb_vec);
      pa_vec = vec_abs(pa_vec);
      pb_vec = vec_abs(pb_vec);
      pc_vec = vec_abs(pc_vec);
      smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
      nearest_vec =  if_then_else(
            vec_cmpeq(pa_vec,smallest_vec),
            a_vec,
            if_then_else(
              vec_cmpeq(pb_vec,smallest_vec),
              b_vec,
              c_vec
              )
            );
      rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,1,4)));

      a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED2_4),2,4);
      b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED2_4),2,4);
      c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED2_4),2,4);
      pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
      pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
      pc_vec = vec_add(pa_vec,pb_vec);
      pa_vec = vec_abs(pa_vec);
      pb_vec = vec_abs(pb_vec);
      pc_vec = vec_abs(pc_vec);
      smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
      nearest_vec =  if_then_else(
            vec_cmpeq(pa_vec,smallest_vec),
            a_vec,
            if_then_else(
              vec_cmpeq(pb_vec,smallest_vec),
              b_vec,
              c_vec
              )
            );
      rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,2,4)));

      a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED3_4),3,4);
      b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED3_4),3,4);
      c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED3_4),3,4);
      pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
      pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
      pc_vec = vec_add(pa_vec,pb_vec);
      pa_vec = vec_abs(pa_vec);
      pb_vec = vec_abs(pb_vec);
      pc_vec = vec_abs(pc_vec);
      smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
      nearest_vec =  if_then_else(
            vec_cmpeq(pa_vec,smallest_vec),
            a_vec,
            if_then_else(
              vec_cmpeq(pb_vec,smallest_vec),
              b_vec,
              c_vec
              )
            );
      rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,3,4)));

      vec_st(rp_vec,0,rp);

      rp += 16;
      pp += 16;
      istop -= 16;
   }

   if(istop > 0)
      for (i = 0; i < istop % 16; i++)
      {
         vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
      }
}

void png_read_filter_row_paeth3_vsx(png_row_infop row_info, png_bytep row,
   png_const_bytep prev_row)
{
  png_byte bpp = 3;

  int a, b, c, pa, pb, pc, p;
  vector unsigned char rp_vec;
  vector unsigned char pp_vec;
  vector unsigned short a_vec,b_vec,c_vec,nearest_vec;
  vector signed short pa_vec,pb_vec,pc_vec,smallest_vec;

  vsx_declare_common_vars(row_info,row,prev_row,bpp)
  rp -= bpp;
  if(istop >= bpp)
     istop -= bpp;

  /* Process the first pixel in the row completely (this is the same as 'up'
   * because there is only one candidate predictor for the first row).
   */
  for(i = 0; i < bpp ; i++)
  {
     *rp = (png_byte)( *rp + *pp);
     rp++;
     pp++;
  }

  for(i = 0; i < unaligned_top ; i++)
  {
     vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
  }

  while( istop >= 16)
  {
     for(i = 0; i < bpp ; i++)
     {
        vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
     }

     rp -= bpp;
     pp -= bpp;
     rp_vec = vec_ld(0,rp);
     vec_ld_unaligned(pp_vec,pp);

     a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED1_3),1,3);
     b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED1_3),1,3);
     c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED1_3),1,3);
     pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
     pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
     pc_vec = vec_add(pa_vec,pb_vec);
     pa_vec = vec_abs(pa_vec);
     pb_vec = vec_abs(pb_vec);
     pc_vec = vec_abs(pc_vec);
     smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
     nearest_vec =  if_then_else(
           vec_cmpeq(pa_vec,smallest_vec),
           a_vec,
           if_then_else(
             vec_cmpeq(pb_vec,smallest_vec),
             b_vec,
             c_vec
             )
           );
     rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,1,3)));

     a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED2_3),2,3);
     b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED2_3),2,3);
     c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED2_3),2,3);
     pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
     pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
     pc_vec = vec_add(pa_vec,pb_vec);
     pa_vec = vec_abs(pa_vec);
     pb_vec = vec_abs(pb_vec);
     pc_vec = vec_abs(pc_vec);
     smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
     nearest_vec =  if_then_else(
           vec_cmpeq(pa_vec,smallest_vec),
           a_vec,
           if_then_else(
             vec_cmpeq(pb_vec,smallest_vec),
             b_vec,
             c_vec
             )
           );
     rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,2,3)));

     a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED3_3),3,3);
     b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED3_3),3,3);
     c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED3_3),3,3);
     pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
     pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
     pc_vec = vec_add(pa_vec,pb_vec);
     pa_vec = vec_abs(pa_vec);
     pb_vec = vec_abs(pb_vec);
     pc_vec = vec_abs(pc_vec);
     smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
     nearest_vec =  if_then_else(
           vec_cmpeq(pa_vec,smallest_vec),
           a_vec,
           if_then_else(
             vec_cmpeq(pb_vec,smallest_vec),
             b_vec,
             c_vec
             )
           );
     rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,3,3)));

     a_vec = vsx_char_to_short(vec_perm(rp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED4_3),4,3);
     b_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_NOT_SHIFTED4_3),4,3);
     c_vec = vsx_char_to_short(vec_perm(pp_vec , VSX_CHAR_ZERO , VSX_LEFTSHIFTED4_3),4,3);
     pa_vec = (vector signed short) vec_sub(b_vec,c_vec);
     pb_vec = (vector signed short) vec_sub(a_vec , c_vec);
     pc_vec = vec_add(pa_vec,pb_vec);
     pa_vec = vec_abs(pa_vec);
     pb_vec = vec_abs(pb_vec);
     pc_vec = vec_abs(pc_vec);
     smallest_vec = vec_min(pc_vec, vec_min(pa_vec,pb_vec));
     nearest_vec =  if_then_else(
           vec_cmpeq(pa_vec,smallest_vec),
           a_vec,
           if_then_else(
             vec_cmpeq(pb_vec,smallest_vec),
             b_vec,
             c_vec
             )
           );
     rp_vec = vec_add(rp_vec,(vsx_short_to_char(nearest_vec,4,3)));

     vec_st(rp_vec,0,rp);

     rp += 15;
     pp += 15;
     istop -= 16;

     /* Since 16 % bpp = 16 % 3 = 1, last element of array must
      * be proceeded manually
      */
     vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
  }

  if(istop > 0)
     for (i = 0; i < istop % 16; i++)
     {
        vsx_paeth_process(rp,pp,a,b,c,pa,pb,pc,bpp)
     }
}

#endif /* PNG_POWERPC_VSX_OPT > 0 */
#endif /* PNG_POWERPC_VSX_IMPLEMENTATION == 1 (intrinsics) */
#endif /* READ */
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

- **__VSX__()**: A function/method defined in this file
- **PNG_READ_SUPPORTED()**: A function/method defined in this file
- **__LITTLE_ENDIAN__()**: A function/method defined in this file
- **PNG_USE_ABS()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../pngpriv.h`
- `stdint.h`
- `stdio.h`
- `altivec.h`

**Python Imports:**
- `pngpriv.h`
- `..`


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

