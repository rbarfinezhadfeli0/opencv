# Documentation for `modules/imgproc/src/resize.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/src/resize.cpp`
- **File Name**: `resize.cpp`
- **File Size**: 171,787 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/src/resize.cpp](../../../modules/imgproc/src/resize.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, 2017, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Copyright (C) 2014-2015, Itseez Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

/* ////////////////////////////////////////////////////////////////////
//
//  Geometrical transforms on images and matrices: rotation, zoom etc.
//
// */

#include "precomp.hpp"
#include "opencl_kernels_imgproc.hpp"
#include "hal_replacement.hpp"
#include "opencv2/core/hal/intrin.hpp"
#include "opencv2/core/utils/buffer_area.private.hpp"

#include "opencv2/core/openvx/ovx_defs.hpp"
#include "resize.hpp"

#include "opencv2/core/softfloat.hpp"
#include "fixedpoint.inl.hpp"

using namespace cv;

namespace
{

template <typename ET, bool needsign> struct fixedtype { typedef fixedpoint64 type; };
template <> struct fixedtype<uint32_t, false> { typedef ufixedpoint64 type; };
template <bool needsign> struct fixedtype<int16_t, needsign> { typedef fixedpoint32 type; };
template <> struct fixedtype<uint16_t, false> { typedef ufixedpoint32 type; };
template <bool needsign> struct fixedtype<int8_t, needsign> { typedef fixedpoint32 type; };
template <> struct fixedtype<uint8_t, false> { typedef ufixedpoint16 type; };

//FT is fixedtype<ET, needsign>::type
template <typename ET, typename FT, int n, bool mulall>
static void hlineResize(ET* src, int cn, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    for (; i < dst_min; i++, m += n) // Points that fall left from src image so became equal to leftmost src point
    {
        for (int j = 0; j < cn; j++, dst++)
        {
            *dst = src[j];
        }
    }
    for (; i < dst_max; i++, m += n)
    {
        ET* src_ofst = src + cn*ofst[i];
        for (int j = 0; j < cn; j++, dst++)
        {
            *dst = (mulall || !m[0].isZero()) ? m[0] * src_ofst[j] : FT::zero();
            for (int k = 1; k < n; k++)
            {
                *dst = *dst + ((mulall || !m[k].isZero()) ? m[k] * src_ofst[j+k*cn] : FT::zero());
            }
        }
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    ET* src_last = src + cn*ofst[dst_width - 1];
    for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
    {
        for (int j = 0; j < cn; j++, dst++)
        {
            *dst = src_last[j];
        }
    }
}
template <typename ET, typename FT, int n, bool mulall, int cncnt> struct hline
{
    static void ResizeCn(ET* src, int cn, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        hlineResize<ET, FT, n, mulall>(src, cn, ofst, m, dst, dst_min, dst_max, dst_width);
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 2, true, 1>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]);
        for (; i < dst_min; i++, m += 2) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
        }
        for (; i < dst_max; i++, m += 2)
        {
            ET* px = src + ofst[i];
            *(dst++) = m[0] * px[0] + m[1] * px[1];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + ofst[dst_width - 1])[0];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 2, true, 2>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]);
        for (; i < dst_min; i++, m += 2) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
        }
        for (; i < dst_max; i++, m += 2)
        {
            ET* px = src + 2*ofst[i];
            *(dst++) = m[0] * px[0] + m[1] * px[2];
            *(dst++) = m[0] * px[1] + m[1] * px[3];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 2*ofst[dst_width - 1])[0];
        src1 = (src + 2*ofst[dst_width - 1])[1];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 2, true, 3>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]), src2(src[2]);
        for (; i < dst_min; i++, m += 2) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
        }
        for (; i < dst_max; i++, m += 2)
        {
            ET* px = src + 3*ofst[i];
            *(dst++) = m[0] * px[0] + m[1] * px[3];
            *(dst++) = m[0] * px[1] + m[1] * px[4];
            *(dst++) = m[0] * px[2] + m[1] * px[5];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 3*ofst[dst_width - 1])[0];
        src1 = (src + 3*ofst[dst_width - 1])[1];
        src2 = (src + 3*ofst[dst_width - 1])[2];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 2, true, 4>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]), src2(src[2]), src3(src[3]);
        for (; i < dst_min; i++, m += 2) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
            *(dst++) = src3;
        }
        for (; i < dst_max; i++, m += 2)
        {
            ET* px = src + 4*ofst[i];
            *(dst++) = m[0] * px[0] + m[1] * px[4];
            *(dst++) = m[0] * px[1] + m[1] * px[5];
            *(dst++) = m[0] * px[2] + m[1] * px[6];
            *(dst++) = m[0] * px[3] + m[1] * px[7];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 4*ofst[dst_width - 1])[0];
        src1 = (src + 4*ofst[dst_width - 1])[1];
        src2 = (src + 4*ofst[dst_width - 1])[2];
        src3 = (src + 4*ofst[dst_width - 1])[3];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
            *(dst++) = src3;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 4, true, 1>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]);
        for (; i < dst_min; i++, m += 4) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
        }
        for (; i < dst_max; i++, m += 4)
        {
            ET* px = src + ofst[i];
            *(dst++) = m[0] * src[0] + m[1] * src[1] + m[2] * src[2] + m[3] * src[3];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + ofst[dst_width - 1])[0];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 4, true, 2>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]);
        for (; i < dst_min; i++, m += 4) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
        }
        for (; i < dst_max; i++, m += 4)
        {
            ET* px = src + 2*ofst[i];
            *(dst++) = m[0] * src[0] + m[1] * src[2] + m[2] * src[4] + m[3] * src[6];
            *(dst++) = m[0] * src[1] + m[1] * src[3] + m[2] * src[5] + m[3] * src[7];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 2*ofst[dst_width - 1])[0];
        src1 = (src + 2*ofst[dst_width - 1])[1];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 4, true, 3>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]), src2(src[2]);
        for (; i < dst_min; i++, m += 4) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
        }
        for (; i < dst_max; i++, m += 4)
        {
            ET* px = src + 3*ofst[i];
            *(dst++) = m[0] * src[0] + m[1] * src[3] + m[2] * src[6] + m[3] * src[ 9];
            *(dst++) = m[0] * src[1] + m[1] * src[4] + m[2] * src[7] + m[3] * src[10];
            *(dst++) = m[0] * src[2] + m[1] * src[5] + m[2] * src[8] + m[3] * src[11];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 3*ofst[dst_width - 1])[0];
        src1 = (src + 3*ofst[dst_width - 1])[1];
        src2 = (src + 3*ofst[dst_width - 1])[2];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
        }
    }
};
template <typename ET, typename FT> struct hline<ET, FT, 4, true, 4>
{
    static void ResizeCn(ET* src, int, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
    {
        int i = 0;
        FT src0(src[0]), src1(src[1]), src2(src[2]), src3(src[3]);
        for (; i < dst_min; i++, m += 4) // Points that fall left from src image so became equal to leftmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
            *(dst++) = src3;
        }
        for (; i < dst_max; i++, m += 4)
        {
            ET* px = src + 4*ofst[i];
            *(dst++) = m[0] * src[0] + m[1] * src[4] + m[2] * src[ 8] + m[3] * src[12];
            *(dst++) = m[0] * src[1] + m[1] * src[5] + m[2] * src[ 9] + m[3] * src[13];
            *(dst++) = m[0] * src[2] + m[1] * src[6] + m[2] * src[10] + m[3] * src[14];
            *(dst++) = m[0] * src[3] + m[1] * src[7] + m[2] * src[11] + m[3] * src[15];
        }
        // Avoid reading a potentially unset ofst, leading to a random memory read.
        if (i >= dst_width) {
            return;
        }
        src0 = (src + 4*ofst[dst_width - 1])[0];
        src1 = (src + 4*ofst[dst_width - 1])[1];
        src2 = (src + 4*ofst[dst_width - 1])[2];
        src3 = (src + 4*ofst[dst_width - 1])[3];
        for (; i < dst_width; i++) // Points that fall right from src image so became equal to rightmost src point
        {
            *(dst++) = src0;
            *(dst++) = src1;
            *(dst++) = src2;
            *(dst++) = src3;
        }
    }
};
template <typename ET, typename FT, int n, bool mulall, int cncnt>
static void hlineResizeCn(ET* src, int cn, int *ofst, FT* m, FT* dst, int dst_min, int dst_max, int dst_width)
{
    hline<ET, FT, n, mulall, cncnt>::ResizeCn(src, cn, ofst, m, dst, dst_min, dst_max, dst_width);
}

template <>
void hlineResizeCn<uint8_t, ufixedpoint16, 2, true, 1>(uint8_t* src, int, int *ofst, ufixedpoint16* m, ufixedpoint16* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    ufixedpoint16 src_0(src[0]);
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint16>::vlanes();
    v_uint16 v_src_0 = vx_setall_u16(*((uint16_t*)&src_0));
    for (; i <= dst_min - VECSZ; i += VECSZ, m += 2*VECSZ, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_src_0);
    }
#endif
    for (; i < dst_min; i++, m += 2)
    {
        *(dst++) = src_0;
    }
#if (CV_SIMD || CV_SIMD_SCALABLE)
    for (; i <= dst_max - 2*VECSZ; i += 2*VECSZ, m += 4*VECSZ, dst += 2*VECSZ)
    {
        v_uint16 v_src0, v_src1;
        v_expand(vx_lut_pairs(src, ofst + i), v_src0, v_src1);
        v_store((uint16_t*)dst      , v_pack(v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), vx_load((int16_t*)m))),
                                             v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), vx_load((int16_t*)m + VECSZ)))));
        v_expand(vx_lut_pairs(src, ofst + i + VECSZ), v_src0, v_src1);
        v_store((uint16_t*)dst+VECSZ, v_pack(v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), vx_load((int16_t*)m + 2*VECSZ))),
                                             v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), vx_load((int16_t*)m + 3*VECSZ)))));
    }
    if (i <= dst_max - VECSZ)
    {
        v_uint16 v_src0, v_src1;
        v_expand(vx_lut_pairs(src, ofst + i), v_src0, v_src1);
        v_store((uint16_t*)dst, v_pack(v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), vx_load((int16_t*)m))),
                                       v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), vx_load((int16_t*)m + VECSZ)))));
        i += VECSZ; m += 2*VECSZ; dst += VECSZ;
    }
#endif
    for (; i < dst_max; i += 1, m += 2)
    {
        uint8_t* px = src + ofst[i];
        *(dst++) = m[0] * px[0] + m[1] * px[1];
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    src_0 = (src + ofst[dst_width - 1])[0];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    v_src_0 = vx_setall_u16(*((uint16_t*)&src_0));
    for (; i <= dst_width - VECSZ; i += VECSZ, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_src_0);
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = src_0;
    }
}
template <>
void hlineResizeCn<uint8_t, ufixedpoint16, 2, true, 2>(uint8_t* src, int, int *ofst, ufixedpoint16* m, ufixedpoint16* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    union {
        uint32_t d;
        uint16_t w[2];
    } srccn;
    ((ufixedpoint16*)(srccn.w))[0] = src[0];
    ((ufixedpoint16*)(srccn.w))[1] = src[1];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint16>::vlanes();
    v_uint16 v_srccn = v_reinterpret_as_u16(vx_setall_u32(srccn.d));
    for (; i <= dst_min - VECSZ/2; i += VECSZ/2, m += VECSZ, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_min; i++, m += 2)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
    }
#if (CV_SIMD || CV_SIMD_SCALABLE)
    for (; i <= dst_max - VECSZ/2; i += VECSZ/2, m += VECSZ, dst += VECSZ)
    {
        v_uint16 v_src0, v_src1;
        v_expand(v_interleave_pairs(v_reinterpret_as_u8(vx_lut_pairs((uint16_t*)src, ofst + i))), v_src0, v_src1);

        v_uint32 v_mul = vx_load((uint32_t*)m);//AaBbCcDd
        v_uint32 v_zip0, v_zip1;
        v_zip(v_mul, v_mul, v_zip0, v_zip1);//AaAaBbBb CcCcDdDd
        v_uint32 v_res0 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), v_reinterpret_as_s16(v_zip0)));
        v_uint32 v_res1 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), v_reinterpret_as_s16(v_zip1)));
        v_store((uint16_t*)dst, v_pack(v_res0, v_res1));//AB1AB2CD1CD2
    }
