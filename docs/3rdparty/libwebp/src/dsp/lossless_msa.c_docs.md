# Documentation for `3rdparty/libwebp/src/dsp/lossless_msa.c`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dsp/lossless_msa.c`
- **File Name**: `lossless_msa.c`
- **File Size**: 13,034 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libwebp/src/dsp/lossless_msa.c](../../../../3rdparty/libwebp/src/dsp/lossless_msa.c)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/dsp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Copyright 2016 Google Inc. All Rights Reserved.
//
// Use of this source code is governed by a BSD-style license
// that can be found in the COPYING file in the root of the source
// tree. An additional intellectual property rights grant can be found
// in the file PATENTS. All contributing project authors may
// be found in the AUTHORS file in the root of the source tree.
// -----------------------------------------------------------------------------
//
// MSA variant of methods for lossless decoder
//
// Author: Prashant Patil (prashant.patil@imgtec.com)

#include "src/dsp/dsp.h"

#if defined(WEBP_USE_MSA)

#include "src/dsp/lossless.h"
#include "src/dsp/msa_macro.h"

//------------------------------------------------------------------------------
// Colorspace conversion functions

#define CONVERT16_BGRA_XXX(psrc, pdst, m0, m1, m2) do {    \
  v16u8 src0, src1, src2, src3, dst0, dst1, dst2;          \
  LD_UB4(psrc, 16, src0, src1, src2, src3);                \
  VSHF_B2_UB(src0, src1, src1, src2, m0, m1, dst0, dst1);  \
  dst2 = VSHF_UB(src2, src3, m2);                          \
  ST_UB2(dst0, dst1, pdst, 16);                            \
  ST_UB(dst2, pdst + 32);                                  \
} while (0)

#define CONVERT12_BGRA_XXX(psrc, pdst, m0, m1, m2) do {    \
  uint32_t pix_w;                                          \
  v16u8 src0, src1, src2, dst0, dst1, dst2;                \
  LD_UB3(psrc, 16, src0, src1, src2);                      \
  VSHF_B2_UB(src0, src1, src1, src2, m0, m1, dst0, dst1);  \
  dst2 = VSHF_UB(src2, src2, m2);                          \
  ST_UB2(dst0, dst1, pdst, 16);                            \
  pix_w = __msa_copy_s_w((v4i32)dst2, 0);                  \
  SW(pix_w, pdst + 32);                                    \
} while (0)

#define CONVERT8_BGRA_XXX(psrc, pdst, m0, m1) do {         \
  uint64_t pix_d;                                          \
  v16u8 src0, src1, src2 = { 0 }, dst0, dst1;              \
  LD_UB2(psrc, 16, src0, src1);                            \
  VSHF_B2_UB(src0, src1, src1, src2, m0, m1, dst0, dst1);  \
  ST_UB(dst0, pdst);                                       \
  pix_d = __msa_copy_s_d((v2i64)dst1, 0);                  \
  SD(pix_d, pdst + 16);                                    \
} while (0)

#define CONVERT4_BGRA_XXX(psrc, pdst, m) do {       \
  const v16u8 src0 = LD_UB(psrc);                   \
  const v16u8 dst0 = VSHF_UB(src0, src0, m);        \
  uint64_t pix_d = __msa_copy_s_d((v2i64)dst0, 0);  \
  uint32_t pix_w = __msa_copy_s_w((v4i32)dst0, 2);  \
  SD(pix_d, pdst + 0);                              \
  SW(pix_w, pdst + 8);                              \
} while (0)

#define CONVERT1_BGRA_BGR(psrc, pdst) do {  \
  const int32_t b = (psrc)[0];              \
  const int32_t g = (psrc)[1];              \
  const int32_t r = (psrc)[2];              \
  (pdst)[0] = b;                            \
  (pdst)[1] = g;                            \
  (pdst)[2] = r;                            \
} while (0)

#define CONVERT1_BGRA_RGB(psrc, pdst) do {  \
  const int32_t b = (psrc)[0];              \
  const int32_t g = (psrc)[1];              \
  const int32_t r = (psrc)[2];              \
  (pdst)[0] = r;                            \
  (pdst)[1] = g;                            \
  (pdst)[2] = b;                            \
} while (0)

