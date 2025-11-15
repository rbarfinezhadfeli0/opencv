# Documentation for `3rdparty/libwebp/src/dsp/lossless_common.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dsp/lossless_common.h`
- **File Name**: `lossless_common.h`
- **File Size**: 7,390 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/dsp/lossless_common.h](../../../../3rdparty/libwebp/src/dsp/lossless_common.h)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/dsp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Copyright 2012 Google Inc. All Rights Reserved.
//
// Use of this source code is governed by a BSD-style license
// that can be found in the COPYING file in the root of the source
// tree. An additional intellectual property rights grant can be found
// in the file PATENTS. All contributing project authors may
// be found in the AUTHORS file in the root of the source tree.
// -----------------------------------------------------------------------------
//
// Image transforms and color space conversion methods for lossless decoder.
//
// Authors: Vikas Arora (vikaas.arora@gmail.com)
//          Jyrki Alakuijala (jyrki@google.com)
//          Vincent Rabaud (vrabaud@google.com)

#ifndef WEBP_DSP_LOSSLESS_COMMON_H_
#define WEBP_DSP_LOSSLESS_COMMON_H_

#include "src/dsp/cpu.h"
#include "src/utils/utils.h"
#include "src/webp/types.h"

#ifdef __cplusplus
extern "C" {
#endif

//------------------------------------------------------------------------------
// Decoding

// color mapping related functions.
static WEBP_INLINE uint32_t VP8GetARGBIndex(uint32_t idx) {
  return (idx >> 8) & 0xff;
}

static WEBP_INLINE uint8_t VP8GetAlphaIndex(uint8_t idx) {
  return idx;
}

static WEBP_INLINE uint32_t VP8GetARGBValue(uint32_t val) {
  return val;
}

static WEBP_INLINE uint8_t VP8GetAlphaValue(uint32_t val) {
  return (val >> 8) & 0xff;
}

//------------------------------------------------------------------------------
// Misc methods.

// Computes sampled size of 'size' when sampling using 'sampling bits'.
static WEBP_INLINE uint32_t VP8LSubSampleSize(uint32_t size,
                                              uint32_t sampling_bits) {
  return (size + (1 << sampling_bits) - 1) >> sampling_bits;
}

// Converts near lossless quality into max number of bits shaved off.
static WEBP_INLINE int VP8LNearLosslessBits(int near_lossless_quality) {
  //    100 -> 0
  // 80..99 -> 1
  // 60..79 -> 2
  // 40..59 -> 3
  // 20..39 -> 4
  //  0..19 -> 5
  return 5 - near_lossless_quality / 20;
}

// -----------------------------------------------------------------------------
// Faster logarithm for integers. Small values use a look-up table.

// The threshold till approximate version of log_2 can be used.
// Practically, we can get rid of the call to log() as the two values match to
// very high degree (the ratio of these two is 0.99999x).
// Keeping a high threshold for now.
#define APPROX_LOG_WITH_CORRECTION_MAX  65536
#define APPROX_LOG_MAX                   4096
#define LOG_2_RECIPROCAL 1.44269504088896338700465094007086
#define LOG_LOOKUP_IDX_MAX 256
extern const float kLog2Table[LOG_LOOKUP_IDX_MAX];
extern const float kSLog2Table[LOG_LOOKUP_IDX_MAX];
typedef float (*VP8LFastLog2SlowFunc)(uint32_t v);

extern VP8LFastLog2SlowFunc VP8LFastLog2Slow;
extern VP8LFastLog2SlowFunc VP8LFastSLog2Slow;

static WEBP_INLINE float VP8LFastLog2(uint32_t v) {
  return (v < LOG_LOOKUP_IDX_MAX) ? kLog2Table[v] : VP8LFastLog2Slow(v);
}
// Fast calculation of v * log2(v) for integer input.
static WEBP_INLINE float VP8LFastSLog2(uint32_t v) {
  return (v < LOG_LOOKUP_IDX_MAX) ? kSLog2Table[v] : VP8LFastSLog2Slow(v);
}

// -----------------------------------------------------------------------------
// PrefixEncode()

// Splitting of distance and length codes into prefixes and
// extra bits. The prefixes are encoded with an entropy code
// while the extra bits are stored just as normal bits.
static WEBP_INLINE void VP8LPrefixEncodeBitsNoLUT(int distance, int* const code,
                                                  int* const extra_bits) {
  const int highest_bit = BitsLog2Floor(--distance);
  const int second_highest_bit = (distance >> (highest_bit - 1)) & 1;
  *extra_bits = highest_bit - 1;
  *code = 2 * highest_bit + second_highest_bit;
}

static WEBP_INLINE void VP8LPrefixEncodeNoLUT(int distance, int* const code,
                                              int* const extra_bits,
                                              int* const extra_bits_value) {
  const int highest_bit = BitsLog2Floor(--distance);
  const int second_highest_bit = (distance >> (highest_bit - 1)) & 1;
  *extra_bits = highest_bit - 1;
  *extra_bits_value = distance & ((1 << *extra_bits) - 1);
  *code = 2 * highest_bit + second_highest_bit;
}

#define PREFIX_LOOKUP_IDX_MAX   512
typedef struct {
  int8_t code_;
  int8_t extra_bits_;
} VP8LPrefixCode;

// These tables are derived using VP8LPrefixEncodeNoLUT.
extern const VP8LPrefixCode kPrefixEncodeCode[PREFIX_LOOKUP_IDX_MAX];
extern const uint8_t kPrefixEncodeExtraBitsValue[PREFIX_LOOKUP_IDX_MAX];
static WEBP_INLINE void VP8LPrefixEncodeBits(int distance, int* const code,
                                             int* const extra_bits) {
  if (distance < PREFIX_LOOKUP_IDX_MAX) {
    const VP8LPrefixCode prefix_code = kPrefixEncodeCode[distance];
    *code = prefix_code.code_;
    *extra_bits = prefix_code.extra_bits_;
  } else {
    VP8LPrefixEncodeBitsNoLUT(distance, code, extra_bits);
  }
}

static WEBP_INLINE void VP8LPrefixEncode(int distance, int* const code,
                                         int* const extra_bits,
                                         int* const extra_bits_value) {
  if (distance < PREFIX_LOOKUP_IDX_MAX) {
    const VP8LPrefixCode prefix_code = kPrefixEncodeCode[distance];
    *code = prefix_code.code_;
    *extra_bits = prefix_code.extra_bits_;
    *extra_bits_value = kPrefixEncodeExtraBitsValue[distance];
  } else {
    VP8LPrefixEncodeNoLUT(distance, code, extra_bits, extra_bits_value);
  }
}

// Sum of each component, mod 256.
static WEBP_UBSAN_IGNORE_UNSIGNED_OVERFLOW WEBP_INLINE
uint32_t VP8LAddPixels(uint32_t a, uint32_t b) {
  const uint32_t alpha_and_green = (a & 0xff00ff00u) + (b & 0xff00ff00u);
  const uint32_t red_and_blue = (a & 0x00ff00ffu) + (b & 0x00ff00ffu);
  return (alpha_and_green & 0xff00ff00u) | (red_and_blue & 0x00ff00ffu);
}

// Difference of each component, mod 256.
static WEBP_UBSAN_IGNORE_UNSIGNED_OVERFLOW WEBP_INLINE
uint32_t VP8LSubPixels(uint32_t a, uint32_t b) {
  const uint32_t alpha_and_green =
      0x00ff00ffu + (a & 0xff00ff00u) - (b & 0xff00ff00u);
  const uint32_t red_and_blue =
      0xff00ff00u + (a & 0x00ff00ffu) - (b & 0x00ff00ffu);
  return (alpha_and_green & 0xff00ff00u) | (red_and_blue & 0x00ff00ffu);
}

//------------------------------------------------------------------------------
// Transform-related functions used in both encoding and decoding.

// Macros used to create a batch predictor that iteratively uses a
// one-pixel predictor.

// The predictor is added to the output pixel (which
// is therefore considered as a residual) to get the final prediction.
#define GENERATE_PREDICTOR_ADD(PREDICTOR, PREDICTOR_ADD)             \
static void PREDICTOR_ADD(const uint32_t* in, const uint32_t* upper, \
                          int num_pixels, uint32_t* out) {           \
  int x;                                                             \
  assert(upper != NULL);                                             \
  for (x = 0; x < num_pixels; ++x) {                                 \
    const uint32_t pred = (PREDICTOR)(&out[x - 1], upper + x);       \
    out[x] = VP8LAddPixels(in[x], pred);                             \
  }                                                                  \
}

#ifdef __cplusplus
}    // extern "C"
#endif

#endif  // WEBP_DSP_LOSSLESS_COMMON_H_
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

- **struct()**: A function/method defined in this file
- **float()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **WEBP_DSP_LOSSLESS_COMMON_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/utils/utils.h`
- `src/dsp/cpu.h`
- `src/webp/types.h`


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