#endif
    for (; i < dst_max; i += 1, m += 2)
    {
        uint8_t* px = src + 2 * ofst[i];
        *(dst++) = m[0] * px[0] + m[1] * px[2];
        *(dst++) = m[0] * px[1] + m[1] * px[3];
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    ((ufixedpoint16*)(srccn.w))[0] = (src + 2 * ofst[dst_width - 1])[0]; ((ufixedpoint16*)(srccn.w))[1] = (src + 2 * ofst[dst_width - 1])[1];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    v_srccn = v_reinterpret_as_u16(vx_setall_u32(srccn.d));
    for (; i <= dst_width - VECSZ/2; i += VECSZ/2, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
    }
}
template <>
void hlineResizeCn<uint8_t, ufixedpoint16, 2, true, 3>(uint8_t* src, int, int *ofst, ufixedpoint16* m, ufixedpoint16* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    union {
        uint64_t q;
        uint16_t w[4];
    } srccn;
    ((ufixedpoint16*)(srccn.w))[0] = src[0];
    ((ufixedpoint16*)(srccn.w))[1] = src[1];
    ((ufixedpoint16*)(srccn.w))[2] = src[2];
    ((ufixedpoint16*)(srccn.w))[3] = 0;
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint16>::vlanes();
    v_uint16 v_srccn = v_pack_triplets(v_reinterpret_as_u16(vx_setall_u64(srccn.q)));
    for (; i <= dst_min - (VECSZ+2)/3; i += VECSZ/4, m += VECSZ/2, dst += 3*VECSZ/4) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_min; i++, m += 2)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[2];
    }
#if (CV_SIMD || CV_SIMD_SCALABLE)
    CV_DECL_ALIGNED(CV_SIMD_WIDTH) int ofst3[VTraits<v_uint16>::max_nlanes/2];
    for (; i <= dst_max - (3*VECSZ/4 + (VECSZ+2)/3); i += VECSZ/2, m += VECSZ, dst += 3*VECSZ/2)
    {
        v_store(ofst3, v_mul(vx_load(ofst + i), vx_setall_s32(3)));
        v_uint8 v_src01, v_src23;
        v_uint16 v_src0, v_src1, v_src2, v_src3;
        v_zip(vx_lut_quads(src, ofst3), v_reinterpret_as_u8(v_shr<8>(v_reinterpret_as_u32(vx_lut_quads(src+2, ofst3)))), v_src01, v_src23);
        v_expand(v_src01, v_src0, v_src1);
        v_expand(v_src23, v_src2, v_src3);

        v_uint32 v_mul0, v_mul1, v_mul2, v_mul3, v_tmp;
        v_mul0 = vx_load((uint32_t*)m);//AaBbCcDd
        v_zip(v_mul0, v_mul0, v_mul3, v_tmp );//AaAaBbBb CcCcDdDd
        v_zip(v_mul3, v_mul3, v_mul0, v_mul1);//AaAaAaAa BbBbBbBb
        v_zip(v_tmp , v_tmp , v_mul2, v_mul3);//CcCcCcCc DdDdDdDd

        v_uint32 v_res0 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), v_reinterpret_as_s16(v_mul0)));
        v_uint32 v_res1 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), v_reinterpret_as_s16(v_mul1)));
        v_uint32 v_res2 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src2), v_reinterpret_as_s16(v_mul2)));
        v_uint32 v_res3 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src3), v_reinterpret_as_s16(v_mul3)));
        v_store((uint16_t*)dst            , v_pack_triplets(v_pack(v_res0, v_res1)));
        v_store((uint16_t*)dst + 3*VECSZ/4, v_pack_triplets(v_pack(v_res2, v_res3)));
    }
#endif
    for (; i < dst_max; i += 1, m += 2)
    {
        uint8_t* px = src + 3 * ofst[i];
        *(dst++) = m[0] * px[0] + m[1] * px[3];
        *(dst++) = m[0] * px[1] + m[1] * px[4];
        *(dst++) = m[0] * px[2] + m[1] * px[5];
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    ((ufixedpoint16*)(srccn.w))[0] = (src + 3*ofst[dst_width - 1])[0];
    ((ufixedpoint16*)(srccn.w))[1] = (src + 3*ofst[dst_width - 1])[1];
    ((ufixedpoint16*)(srccn.w))[2] = (src + 3*ofst[dst_width - 1])[2];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    v_srccn = v_pack_triplets(v_reinterpret_as_u16(vx_setall_u64(srccn.q)));
    for (; i <= dst_width - (VECSZ+2)/3; i += VECSZ/4, dst += 3*VECSZ/4) // Points that fall right from src image so became equal to rightmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[2];
    }
}
template <>
void hlineResizeCn<uint8_t, ufixedpoint16, 2, true, 4>(uint8_t* src, int, int *ofst, ufixedpoint16* m, ufixedpoint16* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    union {
        uint64_t q;
        uint16_t w[4];
    } srccn;
    ((ufixedpoint16*)(srccn.w))[0] = src[0];
    ((ufixedpoint16*)(srccn.w))[1] = src[1];
    ((ufixedpoint16*)(srccn.w))[2] = src[2];
    ((ufixedpoint16*)(srccn.w))[3] = src[3];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint16>::vlanes();
    v_uint16 v_srccn = v_reinterpret_as_u16(vx_setall_u64(srccn.q));
    for (; i <= dst_min - VECSZ/4; i += VECSZ/4, m += VECSZ/2, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_min; i++, m += 2)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[2];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[3];
    }
#if (CV_SIMD || CV_SIMD_SCALABLE)
    for (; i <= dst_max - VECSZ/2; i += VECSZ/2, m += VECSZ, dst += 2*VECSZ)
    {
        v_uint16 v_src0, v_src1, v_src2, v_src3;
        v_expand(v_interleave_quads(v_reinterpret_as_u8(vx_lut_pairs((uint32_t*)src, ofst + i))), v_src0, v_src1);
        v_expand(v_interleave_quads(v_reinterpret_as_u8(vx_lut_pairs((uint32_t*)src, ofst + i + VECSZ/4))), v_src2, v_src3);

        v_uint32 v_mul0, v_mul1, v_mul2, v_mul3, v_tmp;
        v_mul0 = vx_load((uint32_t*)m);//AaBbCcDd
        v_zip(v_mul0, v_mul0, v_mul3, v_tmp );//AaAaBbBb CcCcDdDd
        v_zip(v_mul3, v_mul3, v_mul0, v_mul1);//AaAaAaAa BbBbBbBb
        v_zip(v_tmp , v_tmp , v_mul2, v_mul3);//CcCcCcCc DdDdDdDd

        v_uint32 v_res0 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src0), v_reinterpret_as_s16(v_mul0)));
        v_uint32 v_res1 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src1), v_reinterpret_as_s16(v_mul1)));
        v_uint32 v_res2 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src2), v_reinterpret_as_s16(v_mul2)));
        v_uint32 v_res3 = v_reinterpret_as_u32(v_dotprod(v_reinterpret_as_s16(v_src3), v_reinterpret_as_s16(v_mul3)));
        v_store((uint16_t*)dst        , v_pack(v_res0, v_res1));
        v_store((uint16_t*)dst + VECSZ, v_pack(v_res2, v_res3));
    }
#endif
    for (; i < dst_max; i += 1, m += 2)
    {
        uint8_t* px = src + 4 * ofst[i];
        *(dst++) = m[0] * px[0] + m[1] * px[4];
        *(dst++) = m[0] * px[1] + m[1] * px[5];
        *(dst++) = m[0] * px[2] + m[1] * px[6];
        *(dst++) = m[0] * px[3] + m[1] * px[7];
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    ((ufixedpoint16*)(srccn.w))[0] = (src + 4 * ofst[dst_width - 1])[0]; ((ufixedpoint16*)(srccn.w))[1] = (src + 4 * ofst[dst_width - 1])[1];
    ((ufixedpoint16*)(srccn.w))[2] = (src + 4 * ofst[dst_width - 1])[2]; ((ufixedpoint16*)(srccn.w))[3] = (src + 4 * ofst[dst_width - 1])[3];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    v_srccn = v_reinterpret_as_u16(vx_setall_u64(srccn.q));
    for (; i <= dst_width - VECSZ/4; i += VECSZ/4, dst += VECSZ) // Points that fall right from src image so became equal to rightmost src point
    {
        v_store((uint16_t*)dst, v_srccn);
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = ((ufixedpoint16*)(srccn.w))[0];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[1];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[2];
        *(dst++) = ((ufixedpoint16*)(srccn.w))[3];
    }
}
template <>
void hlineResizeCn<uint16_t, ufixedpoint32, 2, true, 1>(uint16_t* src, int, int *ofst, ufixedpoint32* m, ufixedpoint32* dst, int dst_min, int dst_max, int dst_width)
{
    int i = 0;
    ufixedpoint32 src_0(src[0]);
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint32>::vlanes();
    v_uint32 v_src_0 = vx_setall_u32(*((uint32_t*)&src_0));
    for (; i <= dst_min - VECSZ; i += VECSZ, m += 2*VECSZ, dst += VECSZ) // Points that fall left from src image so became equal to leftmost src point
    {
        v_store((uint32_t*)dst, v_src_0);
    }
#endif
    for (; i < dst_min; i++, m += 2)
    {
        *(dst++) = src_0;
    }
#if (CV_SIMD || CV_SIMD_SCALABLE)
    for (; i <= dst_max - VECSZ; i += VECSZ, m += 2*VECSZ, dst += VECSZ)
    {
        v_uint32 v_src0, v_src1;
        v_expand(vx_lut_pairs(src, ofst + i), v_src0, v_src1);

        v_uint64 v_res0 = v_reinterpret_as_u64(v_mul(v_src0, vx_load((uint32_t *)m)));
        v_uint64 v_res1 = v_reinterpret_as_u64(v_mul(v_src1, vx_load((uint32_t *)m + VECSZ)));
        v_store((uint32_t*)dst, v_pack(v_add(v_and(v_res0, vx_setall_u64(0xFFFFFFFF)), v_shr<32>(v_res0)),
                                       v_add(v_and(v_res1, vx_setall_u64(0xFFFFFFFF)), v_shr<32>(v_res1))));
    }
#endif
    for (; i < dst_max; i += 1, m += 2)
    {
        uint16_t* px = src + ofst[i];
        *(dst++) = m[0] * px[0] + m[1] * px[1];
    }
    // Avoid reading a potentially unset ofst, leading to a random memory read.
    if (i >= dst_width) {
        return;
    }
    src_0 = (src + ofst[dst_width - 1])[0];
#if (CV_SIMD || CV_SIMD_SCALABLE)
    v_src_0 = vx_setall_u32(*((uint32_t*)&src_0));
    for (; i <= dst_width - VECSZ; i += VECSZ, dst += VECSZ)
    {
        v_store((uint32_t*)dst, v_src_0);
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = src_0;
    }
}

template <typename ET, typename FT>
void vlineSet(FT* src, ET* dst, int dst_width)
{
    for (int i = 0; i < dst_width; i++)
        dst[i] = src[i];
}
template <>
void vlineSet<uint8_t, ufixedpoint16>(ufixedpoint16* src, uint8_t* dst, int dst_width)
{
    int i = 0;
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint8>::vlanes();
    const v_uint16 v_fixedRound = vx_setall_u16((uint16_t)((1U << 8) >> 1));
    for (; i <= dst_width - VECSZ; i += VECSZ, src += VECSZ, dst += VECSZ)
    {
        v_uint16 v_src0 = vx_load((uint16_t*)src);
        v_uint16 v_src1 = vx_load((uint16_t*)src + VECSZ/2);

        v_uint16 v_res0 = v_shr<8>(v_add(v_src0, v_fixedRound));
        v_uint16 v_res1 = v_shr<8>(v_add(v_src1, v_fixedRound));

        v_store(dst, v_pack(v_res0, v_res1));
    }
#endif
    for (; i < dst_width; i++)
        *(dst++) = *(src++);
}