#define TRANSFORM_COLOR_INVERSE_8(src0, src1, dst0, dst1,     \
                                  c0, c1, mask0, mask1) do {  \
  v8i16 g0, g1, t0, t1, t2, t3;                               \
  v4i32 t4, t5;                                               \
  VSHF_B2_SH(src0, src0, src1, src1, mask0, mask0, g0, g1);   \
  DOTP_SB2_SH(g0, g1, c0, c0, t0, t1);                        \
  SRAI_H2_SH(t0, t1, 5);                                      \
  t0 = __msa_addv_h(t0, (v8i16)src0);                         \
  t1 = __msa_addv_h(t1, (v8i16)src1);                         \
  t4 = __msa_srli_w((v4i32)t0, 16);                           \
  t5 = __msa_srli_w((v4i32)t1, 16);                           \
  DOTP_SB2_SH(t4, t5, c1, c1, t2, t3);                        \
  SRAI_H2_SH(t2, t3, 5);                                      \
  ADD2(t0, t2, t1, t3, t0, t1);                               \
  VSHF_B2_UB(src0, t0, src1, t1, mask1, mask1, dst0, dst1);   \
} while (0)

#define TRANSFORM_COLOR_INVERSE_4(src, dst, c0, c1, mask0, mask1) do {  \
  const v16i8 g0 = VSHF_SB(src, src, mask0);                            \
  v8i16 t0 = __msa_dotp_s_h(c0, g0);                                    \
  v8i16 t1;                                                             \
  v4i32 t2;                                                             \
  t0 = SRAI_H(t0, 5);                                                   \
  t0 = __msa_addv_h(t0, (v8i16)src);                                    \
  t2 = __msa_srli_w((v4i32)t0, 16);                                     \
  t1 = __msa_dotp_s_h(c1, (v16i8)t2);                                   \
  t1 = SRAI_H(t1, 5);                                                   \
  t0 = t0 + t1;                                                         \
  dst = VSHF_UB(src, t0, mask1);                                        \
} while (0)

static void ConvertBGRAToRGBA_MSA(const uint32_t* src,
                                  int num_pixels, uint8_t* dst) {
  int i;
  const uint8_t* ptemp_src = (const uint8_t*)src;
  uint8_t* ptemp_dst = (uint8_t*)dst;
  v16u8 src0, dst0;
  const v16u8 mask = { 2, 1, 0, 3, 6, 5, 4, 7, 10, 9, 8, 11, 14, 13, 12, 15 };

  while (num_pixels >= 8) {
    v16u8 src1, dst1;
    LD_UB2(ptemp_src, 16, src0, src1);
    VSHF_B2_UB(src0, src0, src1, src1, mask, mask, dst0, dst1);
    ST_UB2(dst0, dst1, ptemp_dst, 16);
    ptemp_src += 32;
    ptemp_dst += 32;
    num_pixels -= 8;
  }
  if (num_pixels > 0) {
    if (num_pixels >= 4) {
      src0 = LD_UB(ptemp_src);
      dst0 = VSHF_UB(src0, src0, mask);
      ST_UB(dst0, ptemp_dst);
      ptemp_src += 16;
      ptemp_dst += 16;
      num_pixels -= 4;
    }
    for (i = 0; i < num_pixels; i++) {
      const uint8_t b = ptemp_src[2];
      const uint8_t g = ptemp_src[1];
      const uint8_t r = ptemp_src[0];
      const uint8_t a = ptemp_src[3];
      ptemp_dst[0] = b;
      ptemp_dst[1] = g;
      ptemp_dst[2] = r;
      ptemp_dst[3] = a;
      ptemp_src += 4;
      ptemp_dst += 4;
    }
  }
}

