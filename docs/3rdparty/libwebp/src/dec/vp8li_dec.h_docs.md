# Documentation for `3rdparty/libwebp/src/dec/vp8li_dec.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dec/vp8li_dec.h`
- **File Name**: `vp8li_dec.h`
- **File Size**: 5,551 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/dec/vp8li_dec.h](../../../../3rdparty/libwebp/src/dec/vp8li_dec.h)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/dec` directory and serves as part of the OpenCV library infrastructure.

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
// Lossless decoder: internal header.
//
// Author: Skal (pascal.massimino@gmail.com)
//         Vikas Arora(vikaas.arora@gmail.com)

#ifndef WEBP_DEC_VP8LI_DEC_H_
#define WEBP_DEC_VP8LI_DEC_H_

#include <string.h>     // for memcpy()
#include "src/dec/webpi_dec.h"
#include "src/utils/bit_reader_utils.h"
#include "src/utils/color_cache_utils.h"
#include "src/utils/huffman_utils.h"
#include "src/webp/types.h"

#ifdef __cplusplus
extern "C" {
#endif

typedef enum {
  READ_DATA = 0,
  READ_HDR = 1,
  READ_DIM = 2
} VP8LDecodeState;

typedef struct VP8LTransform VP8LTransform;
struct VP8LTransform {
  VP8LImageTransformType type_;   // transform type.
  int                    bits_;   // subsampling bits defining transform window.
  int                    xsize_;  // transform window X index.
  int                    ysize_;  // transform window Y index.
  uint32_t*              data_;   // transform data.
};

typedef struct {
  int             color_cache_size_;
  VP8LColorCache  color_cache_;
  VP8LColorCache  saved_color_cache_;  // for incremental

  int             huffman_mask_;
  int             huffman_subsample_bits_;
  int             huffman_xsize_;
  uint32_t*       huffman_image_;
  int             num_htree_groups_;
  HTreeGroup*     htree_groups_;
  HuffmanTables   huffman_tables_;
} VP8LMetadata;

typedef struct VP8LDecoder VP8LDecoder;
struct VP8LDecoder {
  VP8StatusCode    status_;
  VP8LDecodeState  state_;
  VP8Io*           io_;

  const WebPDecBuffer* output_;    // shortcut to io->opaque->output

  uint32_t*        pixels_;        // Internal data: either uint8_t* for alpha
                                   // or uint32_t* for BGRA.
  uint32_t*        argb_cache_;    // Scratch buffer for temporary BGRA storage.

  VP8LBitReader    br_;
  int              incremental_;   // if true, incremental decoding is expected
  VP8LBitReader    saved_br_;      // note: could be local variables too
  int              saved_last_pixel_;

  int              width_;
  int              height_;
  int              last_row_;      // last input row decoded so far.
  int              last_pixel_;    // last pixel decoded so far. However, it may
                                   // not be transformed, scaled and
                                   // color-converted yet.
  int              last_out_row_;  // last row output so far.

  VP8LMetadata     hdr_;

  int              next_transform_;
  VP8LTransform    transforms_[NUM_TRANSFORMS];
  // or'd bitset storing the transforms types.
  uint32_t         transforms_seen_;

  uint8_t*         rescaler_memory;  // Working memory for rescaling work.
  WebPRescaler*    rescaler;         // Common rescaler for all channels.
};

//------------------------------------------------------------------------------
// internal functions. Not public.

struct ALPHDecoder;  // Defined in dec/alphai.h.

// in vp8l.c

// Decodes image header for alpha data stored using lossless compression.
// Returns false in case of error.
WEBP_NODISCARD int VP8LDecodeAlphaHeader(struct ALPHDecoder* const alph_dec,
                                         const uint8_t* const data,
                                         size_t data_size);

// Decodes *at least* 'last_row' rows of alpha. If some of the initial rows are
// already decoded in previous call(s), it will resume decoding from where it
// was paused.
// Returns false in case of bitstream error.
WEBP_NODISCARD int VP8LDecodeAlphaImageStream(
    struct ALPHDecoder* const alph_dec, int last_row);

// Allocates and initialize a new lossless decoder instance.
WEBP_NODISCARD VP8LDecoder* VP8LNew(void);

// Decodes the image header. Returns false in case of error.
WEBP_NODISCARD int VP8LDecodeHeader(VP8LDecoder* const dec, VP8Io* const io);

// Decodes an image. It's required to decode the lossless header before calling
// this function. Returns false in case of error, with updated dec->status_.
WEBP_NODISCARD int VP8LDecodeImage(VP8LDecoder* const dec);

// Resets the decoder in its initial state, reclaiming memory.
// Preserves the dec->status_ value.
void VP8LClear(VP8LDecoder* const dec);

// Clears and deallocate a lossless decoder instance.
void VP8LDelete(VP8LDecoder* const dec);

// Helper function for reading the different Huffman codes and storing them in
// 'huffman_tables' and 'htree_groups'.
// If mapping is NULL 'num_htree_groups_max' must equal 'num_htree_groups'.
// If it is not NULL, it maps 'num_htree_groups_max' indices to the
// 'num_htree_groups' groups. If 'num_htree_groups_max' > 'num_htree_groups',
// some of those indices map to -1. This is used for non-balanced codes to
// limit memory usage.
WEBP_NODISCARD int ReadHuffmanCodesHelper(
    int color_cache_bits, int num_htree_groups, int num_htree_groups_max,
    const int* const mapping, VP8LDecoder* const dec,
    HuffmanTables* const huffman_tables, HTreeGroup** const htree_groups);

//------------------------------------------------------------------------------

#ifdef __cplusplus
}    // extern "C"
#endif

#endif  // WEBP_DEC_VP8LI_DEC_H_
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

- **VP8LDecoder**: A class/struct defined in this file
- **VP8LTransform**: A class/struct defined in this file
- **ALPHDecoder**: A class/struct defined in this file

### Functions and Methods

- **for()**: A function/method defined in this file
- **enum()**: A function/method defined in this file
- **WEBP_DEC_VP8LI_DEC_H_()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **struct()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/dec/webpi_dec.h`
- `src/webp/types.h`
- `src/utils/huffman_utils.h`
- `string.h`
- `src/utils/color_cache_utils.h`
- `src/utils/bit_reader_utils.h`

**Python Imports:**
- `where`


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