template <typename ET, typename FT, int n>
void vlineResize(FT* src, size_t src_step, FT* m, ET* dst, int dst_width)
{
    for (int i = 0; i < dst_width; i++)
    {
        typename FT::WT res = src[i] * m[0];
        for (int k = 1; k < n; k++)
            res = res + src[i + k*src_step] * m[k];
        dst[i] = res;
    }
}
template <>
void vlineResize<uint8_t, ufixedpoint16, 2>(ufixedpoint16* src, size_t src_step, ufixedpoint16* m, uint8_t* dst, int dst_width)
{
    int i = 0;
    ufixedpoint16* src1 = src + src_step;
#if (CV_SIMD || CV_SIMD_SCALABLE)
    const int VECSZ = VTraits<v_uint8>::vlanes();
    const v_int32 v_fixedRound = vx_setall_s32((int32_t)((1 << 16) >> 1));
    const v_int16 v_128    = v_reinterpret_as_s16(vx_setall_u16((uint16_t)1<<15));
    const v_int8  v_128_16 = v_reinterpret_as_s8 (vx_setall_u8 ((uint8_t) 1<<7));

    v_int16 v_mul = v_reinterpret_as_s16(vx_setall_u32(((uint32_t*)m)[0]));
    for (; i <= dst_width - VECSZ; i += VECSZ, src += VECSZ, src1 += VECSZ, dst += VECSZ)
    {
        v_int16 v_src00 = vx_load((int16_t*)src);
        v_int16 v_src10 = vx_load((int16_t*)src1);
        v_int16 v_tmp0, v_tmp1;
        v_zip(v_add_wrap(v_src00,v_128), v_add_wrap(v_src10,v_128), v_tmp0, v_tmp1);

        v_int32 v_res0 = v_dotprod(v_tmp0, v_mul);
        v_int32 v_res1 = v_dotprod(v_tmp1, v_mul);

        v_int16 v_src01 = vx_load((int16_t*)src + VECSZ/2);
        v_int16 v_src11 = vx_load((int16_t*)src1 + VECSZ/2);
        v_zip(v_add_wrap(v_src01,v_128), v_add_wrap(v_src11,v_128), v_tmp0, v_tmp1);
        v_int32 v_res2 = v_dotprod(v_tmp0, v_mul);
        v_int32 v_res3 = v_dotprod(v_tmp1, v_mul);

        v_int8 v_res = v_pack(v_pack(v_shr<16>(v_add(v_res0, v_fixedRound)),
                                     v_shr<16>(v_add(v_res1, v_fixedRound))),
                              v_pack(v_shr<16>(v_add(v_res2, v_fixedRound)),
                                     v_shr<16>(v_add(v_res3, v_fixedRound))));

        v_store(dst, v_reinterpret_as_u8(v_sub_wrap(v_res, v_128_16)));
    }
#endif
    for (; i < dst_width; i++)
    {
        *(dst++) = (uint8_t)(*(src++) * m[0] + *(src1++) * m[1]);
    }
}

template <typename ET> class interpolationLinear
{
public:
    static const int len = 2;
    static const bool needsign = false;
    interpolationLinear(double inv_scale, int srcsize, int dstsize) : scale(softdouble::one() / softdouble(inv_scale)), maxsize(srcsize), minofst(0), maxofst(dstsize) {}
    void getCoeffs(int val, int* offset, typename fixedtype<ET, needsign>::type* coeffs)
    {
        typedef typename fixedtype<ET, needsign>::type fixedpoint;
        softdouble fval = scale*(softdouble(val)+softdouble(0.5))-softdouble(0.5);
        int ival = cvFloor(fval);
        if (ival >= 0 && maxsize > 1)
        {
            if (ival < maxsize - 1)
            {
                *offset = ival;
                coeffs[1] = fval - softdouble(ival);
                coeffs[0] = fixedpoint::one() - coeffs[1];
            }
            else
            {
                *offset = maxsize - 1;
                maxofst = min(maxofst, val);
            }
        }
        else
        {
            minofst = max(minofst, val + 1);
        }
    }
    void getMinMax(int &min, int &max)
    {
        min = minofst;
        max = maxofst;
    }
protected:
    softdouble scale;
    int maxsize;
    int minofst, maxofst;
};

template <typename ET, typename FT, int interp_y_len>
class resize_bitExactInvoker :
    public ParallelLoopBody
{
public:
    typedef FT fixedpoint;
    typedef void(*hResizeFunc)(ET* src, int cn, int *ofst, fixedpoint* m, fixedpoint* dst, int dst_min, int dst_max, int dst_width);
    resize_bitExactInvoker(const uchar* _src, size_t _src_step, int _src_width, int _src_height,
                           uchar* _dst, size_t _dst_step, int _dst_width, int _dst_height,
                           int _cn, int *_xoffsets, int *_yoffsets, fixedpoint *_xcoeffs, fixedpoint *_ycoeffs,
                           int _min_x, int _max_x, int _min_y, int _max_y, hResizeFunc _hResize) : ParallelLoopBody(),
                           src(_src), src_step(_src_step), src_width(_src_width), src_height(_src_height),
                           dst(_dst), dst_step(_dst_step), dst_width(_dst_width), dst_height(_dst_height),
                           cn(_cn), xoffsets(_xoffsets), yoffsets(_yoffsets), xcoeffs(_xcoeffs), ycoeffs(_ycoeffs),
                           min_x(_min_x), max_x(_max_x), min_y(_min_y), max_y(_max_y), hResize(_hResize) {}

    virtual void operator() (const Range& range) const CV_OVERRIDE
    {
        AutoBuffer<fixedpoint> linebuf(interp_y_len * dst_width * cn);
        int last_eval = - interp_y_len;
        int evalbuf_start = 0;
        int rmin_y = max(min_y, range.start);
        int rmax_y = min(max_y, range.end);
        if (range.start < min_y)
        {
            last_eval = 1 - interp_y_len;
            evalbuf_start = 1;
            hResize((ET*)src, cn, xoffsets, xcoeffs, linebuf.data(), min_x, max_x, dst_width);
        }
        int dy = range.start;
        for (; dy < rmin_y; dy++)
            vlineSet<ET, FT>(linebuf.data(), (ET*)(dst + dst_step * dy), dst_width*cn);
        for (; dy < rmax_y; dy++)
        {
            int &iy = yoffsets[dy];

            int i;
            for (i = max(iy, last_eval + interp_y_len); i < min(iy + interp_y_len, src_height); i++, evalbuf_start = (evalbuf_start + 1) % interp_y_len)
                hResize((ET*)(src + i * src_step), cn, xoffsets, xcoeffs, linebuf.data() + evalbuf_start*(dst_width * cn), min_x, max_x, dst_width);
            evalbuf_start = (evalbuf_start + max(iy, src_height - interp_y_len) - max(last_eval, src_height - interp_y_len)) % interp_y_len;
            last_eval = iy;

            fixedpoint curcoeffs[interp_y_len];
            for (i = 0; i < evalbuf_start; i++)
                curcoeffs[i] = ycoeffs[ dy*interp_y_len - evalbuf_start + interp_y_len + i];
            for (; i < interp_y_len; i++)
                curcoeffs[i] = ycoeffs[ dy*interp_y_len - evalbuf_start + i];

            vlineResize<ET, FT, interp_y_len>(linebuf.data(), dst_width*cn, curcoeffs, (ET*)(dst + dst_step * dy), dst_width*cn);
        }
        fixedpoint *endline = linebuf.data();
        if (last_eval + interp_y_len > src_height)
            endline += dst_width*cn*((evalbuf_start + src_height - 1 - last_eval) % interp_y_len);
        else
            hResize((ET*)(src + (src_height - 1) * src_step), cn, xoffsets, xcoeffs, endline, min_x, max_x, dst_width);
        for (; dy < range.end; dy++)
            vlineSet<ET, FT>(endline, (ET*)(dst + dst_step * dy), dst_width*cn);
#if (CV_SIMD || CV_SIMD_SCALABLE)
        vx_cleanup();
#endif
    }

private:
    const uchar* src;
    size_t src_step;
    int src_width, src_height;
    uchar* dst;
    size_t dst_step;
    int dst_width, dst_height, cn;
    int *xoffsets, *yoffsets;
    fixedpoint *xcoeffs, *ycoeffs;
    int min_x, max_x, min_y, max_y;
    hResizeFunc hResize;

    resize_bitExactInvoker(const resize_bitExactInvoker&);
    resize_bitExactInvoker& operator=(const resize_bitExactInvoker&);
};

template <typename ET, typename interpolation>
void resize_bitExact(const uchar* src, size_t src_step, int src_width, int src_height,
                           uchar* dst, size_t dst_step, int dst_width, int dst_height,
                     int cn, double inv_scale_x, double inv_scale_y)
{
    typedef typename fixedtype<ET, interpolation::needsign>::type fixedpoint;
    void(*hResize)(ET* src, int cn, int *ofst, fixedpoint* m, fixedpoint* dst, int dst_min, int dst_max, int dst_width);
    switch (cn)
    {
    case  1: hResize = src_width > interpolation::len ? hlineResizeCn<ET, fixedpoint, interpolation::len, true, 1> : hlineResizeCn<ET, fixedpoint, interpolation::len, false, 1>; break;
    case  2: hResize = src_width > interpolation::len ? hlineResizeCn<ET, fixedpoint, interpolation::len, true, 2> : hlineResizeCn<ET, fixedpoint, interpolation::len, false, 2>; break;
    case  3: hResize = src_width > interpolation::len ? hlineResizeCn<ET, fixedpoint, interpolation::len, true, 3> : hlineResizeCn<ET, fixedpoint, interpolation::len, false, 3>; break;
    case  4: hResize = src_width > interpolation::len ? hlineResizeCn<ET, fixedpoint, interpolation::len, true, 4> : hlineResizeCn<ET, fixedpoint, interpolation::len, false, 4>; break;
    default: hResize = src_width > interpolation::len ? hlineResize<ET, fixedpoint, interpolation::len, true>      : hlineResize<ET, fixedpoint, interpolation::len, false>     ; break;
    }

    interpolation interp_x(inv_scale_x, src_width, dst_width);
    interpolation interp_y(inv_scale_y, src_height, dst_height);

    AutoBuffer<uchar> buf( dst_width * sizeof(int) +
                           dst_height * sizeof(int) +
                           dst_width * interp_x.len*sizeof(fixedpoint) +
                           dst_height * interp_y.len * sizeof(fixedpoint) );
    int* xoffsets = (int*)buf.data();
    int* yoffsets = xoffsets + dst_width;
    fixedpoint* xcoeffs = (fixedpoint*)(yoffsets + dst_height);
    fixedpoint* ycoeffs = xcoeffs + dst_width * interp_x.len;

    int min_x, max_x, min_y, max_y;
    for (int dx = 0; dx < dst_width; dx++)
        interp_x.getCoeffs(dx, xoffsets+dx, xcoeffs+dx*interp_x.len);
    interp_x.getMinMax(min_x, max_x);
    for (int dy = 0; dy < dst_height; dy++)
        interp_y.getCoeffs(dy, yoffsets+dy, ycoeffs+dy*interp_y.len);
    interp_y.getMinMax(min_y, max_y);

    resize_bitExactInvoker<ET, fixedpoint, interpolation::len> invoker(src, src_step, src_width, src_height, dst, dst_step, dst_width, dst_height, cn,
                                                                       xoffsets, yoffsets, xcoeffs, ycoeffs, min_x, max_x, min_y, max_y, hResize);
    Range range(0, dst_height);
    parallel_for_(range, invoker, dst_width * dst_height / (double)(1 << 16));
}

typedef void(*be_resize_func)(const uchar* src, size_t src_step, int src_width, int src_height,
                                    uchar* dst, size_t dst_step, int dst_width, int dst_height,
                              int cn, double inv_scale_x, double inv_scale_y);

}