static void ConvertBGRAToBGR_MSA(const uint32_t* src,
                                 int num_pixels, uint8_t* dst) {
  const uint8_t* ptemp_src = (const uint8_t*)src;
  uint8_t* ptemp_dst = (uint8_t*)dst;
  const v16u8 mask0 = { 0, 1, 2, 4, 5, 6, 8, 9, 10, 12, 13, 14,
                        16, 17, 18, 20 };
  const v16u8 mask1 = { 5, 6, 8, 9, 10, 12, 13, 14, 16, 17, 18, 20,
                        21, 22, 24, 25 };
  const v16u8 mask2 = { 10, 12, 13, 14, 16, 17, 18, 20, 21, 22, 24, 25,
                        26, 28, 29, 30 };

  while (num_pixels >= 16) {
    CONVERT16_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1, mask2);
    ptemp_src += 64;
    ptemp_dst += 48;
    num_pixels -= 16;
  }
  if (num_pixels > 0) {
    if (num_pixels >= 12) {
      CONVERT12_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1, mask2);
      ptemp_src += 48;
      ptemp_dst += 36;
      num_pixels -= 12;
    } else if (num_pixels >= 8) {
      CONVERT8_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1);
      ptemp_src += 32;
      ptemp_dst += 24;
      num_pixels -= 8;
    } else if (num_pixels >= 4) {
      CONVERT4_BGRA_XXX(ptemp_src, ptemp_dst, mask0);
      ptemp_src += 16;
      ptemp_dst += 12;
      num_pixels -= 4;
    }
    if (num_pixels == 3) {
      CONVERT1_BGRA_BGR(ptemp_src + 0, ptemp_dst + 0);
      CONVERT1_BGRA_BGR(ptemp_src + 4, ptemp_dst + 3);
      CONVERT1_BGRA_BGR(ptemp_src + 8, ptemp_dst + 6);
    } else if (num_pixels == 2) {
      CONVERT1_BGRA_BGR(ptemp_src + 0, ptemp_dst + 0);
      CONVERT1_BGRA_BGR(ptemp_src + 4, ptemp_dst + 3);
    } else if (num_pixels == 1) {
      CONVERT1_BGRA_BGR(ptemp_src, ptemp_dst);
    }
  }
}

static void ConvertBGRAToRGB_MSA(const uint32_t* src,
                                 int num_pixels, uint8_t* dst) {
  const uint8_t* ptemp_src = (const uint8_t*)src;
  uint8_t* ptemp_dst = (uint8_t*)dst;
  const v16u8 mask0 = { 2, 1, 0, 6, 5, 4, 10, 9, 8, 14, 13, 12,
                        18, 17, 16, 22 };
  const v16u8 mask1 = { 5, 4, 10, 9, 8, 14, 13, 12, 18, 17, 16, 22,
                        21, 20, 26, 25 };
  const v16u8 mask2 = { 8, 14, 13, 12, 18, 17, 16, 22, 21, 20, 26, 25,
                        24, 30, 29, 28 };

  while (num_pixels >= 16) {
    CONVERT16_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1, mask2);
    ptemp_src += 64;
    ptemp_dst += 48;
    num_pixels -= 16;
  }
  if (num_pixels) {
    if (num_pixels >= 12) {
      CONVERT12_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1, mask2);
      ptemp_src += 48;
      ptemp_dst += 36;
      num_pixels -= 12;
    } else if (num_pixels >= 8) {
      CONVERT8_BGRA_XXX(ptemp_src, ptemp_dst, mask0, mask1);
      ptemp_src += 32;
      ptemp_dst += 24;
      num_pixels -= 8;
    } else if (num_pixels >= 4) {
      CONVERT4_BGRA_XXX(ptemp_src, ptemp_dst, mask0);
      ptemp_src += 16;
      ptemp_dst += 12;
      num_pixels -= 4;
    }
    if (num_pixels == 3) {
      CONVERT1_BGRA_RGB(ptemp_src + 0, ptemp_dst + 0);
      CONVERT1_BGRA_RGB(ptemp_src + 4, ptemp_dst + 3);
      CONVERT1_BGRA_RGB(ptemp_src + 8, ptemp_dst + 6);
    } else if (num_pixels == 2) {
      CONVERT1_BGRA_RGB(ptemp_src + 0, ptemp_dst + 0);
      CONVERT1_BGRA_RGB(ptemp_src + 4, ptemp_dst + 3);
    } else if (num_pixels == 1) {
      CONVERT1_BGRA_RGB(ptemp_src, ptemp_dst);
    }
  }
}

