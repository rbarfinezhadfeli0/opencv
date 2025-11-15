# Documentation for `3rdparty/libwebp/src/utils/bit_reader_inl_utils.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/utils/bit_reader_inl_utils.h`
- **File Name**: `bit_reader_inl_utils.h`
- **File Size**: 5,894 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/utils/bit_reader_inl_utils.h](../../../../3rdparty/libwebp/src/utils/bit_reader_inl_utils.h)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/utils` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Copyright 2014 Google Inc. All Rights Reserved.
//
// Use of this source code is governed by a BSD-style license
// that can be found in the COPYING file in the root of the source
// tree. An additional intellectual property rights grant can be found
// in the file PATENTS. All contributing project authors may
// be found in the AUTHORS file in the root of the source tree.
// -----------------------------------------------------------------------------
//
// Specific inlined methods for boolean decoder [VP8GetBit() ...]
// This file should be included by the .c sources that actually need to call
// these methods.
//
// Author: Skal (pascal.massimino@gmail.com)

#ifndef WEBP_UTILS_BIT_READER_INL_UTILS_H_
#define WEBP_UTILS_BIT_READER_INL_UTILS_H_

#ifdef HAVE_CONFIG_H
#include "src/webp/config.h"
#endif

#include <string.h>  // for memcpy

#include "src/dsp/dsp.h"
#include "src/utils/bit_reader_utils.h"
#include "src/utils/endian_inl_utils.h"
#include "src/utils/utils.h"

#ifdef __cplusplus
extern "C" {
#endif

//------------------------------------------------------------------------------
// Derived type lbit_t = natural type for memory I/O

#if   (BITS > 32)
typedef uint64_t lbit_t;
#elif (BITS > 16)
typedef uint32_t lbit_t;
#elif (BITS >  8)
typedef uint16_t lbit_t;
#else
typedef uint8_t lbit_t;
#endif

extern const uint8_t kVP8Log2Range[128];
extern const uint8_t kVP8NewRange[128];

// special case for the tail byte-reading
void VP8LoadFinalBytes(VP8BitReader* const br);

//------------------------------------------------------------------------------
// Inlined critical functions

// makes sure br->value_ has at least BITS bits worth of data
static WEBP_UBSAN_IGNORE_UNDEF WEBP_INLINE
void VP8LoadNewBytes(VP8BitReader* WEBP_RESTRICT const br) {
  assert(br != NULL && br->buf_ != NULL);
  // Read 'BITS' bits at a time if possible.
  if (br->buf_ < br->buf_max_) {
    // convert memory type to register type (with some zero'ing!)
    bit_t bits;
#if defined(WEBP_USE_MIPS32)
    // This is needed because of un-aligned read.
    lbit_t in_bits;
    lbit_t* p_buf_ = (lbit_t*)br->buf_;
    __asm__ volatile(
      ".set   push                             \n\t"
      ".set   at                               \n\t"
      ".set   macro                            \n\t"
      "ulw    %[in_bits], 0(%[p_buf_])         \n\t"
      ".set   pop                              \n\t"
      : [in_bits]"=r"(in_bits)
      : [p_buf_]"r"(p_buf_)
      : "memory", "at"
    );
#else
    lbit_t in_bits;
    memcpy(&in_bits, br->buf_, sizeof(in_bits));
#endif
    br->buf_ += BITS >> 3;
#if !defined(WORDS_BIGENDIAN)
#if (BITS > 32)
    bits = BSwap64(in_bits);
    bits >>= 64 - BITS;
#elif (BITS >= 24)
    bits = BSwap32(in_bits);
    bits >>= (32 - BITS);
#elif (BITS == 16)
    bits = BSwap16(in_bits);
#else   // BITS == 8
    bits = (bit_t)in_bits;
#endif  // BITS > 32
#else    // WORDS_BIGENDIAN
    bits = (bit_t)in_bits;
    if (BITS != 8 * sizeof(bit_t)) bits >>= (8 * sizeof(bit_t) - BITS);
#endif
    br->value_ = bits | (br->value_ << BITS);
    br->bits_ += BITS;
  } else {
    VP8LoadFinalBytes(br);    // no need to be inlined
  }
}

// Read a bit with proba 'prob'. Speed-critical function!
static WEBP_INLINE int VP8GetBit(VP8BitReader* WEBP_RESTRICT const br,
                                 int prob, const char label[]) {
  // Don't move this declaration! It makes a big speed difference to store
  // 'range' *before* calling VP8LoadNewBytes(), even if this function doesn't
  // alter br->range_ value.
  range_t range = br->range_;
  if (br->bits_ < 0) {
    VP8LoadNewBytes(br);
  }
  {
    const int pos = br->bits_;
    const range_t split = (range * prob) >> 8;
    const range_t value = (range_t)(br->value_ >> pos);
    const int bit = (value > split);
    if (bit) {
      range -= split;
      br->value_ -= (bit_t)(split + 1) << pos;
    } else {
      range = split + 1;
    }
    {
      const int shift = 7 ^ BitsLog2Floor(range);
      range <<= shift;
      br->bits_ -= shift;
    }
    br->range_ = range - 1;
    BT_TRACK(br);
    return bit;
  }
}

// simplified version of VP8GetBit() for prob=0x80 (note shift is always 1 here)
static WEBP_UBSAN_IGNORE_UNSIGNED_OVERFLOW WEBP_INLINE
int VP8GetSigned(VP8BitReader* WEBP_RESTRICT const br, int v,
                 const char label[]) {
  if (br->bits_ < 0) {
    VP8LoadNewBytes(br);
  }
  {
    const int pos = br->bits_;
    const range_t split = br->range_ >> 1;
    const range_t value = (range_t)(br->value_ >> pos);
    const int32_t mask = (int32_t)(split - value) >> 31;  // -1 or 0
    br->bits_ -= 1;
    br->range_ += (range_t)mask;
    br->range_ |= 1;
    br->value_ -= (bit_t)((split + 1) & (uint32_t)mask) << pos;
    BT_TRACK(br);
    return (v ^ mask) - mask;
  }
}

static WEBP_INLINE int VP8GetBitAlt(VP8BitReader* WEBP_RESTRICT const br,
                                    int prob, const char label[]) {
  // Don't move this declaration! It makes a big speed difference to store
  // 'range' *before* calling VP8LoadNewBytes(), even if this function doesn't
  // alter br->range_ value.
  range_t range = br->range_;
  if (br->bits_ < 0) {
    VP8LoadNewBytes(br);
  }
  {
    const int pos = br->bits_;
    const range_t split = (range * prob) >> 8;
    const range_t value = (range_t)(br->value_ >> pos);
    int bit;  // Don't use 'const int bit = (value > split);", it's slower.
    if (value > split) {
      range -= split + 1;
      br->value_ -= (bit_t)(split + 1) << pos;
      bit = 1;
    } else {
      range = split;
      bit = 0;
    }
    if (range <= (range_t)0x7e) {
      const int shift = kVP8Log2Range[range];
      range = kVP8NewRange[range];
      br->bits_ -= shift;
    }
    br->range_ = range;
    BT_TRACK(br);
    return bit;
  }
}

#ifdef __cplusplus
}    // extern "C"
#endif

#endif  // WEBP_UTILS_BIT_READER_INL_UTILS_H_
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

- **uint32_t()**: A function/method defined in this file
- **doesn()**: A function/method defined in this file
- **HAVE_CONFIG_H()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **uint64_t()**: A function/method defined in this file
- **WEBP_UTILS_BIT_READER_INL_UTILS_H_()**: A function/method defined in this file
- **uint8_t()**: A function/method defined in this file
- **uint16_t()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/webp/config.h`
- `src/utils/endian_inl_utils.h`
- `string.h`
- `src/dsp/dsp.h`
- `src/utils/utils.h`
- `src/utils/bit_reader_utils.h`


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