namespace cv
{

/************** interpolation formulas and tables ***************/

const int INTER_RESIZE_COEF_BITS=11;
const int INTER_RESIZE_COEF_SCALE=1 << INTER_RESIZE_COEF_BITS;

static inline void interpolateCubic( float x, float* coeffs )
{
    const float A = -0.75f;

    coeffs[0] = ((A*(x + 1) - 5*A)*(x + 1) + 8*A)*(x + 1) - 4*A;
    coeffs[1] = ((A + 2)*x - (A + 3))*x*x + 1;
    coeffs[2] = ((A + 2)*(1 - x) - (A + 3))*(1 - x)*(1 - x) + 1;
    coeffs[3] = 1.f - coeffs[0] - coeffs[1] - coeffs[2];
}

static inline void interpolateLanczos4( float x, float* coeffs )
{
    static const double s45 = 0.70710678118654752440084436210485;
    static const double cs[][2]=
    {{1, 0}, {-s45, -s45}, {0, 1}, {s45, -s45}, {-1, 0}, {s45, s45}, {0, -1}, {-s45, s45}};

    float sum = 0;
    double y0=-(x+3)*CV_PI*0.25, s0 = std::sin(y0), c0= std::cos(y0);
    for(int i = 0; i < 8; i++ )
    {
        float y0_ = (x+3-i);
        if (fabs(y0_) >= 1e-6f)
        {
            double y = -y0_*CV_PI*0.25;
            coeffs[i] = (float)((cs[i][0]*s0 + cs[i][1]*c0)/(y*y));
        }
        else
        {
            // special handling for 'x' values:
            // - ~0.0: 0 0 0 1 0 0 0 0
            // - ~1.0: 0 0 0 0 1 0 0 0
            coeffs[i] = 1e30f;
        }
        sum += coeffs[i];
    }

    sum = 1.f/sum;
    for(int i = 0; i < 8; i++ )
        coeffs[i] *= sum;
}

template<typename ST, typename DT> struct Cast
{
    typedef ST type1;
    typedef DT rtype;

    DT operator()(ST val) const { return saturate_cast<DT>(val); }
};

template<typename ST, typename DT, int bits> struct FixedPtCast
{
    typedef ST type1;
    typedef DT rtype;
    enum { SHIFT = bits, DELTA = 1 << (bits-1) };

    DT operator()(ST val) const { return saturate_cast<DT>((val + DELTA)>>SHIFT); }
};

/****************************************************************************************\
*                                         Resize                                         *
\****************************************************************************************/

class resizeNNInvoker :
    public ParallelLoopBody
{
public:
    resizeNNInvoker(const Mat& _src, Mat &_dst, int *_x_ofs, double _ify) :
        ParallelLoopBody(), src(_src), dst(_dst), x_ofs(_x_ofs),
        ify(_ify)
    {
    }

    virtual void operator() (const Range& range) const CV_OVERRIDE
    {
        Size ssize = src.size(), dsize = dst.size();
        int y, x, pix_size = (int)src.elemSize();

        for( y = range.start; y < range.end; y++ )
        {
            uchar* D = dst.data + dst.step*y;
            int sy = std::min(cvFloor(y*ify), ssize.height-1);
            const uchar* S = src.ptr(sy);

            switch( pix_size )
            {
            case 1:
                for( x = 0; x <= dsize.width - 2; x += 2 )
                {
                    uchar t0 = S[x_ofs[x]];
                    uchar t1 = S[x_ofs[x+1]];
                    D[x] = t0;
                    D[x+1] = t1;
                }

                for( ; x < dsize.width; x++ )
                    D[x] = S[x_ofs[x]];
                break;
            case 2:
                for( x = 0; x < dsize.width; x++ )
                    *(ushort*)(D + x*2) = *(ushort*)(S + x_ofs[x]);
                break;
            case 3:
                for( x = 0; x < dsize.width; x++, D += 3 )
                {
                    const uchar* _tS = S + x_ofs[x];
                    D[0] = _tS[0]; D[1] = _tS[1]; D[2] = _tS[2];
                }
                break;
            case 4:
                for( x = 0; x < dsize.width; x++ )
                    *(int*)(D + x*4) = *(int*)(S + x_ofs[x]);
                break;
            case 6:
                for( x = 0; x < dsize.width; x++, D += 6 )
                {
                    const ushort* _tS = (const ushort*)(S + x_ofs[x]);
                    ushort* _tD = (ushort*)D;
                    _tD[0] = _tS[0]; _tD[1] = _tS[1]; _tD[2] = _tS[2];
                }
                break;
            case 8:
                for( x = 0; x < dsize.width; x++, D += 8 )
                {
                    const int* _tS = (const int*)(S + x_ofs[x]);
                    int* _tD = (int*)D;
                    _tD[0] = _tS[0]; _tD[1] = _tS[1];
                }
                break;
            case 12:
                for( x = 0; x < dsize.width; x++, D += 12 )
                {
                    const int* _tS = (const int*)(S + x_ofs[x]);
                    int* _tD = (int*)D;
                    _tD[0] = _tS[0]; _tD[1] = _tS[1]; _tD[2] = _tS[2];
                }
                break;
            default:
                for( x = 0; x < dsize.width; x++, D += pix_size )
                {
                    const uchar* _tS = S + x_ofs[x];
                    for (int k = 0; k < pix_size; k++)
                        D[k] = _tS[k];
                }
            }
        }
    }

private:
    const Mat& src;
    Mat& dst;
    int* x_ofs;
    double ify;

    resizeNNInvoker(const resizeNNInvoker&);
    resizeNNInvoker& operator=(const resizeNNInvoker&);
};

static void
resizeNN( const Mat& src, Mat& dst, double fx, double fy )
{
    Size ssize = src.size(), dsize = dst.size();
    AutoBuffer<int> _x_ofs(dsize.width);
    int* x_ofs = _x_ofs.data();
    int pix_size = (int)src.elemSize();
    double ifx = 1./fx, ify = 1./fy;
    int x;

    for( x = 0; x < dsize.width; x++ )
    {
        int sx = cvFloor(x*ifx);
        x_ofs[x] = std::min(sx, ssize.width-1)*pix_size;
    }

    Range range(0, dsize.height);
#if CV_TRY_AVX2
    if(CV_CPU_HAS_SUPPORT_AVX2 && ((pix_size == 2) || (pix_size == 4)))
    {
        if(pix_size == 2)
            opt_AVX2::resizeNN2_AVX2(range, src, dst, x_ofs, ify);
        else
            opt_AVX2::resizeNN4_AVX2(range, src, dst, x_ofs, ify);
    }
    else
#endif
#if CV_TRY_SSE4_1
    if(CV_CPU_HAS_SUPPORT_SSE4_1 && ((pix_size == 2) || (pix_size == 4)))
    {
        if(pix_size == 2)
            opt_SSE4_1::resizeNN2_SSE4_1(range, src, dst, x_ofs, ify);
        else
            opt_SSE4_1::resizeNN4_SSE4_1(range, src, dst, x_ofs, ify);
    }
    else
#endif
#if CV_TRY_LASX
    if(CV_CPU_HAS_SUPPORT_LASX && ((pix_size == 2) || (pix_size == 4)))
    {
        if(pix_size == 2)
            opt_LASX::resizeNN2_LASX(range, src, dst, x_ofs, ify);
        else
            opt_LASX::resizeNN4_LASX(range, src, dst, x_ofs, ify);
    }
    else
#endif
    {
        resizeNNInvoker invoker(src, dst, x_ofs, ify);
        parallel_for_(range, invoker, dst.total()/(double)(1<<16));
    }
}

class resizeNN_bitexactInvoker : public ParallelLoopBody
{
public:
    resizeNN_bitexactInvoker(const Mat& _src, Mat& _dst, int* _x_ofse, int _ify, int _ify0)
        : src(_src), dst(_dst), x_ofse(_x_ofse), ify(_ify), ify0(_ify0) {}

    virtual void operator() (const Range& range) const CV_OVERRIDE
    {
        Size ssize = src.size(), dsize = dst.size();
        int pix_size = (int)src.elemSize();
        for( int y = range.start; y < range.end; y++ )
        {
            uchar* D = dst.ptr(y);
            int _sy = (ify * y + ify0) >> 16;
            int sy = std::min(_sy, ssize.height-1);
            const uchar* S = src.ptr(sy);

            int x = 0;
            switch( pix_size )
            {
            case 1:
#if (CV_SIMD || CV_SIMD_SCALABLE)
                for( ; x <= dsize.width - VTraits<v_uint8>::vlanes(); x += VTraits<v_uint8>::vlanes() )
                    v_store(D + x, vx_lut(S, x_ofse + x));
#endif
                for( ; x < dsize.width; x++ )
                    D[x] = S[x_ofse[x]];
                break;
            case 2:
#if (CV_SIMD || CV_SIMD_SCALABLE)
                for( ; x <= dsize.width - VTraits<v_uint16>::vlanes(); x += VTraits<v_uint16>::vlanes() )
                    v_store((ushort*)D + x, vx_lut((ushort*)S, x_ofse + x));
#endif
                for( ; x < dsize.width; x++ )
                    *((ushort*)D + x) = *((ushort*)S + x_ofse[x]);
                break;
            case 3:
                for( ; x < dsize.width; x++, D += 3 )
                {
                    const uchar* _tS = S + x_ofse[x] * 3;
                    D[0] = _tS[0]; D[1] = _tS[1]; D[2] = _tS[2];
                }
                break;
            case 4:
#if (CV_SIMD || CV_SIMD_SCALABLE)
                for( ; x <= dsize.width - VTraits<v_uint32>::vlanes(); x += VTraits<v_uint32>::vlanes() )
                    v_store((uint32_t*)D + x, vx_lut((uint32_t*)S, x_ofse + x));
#endif
                for( ; x < dsize.width; x++ )
                    *((uint32_t*)D + x) = *((uint32_t*)S + x_ofse[x]);
                break;
            case 6:
                for( ; x < dsize.width; x++, D += 6 )
                {
                    const ushort* _tS = (const ushort*)(S + x_ofse[x]*6);
                    ushort* _tD = (ushort*)D;
                    _tD[0] = _tS[0]; _tD[1] = _tS[1]; _tD[2] = _tS[2];
                }
                break;
            case 8:
#if (CV_SIMD || CV_SIMD_SCALABLE)
                for( ; x <= dsize.width - VTraits<v_uint64>::vlanes(); x += VTraits<v_uint64>::vlanes() )
                    v_store((uint64_t*)D + x, vx_lut((uint64_t*)S, x_ofse + x));
#endif
                for( ; x < dsize.width; x++ )
                    *((uint64_t*)D + x) = *((uint64_t*)S + x_ofse[x]);
                break;
            case 12:
                for( ; x < dsize.width; x++, D += 12 )
                {
                    const int* _tS = (const int*)(S + x_ofse[x]*12);
                    int* _tD = (int*)D;
                    _tD[0] = _tS[0]; _tD[1] = _tS[1]; _tD[2] = _tS[2];
                }
                break;
            default:
                for( x = 0; x < dsize.width; x++, D += pix_size )
                {
                    const uchar* _tS = S + x_ofse[x] * pix_size;
                    for (int k = 0; k < pix_size; k++)
                        D[k] = _tS[k];
                }
            }
        }
    }
private:
    const Mat& src;
    Mat& dst;
    int* x_ofse;
    const int ify;
    const int ify0;
};

static void resizeNN_bitexact( const Mat& src, Mat& dst, double /*fx*/, double /*fy*/ )
{
    Size ssize = src.size(), dsize = dst.size();
    int ifx = ((ssize.width << 16) + dsize.width / 2) / dsize.width; // 16bit fixed-point arithmetic
    int ifx0 = ifx / 2 - ssize.width % 2;                       // This method uses center pixel coordinate as Pillow and scikit-images do.
    int ify = ((ssize.height << 16) + dsize.height / 2) / dsize.height;
    int ify0 = ify / 2 - ssize.height % 2;

    cv::utils::BufferArea area;
    int* x_ofse = 0;
    area.allocate(x_ofse, dsize.width, CV_SIMD_WIDTH);
    area.commit();

    for( int x = 0; x < dsize.width; x++ )
    {
        int sx = (ifx * x + ifx0) >> 16;
        x_ofse[x] = std::min(sx, ssize.width-1);    // offset in element (not byte)
    }
    Range range(0, dsize.height);
    resizeNN_bitexactInvoker invoker(src, dst, x_ofse, ify, ify0);
    parallel_for_(range, invoker, dst.total()/(double)(1<<16));
}

struct VResizeNoVec
{
    template<typename WT, typename T, typename BT>
    int operator()(const WT**, T*, const BT*, int ) const
    {
        return 0;
    }
};

struct HResizeNoVec
{
    template<typename T, typename WT, typename AT> inline
    int operator()(const T**, WT**, int, const int*,
        const AT*, int, int, int, int, int) const
    {
        return 0;
    }
};

#if (CV_SIMD || CV_SIMD_SCALABLE)

struct VResizeLinearVec_32s8u
{
    int operator()(const int** src, uchar* dst, const short* beta, int width) const
    {
        const int *S0 = src[0], *S1 = src[1];
        int x = 0;
        v_int16 b0 = vx_setall_s16(beta[0]), b1 = vx_setall_s16(beta[1]);

        if( (((size_t)S0|(size_t)S1)&(VTraits<v_uint8>::vlanes() - 1)) == 0 )
            for( ; x <= width - VTraits<v_uint8>::vlanes(); x += VTraits<v_uint8>::vlanes())
                v_store(dst + x, v_rshr_pack_u<2>(v_add(v_mul_hi(v_pack(v_shr<4>(vx_load_aligned(S0 + x)), v_shr<4>(vx_load_aligned(S0 + x + VTraits<v_int32>::vlanes()))), b0), v_mul_hi(v_pack(v_shr<4>(vx_load_aligned(S1 + x)), v_shr<4>(vx_load_aligned(S1 + x + VTraits<v_int32>::vlanes()))), b1)),
                                                  v_add(v_mul_hi(v_pack(v_shr<4>(vx_load_aligned(S0 + x + 2 * VTraits<v_int32>::vlanes())), v_shr<4>(vx_load_aligned(S0 + x + 3 * VTraits<v_int32>::vlanes()))), b0), v_mul_hi(v_pack(v_shr<4>(vx_load_aligned(S1 + x + 2 * VTraits<v_int32>::vlanes())), v_shr<4>(vx_load_aligned(S1 + x + 3 * VTraits<v_int32>::vlanes()))), b1))));
        else
            for( ; x <= width - VTraits<v_uint8>::vlanes(); x += VTraits<v_uint8>::vlanes())
                v_store(dst + x, v_rshr_pack_u<2>(v_add(v_mul_hi(v_pack(v_shr<4>(vx_load(S0 + x)), v_shr<4>(vx_load(S0 + x + VTraits<v_int32>::vlanes()))), b0), v_mul_hi(v_pack(v_shr<4>(vx_load(S1 + x)), v_shr<4>(vx_load(S1 + x + VTraits<v_int32>::vlanes()))), b1)),
                                                  v_add(v_mul_hi(v_pack(v_shr<4>(vx_load(S0 + x + 2 * VTraits<v_int32>::vlanes())), v_shr<4>(vx_load(S0 + x + 3 * VTraits<v_int32>::vlanes()))), b0), v_mul_hi(v_pack(v_shr<4>(vx_load(S1 + x + 2 * VTraits<v_int32>::vlanes())), v_shr<4>(vx_load(S1 + x + 3 * VTraits<v_int32>::vlanes()))), b1))));

        for( ; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
            v_rshr_pack_u_store<2>(dst + x, v_add(v_mul_hi(v_pack(v_shr<4>(vx_load(S0 + x)), v_shr<4>(vx_load(S0 + x + VTraits<v_int32>::vlanes()))), b0), v_mul_hi(v_pack(v_shr<4>(vx_load(S1 + x)), v_shr<4>(vx_load(S1 + x + VTraits<v_int32>::vlanes()))), b1)));

        return x;
    }
};

struct VResizeLinearVec_32f16u
{
    int operator()(const float** src, ushort* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1];
        int x = 0;

        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]);