static void AddGreenToBlueAndRed_MSA(const uint32_t* const src, int num_pixels,
                                     uint32_t* dst) {
  int i;
  const uint8_t* in = (const uint8_t*)src;
  uint8_t* out = (uint8_t*)dst;
  v16u8 src0, dst0, tmp0;
  const v16u8 mask = { 1, 255, 1, 255, 5, 255, 5, 255, 9, 255, 9, 255,
                       13, 255, 13, 255 };

  while (num_pixels >= 8) {
    v16u8 src1, dst1, tmp1;
    LD_UB2(in, 16, src0, src1);
    VSHF_B2_UB(src0, src1, src1, src0, mask, mask, tmp0, tmp1);
    ADD2(src0, tmp0, src1, tmp1, dst0, dst1);
    ST_UB2(dst0, dst1, out, 16);
    in += 32;
    out += 32;
    num_pixels -= 8;
  }
  if (num_pixels > 0) {
    if (num_pixels >= 4) {
      src0 = LD_UB(in);
      tmp0 = VSHF_UB(src0, src0, mask);
      dst0 = src0 + tmp0;
      ST_UB(dst0, out);
      in += 16;
      out += 16;
      num_pixels -= 4;
    }
    for (i = 0; i < num_pixels; i++) {
      const uint8_t b = in[0];
      const uint8_t g = in[1];
      const uint8_t r = in[2];
      out[0] = (b + g) & 0xff;
      out[1] = g;
      out[2] = (r + g) & 0xff;
      out[4] = in[4];
      out += 4;
    }
  }
}

static void TransformColorInverse_MSA(const VP8LMultipliers* const m,
                                      const uint32_t* src, int num_pixels,
                                      uint32_t* dst) {
  v16u8 src0, dst0;
  const v16i8 g2br = (v16i8)__msa_fill_w(m->green_to_blue_ |
                                         (m->green_to_red_ << 16));
  const v16i8 r2b = (v16i8)__msa_fill_w(m->red_to_blue_);
  const v16u8 mask0 = { 1, 255, 1, 255, 5, 255, 5, 255, 9, 255, 9, 255,
                        13, 255, 13, 255 };
  const v16u8 mask1 = { 16, 1, 18, 3, 20, 5, 22, 7, 24, 9, 26, 11,
                        28, 13, 30, 15 };

  while (num_pixels >= 8) {
    v16u8 src1, dst1;
    LD_UB2(src, 4, src0, src1);
    TRANSFORM_COLOR_INVERSE_8(src0, src1, dst0, dst1, g2br, r2b, mask0, mask1);
    ST_UB2(dst0, dst1, dst, 4);
    src += 8;
    dst += 8;
    num_pixels -= 8;
  }
  if (num_pixels > 0) {
    if (num_pixels >= 4) {
      src0 = LD_UB(src);
      TRANSFORM_COLOR_INVERSE_4(src0, dst0, g2br, r2b, mask0, mask1);
      ST_UB(dst0, dst);
      src += 4;
      dst += 4;
      num_pixels -= 4;
    }
    if (num_pixels > 0) {
      src0 = LD_UB(src);
      TRANSFORM_COLOR_INVERSE_4(src0, dst0, g2br, r2b, mask0, mask1);
      if (num_pixels == 3) {
        const uint64_t pix_d = __msa_copy_s_d((v2i64)dst0, 0);
        const uint32_t pix_w = __msa_copy_s_w((v4i32)dst0, 2);
        SD(pix_d, dst + 0);
        SW(pix_w, dst + 2);
      } else if (num_pixels == 2) {
        const uint64_t pix_d = __msa_copy_s_d((v2i64)dst0, 0);
        SD(pix_d, dst);
      } else {
        const uint32_t pix_w = __msa_copy_s_w((v4i32)dst0, 0);
        SW(pix_w, dst);
      }
    }
  }
}

//------------------------------------------------------------------------------
// Entry point

extern void VP8LDspInitMSA(void);

WEBP_TSAN_IGNORE_FUNCTION void VP8LDspInitMSA(void) {
  VP8LConvertBGRAToRGBA = ConvertBGRAToRGBA_MSA;
  VP8LConvertBGRAToBGR = ConvertBGRAToBGR_MSA;
  VP8LConvertBGRAToRGB = ConvertBGRAToRGB_MSA;

  VP8LAddGreenToBlueAndRed = AddGreenToBlueAndRed_MSA;
  VP8LTransformColorInverse = TransformColorInverse_MSA;
}

#else  // !WEBP_USE_MSA

WEBP_DSP_INIT_STUB(VP8LDspInitMSA)

#endif  // WEBP_USE_MSA
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
- `src/dsp/dsp.h`
- `src/dsp/msa_macro.h`
- `src/dsp/lossless.h`


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

