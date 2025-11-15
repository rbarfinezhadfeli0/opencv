# Documentation for `hal/carotene/src/convert_scale.cpp`

## File Metadata

- **Full Path**: `hal/carotene/src/convert_scale.cpp`
- **File Name**: `convert_scale.cpp`
- **File Size**: 126,846 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/carotene/src/convert_scale.cpp](../../../hal/carotene/src/convert_scale.cpp)

## Purpose and Role

This file is located in the `hal/carotene/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * By downloading, copying, installing or using the software you agree to this license.
 * If you do not agree to this license, do not download, install,
 * copy or use the software.
 *
 *
 *                           License Agreement
 *                For Open Source Computer Vision Library
 *                        (3-clause BSD License)
 *
 * Copyright (C) 2012-2015, NVIDIA Corporation, all rights reserved.
 * Third party copyrights are property of their respective owners.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 *   * Redistributions of source code must retain the above copyright notice,
 *     this list of conditions and the following disclaimer.
 *
 *   * Redistributions in binary form must reproduce the above copyright notice,
 *     this list of conditions and the following disclaimer in the documentation
 *     and/or other materials provided with the distribution.
 *
 *   * Neither the names of the copyright holders nor the names of the contributors
 *     may be used to endorse or promote products derived from this software
 *     without specific prior written permission.
 *
 * This software is provided by the copyright holders and contributors "as is" and
 * any express or implied warranties, including, but not limited to, the implied
 * warranties of merchantability and fitness for a particular purpose are disclaimed.
 * In no event shall copyright holders or contributors be liable for any direct,
 * indirect, incidental, special, exemplary, or consequential damages
 * (including, but not limited to, procurement of substitute goods or services;
 * loss of use, data, or profits; or business interruption) however caused
 * and on any theory of liability, whether in contract, strict liability,
 * or tort (including negligence or otherwise) arising in any way out of
 * the use of this software, even if advised of the possibility of such damage.
 */

#include "common.hpp"
#include "vround_helper.hpp"

namespace CAROTENE_NS {

#ifdef CAROTENE_NEON

#define CVTS_FUNC(T1, T2, SIMD_SIZE, CVTINIT, CVTROW)                            \
    void convertScale(const Size2D &_size,                                       \
                      const T1 * srcBase, ptrdiff_t srcStride,                   \
                      T2 * dstBase, ptrdiff_t dstStride,                         \
                      f64 alpha, f64 beta)                                       \
    {                                                                            \
        internal::assertSupportedConfiguration();                                \
        Size2D size(_size);                                                      \
        if (srcStride == dstStride &&                                            \
            srcStride == (ptrdiff_t)(size.width))                                \
        {                                                                        \
            size.width *= size.height;                                           \
            size.height = 1;                                                     \
        }                                                                        \
        const ptrdiff_t sstep = srcStride / sizeof(T1);                          \
        const ptrdiff_t dstep = dstStride / sizeof(T2);                          \
        const size_t w = size.width & ~(SIMD_SIZE-1);                            \
        if (size.width >= SIMD_SIZE)                                             \
        {                                                                        \
            const T1* _src = srcBase;                                            \
            T2* _dst = dstBase;                                                  \
            CVTINIT                                                              \
            for (ptrdiff_t h = size.height; h--; _src += sstep, _dst += dstep )  \
                CVTROW                                                           \
        }                                                                        \
        if(w < size.width)                                                       \
        {                                                                        \
            const T1* _src = srcBase;                                            \
            T2* _dst = dstBase;                                                  \
            for (ptrdiff_t h = size.height; h--; _src += sstep, _dst += dstep )  \
                for(size_t i = w; i < size.width; i++ )                          \
                    _dst[i] = internal::saturate_cast<T2>(_src[i]*alpha + beta); \
        }                                                                        \
    }

#define CVTS_FUNC1(T1, SIMD_SIZE, CVTSINIT, CVTSROW)                             \
    void convertScale(const Size2D &_size,                                       \
                      const T1 * srcBase, ptrdiff_t srcStride,                   \
                      T1 * dstBase, ptrdiff_t dstStride,                         \
                      f64 alpha, f64 beta)                                       \
    {                                                                            \
        internal::assertSupportedConfiguration();                                \
        Size2D size(_size);                                                      \
        if (srcStride == dstStride &&                                            \
            srcStride == (ptrdiff_t)(size.width))                                \
        {                                                                        \
            size.width *= size.height;                                           \
            size.height = 1;                                                     \
        }                                                                        \
        const ptrdiff_t sstep = srcStride / sizeof(T1);                          \
        const ptrdiff_t dstep = dstStride / sizeof(T1);                          \
        const size_t w = size.width & ~(SIMD_SIZE-1);                            \
        if (size.width >= SIMD_SIZE)                                             \
        {                                                                        \
            const T1* _src = srcBase;                                            \
            T1* _dst = dstBase;                                                  \
            CVTSINIT                                                             \
            for (ptrdiff_t h = size.height; h--; _src += sstep, _dst += dstep )  \
                CVTSROW                                                          \
        }                                                                        \
        if(w < size.width)                                                       \
        {                                                                        \
            const T1* _src = srcBase;                                            \
            T1* _dst = dstBase;                                                  \
            for (ptrdiff_t h = size.height; h--; _src += sstep, _dst += dstep )  \
                for(size_t i = w; i < size.width; i++ )                          \
                    _dst[i] = internal::saturate_cast<T1>(_src[i]*alpha + beta); \
        }                                                                        \
    }

#else

#define CVTS_FUNC(T1, T2, SIMD_SIZE, CVTINIT, CVTROW)                            \
    void convertScale(const Size2D &,                                            \
                      const T1 *, ptrdiff_t,                                     \
                      T2 *, ptrdiff_t,                                           \
                      f64, f64)                                                  \
    {                                                                            \
        internal::assertSupportedConfiguration();                                \
    }

#define CVTS_FUNC1(T1, SIMD_SIZE, CVTSINIT, CVTSROW)                             \
    void convertScale(const Size2D &,                                            \
                      const T1 *, ptrdiff_t,                                     \
                      T1 *, ptrdiff_t,                                           \
                      f64, f64)                                                  \
    {                                                                            \
        internal::assertSupportedConfiguration();                                \
    }

#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC1(u8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovun.s32 d22, q7                                   \n\t"
            "vqmovun.s32 d23, q8                                   \n\t"
            "vqmovun.s32 d24, q9                                   \n\t"
            "vqmovun.s32 d25, q10                                  \n\t"
            "vqmovn.u16 d26, q11                                   \n\t"
            "vqmovn.u16 d27, q12                                   \n\t"
            "vst1.8 {d26-d27}, [%[dst1]]                           \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC1(u8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int32x4_t vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        int32x4_t vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        uint16x8_t vRes1_u16 = vcombine_u16(vqmovun_s32(vline1_s32), vqmovun_s32(vline2_s32));
        uint16x8_t vRes2_u16 = vcombine_u16(vqmovun_s32(vline3_s32), vqmovun_s32(vline4_s32));
        vst1q_u8(_dst + i, vcombine_u8(vqmovn_u16(vRes1_u16), vqmovn_u16(vRes2_u16)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(u8, s8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovn.s32 d22, q7                                    \n\t"
            "vqmovn.s32 d23, q8                                    \n\t"
            "vqmovn.s32 d24, q9                                    \n\t"
            "vqmovn.s32 d25, q10                                   \n\t"
            "vqmovn.s16 d26, q11                                   \n\t"
            "vqmovn.s16 d27, q12                                   \n\t"
            "vst1.8 {d26-d27}, [%[dst1]]                           \n\t"
            : //no output
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(u8, s8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int32x4_t vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        int32x4_t vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        int16x8_t vRes1_u16 = vcombine_s16(vqmovn_s32(vline1_s32), vqmovn_s32(vline2_s32));
        int16x8_t vRes2_u16 = vcombine_s16(vqmovn_s32(vline3_s32), vqmovn_s32(vline4_s32));
        vst1q_s8(_dst + i, vcombine_s8(vqmovn_s16(vRes1_u16), vqmovn_s16(vRes2_u16)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(u8, u16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovun.s32 d22, q7                                   \n\t"
            "vqmovun.s32 d23, q8                                   \n\t"
            "vqmovun.s32 d24, q9                                   \n\t"
            "vqmovun.s32 d25, q10                                  \n\t"
            "vst1.16 {d22-d23}, [%[dst1]]                          \n\t"
            "vst1.16 {d24-d25}, [%[dst2]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 8),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(u8, u16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int32x4_t vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        int32x4_t vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        vst1q_u16(_dst + i + 0, vcombine_u16(vqmovun_s32(vline1_s32), vqmovun_s32(vline2_s32)));
        vst1q_u16(_dst + i + 8, vcombine_u16(vqmovun_s32(vline3_s32), vqmovun_s32(vline4_s32)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(u8, s16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovn.s32 d22, q7                                    \n\t"
            "vqmovn.s32 d23, q8                                    \n\t"
            "vqmovn.s32 d24, q9                                    \n\t"
            "vqmovn.s32 d25, q10                                   \n\t"
            "vst1.16 {d22-d23}, [%[dst1]]                          \n\t"
            "vst1.16 {d24-d25}, [%[dst2]]                          \n\t"
            : //no output
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 8),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(u8, s16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int32x4_t vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        int32x4_t vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        vst1q_s16(_dst + i + 0, vcombine_s16(vqmovn_s32(vline1_s32), vqmovn_s32(vline2_s32)));
        vst1q_s16(_dst + i + 8, vcombine_s16(vqmovn_s32(vline3_s32), vqmovn_s32(vline4_s32)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u8, s32, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vst1.32 {d14-d15}, [%[dst1]]                          \n\t"
            "vst1.32 {d16-d17}, [%[dst2]]                          \n\t"
            "vst1.32 {d18-d19}, [%[dst3]]                          \n\t"
            "vst1.32 {d20-d21}, [%[dst4]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              [dst3] "r" (_dst + i + 8),
              [dst4] "r" (_dst + i + 12),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10",
            "d11","d12","d13","d14","d15","d16","d17",
            "d18","d19","d20","d21","d22","d23","d24",
            "d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(u8, s32, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int32x4_t vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        int32x4_t vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        vst1q_s32(_dst + i + 0,  vline1_s32);
        vst1q_s32(_dst + i + 4,  vline2_s32);
        vst1q_s32(_dst + i + 8,  vline3_s32);
        vst1q_s32(_dst + i + 12, vline4_s32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u8, f32, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u8 q3, d4                                       \n\t"
            "vmovl.u8 q4, d5                                       \n\t"
            "vmovl.u16 q5, d6                                      \n\t"
            "vmovl.u16 q6, d7                                      \n\t"
            "vmovl.u16 q7, d8                                      \n\t"
            "vmovl.u16 q8, d9                                      \n\t"
            "vcvt.f32.u32 q9, q5                                   \n\t"
            "vcvt.f32.u32 q10, q6                                  \n\t"
            "vcvt.f32.u32 q11, q7                                  \n\t"
            "vcvt.f32.u32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vst1.32 {d6-d7}, [%[dst1]]                            \n\t"
            "vst1.32 {d8-d9}, [%[dst2]]                            \n\t"
            "vst1.32 {d10-d11}, [%[dst3]]                          \n\t"
            "vst1.32 {d12-d13}, [%[dst4]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              [dst3] "r" (_dst + i + 8),
              [dst4] "r" (_dst + i + 12),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10",
            "d11","d12","d13","d14","d15","d16","d17",
            "d18","d19","d20","d21","d22","d23","d24",
            "d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(u8, f32, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        uint8x16_t vline = vld1q_u8(_src + i);
        uint16x8_t vline1_u16 = vmovl_u8(vget_low_u8 (vline));
        uint16x8_t vline2_u16 = vmovl_u8(vget_high_u8(vline));
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline1_u16));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline1_u16));
        uint32x4_t vline3_u32 = vmovl_u16(vget_low_u16 (vline2_u16));
        uint32x4_t vline4_u32 = vmovl_u16(vget_high_u16(vline2_u16));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        float32x4_t vline3_f32 = vcvtq_f32_u32(vline3_u32);
        float32x4_t vline4_f32 = vcvtq_f32_u32(vline4_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vst1q_f32(_dst + i + 0,  vline1_f32);
        vst1q_f32(_dst + i + 4,  vline2_f32);
        vst1q_f32(_dst + i + 8,  vline3_f32);
        vst1q_f32(_dst + i + 12, vline4_f32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(s8, u8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovun.s32 d22, q7                                   \n\t"
            "vqmovun.s32 d23, q8                                   \n\t"
            "vqmovun.s32 d24, q9                                   \n\t"
            "vqmovun.s32 d25, q10                                  \n\t"
            "vqmovn.u16 d26, q11                                   \n\t"
            "vqmovn.u16 d27, q12                                   \n\t"
            "vst1.8 {d26-d27}, [%[dst1]]                           \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(s8, u8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        uint16x8_t vRes1_u16 = vcombine_u16(vqmovun_s32(vline1_s32), vqmovun_s32(vline2_s32));
        uint16x8_t vRes2_u16 = vcombine_u16(vqmovun_s32(vline3_s32), vqmovun_s32(vline4_s32));
        vst1q_u8(_dst + i, vcombine_u8(vqmovn_u16(vRes1_u16), vqmovn_u16(vRes2_u16)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC1(s8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovn.s32 d22, q7                                    \n\t"
            "vqmovn.s32 d23, q8                                    \n\t"
            "vqmovn.s32 d24, q9                                    \n\t"
            "vqmovn.s32 d25, q10                                   \n\t"
            "vqmovn.s16 d26, q11                                   \n\t"
            "vqmovn.s16 d27, q12                                   \n\t"
            "vst1.8 {d26-d27}, [%[dst1]]                           \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC1(s8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        int16x8_t vRes1_s16 = vcombine_s16(vqmovn_s32(vline1_s32), vqmovn_s32(vline2_s32));
        int16x8_t vRes2_s16 = vcombine_s16(vqmovn_s32(vline3_s32), vqmovn_s32(vline4_s32));
        vst1q_s8(_dst + i, vcombine_s8(vqmovn_s16(vRes1_s16), vqmovn_s16(vRes2_s16)));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(s8, u16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovun.s32 d22, q7                                   \n\t"
            "vqmovun.s32 d23, q8                                   \n\t"
            "vqmovun.s32 d24, q9                                   \n\t"
            "vqmovun.s32 d25, q10                                  \n\t"
            "vst1.16 {d22-d23}, [%[dst1]]                          \n\t"
            "vst1.16 {d24-d25}, [%[dst2]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 8),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(s8, u16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        uint16x8_t vRes1_u16 = vcombine_u16(vqmovun_s32(vline1_s32), vqmovun_s32(vline2_s32));
        uint16x8_t vRes2_u16 = vcombine_u16(vqmovun_s32(vline3_s32), vqmovun_s32(vline4_s32));
        vst1q_u16(_dst + i + 0, vRes1_u16);
        vst1q_u16(_dst + i + 8, vRes2_u16);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && defined(__arm__)
CVTS_FUNC(s8, s16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vqmovn.s32 d22, q7                                    \n\t"
            "vqmovn.s32 d23, q8                                    \n\t"
            "vqmovn.s32 d24, q9                                    \n\t"
            "vqmovn.s32 d25, q10                                   \n\t"
            "vst1.16 {d22-d23}, [%[dst1]]                          \n\t"
            "vst1.16 {d24-d25}, [%[dst2]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 8),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(s8, s16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        int16x8_t vRes1_s16 = vcombine_s16(vqmovn_s32(vline1_s32), vqmovn_s32(vline2_s32));
        int16x8_t vRes2_s16 = vcombine_s16(vqmovn_s32(vline3_s32), vqmovn_s32(vline4_s32));
        vst1q_s16(_dst + i + 0, vRes1_s16);
        vst1q_s16(_dst + i + 8, vRes2_s16);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s8, s32, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vcvt.s32.f32 q7, q3                                   \n\t"
            "vcvt.s32.f32 q8, q4                                   \n\t"
            "vcvt.s32.f32 q9, q5                                   \n\t"
            "vcvt.s32.f32 q10, q6                                  \n\t"
            "vst1.32 {d14-d15}, [%[dst1]]                          \n\t"
            "vst1.32 {d16-d17}, [%[dst2]]                          \n\t"
            "vst1.32 {d18-d19}, [%[dst3]]                          \n\t"
            "vst1.32 {d20-d21}, [%[dst4]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              [dst3] "r" (_dst + i + 8),
              [dst4] "r" (_dst + i + 12),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10",
            "d11","d12","d13","d14","d15","d16","d17",
            "d18","d19","d20","d21","d22","d23","d24",
            "d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(s8, s32, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vline3_s32 = internal::vroundq_s32_f32(vline3_f32);
        vline4_s32 = internal::vroundq_s32_f32(vline4_f32);
        vst1q_s32(_dst + i + 0,  vline1_s32);
        vst1q_s32(_dst + i + 4,  vline2_s32);
        vst1q_s32(_dst + i + 8,  vline3_s32);
        vst1q_s32(_dst + i + 12, vline4_s32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s8, f32, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s8 q3, d4                                       \n\t"
            "vmovl.s8 q4, d5                                       \n\t"
            "vmovl.s16 q5, d6                                      \n\t"
            "vmovl.s16 q6, d7                                      \n\t"
            "vmovl.s16 q7, d8                                      \n\t"
            "vmovl.s16 q8, d9                                      \n\t"
            "vcvt.f32.s32 q9, q5                                   \n\t"
            "vcvt.f32.s32 q10, q6                                  \n\t"
            "vcvt.f32.s32 q11, q7                                  \n\t"
            "vcvt.f32.s32 q12, q8                                  \n\t"
            "vmul.f32 q13, q9, q0                                  \n\t"
            "vmul.f32 q14, q10, q0                                 \n\t"
            "vmul.f32 q15, q11, q0                                 \n\t"
            "vmul.f32 q2, q12, q0                                  \n\t"
            "vadd.f32 q3, q13, q1                                  \n\t"
            "vadd.f32 q4, q14, q1                                  \n\t"
            "vadd.f32 q5, q15, q1                                  \n\t"
            "vadd.f32 q6, q2, q1                                   \n\t"
            "vst1.32 {d6-d7}, [%[dst1]]                            \n\t"
            "vst1.32 {d8-d9}, [%[dst2]]                            \n\t"
            "vst1.32 {d10-d11}, [%[dst3]]                          \n\t"
            "vst1.32 {d12-d13}, [%[dst4]]                          \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              [dst3] "r" (_dst + i + 8),
              [dst4] "r" (_dst + i + 12),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10",
            "d11","d12","d13","d14","d15","d16","d17",
            "d18","d19","d20","d21","d22","d23","d24",
            "d25","d26","d27","d28","d29","d30","d31"
        );
    }
})
#else
CVTS_FUNC(s8, f32, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 16)
    {
        internal::prefetch(_src + i);
        int8x16_t vline = vld1q_s8(_src + i);
        int16x8_t vline1_s16 = vmovl_s8(vget_low_s8 (vline));
        int16x8_t vline2_s16 = vmovl_s8(vget_high_s8(vline));
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline1_s16));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline1_s16));
        int32x4_t vline3_s32 = vmovl_s16(vget_low_s16 (vline2_s16));
        int32x4_t vline4_s32 = vmovl_s16(vget_high_s16(vline2_s16));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        float32x4_t vline3_f32 = vcvtq_f32_s32(vline3_s32);
        float32x4_t vline4_f32 = vcvtq_f32_s32(vline4_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline3_f32 = vmulq_f32(vline3_f32, vscale);
        vline4_f32 = vmulq_f32(vline4_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline3_f32 = vaddq_f32(vline3_f32, vshift);
        vline4_f32 = vaddq_f32(vline4_f32, vshift);
        vst1q_f32(_dst + i + 0,  vline1_f32);
        vst1q_f32(_dst + i + 4,  vline2_f32);
        vst1q_f32(_dst + i + 8,  vline3_f32);
        vst1q_f32(_dst + i + 12, vline4_f32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u16, u8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src1]]                             \n\t"
            "vmovl.u16 q3, d4                                      \n\t"
            "vmovl.u16 q4, d5                                      \n\t"
            "vcvt.f32.u32 q5, q3                                   \n\t"
            "vcvt.f32.u32 q6, q4                                   \n\t"
            "vmul.f32 q7, q5, q0                                   \n\t"
            "vmul.f32 q8, q6, q0                                   \n\t"
            "vadd.f32 q9, q7, q1                                   \n\t"
            "vadd.f32 q10, q8, q1                                  \n\t"
            "vcvt.s32.f32 q11, q9                                  \n\t"
            "vcvt.s32.f32 q12, q10                                 \n\t"
            "vqmovn.s32 d26, q11                                   \n\t"
            "vqmovn.s32 d27, q12                                   \n\t"
            "vqmovun.s16 d28, q13                                  \n\t"
             "vst1.8 {d28}, [%[dst]]                               \n\t"
            : /*no output*/
            : [src1] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
               "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28"
        );
    }
})
#else
CVTS_FUNC(u16, u8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        uint8x8_t vRes = vqmovun_s16(vcombine_s16(vRes1, vRes2));
        vst1_u8(_dst + i, vRes);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u16, s8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src1]]                             \n\t"
            "vmovl.u16 q3, d4                                      \n\t"
            "vmovl.u16 q4, d5                                      \n\t"
            "vcvt.f32.u32 q5, q3                                   \n\t"
            "vcvt.f32.u32 q6, q4                                   \n\t"
            "vmul.f32 q7, q5, q0                                   \n\t"
            "vmul.f32 q8, q6, q0                                   \n\t"
            "vadd.f32 q9, q7, q1                                   \n\t"
            "vadd.f32 q10, q8, q1                                  \n\t"
            "vcvt.s32.f32 q11, q9                                  \n\t"
            "vcvt.s32.f32 q12, q10                                 \n\t"
            "vqmovn.s32 d26, q11                                   \n\t"
            "vqmovn.s32 d27, q12                                   \n\t"
            "vqmovn.s16 d28, q13                                   \n\t"
            "vst1.8 {d28}, [%[dst]]                                \n\t"
            : /*no output*/
            : [src1] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
               "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28"
        );
    }
})
#else
CVTS_FUNC(u16, s8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        int8x8_t vRes = vqmovn_s16(vcombine_s16(vRes1, vRes2));
        vst1_s8(_dst + i, vRes);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC1(u16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u16 q3, d4                                       \n\t"
            "vmovl.u16 q4, d5                                       \n\t"
            "vcvt.f32.u32 q5, q3                                    \n\t"
            "vcvt.f32.u32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vcvt.s32.f32 q11, q9                                   \n\t"
            "vcvt.s32.f32 q12, q10                                  \n\t"
            "vqmovun.s32 d26, q11                                   \n\t"
            "vqmovun.s32 d27, q12                                   \n\t"
            "vst1.16 {d26-d27}, [%[dst]]                            \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
              "w" (vshift), "w" (vscale)
            : "d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27"
        );
    }
})
#else
CVTS_FUNC1(u16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        uint16x4_t vRes1 = vqmovun_s32(vline1_s32);
        uint16x4_t vRes2 = vqmovun_s32(vline2_s32);
        vst1q_u16(_dst + i, vcombine_u16(vRes1, vRes2));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u16, s16, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u16 q3, d4                                       \n\t"
            "vmovl.u16 q4, d5                                       \n\t"
            "vcvt.f32.u32 q5, q3                                    \n\t"
            "vcvt.f32.u32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vcvt.s32.f32 q11, q9                                   \n\t"
            "vcvt.s32.f32 q12, q10                                  \n\t"
            "vqmovn.s32 d26, q11                                    \n\t"
            "vqmovn.s32 d27, q12                                    \n\t"
            "vst1.16 {d26-d27}, [%[dst]]                            \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
              "w" (vshift), "w" (vscale)
            : "d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27"
        );
    }
})
#else
CVTS_FUNC(u16, s16, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        vst1q_s16(_dst + i, vcombine_s16(vRes1, vRes2));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u16, s32, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                        \n\t"
            "vmovl.u16 q3, d4                                 \n\t"
            "vmovl.u16 q4, d5                                 \n\t"
            "vcvt.f32.u32 q5, q3                              \n\t"
            "vcvt.f32.u32 q6, q4                              \n\t"
            "vmul.f32 q7, q5, q0                              \n\t"
            "vmul.f32 q8, q6, q0                              \n\t"
            "vadd.f32 q9, q7, q1                              \n\t"
            "vadd.f32 q10, q8, q1                             \n\t"
            "vcvt.s32.f32 q11, q9                             \n\t"
            "vcvt.s32.f32 q12, q10                            \n\t"
            "vst1.32 {d22-d23}, [%[dst1]]                     \n\t"
            "vst1.32 {d24-d25}, [%[dst2]]                     \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i),
              [dst2] "r" (_dst + i + 4),
              "w" (vshift), "w" (vscale)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25"
        );
    }
})
#else
CVTS_FUNC(u16, s32, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        int32x4_t vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        int32x4_t vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vst1q_s32(_dst + i + 0, vline1_s32);
        vst1q_s32(_dst + i + 4, vline2_s32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(u16, f32, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.u16 q3, d4                                       \n\t"
            "vmovl.u16 q4, d5                                       \n\t"
             "vcvt.f32.u32 q5, q3                                    \n\t"
            "vcvt.f32.u32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vst1.32 {d18-d19}, [%[dst1]]                           \n\t"
            "vst1.32 {d20-d21}, [%[dst2]]                           \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21"
        );
    }
})
#else
CVTS_FUNC(u16, f32, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        uint16x8_t vline = vld1q_u16(_src + i);
        uint32x4_t vline1_u32 = vmovl_u16(vget_low_u16 (vline));
        uint32x4_t vline2_u32 = vmovl_u16(vget_high_u16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_u32(vline1_u32);
        float32x4_t vline2_f32 = vcvtq_f32_u32(vline2_u32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vst1q_f32(_dst + i + 0, vline1_f32);
        vst1q_f32(_dst + i + 4, vline2_f32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s16, u8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src1]]                             \n\t"
            "vmovl.s16 q3, d4                                      \n\t"
            "vmovl.s16 q4, d5                                      \n\t"
            "vcvt.f32.s32 q5, q3                                   \n\t"
            "vcvt.f32.s32 q6, q4                                   \n\t"
            "vmul.f32 q7, q5, q0                                   \n\t"
            "vmul.f32 q8, q6, q0                                   \n\t"
            "vadd.f32 q9, q7, q1                                   \n\t"
            "vadd.f32 q10, q8, q1                                  \n\t"
            "vcvt.s32.f32 q11, q9                                  \n\t"
            "vcvt.s32.f32 q12, q10                                 \n\t"
            "vqmovn.s32 d26, q11                                   \n\t"
            "vqmovn.s32 d27, q12                                   \n\t"
            "vqmovun.s16 d28, q13                                  \n\t"
            "vst1.8 {d28}, [%[dst]]                                \n\t"
            : /*no output*/
            : [src1] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
               "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28"
        );
    }
})
#else
CVTS_FUNC(s16, u8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        uint8x8_t vRes = vqmovun_s16(vcombine_s16(vRes1, vRes2));
        vst1_u8(_dst + i, vRes);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s16, s8, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.8 {d4-d5}, [%[src1]]                             \n\t"
            "vmovl.s16 q3, d4                                      \n\t"
            "vmovl.s16 q4, d5                                      \n\t"
            "vcvt.f32.s32 q5, q3                                   \n\t"
            "vcvt.f32.s32 q6, q4                                   \n\t"
            "vmul.f32 q7, q5, q0                                   \n\t"
            "vmul.f32 q8, q6, q0                                   \n\t"
            "vadd.f32 q9, q7, q1                                   \n\t"
            "vadd.f32 q10, q8, q1                                  \n\t"
            "vcvt.s32.f32 q11, q9                                  \n\t"
            "vcvt.s32.f32 q12, q10                                 \n\t"
            "vqmovn.s32 d26, q11                                   \n\t"
            "vqmovn.s32 d27, q12                                   \n\t"
            "vqmovn.s16 d28, q13                                   \n\t"
            "vst1.8 {d28}, [%[dst]]                                \n\t"
            : /*no output*/
            : [src1] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
               "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27","d28"
        );
    }
})
#else
CVTS_FUNC(s16, s8, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        int8x8_t vRes = vqmovn_s16(vcombine_s16(vRes1, vRes2));
        vst1_s8(_dst + i, vRes);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s16, u16, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s16 q3, d4                                       \n\t"
            "vmovl.s16 q4, d5                                       \n\t"
            "vcvt.f32.s32 q5, q3                                    \n\t"
            "vcvt.f32.s32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vcvt.s32.f32 q11, q9                                   \n\t"
            "vcvt.s32.f32 q12, q10                                  \n\t"
            "vqmovun.s32 d26, q11                                   \n\t"
            "vqmovun.s32 d27, q12                                   \n\t"
            "vst1.16 {d26-d27}, [%[dst]]                            \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27"
        );
    }
})
#else
CVTS_FUNC(s16, u16, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        uint16x4_t vRes1 = vqmovun_s32(vline1_s32);
        uint16x4_t vRes2 = vqmovun_s32(vline2_s32);
        vst1q_u16(_dst + i, vcombine_u16(vRes1, vRes2));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC1(s16, 16,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s16 q3, d4                                       \n\t"
            "vmovl.s16 q4, d5                                       \n\t"
            "vcvt.f32.s32 q5, q3                                    \n\t"
            "vcvt.f32.s32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vcvt.s32.f32 q11, q9                                   \n\t"
            "vcvt.s32.f32 q12, q10                                  \n\t"
            "vqmovn.s32 d26, q11                                    \n\t"
            "vqmovn.s32 d27, q12                                    \n\t"
            "vst1.16 {d26-d27}, [%[dst]]                            \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst] "r" (_dst + i + 0),
              "w" (vshift), "w" (vscale)
            : "d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26","d27"
        );
    }
})
#else
CVTS_FUNC1(s16, 16,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        int16x4_t vRes1 = vqmovn_s32(vline1_s32);
        int16x4_t vRes2 = vqmovn_s32(vline2_s32);
        vst1q_s16(_dst + i, vcombine_s16(vRes1, vRes2));
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s16, s32, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s16 q3, d4                                       \n\t"
            "vmovl.s16 q4, d5                                       \n\t"
            "vcvt.f32.s32 q5, q3                                    \n\t"
            "vcvt.f32.s32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                    \n\t"
            "vadd.f32 q10, q8, q1                                   \n\t"
            "vcvt.s32.f32 q11, q9                                   \n\t"
            "vcvt.s32.f32 q12, q10                                  \n\t"
            "vst1.32 {d22-d23}, [%[dst1]]                           \n\t"
            "vst1.32 {d24-d25}, [%[dst2]]                           \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25"
        );
    }
})
#else
CVTS_FUNC(s16, s32, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        vst1q_s32(_dst + i + 0, vline1_s32);
        vst1q_s32(_dst + i + 4, vline2_s32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s16, f32, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.16 {d4-d5}, [%[src]]                              \n\t"
            "vmovl.s16 q3, d4                                       \n\t"
            "vmovl.s16 q4, d5                                       \n\t"
            "vcvt.f32.s32 q5, q3                                    \n\t"
            "vcvt.f32.s32 q6, q4                                    \n\t"
            "vmul.f32 q7, q5, q0                                    \n\t"
            "vmul.f32 q8, q6, q0                                    \n\t"
            "vadd.f32 q9, q7, q1                                     \n\t"
            "vadd.f32 q10, q8, q1                                     \n\t"
            "vst1.32 {d18-d19}, [%[dst1]]                             \n\t"
            "vst1.32 {d20-d21}, [%[dst2]]                             \n\t"
            : /*no output*/
            : [src] "r" (_src + i),
              [dst1] "r" (_dst + i + 0),
              [dst2] "r" (_dst + i + 4),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21"
        );
    }
})
#else
CVTS_FUNC(s16, f32, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int16x8_t vline = vld1q_s16(_src + i);
        int32x4_t vline1_s32 = vmovl_s16(vget_low_s16 (vline));
        int32x4_t vline2_s32 = vmovl_s16(vget_high_s16(vline));
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vst1q_f32(_dst + i + 0, vline1_f32);
        vst1q_f32(_dst + i + 4, vline2_f32);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s32, u8, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.32 {d4-d5}, [%[src1]]                              \n\t"
            "vld1.32 {d6-d7}, [%[src2]]                              \n\t"
            "vcvt.f32.s32 q4, q2                                     \n\t"
            "vcvt.f32.s32 q5, q3                                     \n\t"
            "vmul.f32 q6, q4, q0                                     \n\t"
            "vmul.f32 q7, q5, q0                                     \n\t"
            "vadd.f32 q8, q6, q1                                     \n\t"
            "vadd.f32 q9, q7, q1                                     \n\t"
            "vcvt.s32.f32 q10, q8                                    \n\t"
            "vcvt.s32.f32 q11, q9                                    \n\t"
            "vqmovun.s32 d24, q10                                    \n\t"
            "vqmovun.s32 d25, q11                                    \n\t"
            "vqmovn.u16  d26, q12                                    \n\t"
            "vst1.8 {d26}, [%[dst]]                                  \n\t"
            : /*no output*/
            : [src1] "r" (_src + i + 0),
              [src2] "r" (_src + i + 4),
              [dst] "r" (_dst + i),
              "w"  (vscale), "w" (vshift)
            : "d4","d5","d6","d7","d8","d9","d10","d11","d12","d13","d14","d15","d16","d17","d18","d19","d20","d21","d22","d23","d24","d25","d26"
        );
    }
})
#else
CVTS_FUNC(s32, u8, 8,
    float32x4_t vscale = vdupq_n_f32((f32)alpha);
    float32x4_t vshift = vdupq_n_f32((f32)beta);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        int32x4_t vline1_s32 = vld1q_s32(_src + i + 0);
        int32x4_t vline2_s32 = vld1q_s32(_src + i + 4);
        float32x4_t vline1_f32 = vcvtq_f32_s32(vline1_s32);
        float32x4_t vline2_f32 = vcvtq_f32_s32(vline2_s32);
        vline1_f32 = vmulq_f32(vline1_f32, vscale);
        vline2_f32 = vmulq_f32(vline2_f32, vscale);
        vline1_f32 = vaddq_f32(vline1_f32, vshift);
        vline2_f32 = vaddq_f32(vline2_f32, vshift);
        vline1_s32 = internal::vroundq_s32_f32(vline1_f32);
        vline2_s32 = internal::vroundq_s32_f32(vline2_f32);
        uint16x4_t vRes1 = vqmovun_s32(vline1_s32);
        uint16x4_t vRes2 = vqmovun_s32(vline2_s32);
        uint8x8_t vRes = vqmovn_u16(vcombine_u16(vRes1, vRes2));
        vst1_u8(_dst + i, vRes);
    }
})
#endif

#if !defined(__aarch64__) && defined(__GNUC__) && __GNUC__ == 4 &&  __GNUC_MINOR__ < 7 && !defined(__clang__)
CVTS_FUNC(s32, s8, 8,
    register float32x4_t vscale asm ("q0") = vdupq_n_f32((f32)alpha);
    register float32x4_t vshift asm ("q1") = vdupq_n_f32((f32)beta + 0.5f);,
{
    for (size_t i = 0; i < w; i += 8)
    {
        internal::prefetch(_src + i);
        __asm__ (
            "vld1.32 {d4-d5}, [%[src1]]                              \n\t"
            "vld1.32 {d6-d7}, [%[src2]]                              \n\t"
            "vcvt.f32.s32 q4, q2                                     \n\t"
            "vcvt.f32.s32 q5, q3                                     \n\t"
            "vmul.f32 q6, q4, q0                                     \n\t"
            "vmul.f32 q7, q5, q0                                     \n\t"
            "vadd.f32 q8, q6, q1                                     \n\t"
            "vadd.f32 q9, q7, q1                                     \n\t"
            "vcvt.s32.f3
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

### Functions and Methods

- **CAROTENE_NEON()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `common.hpp`
- `vround_helper.hpp`

**Python Imports:**
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