        if( (((size_t)S0|(size_t)S1)&(VTraits<v_uint8>::vlanes() - 1)) == 0 )
            for( ; x <= width - VTraits<v_uint16>::vlanes(); x += VTraits<v_uint16>::vlanes())
                v_store(dst + x, v_pack_u(v_round(v_muladd(vx_load_aligned(S0 + x                    ), b0, v_mul(vx_load_aligned(S1 + x), b1))),
                                          v_round(v_muladd(vx_load_aligned(S0 + x + VTraits<v_float32>::vlanes()), b0, v_mul(vx_load_aligned(S1 + x + VTraits<v_float32>::vlanes()), b1)))));
        else
            for (; x <= width - VTraits<v_uint16>::vlanes(); x += VTraits<v_uint16>::vlanes())
                v_store(dst + x, v_pack_u(v_round(v_muladd(vx_load(S0 + x                    ), b0, v_mul(vx_load(S1 + x), b1))),
                                          v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()), b0, v_mul(vx_load(S1 + x + VTraits<v_float32>::vlanes()), b1)))));
        for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
        {
            v_int32 t0 = v_round(v_muladd(vx_load(S0 + x), b0, v_mul(vx_load(S1 + x), b1)));
            v_store_low(dst + x, v_pack_u(t0, t0));
        }

        return x;
    }
};

struct VResizeLinearVec_32f16s
{
    int operator()(const float** src, short* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1];
        int x = 0;

        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]);

        if( (((size_t)S0|(size_t)S1)&(VTraits<v_uint8>::vlanes() - 1)) == 0 )
            for( ; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
                v_store(dst + x, v_pack(v_round(v_muladd(vx_load_aligned(S0 + x                    ), b0, v_mul(vx_load_aligned(S1 + x), b1))),
                                        v_round(v_muladd(vx_load_aligned(S0 + x + VTraits<v_float32>::vlanes()), b0, v_mul(vx_load_aligned(S1 + x + VTraits<v_float32>::vlanes()), b1)))));
        else
            for (; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
                v_store(dst + x, v_pack(v_round(v_muladd(vx_load(S0 + x                    ), b0, v_mul(vx_load(S1 + x), b1))),
                                        v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()), b0, v_mul(vx_load(S1 + x + VTraits<v_float32>::vlanes()), b1)))));
        for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
        {
            v_int32 t0 = v_round(v_muladd(vx_load(S0 + x), b0, v_mul(vx_load(S1 + x), b1)));
            v_store_low(dst + x, v_pack(t0, t0));
        }

        return x;
    }
};

struct VResizeLinearVec_32f
{
    int operator()(const float** src, float* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1];
        int x = 0;

        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]);

        if( (((size_t)S0|(size_t)S1)&(VTraits<v_uint8>::vlanes() - 1)) == 0 )
            for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
                v_store(dst + x, v_muladd(vx_load_aligned(S0 + x), b0, v_mul(vx_load_aligned(S1 + x), b1)));
        else
            for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
                v_store(dst + x, v_muladd(vx_load(S0 + x), b0, v_mul(vx_load(S1 + x), b1)));

        return x;
    }
};


struct VResizeCubicVec_32s8u
{
    int operator()(const int** src, uchar* dst, const short* beta, int width) const
    {
        const int *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3];
        int x = 0;
        float scale = 1.f/(INTER_RESIZE_COEF_SCALE*INTER_RESIZE_COEF_SCALE);

        v_float32 b0 = vx_setall_f32(beta[0] * scale), b1 = vx_setall_f32(beta[1] * scale),
                  b2 = vx_setall_f32(beta[2] * scale), b3 = vx_setall_f32(beta[3] * scale);

        if( (((size_t)S0|(size_t)S1|(size_t)S2|(size_t)S3)&(VTraits<v_uint8>::vlanes() - 1)) == 0 )
            for( ; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
                v_pack_u_store(dst + x, v_pack(v_round(v_muladd(v_cvt_f32(vx_load_aligned(S0 + x                    )),  b0,
                                                       v_muladd(v_cvt_f32(vx_load_aligned(S1 + x                    )),  b1,
                                                       v_muladd(v_cvt_f32(vx_load_aligned(S2 + x                    )),  b2,
                                                                v_mul(v_cvt_f32(vx_load_aligned(S3 + x)), b3))))),
                                               v_round(v_muladd(v_cvt_f32(vx_load_aligned(S0 + x + VTraits<v_float32>::vlanes())),  b0,
                                                       v_muladd(v_cvt_f32(vx_load_aligned(S1 + x + VTraits<v_float32>::vlanes())),  b1,
                                                       v_muladd(v_cvt_f32(vx_load_aligned(S2 + x + VTraits<v_float32>::vlanes())),  b2,
                                                                v_mul(v_cvt_f32(vx_load_aligned(S3 + x + VTraits<v_float32>::vlanes())), b3)))))));
        else
            for( ; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
                v_pack_u_store(dst + x, v_pack(v_round(v_muladd(v_cvt_f32(vx_load(S0 + x                    )),  b0,
                                                       v_muladd(v_cvt_f32(vx_load(S1 + x                    )),  b1,
                                                       v_muladd(v_cvt_f32(vx_load(S2 + x                    )),  b2,
                                                                v_mul(v_cvt_f32(vx_load(S3 + x)), b3))))),
                                               v_round(v_muladd(v_cvt_f32(vx_load(S0 + x + VTraits<v_float32>::vlanes())),  b0,
                                                       v_muladd(v_cvt_f32(vx_load(S1 + x + VTraits<v_float32>::vlanes())),  b1,
                                                       v_muladd(v_cvt_f32(vx_load(S2 + x + VTraits<v_float32>::vlanes())),  b2,
                                                                v_mul(v_cvt_f32(vx_load(S3 + x + VTraits<v_float32>::vlanes())), b3)))))));
        return x;
    }
};

struct VResizeCubicVec_32f16u
{
    int operator()(const float** src, ushort* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3];
        int x = 0;
        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]);

        for (; x <= width - VTraits<v_uint16>::vlanes(); x += VTraits<v_uint16>::vlanes())
            v_store(dst + x, v_pack_u(v_round(v_muladd(vx_load(S0 + x                    ),  b0,
                                              v_muladd(vx_load(S1 + x                    ),  b1,
                                              v_muladd(vx_load(S2 + x                    ),  b2,
                                                       v_mul(vx_load(S3 + x), b3))))),
                                      v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()),  b0,
                                              v_muladd(vx_load(S1 + x + VTraits<v_float32>::vlanes()),  b1,
                                              v_muladd(vx_load(S2 + x + VTraits<v_float32>::vlanes()),  b2,
                                                       v_mul(vx_load(S3 + x + VTraits<v_float32>::vlanes()), b3)))))));

        return x;
    }
};

struct VResizeCubicVec_32f16s
{
    int operator()(const float** src, short* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3];
        int x = 0;
        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]);

        for (; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
            v_store(dst + x, v_pack(v_round(v_muladd(vx_load(S0 + x                    ),  b0,
                                            v_muladd(vx_load(S1 + x                    ),  b1,
                                            v_muladd(vx_load(S2 + x                    ),  b2,
                                                     v_mul(vx_load(S3 + x), b3))))),
                                    v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()),  b0,
                                            v_muladd(vx_load(S1 + x + VTraits<v_float32>::vlanes()),  b1,
                                            v_muladd(vx_load(S2 + x + VTraits<v_float32>::vlanes()),  b2,
                                                     v_mul(vx_load(S3 + x + VTraits<v_float32>::vlanes()), b3)))))));

        return x;
    }
};

struct VResizeCubicVec_32f
{
    int operator()(const float** src, float* dst, const float* beta, int width) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3];
        int x = 0;
        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]);

        for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
            v_store(dst + x, v_muladd(vx_load(S0 + x),  b0,
                             v_muladd(vx_load(S1 + x),  b1,
                             v_muladd(vx_load(S2 + x),  b2,
                                      v_mul(vx_load(S3 + x), b3)))));

        return x;
    }
};


#if CV_TRY_SSE4_1

struct VResizeLanczos4Vec_32f16u
{
    int operator()(const float** src, ushort* dst, const float* beta, int width) const
    {
        if (CV_CPU_HAS_SUPPORT_SSE4_1)
            return opt_SSE4_1::VResizeLanczos4Vec_32f16u_SSE41(src, dst, beta, width);
        else
            return 0;
    }
};

#else

struct VResizeLanczos4Vec_32f16u
{
    int operator()(const float** src, ushort* dst, const float* beta, int width ) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3],
                    *S4 = src[4], *S5 = src[5], *S6 = src[6], *S7 = src[7];
        int x = 0;
        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]),
                  b4 = vx_setall_f32(beta[4]), b5 = vx_setall_f32(beta[5]),
                  b6 = vx_setall_f32(beta[6]), b7 = vx_setall_f32(beta[7]);

        for( ; x <= width - VTraits<v_uint16>::vlanes(); x += VTraits<v_uint16>::vlanes())
            v_store(dst + x, v_pack_u(v_round(v_muladd(vx_load(S0 + x                    ),  b0,
                                              v_muladd(vx_load(S1 + x                    ),  b1,
                                              v_muladd(vx_load(S2 + x                    ),  b2,
                                              v_muladd(vx_load(S3 + x                    ),  b3,
                                              v_muladd(vx_load(S4 + x                    ),  b4,
                                              v_muladd(vx_load(S5 + x                    ),  b5,
                                              v_muladd(vx_load(S6 + x                    ),  b6,
                                                       v_mul(vx_load(S7 + x                    ), b7))))))))),
                                      v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()),  b0,
                                              v_muladd(vx_load(S1 + x + VTraits<v_float32>::vlanes()),  b1,
                                              v_muladd(vx_load(S2 + x + VTraits<v_float32>::vlanes()),  b2,
                                              v_muladd(vx_load(S3 + x + VTraits<v_float32>::vlanes()),  b3,
                                              v_muladd(vx_load(S4 + x + VTraits<v_float32>::vlanes()),  b4,
                                              v_muladd(vx_load(S5 + x + VTraits<v_float32>::vlanes()),  b5,
                                              v_muladd(vx_load(S6 + x + VTraits<v_float32>::vlanes()),  b6,
                                                       v_mul(vx_load(S7 + x + VTraits<v_float32>::vlanes()), b7)))))))))));

        return x;
    }
};

#endif

struct VResizeLanczos4Vec_32f16s
{
    int operator()(const float** src, short* dst, const float* beta, int width ) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3],
                    *S4 = src[4], *S5 = src[5], *S6 = src[6], *S7 = src[7];
        int x = 0;
        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]),
                  b4 = vx_setall_f32(beta[4]), b5 = vx_setall_f32(beta[5]),
                  b6 = vx_setall_f32(beta[6]), b7 = vx_setall_f32(beta[7]);

        for( ; x <= width - VTraits<v_int16>::vlanes(); x += VTraits<v_int16>::vlanes())
            v_store(dst + x, v_pack(v_round(v_muladd(vx_load(S0 + x                    ),  b0,
                                            v_muladd(vx_load(S1 + x                    ),  b1,
                                            v_muladd(vx_load(S2 + x                    ),  b2,
                                            v_muladd(vx_load(S3 + x                    ),  b3,
                                            v_muladd(vx_load(S4 + x                    ),  b4,
                                            v_muladd(vx_load(S5 + x                    ),  b5,
                                            v_muladd(vx_load(S6 + x                    ),  b6,
                                                     v_mul(vx_load(S7 + x), b7))))))))),
                                    v_round(v_muladd(vx_load(S0 + x + VTraits<v_float32>::vlanes()),  b0,
                                            v_muladd(vx_load(S1 + x + VTraits<v_float32>::vlanes()),  b1,
                                            v_muladd(vx_load(S2 + x + VTraits<v_float32>::vlanes()),  b2,
                                            v_muladd(vx_load(S3 + x + VTraits<v_float32>::vlanes()),  b3,
                                            v_muladd(vx_load(S4 + x + VTraits<v_float32>::vlanes()),  b4,
                                            v_muladd(vx_load(S5 + x + VTraits<v_float32>::vlanes()),  b5,
                                            v_muladd(vx_load(S6 + x + VTraits<v_float32>::vlanes()),  b6,
                                                     v_mul(vx_load(S7 + x + VTraits<v_float32>::vlanes()), b7)))))))))));

        return x;
    }
};

