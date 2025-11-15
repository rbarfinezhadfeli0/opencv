# Documentation for `3rdparty/libwebp/src/dsp/alpha_processing_neon.c`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dsp/alpha_processing_neon.c`
- **File Name**: `alpha_processing_neon.c`
- **File Size**: 7,261 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libwebp/src/dsp/alpha_processing_neon.c](../../../../3rdparty/libwebp/src/dsp/alpha_processing_neon.c)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/dsp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Copyright 2017 Google Inc. All Rights Reserved.
//
// Use of this source code is governed by a BSD-style license
// that can be found in the COPYING file in the root of the source
// tree. An additional intellectual property rights grant can be found
// in the file PATENTS. All contributing project authors may
// be found in the AUTHORS file in the root of the source tree.
// -----------------------------------------------------------------------------
//
// Utilities for processing transparent channel, NEON version.
//
// Author: Skal (pascal.massimino@gmail.com)

#include "src/dsp/dsp.h"

#if defined(WEBP_USE_NEON)

#include "src/dsp/neon.h"

//------------------------------------------------------------------------------

#define MULTIPLIER(a) ((a) * 0x8081)
#define PREMULTIPLY(x, m) (((x) * (m)) >> 23)

#define MULTIPLY_BY_ALPHA(V, ALPHA, OTHER) do {                        \
  const uint8x8_t alpha = (V).val[(ALPHA)];                            \
  const uint16x8_t r1 = vmull_u8((V).val[1], alpha);                   \
  const uint16x8_t g1 = vmull_u8((V).val[2], alpha);                   \
  const uint16x8_t b1 = vmull_u8((V).val[(OTHER)], alpha);             \
  /* we use: v / 255 = (v + 1 + (v >> 8)) >> 8 */                      \
  const uint16x8_t r2 = vsraq_n_u16(r1, r1, 8);                        \
  const uint16x8_t g2 = vsraq_n_u16(g1, g1, 8);                        \
  const uint16x8_t b2 = vsraq_n_u16(b1, b1, 8);                        \
  const uint16x8_t r3 = vaddq_u16(r2, kOne);                           \
  const uint16x8_t g3 = vaddq_u16(g2, kOne);                           \
  const uint16x8_t b3 = vaddq_u16(b2, kOne);                           \
  (V).val[1] = vshrn_n_u16(r3, 8);                                     \
  (V).val[2] = vshrn_n_u16(g3, 8);                                     \
  (V).val[(OTHER)] = vshrn_n_u16(b3, 8);                               \
} while (0)

static void ApplyAlphaMultiply_NEON(uint8_t* rgba, int alpha_first,
                                    int w, int h, int stride) {
  const uint16x8_t kOne = vdupq_n_u16(1u);
  while (h-- > 0) {
    uint32_t* const rgbx = (uint32_t*)rgba;
    int i = 0;
    if (alpha_first) {
      for (; i + 8 <= w; i += 8) {
        // load aaaa...|rrrr...|gggg...|bbbb...
        uint8x8x4_t RGBX = vld4_u8((const uint8_t*)(rgbx + i));
        MULTIPLY_BY_ALPHA(RGBX, 0, 3);
        vst4_u8((uint8_t*)(rgbx + i), RGBX);
      }
    } else {
      for (; i + 8 <= w; i += 8) {
        uint8x8x4_t RGBX = vld4_u8((const uint8_t*)(rgbx + i));
        MULTIPLY_BY_ALPHA(RGBX, 3, 0);
        vst4_u8((uint8_t*)(rgbx + i), RGBX);
      }
    }
    // Finish with left-overs.
    for (; i < w; ++i) {
      uint8_t* const rgb = rgba + (alpha_first ? 1 : 0);
      const uint8_t* const alpha = rgba + (alpha_first ? 0 : 3);
      const uint32_t a = alpha[4 * i];
      if (a != 0xff) {
        const uint32_t mult = MULTIPLIER(a);
        rgb[4 * i + 0] = PREMULTIPLY(rgb[4 * i + 0], mult);
        rgb[4 * i + 1] = PREMULTIPLY(rgb[4 * i + 1], mult);
        rgb[4 * i + 2] = PREMULTIPLY(rgb[4 * i + 2], mult);
      }
    }
    rgba += stride;
  }
}
#undef MULTIPLY_BY_ALPHA
#undef MULTIPLIER
#undef PREMULTIPLY

//------------------------------------------------------------------------------

