# Documentation for `docs/3rdparty/libwebp/src/dsp/filters_neon.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libwebp/src/dsp/filters_neon.c_docs.md`
- **File Name**: `filters_neon.c_docs.md`
- **File Size**: 14,877 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libwebp/src/dsp/filters_neon.c_docs.md](../../../../../docs/3rdparty/libwebp/src/dsp/filters_neon.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libwebp/src/dsp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libwebp/src/dsp/filters_neon.c`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dsp/filters_neon.c`
- **File Name**: `filters_neon.c`
- **File Size**: 11,715 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libwebp/src/dsp/filters_neon.c](../../../../3rdparty/libwebp/src/dsp/filters_neon.c)

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
// NEON variant of alpha filters
//
// Author: Skal (pascal.massimino@gmail.com)

#include "src/dsp/dsp.h"

#if defined(WEBP_USE_NEON)

#include <assert.h>
#include "src/dsp/neon.h"

//------------------------------------------------------------------------------
// Helpful macros.

#define DCHECK(in, out)                                                        \
  do {                                                                         \
    assert(in != NULL);                                                        \
    assert(out != NULL);                                                       \
    assert(width > 0);                                                         \
    assert(height > 0);                                                        \
    assert(stride >= width);                                                   \
    assert(row >= 0 && num_rows > 0 && row + num_rows <= height);              \
    (void)height;  /* Silence unused warning. */                               \
  } while (0)

// load eight u8 and widen to s16
#define U8_TO_S16(A) vreinterpretq_s16_u16(vmovl_u8(A))
#define LOAD_U8_TO_S16(A) U8_TO_S16(vld1_u8(A))

// shift left or right by N byte, inserting zeros
#define SHIFT_RIGHT_N_Q(A, N) vextq_u8((A), zero, (N))
#define SHIFT_LEFT_N_Q(A, N) vextq_u8(zero, (A), (16 - (N)) % 16)

// rotate left by N bytes
#define ROTATE_LEFT_N(A, N)   vext_u8((A), (A), (N))
// rotate right by N bytes
#define ROTATE_RIGHT_N(A, N)   vext_u8((A), (A), (8 - (N)) % 8)

static void PredictLine_NEON(const uint8_t* src, const uint8_t* pred,
                             uint8_t* dst, int length) {
  int i;
  assert(length >= 0);
  for (i = 0; i + 16 <= length; i += 16) {
    const uint8x16_t A = vld1q_u8(&src[i]);
    const uint8x16_t B = vld1q_u8(&pred[i]);
    const uint8x16_t C = vsubq_u8(A, B);
    vst1q_u8(&dst[i], C);
  }
  for (; i < length; ++i) dst[i] = src[i] - pred[i];
}

// Special case for left-based prediction (when preds==dst-1 or preds==src-1).
static void PredictLineLeft_NEON(const uint8_t* src, uint8_t* dst, int length) {
  PredictLine_NEON(src, src - 1, dst, length);
}

//------------------------------------------------------------------------------
// Horizontal filter.

static WEBP_INLINE void DoHorizontalFilter_NEON(const uint8_t* in,
                                                int width, int height,
                                                int stride,
                                                int row, int num_rows,
                                                uint8_t* out) {
  const size_t start_offset = row * stride;
  const int last_row = row + num_rows;
  DCHECK(in, out);
  in += start_offset;
  out += start_offset;

  if (row == 0) {
    // Leftmost pixel is the same as input for topmost scanline.
    out[0] = in[0];
    PredictLineLeft_NEON(in + 1, out + 1, width - 1);
    row = 1;
    in += stride;
    out += stride;
  }

  // Filter line-by-line.
  while (row < last_row) {
    // Leftmost pixel is predicted from above.
    out[0] = in[0] - in[-stride];
    PredictLineLeft_NEON(in + 1, out + 1, width - 1);
    ++row;
    in += stride;
    out += stride;
  }
}

static void HorizontalFilter_NEON(const uint8_t* data, int width, int height,
                                  int stride, uint8_t* filtered_data) {
  DoHorizontalFilter_NEON(data, width, height, stride, 0, height,
                          filtered_data);
}

//------------------------------------------------------------------------------
// Vertical filter.