struct VResizeLanczos4Vec_32f
{
    int operator()(const float** src, float* dst, const float* beta, int width ) const
    {
        const float *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3],
                    *S4 = src[4], *S5 = src[5], *S6 = src[6], *S7 = src[7];
        int x = 0;

        v_float32 b0 = vx_setall_f32(beta[0]), b1 = vx_setall_f32(beta[1]),
                  b2 = vx_setall_f32(beta[2]), b3 = vx_setall_f32(beta[3]),
                  b4 = vx_setall_f32(beta[4]), b5 = vx_setall_f32(beta[5]),
                  b6 = vx_setall_f32(beta[6]), b7 = vx_setall_f32(beta[7]);

        for( ; x <= width - VTraits<v_float32>::vlanes(); x += VTraits<v_float32>::vlanes())
            v_store(dst + x, v_muladd(vx_load(S0 + x),  b0,
                             v_muladd(vx_load(S1 + x),  b1,
                             v_muladd(vx_load(S2 + x),  b2,
                             v_muladd(vx_load(S3 + x),  b3,
                             v_muladd(vx_load(S4 + x),  b4,
                             v_muladd(vx_load(S5 + x),  b5,
                             v_muladd(vx_load(S6 + x),  b6,
                                      v_mul(vx_load(S7 + x), b7)))))))));

        return x;
    }
};

#else

typedef VResizeNoVec VResizeLinearVec_32s8u;
typedef VResizeNoVec VResizeLinearVec_32f16u;
typedef VResizeNoVec VResizeLinearVec_32f16s;
typedef VResizeNoVec VResizeLinearVec_32f;

typedef VResizeNoVec VResizeCubicVec_32s8u;
typedef VResizeNoVec VResizeCubicVec_32f16u;
typedef VResizeNoVec VResizeCubicVec_32f16s;
typedef VResizeNoVec VResizeCubicVec_32f;

typedef VResizeNoVec VResizeLanczos4Vec_32f16u;
typedef VResizeNoVec VResizeLanczos4Vec_32f16s;
typedef VResizeNoVec VResizeLanczos4Vec_32f;

#endif

#if CV_SIMD128

template<typename ST, typename DT, typename AT, typename DVT>
struct HResizeLinearVec_X4
{
    int operator()(const ST** src, DT** dst, int count, const int* xofs,
        const AT* alpha, int, int, int cn, int, int xmax) const
    {
        const int nlanes = 4;
        const int len0 = xmax & -nlanes;
        int dx = 0, k = 0;

        for( ; k <= (count - 2); k+=2 )
        {
            const ST *S0 = src[k];
            DT *D0 = dst[k];
            const ST *S1 = src[k+1];
            DT *D1 = dst[k+1];

            for( dx = 0; dx < len0; dx += nlanes )
            {
                int sx0 = xofs[dx+0];
                int sx1 = xofs[dx+1];
                int sx2 = xofs[dx+2];
                int sx3 = xofs[dx+3];
                DVT a_even;
                DVT a_odd;

                v_load_deinterleave(&alpha[dx*2], a_even, a_odd);
                DVT s0(S0[sx0], S0[sx1], S0[sx2], S0[sx3]);
                DVT s1(S0[sx0+cn], S0[sx1+cn], S0[sx2+cn], S0[sx3+cn]);
                DVT s0_u(S1[sx0], S1[sx1], S1[sx2], S1[sx3]);
                DVT s1_u(S1[sx0+cn], S1[sx1+cn], S1[sx2+cn], S1[sx3+cn]);
                v_store(&D1[dx], v_add(v_mul(s0_u, a_even), v_mul(s1_u, a_odd)));
                v_store(&D0[dx], v_add(v_mul(s0, a_even), v_mul(s1, a_odd)));
            }
        }
        for( ; k < count; k++ )
        {
            const ST *S = src[k];
            DT *D = dst[k];
            for( dx = 0; dx < len0; dx += nlanes )
            {
                int sx0 = xofs[dx+0];
                int sx1 = xofs[dx+1];
                int sx2 = xofs[dx+2];
                int sx3 = xofs[dx+3];
                DVT a_even;
                DVT a_odd;

                v_load_deinterleave(&alpha[dx*2], a_even, a_odd);
                DVT s0(S[sx0], S[sx1], S[sx2], S[sx3]);
                DVT s1(S[sx0+cn], S[sx1+cn], S[sx2+cn], S[sx3+cn]);
                v_store(&D[dx], v_add(v_mul(s0, a_even), v_mul(s1, a_odd)));
            }
        }
        return dx;
    }
};