static int DispatchAlpha_NEON(const uint8_t* WEBP_RESTRICT alpha,
                              int alpha_stride, int width, int height,
                              uint8_t* WEBP_RESTRICT dst, int dst_stride) {
  uint32_t alpha_mask = 0xffu;
  uint8x8_t mask8 = vdup_n_u8(0xff);
  uint32_t tmp[2];
  int i, j;
  for (j = 0; j < height; ++j) {
    // We don't know if alpha is first or last in dst[] (depending on rgbA/Argb
    // mode). So we must be sure dst[4*i + 8 - 1] is writable for the store.
    // Hence the test with 'width - 1' instead of just 'width'.
    for (i = 0; i + 8 <= width - 1; i += 8) {
      uint8x8x4_t rgbX = vld4_u8((const uint8_t*)(dst + 4 * i));
      const uint8x8_t alphas = vld1_u8(alpha + i);
      rgbX.val[0] = alphas;
      vst4_u8((uint8_t*)(dst + 4 * i), rgbX);
      mask8 = vand_u8(mask8, alphas);
    }
    for (; i < width; ++i) {
      const uint32_t alpha_value = alpha[i];
      dst[4 * i] = alpha_value;
      alpha_mask &= alpha_value;
    }
    alpha += alpha_stride;
    dst += dst_stride;
  }
  vst1_u8((uint8_t*)tmp, mask8);
  alpha_mask *= 0x01010101;
  alpha_mask &= tmp[0];
  alpha_mask &= tmp[1];
  return (alpha_mask != 0xffffffffu);
}

static void DispatchAlphaToGreen_NEON(const uint8_t* WEBP_RESTRICT alpha,
                                      int alpha_stride, int width, int height,
                                      uint32_t* WEBP_RESTRICT dst,
                                      int dst_stride) {
  int i, j;
  uint8x8x4_t greens;   // leave A/R/B channels zero'd.
  greens.val[0] = vdup_n_u8(0);
  greens.val[2] = vdup_n_u8(0);
  greens.val[3] = vdup_n_u8(0);
  for (j = 0; j < height; ++j) {
    for (i = 0; i + 8 <= width; i += 8) {
      greens.val[1] = vld1_u8(alpha + i);
      vst4_u8((uint8_t*)(dst + i), greens);
    }
    for (; i < width; ++i) dst[i] = alpha[i] << 8;
    alpha += alpha_stride;
    dst += dst_stride;
  }
}

static int ExtractAlpha_NEON(const uint8_t* WEBP_RESTRICT argb, int argb_stride,
                             int width, int height,
                             uint8_t* WEBP_RESTRICT alpha, int alpha_stride) {
  uint32_t alpha_mask = 0xffu;
  uint8x8_t mask8 = vdup_n_u8(0xff);
  uint32_t tmp[2];
  int i, j;
  for (j = 0; j < height; ++j) {
    // We don't know if alpha is first or last in dst[] (depending on rgbA/Argb
    // mode). So we must be sure dst[4*i + 8 - 1] is writable for the store.
    // Hence the test with 'width - 1' instead of just 'width'.
    for (i = 0; i + 8 <= width - 1; i += 8) {
      const uint8x8x4_t rgbX = vld4_u8((const uint8_t*)(argb + 4 * i));
      const uint8x8_t alphas = rgbX.val[0];
      vst1_u8((uint8_t*)(alpha + i), alphas);
      mask8 = vand_u8(mask8, alphas);
    }
    for (; i < width; ++i) {
      alpha[i] = argb[4 * i];
      alpha_mask &= alpha[i];
    }
    argb += argb_stride;
    alpha += alpha_stride;
  }
  vst1_u8((uint8_t*)tmp, mask8);
  alpha_mask *= 0x01010101;
  alpha_mask &= tmp[0];
  alpha_mask &= tmp[1];
  return (alpha_mask == 0xffffffffu);
}

static void ExtractGreen_NEON(const uint32_t* WEBP_RESTRICT argb,
                              uint8_t* WEBP_RESTRICT alpha, int size) {
  int i;
  for (i = 0; i + 16 <= size; i += 16) {
    const uint8x16x4_t rgbX = vld4q_u8((const uint8_t*)(argb + i));
    const uint8x16_t greens = rgbX.val[1];
    vst1q_u8(alpha + i, greens);
  }
  for (; i < size; ++i) alpha[i] = (argb[i] >> 8) & 0xff;
}

//------------------------------------------------------------------------------

extern void WebPInitAlphaProcessingNEON(void);

WEBP_TSAN_IGNORE_FUNCTION void WebPInitAlphaProcessingNEON(void) {
  WebPApplyAlphaMultiply = ApplyAlphaMultiply_NEON;
  WebPDispatchAlpha = DispatchAlpha_NEON;
  WebPDispatchAlphaToGreen = DispatchAlphaToGreen_NEON;
  WebPExtractAlpha = ExtractAlpha_NEON;
  WebPExtractGreen = ExtractGreen_NEON;
}

#else  // !WEBP_USE_NEON

WEBP_DSP_INIT_STUB(WebPInitAlphaProcessingNEON)

#endif  // WEBP_USE_NEON
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

- **PREMULTIPLY()**: A function/method defined in this file
- **MULTIPLIER()**: A function/method defined in this file
- **MULTIPLY_BY_ALPHA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/dsp/dsp.h`
- `src/dsp/neon.h`


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