static WEBP_INLINE void DoVerticalFilter_NEON(const uint8_t* in,
                                              int width, int height, int stride,
                                              int row, int num_rows,
                                              uint8_t* out) {
  const size_t start_offset = row * stride;
  const int last_row = row + num_rows;
  DCHECK(in, out);
  in += start_offset;
  out += start_offset;

  if (row == 0) {
    // Very first top-left pixel is copied.
    out[0] = in[0];
    // Rest of top scan-line is left-predicted.
    PredictLineLeft_NEON(in + 1, out + 1, width - 1);
    row = 1;
    in += stride;
    out += stride;
  }

  // Filter line-by-line.
  while (row < last_row) {
    PredictLine_NEON(in, in - stride, out, width);
    ++row;
    in += stride;
    out += stride;
  }
}

static void VerticalFilter_NEON(const uint8_t* data, int width, int height,
                                int stride, uint8_t* filtered_data) {
  DoVerticalFilter_NEON(data, width, height, stride, 0, height,
                        filtered_data);
}

//------------------------------------------------------------------------------
// Gradient filter.

static WEBP_INLINE int GradientPredictor_C(uint8_t a, uint8_t b, uint8_t c) {
  const int g = a + b - c;
  return ((g & ~0xff) == 0) ? g : (g < 0) ? 0 : 255;  // clip to 8bit
}

static void GradientPredictDirect_NEON(const uint8_t* const row,
                                       const uint8_t* const top,
                                       uint8_t* const out, int length) {
  int i;
  for (i = 0; i + 8 <= length; i += 8) {
    const uint8x8_t A = vld1_u8(&row[i - 1]);
    const uint8x8_t B = vld1_u8(&top[i + 0]);
    const int16x8_t C = vreinterpretq_s16_u16(vaddl_u8(A, B));
    const int16x8_t D = LOAD_U8_TO_S16(&top[i - 1]);
    const uint8x8_t E = vqmovun_s16(vsubq_s16(C, D));
    const uint8x8_t F = vld1_u8(&row[i + 0]);
    vst1_u8(&out[i], vsub_u8(F, E));
  }
  for (; i < length; ++i) {
    out[i] = row[i] - GradientPredictor_C(row[i - 1], top[i], top[i - 1]);
  }
}

static WEBP_INLINE void DoGradientFilter_NEON(const uint8_t* in,
                                              int width, int height,
                                              int stride,
                                              int row, int num_rows,
                                              uint8_t* out) {
  const size_t start_offset = row * stride;
  const int last_row = row + num_rows;
  DCHECK(in, out);
  in += start_offset;
  out += start_offset;

  // left prediction for top scan-line
  if (row == 0) {
    out[0] = in[0];
    PredictLineLeft_NEON(in + 1, out + 1, width - 1);
    row = 1;
    in += stride;
    out += stride;
  }

  // Filter line-by-line.
  while (row < last_row) {
    out[0] = in[0] - in[-stride];
    GradientPredictDirect_NEON(in + 1, in + 1 - stride, out + 1, width - 1);
    ++row;
    in += stride;
    out += stride;
  }
}

static void GradientFilter_NEON(const uint8_t* data, int width, int height,
                                int stride, uint8_t* filtered_data) {
  DoGradientFilter_NEON(data, width, height, stride, 0, height,
                        filtered_data);
}

#undef DCHECK

//------------------------------------------------------------------------------
// Inverse transforms

static void HorizontalUnfilter_NEON(const uint8_t* prev, const uint8_t* in,
                                    uint8_t* out, int width) {
  int i;
  const uint8x16_t zero = vdupq_n_u8(0);
  uint8x16_t last;
  out[0] = in[0] + (prev == NULL ? 0 : prev[0]);
  if (width <= 1) return;
  last = vsetq_lane_u8(out[0], zero, 0);
  for (i = 1; i + 16 <= width; i += 16) {
    const uint8x16_t A0 = vld1q_u8(&in[i]);
    const uint8x16_t A1 = vaddq_u8(A0, last);
    const uint8x16_t A2 = SHIFT_LEFT_N_Q(A1, 1);
    const uint8x16_t A3 = vaddq_u8(A1, A2);
    const uint8x16_t A4 = SHIFT_LEFT_N_Q(A3, 2);
    const uint8x16_t A5 = vaddq_u8(A3, A4);
    const uint8x16_t A6 = SHIFT_LEFT_N_Q(A5, 4);
    const uint8x16_t A7 = vaddq_u8(A5, A6);
    const uint8x16_t A8 = SHIFT_LEFT_N_Q(A7, 8);
    const uint8x16_t A9 = vaddq_u8(A7, A8);
    vst1q_u8(&out[i], A9);
    last = SHIFT_RIGHT_N_Q(A9, 15);
  }
  for (; i < width; ++i) out[i] = in[i] + out[i - 1];
}