struct HResizeLinearVecU8_X4
{
    int operator()(const uchar** src, int** dst, int count, const int* xofs,
        const short* alpha/*[xmax]*/, int /*smax*/, int dmax, int cn, int /*xmin*/, int xmax) const
    {
        int dx = 0, k = 0;

        if(cn == 1)
        {
            const int step = 8;
            const int len0 = xmax & -step;
            for( ; k <= (count - 2); k+=2 )
            {
                const uchar *S0 = src[k];
                int *D0 = dst[k];
                const uchar *S1 = src[k+1];
                int *D1 = dst[k+1];

                for( dx = 0; dx < len0; dx += step )
                {
                    v_int16x8 al = v_load(alpha+dx*2);
                    v_int16x8 ah = v_load(alpha+dx*2+8);
                    v_uint16x8 sl, sh;
                    v_expand(v_lut_pairs(S0, xofs+dx), sl, sh);
                    v_store(&D0[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D0[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                    v_expand(v_lut_pairs(S1, xofs+dx), sl, sh);
                    v_store(&D1[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D1[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                }
            }
            for( ; k < count; k++ )
            {
                const uchar *S = src[k];
                int *D = dst[k];
                for( dx = 0; dx < len0; dx += step )
                {
                    v_int16x8 al = v_load(alpha+dx*2);
                    v_int16x8 ah = v_load(alpha+dx*2+8);
                    v_uint16x8 sl, sh;
                    v_expand(v_lut_pairs(S, xofs+dx), sl, sh);
                    v_store(&D[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                }
            }
        }
        else if(cn == 2)
        {
            const int step = 8;
            const int len0 = xmax & -step;
            for( ; k <= (count - 2); k+=2 )
            {
                const uchar *S0 = src[k];
                int *D0 = dst[k];
                const uchar *S1 = src[k+1];
                int *D1 = dst[k+1];

                for( dx = 0; dx < len0; dx += step )
                {
                    int ofs[4] = { xofs[dx], xofs[dx + 2], xofs[dx + 4], xofs[dx + 6] };
                    v_int16x8 al = v_load(alpha+dx*2);
                    v_int16x8 ah = v_load(alpha+dx*2+8);
                    v_uint16x8 sl, sh;
                    v_expand(v_interleave_pairs(v_lut_quads(S0, ofs)), sl, sh);
                    v_store(&D0[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D0[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                    v_expand(v_interleave_pairs(v_lut_quads(S1, ofs)), sl, sh);
                    v_store(&D1[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D1[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                }
            }
            for( ; k < count; k++ )
            {
                const uchar *S = src[k];
                int *D = dst[k];
                for( dx = 0; dx < len0; dx += step )
                {
                    int ofs[4] = { xofs[dx], xofs[dx + 2], xofs[dx + 4], xofs[dx + 6] };
                    v_int16x8 al = v_load(alpha+dx*2);
                    v_int16x8 ah = v_load(alpha+dx*2+8);
                    v_uint16x8 sl, sh;
                    v_expand(v_interleave_pairs(v_lut_quads(S, ofs)), sl, sh);
                    v_store(&D[dx], v_dotprod(v_reinterpret_as_s16(sl), al));
                    v_store(&D[dx+4], v_dotprod(v_reinterpret_as_s16(sh), ah));
                }
            }
        }
        else if(cn == 3)
        {
            /* Peek at the last x offset to find the maximal s offset.  We know the loop
               will terminate prior to value which may be 1 or more elements prior to the
               final valid offset. xofs[] is constucted to be an array of increasingly
               large offsets (i.e xofs[x] <= xofs[x+1] for x < xmax). */
            int smax = xofs[dmax-cn];

            for( ; k <= (count - 2); k+=2 )
            {
                const uchar *S0 = src[k];
                int *D0 = dst[k];
                const uchar *S1 = src[k+1];
                int *D1 = dst[k+1];

                for( dx = 0; (xofs[dx] + cn) < smax; dx += cn )
                {
                    v_int16x8 a = v_load(alpha+dx*2);
                    v_store(&D0[dx], v_dotprod(v_reinterpret_as_s16(v_or(v_load_expand_q(S0 + xofs[dx]), v_shl<16>(v_load_expand_q(S0 + xofs[dx] + cn)))), a));
                    v_store(&D1[dx], v_dotprod(v_reinterpret_as_s16(v_or(v_load_expand_q(S1 + xofs[dx]), v_shl<16>(v_load_expand_q(S1 + xofs[dx] + cn)))), a));
                }
            }
            for( ; k < count; k++ )
            {
                const uchar *S = src[k];
                int *D = dst[k];
                for( dx = 0; (xofs[dx] + cn) < smax; dx += cn )
                {
                    v_int16x8 a = v_load(alpha+dx*2);
                    v_store(&D[dx], v_dotprod(v_reinterpret_as_s16(v_or(v_load_expand_q(S + xofs[dx]), v_shl<16>(v_load_expand_q(S + xofs[dx] + cn)))), a));
                }
            }
            /* Debug check to ensure truthiness that we never vector the final value. */
            CV_DbgAssert(dx < dmax);
        }
        else if(cn == 4)
        {
            const int step = 4;
            const int len0 = xmax & -step;
            for( ; k <= (count - 2); k+=2 )
            {
                const uchar *S0 = src[k];
                int *D0 = dst[k];
                const uchar *S1 = src[k+1];
                int *D1 = dst[k+1];

                for( dx = 0; dx < len0; dx += step )
                {
                    v_int16x8 a = v_load(alpha+dx*2);
                    v_store(&D0[dx], v_dotprod(v_reinterpret_as_s16(v_interleave_quads(v_load_expand(S0+xofs[dx]))), a));
                    v_store(&D1[dx], v_dotprod(v_reinterpret_as_s16(v_interleave_quads(v_load_expand(S1+xofs[dx]))), a));
                }
            }
            for( ; k < count; k++ )
            {
                const uchar *S = src[k];
                int *D = dst[k];
                for( dx = 0; dx < len0; dx += step )
                {
                    v_int16x8 a = v_load(alpha+dx*2);
                    v_store(&D[dx], v_dotprod(v_reinterpret_as_s16(v_interleave_quads(v_load_expand(S+xofs[dx]))), a));
                }
            }
        }
        else
        {
            return 0;  // images with channels >4 are out of optimization scope
        }
        return dx;
    }
};

typedef HResizeLinearVec_X4<float,float,float,v_float32x4> HResizeLinearVec_32f;
typedef HResizeLinearVec_X4<ushort,float,float,v_float32x4> HResizeLinearVec_16u32f;
typedef HResizeLinearVec_X4<short,float,float,v_float32x4> HResizeLinearVec_16s32f;
typedef HResizeLinearVecU8_X4 HResizeLinearVec_8u32s;

#else

typedef HResizeNoVec HResizeLinearVec_8u32s;
typedef HResizeNoVec HResizeLinearVec_16u32f;
typedef HResizeNoVec HResizeLinearVec_16s32f;
typedef HResizeNoVec HResizeLinearVec_32f;

#endif

typedef HResizeNoVec HResizeLinearVec_64f;


template<typename T, typename WT, typename AT, int ONE, class VecOp>
struct HResizeLinear
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const T** src, WT** dst, int count,
                    const int* xofs, const AT* alpha,
                    int swidth, int dwidth, int cn, int xmin, int xmax ) const
    {
        int dx, k;
        VecOp vecOp;

        int dx0 = vecOp(src, dst, count,
            xofs, alpha, swidth, dwidth, cn, xmin, xmax );

        for( k = 0; k <= count - 2; k+=2 )
        {
            const T *S0 = src[k], *S1 = src[k+1];
            WT *D0 = dst[k], *D1 = dst[k+1];
            for( dx = dx0; dx < xmax; dx++ )
            {
                int sx = xofs[dx];
                WT a0 = alpha[dx*2], a1 = alpha[dx*2+1];
                WT t0 = S0[sx]*a0 + S0[sx + cn]*a1;
                WT t1 = S1[sx]*a0 + S1[sx + cn]*a1;
                D0[dx] = t0; D1[dx] = t1;
            }

            for( ; dx < dwidth; dx++ )
            {
                int sx = xofs[dx];
                D0[dx] = WT(S0[sx]*ONE); D1[dx] = WT(S1[sx]*ONE);
            }
        }

        for( ; k < count; k++ )
        {
            const T *S = src[k];
            WT *D = dst[k];
            for( dx = dx0; dx < xmax; dx++ )
            {
                int sx = xofs[dx];
                D[dx] = S[sx]*alpha[dx*2] + S[sx+cn]*alpha[dx*2+1];
            }

            for( ; dx < dwidth; dx++ )
                D[dx] = WT(S[xofs[dx]]*ONE);
        }
    }
};


template<typename T, typename WT, typename AT, class CastOp, class VecOp>
struct VResizeLinear
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const WT** src, T* dst, const AT* beta, int width ) const
    {
        WT b0 = beta[0], b1 = beta[1];
        const WT *S0 = src[0], *S1 = src[1];
        CastOp castOp;
        VecOp vecOp;

        int x = vecOp(src, dst, beta, width);
        #if CV_ENABLE_UNROLLED
        for( ; x <= width - 4; x += 4 )
        {
            WT t0, t1;
            t0 = S0[x]*b0 + S1[x]*b1;
            t1 = S0[x+1]*b0 + S1[x+1]*b1;
            dst[x] = castOp(t0); dst[x+1] = castOp(t1);
            t0 = S0[x+2]*b0 + S1[x+2]*b1;
            t1 = S0[x+3]*b0 + S1[x+3]*b1;
            dst[x+2] = castOp(t0); dst[x+3] = castOp(t1);
        }
        #endif
        for( ; x < width; x++ )
            dst[x] = castOp(S0[x]*b0 + S1[x]*b1);
    }
};

template<>
struct VResizeLinear<uchar, int, short, FixedPtCast<int, uchar, INTER_RESIZE_COEF_BITS*2>, VResizeLinearVec_32s8u>
{
    typedef uchar value_type;
    typedef int buf_type;
    typedef short alpha_type;

    void operator()(const buf_type** src, value_type* dst, const alpha_type* beta, int width ) const
    {
        alpha_type b0 = beta[0], b1 = beta[1];
        const buf_type *S0 = src[0], *S1 = src[1];
        VResizeLinearVec_32s8u vecOp;

        int x = vecOp(src, dst, beta, width);
        #if CV_ENABLE_UNROLLED
        for( ; x <= width - 4; x += 4 )
        {
            dst[x+0] = uchar(( ((b0 * (S0[x+0] >> 4)) >> 16) + ((b1 * (S1[x+0] >> 4)) >> 16) + 2)>>2);
            dst[x+1] = uchar(( ((b0 * (S0[x+1] >> 4)) >> 16) + ((b1 * (S1[x+1] >> 4)) >> 16) + 2)>>2);
            dst[x+2] = uchar(( ((b0 * (S0[x+2] >> 4)) >> 16) + ((b1 * (S1[x+2] >> 4)) >> 16) + 2)>>2);
            dst[x+3] = uchar(( ((b0 * (S0[x+3] >> 4)) >> 16) + ((b1 * (S1[x+3] >> 4)) >> 16) + 2)>>2);
        }
        #endif
        for( ; x < width; x++ )
            dst[x] = uchar(( ((b0 * (S0[x] >> 4)) >> 16) + ((b1 * (S1[x] >> 4)) >> 16) + 2)>>2);
    }
};


template<typename T, typename WT, typename AT>
struct HResizeCubic
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const T** src, WT** dst, int count,
                    const int* xofs, const AT* alpha,
                    int swidth, int dwidth, int cn, int xmin, int xmax ) const
    {
        for( int k = 0; k < count; k++ )
        {
            const T *S = src[k];
            WT *D = dst[k];
            int dx = 0, limit = xmin;
            for(;;)
            {
                for( ; dx < limit; dx++, alpha += 4 )
                {
                    int j, sx = xofs[dx] - cn;
                    WT v = 0;
                    for( j = 0; j < 4; j++ )
                    {
                        int sxj = sx + j*cn;
                        if( (unsigned)sxj >= (unsigned)swidth )
                        {
                            while( sxj < 0 )
                                sxj += cn;
                            while( sxj >= swidth )
                                sxj -= cn;
                        }
                        v += S[sxj]*alpha[j];
                    }
                    D[dx] = v;
                }
                if( limit == dwidth )
                    break;
                for( ; dx < xmax; dx++, alpha += 4 )
                {
                    int sx = xofs[dx];
                    D[dx] = S[sx-cn]*alpha[0] + S[sx]*alpha[1] +
                        S[sx+cn]*alpha[2] + S[sx+cn*2]*alpha[3];
                }
                limit = dwidth;
            }
            alpha -= dwidth*4;
        }
    }
};


template<typename T, typename WT, typename AT, class CastOp, class VecOp>
struct VResizeCubic
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const WT** src, T* dst, const AT* beta, int width ) const
    {
        WT b0 = beta[0], b1 = beta[1], b2 = beta[2], b3 = beta[3];
        const WT *S0 = src[0], *S1 = src[1], *S2 = src[2], *S3 = src[3];
        CastOp castOp;
        VecOp vecOp;

        int x = vecOp(src, dst, beta, width);
        for( ; x < width; x++ )
            dst[x] = castOp(S0[x]*b0 + S1[x]*b1 + S2[x]*b2 + S3[x]*b3);
    }
};


template<typename T, typename WT, typename AT>
struct HResizeLanczos4
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const T** src, WT** dst, int count,
                    const int* xofs, const AT* alpha,
                    int swidth, int dwidth, int cn, int xmin, int xmax ) const
    {
        for( int k = 0; k < count; k++ )
        {
            const T *S = src[k];
            WT *D = dst[k];
            int dx = 0, limit = xmin;
            for(;;)
            {
                for( ; dx < limit; dx++, alpha += 8 )
                {
                    int j, sx = xofs[dx] - cn*3;
                    WT v = 0;
                    for( j = 0; j < 8; j++ )
                    {
                        int sxj = sx + j*cn;
                        if( (unsigned)sxj >= (unsigned)swidth )
                        {
                            while( sxj < 0 )
                                sxj += cn;
                            while( sxj >= swidth )
                                sxj -= cn;
                        }
                        v += S[sxj]*alpha[j];
                    }
                    D[dx] = v;
                }
                if( limit == dwidth )
                    break;
                for( ; dx < xmax; dx++, alpha += 8 )
                {
                    int sx = xofs[dx];
                    D[dx] = S[sx-cn*3]*alpha[0] + S[sx-cn*2]*alpha[1] +
                        S[sx-cn]*alpha[2] + S[sx]*alpha[3] +
                        S[sx+cn]*alpha[4] + S[sx+cn*2]*alpha[5] +
                        S[sx+cn*3]*alpha[6] + S[sx+cn*4]*alpha[7];
                }
                limit = dwidth;
            }
            alpha -= dwidth*8;
        }
    }
};


template<typename T, typename WT, typename AT, class CastOp, class VecOp>
struct VResizeLanczos4
{
    typedef T value_type;
    typedef WT buf_type;
    typedef AT alpha_type;

    void operator()(const WT** src, T* dst, const AT* beta, int width ) const
    {
        CastOp castOp;
        VecOp vecOp;
        int x = vecOp(src, dst, beta, width);
        #if CV_ENABLE_UNROLLED
        for( ; x <= width - 4; x += 4 )
        {
            WT b = beta[0];
            const WT* S = src[0];
            WT s0 = S[x]*b, s1 = S[x+1]*b, s2 = S[x+2]*b, s3 = S[x+3]*b;

            for( int k = 1; k < 8; k++ )
            {
                b = beta[k]; S = src[k];
                s0 += S[x]*b; s1 += S[x+1]*b;
                s2 += S[x+2]*b; s3 += S[x+3]*b;
            }

            dst[x] = castOp(s0); dst[x+1] = castOp(s1);
            dst[x+2] = castOp(s2); dst[x+3] = castOp(s3);
        }
        #endif
        for( ; x < width; x++ )
        {
            dst[x] = castOp(src[0][x]*beta[0] + src[1][x]*beta[1] +
                src[2][x]*beta[2] + src[3][x]*beta[3] + src[4][x]*beta[4] +
                src[5][x]*beta[5] + src[6][x]*beta[6] + src[7][x]*beta[7]);
        }
    }
};


static inline int clip(int x, int a, int b)
{
    return x >= a ? (x < b ? x : b-1) : a;
}

static const int MAX_ESIZE=16;

template <typename HResize, typename VResize>
class resizeGeneric_Invoker :
    public ParallelLoopBody
{
public:
    typedef typename HResize::value_type T;
    typedef typename HResize::buf_type WT;
    typedef typename HResize::alpha_type AT;

    resizeGeneric_Invoker(const Mat& _src, Mat &_dst, const int *_xofs, const int *_yofs,
        const AT* _alpha, const AT* __beta, const Size& _ssize, const Size &_dsize,
        int _ksize, int _xmin, int _xmax) :
        ParallelLoopBody(), src(_src), dst(_dst), xofs(_xofs), yofs(_yofs),
        alpha(_alpha), _beta(__beta), ssize(_ssize), dsize(_dsize),
        ksize(_ksize), xmin(_xmin), xmax(_xmax)
    {
        CV_Assert(ksize <= MAX_ESIZE);
    }

    virtual void operator() (const Range& range) const CV_OVERRIDE
    {
        int dy, cn = src.channels();
        HResize hresize;
        VResize vresize;

        int bufstep = (int)alignSize(dsize.width, 16);
        AutoBuffer<WT> _buffer(bufstep*ksize);
        const T* srows[MAX_ESIZE]={0};
        WT* rows[MAX_ESIZE]={0};
        int prev_sy[MAX_ESIZE];

        for(int k = 0; k < ksize; k++ )
        {
            prev_sy[k] = -1;
            rows[k] = _buffer.data() + bufstep*k;
        }

        const AT* beta = _beta + ksize * range.start;

        for( dy = range.start; dy < range.end; dy++, beta += ksize )
        {
            int sy0 = yofs[dy], k0=ksize, k1=0, ksize2 = ksize/2;

            for(int k = 0; k < ksize; k++ )
            {
                int sy = clip(sy0 - ksize2 + 1 + k, 0, ssize.height);
                for( k1 = std::max(k1, k); k1 < ksize; k1++ )
                {
                    if( k1 < MAX_ESIZE && sy == prev_sy[k1] ) // if the sy-th row has been computed already, reuse it.
                    {
                        if( k1 > k )
                            memcpy( rows[k], rows[k1], bufstep*sizeof(rows[0][0]) );
                        break;
                    }
                }
                if( k1 == ksize )
                    k0 = std::min(k0, k); // remember the first row that needs to be computed
                srows[k] = src.template ptr<T>(sy);
                prev_sy[k] = sy;
            }

            if( k0 < ksize )
                hresize( (const T**)(srows + k0), (WT**)(rows + k0), ksize - k0, xofs, (const AT*)(alpha),
                        ssize.width, dsize.width, cn, xmin, xmax );
            vresize( (const WT**)rows, (T*)(dst.data + dst.step*dy), beta, dsize.width );
        }
    }

private:
    Mat src;
    Mat dst;
    const int* xofs, *yofs;
    const AT* alpha, *_beta;
    Size ssize, dsize;
    const int ksize, xmin, xmax;

    resizeGeneric_Invoker& operator = (const resizeGeneric_Invoker&);
};

template<class HResize, class VResize>
static void resizeGeneric_( const Mat& src, Mat& dst,
                            const int* xofs, const void* _alpha,
                            const int* yofs, const void* _beta,
                            int xmin, int xmax, int ksize )
{
    typedef typename HResize::alpha_type AT;

    const AT* beta = (const AT*)_beta;
    Size ssize = src.size(), dsize = dst.size();
    int cn = src.channels();
    ssize.width *= cn;
    dsize.width *= cn;
    xmin *= cn;
    xmax *= cn;
    // image resize is a separable operation. In case of not too strong

    Range range(0, dsize.height);
    resizeGeneric_Invoker<HResize, VResize> invoker(src, dst, xofs, yofs, (const AT*)_alpha, beta,
        ssize, dsize, ksize, xmin, xmax);
    parallel_for_(range, invoker, dst.total()/(double)(1<<16));
}

template <typename T, typename WT>
struct ResizeAreaFastNoVec
{
    ResizeAreaFastNoVec(int, int) { }
    ResizeAreaFastNoVec(int, int, int, int) { }
    int operator() (const T*, T*, int) const
    { return 0; }
};

#if CV_NEON

class ResizeAreaFastVec_SIMD_8u
{
public:
    ResizeAreaFastVec_SIMD_8u(int _cn, int _step) :
        cn(_cn), step(_step)
    {
    }

    int operator() (const uchar* S, uchar* D, int w) const
    {
        int dx = 0;
        const uchar* S0 = S, * S1 = S0 + step;

        uint16x8_t v_2 = vdupq_n_u16(2);

        if (cn == 1)
        {
            for ( ; dx <= w - 16; dx += 16, S0 += 32, S1 += 32, D += 16)
            {
                uint8x16x2_t v_row0 = vld2q_u8(S0), v_row1 = vld2q_u8(S1);

                uint16x8_t v_dst0 = vaddl_u8(vget_low_u8(v_row0.val[0]), vget_low_u8(v_row0.val[1]));
                v_dst0 = vaddq_u16(v_dst0, vaddl_u8(vget_low_u8(v_row1.val[0]), vget_low_u8(v_row1.val[1])));
                v_dst0 = vshrq_n_u16(vaddq_u16(v_dst0, v_2), 2);

                uint16x8_t v_dst1 = vaddl_u8(vget_high_u8(v_row0.val[0]), vget_high_u8(v_row0.val[1]));
                v_dst1 = vaddq_u16(v_dst1, vaddl_u8(vget_high_u8(v_row1.val[0]), vget_high_u8(v_row1.val[1])));
                v_dst1 = vshrq_n_u16(vaddq_u16(v_dst1, v_2), 2);

                vst1q_u8(D, vcombine_u8(vmovn_u16(v_dst0), vmovn_u16(v_dst1)));
            }
        }
        else if (cn == 4)
        {
            for ( ; dx <= w - 8; dx += 8, S0 += 16, S1 += 16, D += 8)
            {
                uint8x16_t v_row0 = vld1q_u8(S0), v_row1 = vld1q_u8(S1);

                uint16x8_t v_row00 = vmovl_u8(vget_low_u8(v_row0));
                uint16x8_t v_row01 = vmovl_u8(vget_high_u8(v_row0));
                uint16x8_t v_row10 = vmovl_u8(vget_low_u8(v_row1));
                uint16x8_t v_row11 = vmovl_u8(vget_high_u8(v_row1));

                uint16x4_t v_p0 = vadd_u16(vadd_u16(vget_low_u16(v_row00), vget_high_u16(v_row00)),
                                           vadd_u16(vget_low_u16(v_row10), vget_high_u16(v_row10)));
                uint16x4_t v_p1 = vadd_u16(vadd_u16(vget_low_u16(v_row01), vget_high_u16(v_row01)),
                                           vadd_u16(vget_low_u16(v_row11), vget_high_u16(v_row11)));
                uint16x8_t v_dst = vshrq_n_u16(vaddq_u16(vcombine_u16(v_p0, v_p1), v_2), 2);

                vst1_u8(D, vmovn_u16(v_dst));
            }
        }

        return dx;
    }

private:
    int cn, step;
};

class ResizeAreaFastVec_SIMD_16u
{
public:
    ResizeAreaFastVec_SIMD_16u(int _cn, int _step) :
        cn(_cn), step(_step)
    {
    }

    int operator() (const ushort * S, ushort * D, int w) const
    {
        int dx = 0;
        const ushort * S0 = S, * S1 = (const ushort *)((const uchar *)(S0) + step);

        uint32x4_t v_2 = vdupq_n_u32(2);

        if (cn == 1)
        {
            for ( ; dx <= w - 8; dx += 8, S0 += 16, S1 += 16, D += 8)
            {
                uint16x8x2_t v_row0 = vld2q_u16(S0), v_row1 = vld2q_u16(S1);

                uint32x4_t v_dst0 = vaddl_u16(vget_low_u16(v_row0.val[0]), vget_low_u16(v_row0.val[1]));
                v_dst0 = vaddq_u32(v_dst0, vaddl_u16(vget_low_u16(v_row1.val[0]), vget_low_u16(v_row1.val[1])));
                v_dst0 = vshrq_n_u32(vaddq_u32(v_dst0, v_2), 2);

                uint32x4_t v_dst1 = vaddl_u16(vget_high_u16(v_row0.val[0]), vget_high_u16(v_row0.val[1]));
                v_dst1 = vaddq_u32(v_dst1, vaddl_u16(vget_high_u16(v_row1.val[0]), vget_high_u16(v_row1.val[1])));
                v_dst1 = vshrq_n_u32(vaddq_u32(v_dst1, v_2), 2);

                vst1q_u16(D, vcombine_u16(vmovn_u32(v_dst0), vmovn_u32(v_dst1)));
            }
        }
        else if (cn == 4)
        {
            for ( ; dx <= w - 4; dx += 4, S0 += 8, S1 += 8, D += 4)
            {
                uint16x8_t v_row0 = vld1q_u16(S0), v_row1 = vld1q_u16(S1);
                uint32x4_t v_dst = vaddq_u32(vaddl_u16(vget_low_u16(v_row0), vget_high_u16(v_row0)),
                                             vaddl_u16(vget_low_u16(v_row1), vget_high_u16(v_row1)));
                vst1_u16(D, vmovn_u32(vshrq_n_u32(vaddq_u32(v_dst, v_2), 2)));
            }
        }

        return dx;
    }

private:
    int cn, step;
};

class ResizeAreaFastVec_SIMD_16s
{
public:
    ResizeAreaFastVec_SIMD_16s(int _cn, int _step) :
        cn(_cn), step(_step)
    {
    }

    int operator() (const short * S, short * D, int w) const
    {
        int dx = 0;
        const short * S0 = S, * S1 = (const short *)((const uchar *)(S0) + step);

        int32x4_t v_2 = vdupq_n_s32(2);

        if (cn == 1)
        {
            for ( ; dx <= w - 8; dx += 8, S0 += 16, S1 += 16, D += 8)
            {
                int16x8x2_t v_row0 = vld2q_s16(S0), v_row1 = vld2q_s16(S1);

                int32x4_t v_dst0 = vaddl_s16(vget_low_s16(v_row0.val[0]), vget_low_s16(v_row0.val[1]));
                v_dst0 = vaddq_s32(v_dst0, vaddl_s16(vget_low_s16(v_row1.val[0]), vget_low_s16(v_row1.val[1])));
                v_dst0 = vshrq_n_s32(vaddq_s32(v_dst0, v_2), 2);

                int32x4_t v_dst1 = vaddl_s16(vget_high_s16(v_row0.val[0]), vget_high_s16(v_row0.val[1]));
                v_dst1 = vaddq_s32(v_dst1, vaddl_s16(vget_high_s16(v_row1.val[0]), vget_high_s16(v_row1.val[1])));
                v_dst1 = vshrq_n_s32(vaddq_s32(v_dst1, v_2), 2);

                vst1q_s16(D, vcombine_s16(vmovn_s32(v_dst0), vmovn_s32(v_dst1)));
            }
        }
        else if (cn == 4)
        {
            for ( ; dx <= w - 4; dx += 4, S0 += 8, S1 += 8, D += 4)
            {
                int16x8_t v_row0 = vld1q_s16(S0), v_row1 = vld1q_s16(S1);
                int32x4_t v_dst = vaddq_s32(vaddl_s16(vget_low_s16(v_row0), vget_high_s16(v_row0)),
                                            vaddl_s16(vget_low_s16(v_row1), vget_high_s16(v_row1)));
                vst1_s16(D, vmovn_s32(vshrq_n_s32(vaddq_s32(v_dst, v_2), 2)));
            }
        }

        return dx;
    }

private:
    int cn, step;
};

struct ResizeAreaFastVec_SIMD_32f
{
    ResizeAreaFastVec_SIMD_32f(int _scale_x, int _scale_y, int _cn, int _step) :
        cn(_cn), step(_step)
    {
        fast_mode = _scale_x == 2 && _scale_y == 2 && (cn == 1 || cn == 4);
    }

    int operator() (const float * S, float * D, int w) const
    {
        if (!fast_mode)
            return 0;

        const float * S0 = S, * S1 = (const float *)((const uchar *)(S0) + step);
        int dx = 0;

        float32x4_t v_025 = vdupq_n_f32(0.25f);

        if (cn == 1)
        {
            for ( ; dx <= w - 4; dx += 4, S0 += 8, S1 += 8, D += 4)
            {
                float32x4x2_t v_row0 = vld2q_f32(S0), v_row1 = vld2q_f32(S1);

                float32x4_t v_dst0 = vaddq_f32(v_row0.val[0], v_row0.val[1]);
                float32x4_t v_dst1 = vaddq_f32(v_row1.val[0], v_row1.val[1]);

                vst1q_f32(D, vmulq_f32(vaddq_f32(v_dst0, v_dst1), v_025));
            }
        }
        else if (cn == 4)
        {
            for ( ; dx <= w - 4; dx += 4, S0 += 8, S1 += 8, D += 4)
            {
                float32x4_t v_dst0 = vaddq_f32(vld1q_f32(S0), vld1q_f32(S0 + 4));
                float32x4_t v_dst1 = vaddq_f32(vld1q_f32(S1), vld1q_f32(S1 + 4));

                vst1q_f32(D, vmulq_f32(vaddq_f32(v_dst0, v_dst1), v_025));
            }
        }

        return dx;
    }

private:
    int cn;
    bool fast_mode;
    int step;
};

#elif CV_SIMD

class ResizeAreaFastVec_SIMD_8u
{
public:
    ResizeAreaFastVec_SIMD_8u(int _c
... [Content truncated - file is very large]
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

- **VResizeCubicVec_32s8u**: A class/struct defined in this file
- **HResizeNoVec**: A class/struct defined in this file
- **VResizeLanczos4Vec_32f16u**: A class/struct defined in this file
- **CastOp**: A class/struct defined in this file
- **resizeNN_bitexactInvoker**: A class/struct defined in this file
- **VResizeLinearVec_32f16s**: A class/struct defined in this file
- **VResizeCubicVec_32f16u**: A class/struct defined in this file
- **VResizeLinear**: A class/struct defined in this file
- **VResizeLanczos4Vec_32f**: A class/struct defined in this file
- **HResizeLinearVecU8_X4**: A class/struct defined in this file
- **VResizeCubic**: A class/struct defined in this file
- **HResizeLinearVec_X4**: A class/struct defined in this file
- **fixedtype**: A class/struct defined in this file
- **resizeNNInvoker**: A class/struct defined in this file
- **hline**: A class/struct defined in this file
- **interpolationLinear**: A class/struct defined in this file
- **VResizeNoVec**: A class/struct defined in this file
- **resize_bitExactInvoker**: A class/struct defined in this file
- **VResizeLinearVec_32s8u**: A class/struct defined in this file
- **VResizeLinearVec_32f**: A class/struct defined in this file
- **VResizeCubicVec_32f**: A class/struct defined in this file
- **HResizeLinear**: A class/struct defined in this file
- **FixedPtCast**: A class/struct defined in this file
- **VecOp**: A class/struct defined in this file
- **HResizeLanczos4**: A class/struct defined in this file
- **VResizeLinearVec_32f16u**: A class/struct defined in this file
- **VResizeLanczos4Vec_32f16s**: A class/struct defined in this file
- **Cast**: A class/struct defined in this file
- **VResizeCubicVec_32f16s**: A class/struct defined in this file
- **HResizeCubic**: A class/struct defined in this file

### Functions and Methods

- **ufixedpoint32()**: A function/method defined in this file
- **fixedpoint32()**: A function/method defined in this file
- **HResizeNoVec()**: A function/method defined in this file
- **WT()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **func()**: A function/method defined in this file
- **DT()**: A function/method defined in this file
- **fixedpoint64()**: A function/method defined in this file
- **uchar()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **v_float64()**: A function/method defined in this file
- **T()**: A function/method defined in this file
- **ResizeAreaFastNoVec()**: A function/method defined in this file
- **HResizeLinearVecU8_X4()**: A function/method defined in this file
- **HAVE_IPP()**: A function/method defined in this file
- **HResizeLinearVec_X4()**: A function/method defined in this file
- **OPENCV_EXCLUDE_C_API()**: A function/method defined in this file
- **ufixedpoint16()**: A function/method defined in this file
- **VResizeNoVec()**: A function/method defined in this file
- **AT()**: A function/method defined in this file
- **linear_exact_tab()**: A function/method defined in this file
- **int()**: A function/method defined in this file
- **ufixedpoint64()**: A function/method defined in this file
- **typename()**: A function/method defined in this file
- **FT()**: A function/method defined in this file
- **HAVE_IPP_IW()**: A function/method defined in this file
- **v_float32()**: A function/method defined in this file
- **ST()**: A function/method defined in this file
- **short()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `resize.hpp`
- `fixedpoint.inl.hpp`
- `opencv2/core/openvx/ovx_defs.hpp`
- `opencl_kernels_imgproc.hpp`
- `opencv2/core/hal/intrin.hpp`
- `opencv2/core/utils/buffer_area.private.hpp`
- `opencv2/core/softfloat.hpp`
- `precomp.hpp`
- `hal_replacement.hpp`

**Python Imports:**
- `src`
- `this`


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

