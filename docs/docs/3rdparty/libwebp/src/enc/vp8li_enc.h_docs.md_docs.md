# Documentation for `docs/3rdparty/libwebp/src/enc/vp8li_enc.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libwebp/src/enc/vp8li_enc.h_docs.md`
- **File Name**: `vp8li_enc.h_docs.md`
- **File Size**: 8,266 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libwebp/src/enc/vp8li_enc.h_docs.md](../../../../../docs/3rdparty/libwebp/src/enc/vp8li_enc.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libwebp/src/enc` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libwebp/src/enc/vp8li_enc.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/enc/vp8li_enc.h`
- **File Name**: `vp8li_enc.h`
- **File Size**: 4,655 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/enc/vp8li_enc.h](../../../../3rdparty/libwebp/src/enc/vp8li_enc.h)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/enc` directory and serves as part of the OpenCV library infrastructure.

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
// Lossless encoder: internal header.
//
// Author: Vikas Arora (vikaas.arora@gmail.com)

#ifndef WEBP_ENC_VP8LI_ENC_H_
#define WEBP_ENC_VP8LI_ENC_H_

#ifdef HAVE_CONFIG_H
#include "src/webp/config.h"
#endif
// Either WEBP_NEAR_LOSSLESS is defined as 0 in config.h when compiling to
// disable near-lossless, or it is enabled by default.
#ifndef WEBP_NEAR_LOSSLESS
#define WEBP_NEAR_LOSSLESS 1
#endif

#include "src/enc/backward_references_enc.h"
#include "src/enc/histogram_enc.h"
#include "src/utils/bit_writer_utils.h"
#include "src/webp/encode.h"
#include "src/webp/format_constants.h"

#ifdef __cplusplus
extern "C" {
#endif

// maximum value of transform_bits_ in VP8LEncoder.
#define MAX_TRANSFORM_BITS 6

typedef enum {
  kEncoderNone = 0,
  kEncoderARGB,
  kEncoderNearLossless,
  kEncoderPalette
} VP8LEncoderARGBContent;

typedef struct {
  const WebPConfig* config_;      // user configuration and parameters
  const WebPPicture* pic_;        // input picture.

  uint32_t* argb_;                       // Transformed argb image data.
  VP8LEncoderARGBContent argb_content_;  // Content type of the argb buffer.
  uint32_t* argb_scratch_;               // Scratch memory for argb rows
                                         // (used for prediction).
  uint32_t* transform_data_;             // Scratch memory for transform data.
  uint32_t* transform_mem_;              // Currently allocated memory.
  size_t    transform_mem_size_;         // Currently allocated memory size.

  int       current_width_;       // Corresponds to packed image width.

  // Encoding parameters derived from quality parameter.
  int histo_bits_;
  int transform_bits_;    // <= MAX_TRANSFORM_BITS.
  int cache_bits_;        // If equal to 0, don't use color cache.

  // Encoding parameters derived from image characteristics.
  int use_cross_color_;
  int use_subtract_green_;
  int use_predict_;
  int use_palette_;
  int palette_size_;
  uint32_t palette_[MAX_PALETTE_SIZE];
  // Sorted version of palette_ for cache purposes.
  uint32_t palette_sorted_[MAX_PALETTE_SIZE];

  // Some 'scratch' (potentially large) objects.
  struct VP8LBackwardRefs refs_[4];  // Backward Refs array for temporaries.
  VP8LHashChain hash_chain_;         // HashChain data for constructing
                                     // backward references.
} VP8LEncoder;

//------------------------------------------------------------------------------
// internal functions. Not public.

// Encodes the picture.
// Returns 0 if config or picture is NULL or picture doesn't have valid argb
// input.
int VP8LEncodeImage(const WebPConfig* const config,
                    const WebPPicture* const picture);

// Encodes the main image stream using the supplied bit writer.
// Returns false in case of error (stored in picture->error_code).
int VP8LEncodeStream(const WebPConfig* const config,
                     const WebPPicture* const picture, VP8LBitWriter* const bw);

#if (WEBP_NEAR_LOSSLESS == 1)
// in near_lossless.c
// Near lossless preprocessing in RGB color-space.
int VP8ApplyNearLossless(const WebPPicture* const picture, int quality,
                         uint32_t* const argb_dst);
#endif

//------------------------------------------------------------------------------
// Image transforms in predictor.c.

// pic and percent are for progress.
// Returns false in case of error (stored in pic->error_code).
int VP8LResidualImage(int width, int height, int bits, int low_effort,
                      uint32_t* const argb, uint32_t* const argb_scratch,
                      uint32_t* const image, int near_lossless, int exact,
                      int used_subtract_green, const WebPPicture* const pic,
                      int percent_range, int* const percent);

int VP8LColorSpaceTransform(int width, int height, int bits, int quality,
                            uint32_t* const argb, uint32_t* image,
                            const WebPPicture* const pic, int percent_range,
                            int* const percent);

//------------------------------------------------------------------------------

#ifdef __cplusplus
}    // extern "C"
#endif

#endif  // WEBP_ENC_VP8LI_ENC_H_
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

### Classes and Structures

- **VP8LBackwardRefs**: A class/struct defined in this file

### Functions and Methods

- **enum()**: A function/method defined in this file
- **HAVE_CONFIG_H()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **struct()**: A function/method defined in this file
- **WEBP_NEAR_LOSSLESS()**: A function/method defined in this file
- **WEBP_ENC_VP8LI_ENC_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/webp/encode.h`
- `src/enc/histogram_enc.h`
- `src/utils/bit_writer_utils.h`
- `src/webp/config.h`
- `src/enc/backward_references_enc.h`
- `src/webp/format_constants.h`

**Python Imports:**
- `quality`
- `image`


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