static void VerticalUnfilter_NEON(const uint8_t* prev, const uint8_t* in,
                                  uint8_t* out, int width) {
  if (prev == NULL) {
    HorizontalUnfilter_NEON(NULL, in, out, width);
  } else {
    int i;
    assert(width >= 0);
    for (i = 0; i + 16 <= width; i += 16) {
      const uint8x16_t A = vld1q_u8(&in[i]);
      const uint8x16_t B = vld1q_u8(&prev[i]);
      const uint8x16_t C = vaddq_u8(A, B);
      vst1q_u8(&out[i], C);
    }
    for (; i < width; ++i) out[i] = in[i] + prev[i];
  }
}

// GradientUnfilter_NEON is correct but slower than the C-version,
// at least on ARM64. For armv7, it's a wash.
// So best is to disable it for now, but keep the idea around...
#if !defined(USE_GRADIENT_UNFILTER)
#define USE_GRADIENT_UNFILTER 0   // ALTERNATE_CODE
#endif

#if (USE_GRADIENT_UNFILTER == 1)
#define GRAD_PROCESS_LANE(L)  do {                                             \
  const uint8x8_t tmp1 = ROTATE_RIGHT_N(pred, 1);  /* rotate predictor in */   \
  const int16x8_t tmp2 = vaddq_s16(BC, U8_TO_S16(tmp1));                       \
  const uint8x8_t delta = vqmovun_s16(tmp2);                                   \
  pred = vadd_u8(D, delta);                                                    \
  out = vext_u8(out, ROTATE_LEFT_N(pred, (L)), 1);                             \
} while (0)

static void GradientPredictInverse_NEON(const uint8_t* const in,
                                        const uint8_t* const top,
                                        uint8_t* const row, int length) {
  if (length > 0) {
    int i;
    uint8x8_t pred = vdup_n_u8(row[-1]);   // left sample
    uint8x8_t out = vdup_n_u8(0);
    for (i = 0; i + 8 <= length; i += 8) {
      const int16x8_t B = LOAD_U8_TO_S16(&top[i + 0]);
      const int16x8_t C = LOAD_U8_TO_S16(&top[i - 1]);
      const int16x8_t BC = vsubq_s16(B, C);  // unclipped gradient basis B - C
      const uint8x8_t D = vld1_u8(&in[i]);   // base input
      GRAD_PROCESS_LANE(0);
      GRAD_PROCESS_LANE(1);
      GRAD_PROCESS_LANE(2);
      GRAD_PROCESS_LANE(3);
      GRAD_PROCESS_LANE(4);
      GRAD_PROCESS_LANE(5);
      GRAD_PROCESS_LANE(6);
      GRAD_PROCESS_LANE(7);
      vst1_u8(&row[i], out);
    }
    for (; i < length; ++i) {
      row[i] = in[i] + GradientPredictor_C(row[i - 1], top[i], top[i - 1]);
    }
  }
}
#undef GRAD_PROCESS_LANE

static void GradientUnfilter_NEON(const uint8_t* prev, const uint8_t* in,
                                  uint8_t* out, int width) {
  if (prev == NULL) {
    HorizontalUnfilter_NEON(NULL, in, out, width);
  } else {
    out[0] = in[0] + prev[0];  // predict from above
    GradientPredictInverse_NEON(in + 1, prev + 1, out + 1, width - 1);
  }
}

#endif   // USE_GRADIENT_UNFILTER

//------------------------------------------------------------------------------
// Entry point

extern void VP8FiltersInitNEON(void);

WEBP_TSAN_IGNORE_FUNCTION void VP8FiltersInitNEON(void) {
  WebPUnfilters[WEBP_FILTER_HORIZONTAL] = HorizontalUnfilter_NEON;
  WebPUnfilters[WEBP_FILTER_VERTICAL] = VerticalUnfilter_NEON;
#if (USE_GRADIENT_UNFILTER == 1)
  WebPUnfilters[WEBP_FILTER_GRADIENT] = GradientUnfilter_NEON;
#endif

  WebPFilters[WEBP_FILTER_HORIZONTAL] = HorizontalFilter_NEON;
  WebPFilters[WEBP_FILTER_VERTICAL] = VerticalFilter_NEON;
  WebPFilters[WEBP_FILTER_GRADIENT] = GradientFilter_NEON;
}

#else  // !WEBP_USE_NEON

WEBP_DSP_INIT_STUB(VP8FiltersInitNEON)

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

- **GRAD_PROCESS_LANE()**: A function/method defined in this file
- **DCHECK()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/dsp/dsp.h`
- `src/dsp/neon.h`
- `assert.h`

**Python Imports:**
- `above.`
- `above`


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

