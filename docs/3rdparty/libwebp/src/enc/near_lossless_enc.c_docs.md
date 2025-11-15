# Documentation for `3rdparty/libwebp/src/enc/near_lossless_enc.c`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/enc/near_lossless_enc.c`
- **File Name**: `near_lossless_enc.c`
- **File Size**: 5,319 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libwebp/src/enc/near_lossless_enc.c](../../../../3rdparty/libwebp/src/enc/near_lossless_enc.c)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/enc` directory and serves as part of the OpenCV library infrastructure.

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
// Near-lossless image preprocessing adjusts pixel values to help
// compressibility with a guarantee of maximum deviation between original and
// resulting pixel values.
//
// Author: Jyrki Alakuijala (jyrki@google.com)
// Converted to C by Aleksander Kramarz (akramarz@google.com)

#include <assert.h>
#include <stdlib.h>

#include "src/dsp/lossless_common.h"
#include "src/utils/utils.h"
#include "src/enc/vp8li_enc.h"

#if (WEBP_NEAR_LOSSLESS == 1)

#define MIN_DIM_FOR_NEAR_LOSSLESS 64
#define MAX_LIMIT_BITS             5

// Quantizes the value up or down to a multiple of 1<<bits (or to 255),
// choosing the closer one, resolving ties using bankers' rounding.
static uint32_t FindClosestDiscretized(uint32_t a, int bits) {
  const uint32_t mask = (1u << bits) - 1;
  const uint32_t biased = a + (mask >> 1) + ((a >> bits) & 1);
  assert(bits > 0);
  if (biased > 0xff) return 0xff;
  return biased & ~mask;
}

// Applies FindClosestDiscretized to all channels of pixel.
static uint32_t ClosestDiscretizedArgb(uint32_t a, int bits) {
  return
      (FindClosestDiscretized(a >> 24, bits) << 24) |
      (FindClosestDiscretized((a >> 16) & 0xff, bits) << 16) |
      (FindClosestDiscretized((a >> 8) & 0xff, bits) << 8) |
      (FindClosestDiscretized(a & 0xff, bits));
}

// Checks if distance between corresponding channel values of pixels a and b
// is within the given limit.
static int IsNear(uint32_t a, uint32_t b, int limit) {
  int k;
  for (k = 0; k < 4; ++k) {
    const int delta =
        (int)((a >> (k * 8)) & 0xff) - (int)((b >> (k * 8)) & 0xff);
    if (delta >= limit || delta <= -limit) {
      return 0;
    }
  }
  return 1;
}

static int IsSmooth(const uint32_t* const prev_row,
                    const uint32_t* const curr_row,
                    const uint32_t* const next_row,
                    int ix, int limit) {
  // Check that all pixels in 4-connected neighborhood are smooth.
  return (IsNear(curr_row[ix], curr_row[ix - 1], limit) &&
          IsNear(curr_row[ix], curr_row[ix + 1], limit) &&
          IsNear(curr_row[ix], prev_row[ix], limit) &&
          IsNear(curr_row[ix], next_row[ix], limit));
}

// Adjusts pixel values of image with given maximum error.
static void NearLossless(int xsize, int ysize, const uint32_t* argb_src,
                         int stride, int limit_bits, uint32_t* copy_buffer,
                         uint32_t* argb_dst) {
  int x, y;
  const int limit = 1 << limit_bits;
  uint32_t* prev_row = copy_buffer;
  uint32_t* curr_row = prev_row + xsize;
  uint32_t* next_row = curr_row + xsize;
  memcpy(curr_row, argb_src, xsize * sizeof(argb_src[0]));
  memcpy(next_row, argb_src + stride, xsize * sizeof(argb_src[0]));

  for (y = 0; y < ysize; ++y, argb_src += stride, argb_dst += xsize) {
    if (y == 0 || y == ysize - 1) {
      memcpy(argb_dst, argb_src, xsize * sizeof(argb_src[0]));
    } else {
      memcpy(next_row, argb_src + stride, xsize * sizeof(argb_src[0]));
      argb_dst[0] = argb_src[0];
      argb_dst[xsize - 1] = argb_src[xsize - 1];
      for (x = 1; x < xsize - 1; ++x) {
        if (IsSmooth(prev_row, curr_row, next_row, x, limit)) {
          argb_dst[x] = curr_row[x];
        } else {
          argb_dst[x] = ClosestDiscretizedArgb(curr_row[x], limit_bits);
        }
      }
    }
    {
      // Three-way swap.
      uint32_t* const temp = prev_row;
      prev_row = curr_row;
      curr_row = next_row;
      next_row = temp;
    }
  }
}

int VP8ApplyNearLossless(const WebPPicture* const picture, int quality,
                         uint32_t* const argb_dst) {
  int i;
  const int xsize = picture->width;
  const int ysize = picture->height;
  const int stride = picture->argb_stride;
  uint32_t* const copy_buffer =
      (uint32_t*)WebPSafeMalloc(xsize * 3, sizeof(*copy_buffer));
  const int limit_bits = VP8LNearLosslessBits(quality);
  assert(argb_dst != NULL);
  assert(limit_bits > 0);
  assert(limit_bits <= MAX_LIMIT_BITS);
  if (copy_buffer == NULL) {
    return 0;
  }
  // For small icon images, don't attempt to apply near-lossless compression.
  if ((xsize < MIN_DIM_FOR_NEAR_LOSSLESS &&
       ysize < MIN_DIM_FOR_NEAR_LOSSLESS) ||
      ysize < 3) {
    for (i = 0; i < ysize; ++i) {
      memcpy(argb_dst + i * xsize, picture->argb + i * picture->argb_stride,
             xsize * sizeof(*argb_dst));
    }
    WebPSafeFree(copy_buffer);
    return 1;
  }

  NearLossless(xsize, ysize, picture->argb, stride, limit_bits, copy_buffer,
               argb_dst);
  for (i = limit_bits - 1; i != 0; --i) {
    NearLossless(xsize, ysize, argb_dst, xsize, i, copy_buffer, argb_dst);
  }
  WebPSafeFree(copy_buffer);
  return 1;
}
#else  // (WEBP_NEAR_LOSSLESS == 1)

// Define a stub to suppress compiler warnings.
extern void VP8LNearLosslessStub(void);
void VP8LNearLosslessStub(void) {}

#endif  // (WEBP_NEAR_LOSSLESS == 1)
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
- `stdlib.h`
- `src/dsp/lossless_common.h`
- `src/enc/vp8li_enc.h`
- `src/utils/utils.h`
- `assert.h`


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

