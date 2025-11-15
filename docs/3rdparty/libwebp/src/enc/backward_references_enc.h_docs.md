# Documentation for `3rdparty/libwebp/src/enc/backward_references_enc.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/enc/backward_references_enc.h`
- **File Name**: `backward_references_enc.h`
- **File Size**: 8,820 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/enc/backward_references_enc.h](../../../../3rdparty/libwebp/src/enc/backward_references_enc.h)

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
// Author: Jyrki Alakuijala (jyrki@google.com)
//

#ifndef WEBP_ENC_BACKWARD_REFERENCES_ENC_H_
#define WEBP_ENC_BACKWARD_REFERENCES_ENC_H_

#include <assert.h>
#include <stdlib.h>
#include "src/webp/types.h"
#include "src/webp/encode.h"
#include "src/webp/format_constants.h"

#ifdef __cplusplus
extern "C" {
#endif

// The maximum allowed limit is 11.
#define MAX_COLOR_CACHE_BITS 10

// -----------------------------------------------------------------------------
// PixOrCopy

enum Mode {
  kLiteral,
  kCacheIdx,
  kCopy,
  kNone
};

typedef struct {
  // mode as uint8_t to make the memory layout to be exactly 8 bytes.
  uint8_t mode;
  uint16_t len;
  uint32_t argb_or_distance;
} PixOrCopy;

static WEBP_INLINE PixOrCopy PixOrCopyCreateCopy(uint32_t distance,
                                                 uint16_t len) {
  PixOrCopy retval;
  retval.mode = kCopy;
  retval.argb_or_distance = distance;
  retval.len = len;
  return retval;
}

static WEBP_INLINE PixOrCopy PixOrCopyCreateCacheIdx(int idx) {
  PixOrCopy retval;
  assert(idx >= 0);
  assert(idx < (1 << MAX_COLOR_CACHE_BITS));
  retval.mode = kCacheIdx;
  retval.argb_or_distance = idx;
  retval.len = 1;
  return retval;
}

static WEBP_INLINE PixOrCopy PixOrCopyCreateLiteral(uint32_t argb) {
  PixOrCopy retval;
  retval.mode = kLiteral;
  retval.argb_or_distance = argb;
  retval.len = 1;
  return retval;
}

static WEBP_INLINE int PixOrCopyIsLiteral(const PixOrCopy* const p) {
  return (p->mode == kLiteral);
}

static WEBP_INLINE int PixOrCopyIsCacheIdx(const PixOrCopy* const p) {
  return (p->mode == kCacheIdx);
}

static WEBP_INLINE int PixOrCopyIsCopy(const PixOrCopy* const p) {
  return (p->mode == kCopy);
}

static WEBP_INLINE uint32_t PixOrCopyLiteral(const PixOrCopy* const p,
                                             int component) {
  assert(p->mode == kLiteral);
  return (p->argb_or_distance >> (component * 8)) & 0xff;
}

static WEBP_INLINE uint32_t PixOrCopyLength(const PixOrCopy* const p) {
  return p->len;
}

static WEBP_INLINE uint32_t PixOrCopyCacheIdx(const PixOrCopy* const p) {
  assert(p->mode == kCacheIdx);
  assert(p->argb_or_distance < (1U << MAX_COLOR_CACHE_BITS));
  return p->argb_or_distance;
}

static WEBP_INLINE uint32_t PixOrCopyDistance(const PixOrCopy* const p) {
  assert(p->mode == kCopy);
  return p->argb_or_distance;
}

// -----------------------------------------------------------------------------
// VP8LHashChain

#define HASH_BITS 18
#define HASH_SIZE (1 << HASH_BITS)

// If you change this, you need MAX_LENGTH_BITS + WINDOW_SIZE_BITS <= 32 as it
// is used in VP8LHashChain.
#define MAX_LENGTH_BITS 12
#define WINDOW_SIZE_BITS 20
// We want the max value to be attainable and stored in MAX_LENGTH_BITS bits.
#define MAX_LENGTH ((1 << MAX_LENGTH_BITS) - 1)
#if MAX_LENGTH_BITS + WINDOW_SIZE_BITS > 32
#error "MAX_LENGTH_BITS + WINDOW_SIZE_BITS > 32"
#endif

typedef struct VP8LHashChain VP8LHashChain;
struct VP8LHashChain {
  // The 20 most significant bits contain the offset at which the best match
  // is found. These 20 bits are the limit defined by GetWindowSizeForHashChain
  // (through WINDOW_SIZE = 1<<20).
  // The lower 12 bits contain the length of the match. The 12 bit limit is
  // defined in MaxFindCopyLength with MAX_LENGTH=4096.
  uint32_t* offset_length_;
  // This is the maximum size of the hash_chain that can be constructed.
  // Typically this is the pixel count (width x height) for a given image.
  int size_;
};

// Must be called first, to set size.
int VP8LHashChainInit(VP8LHashChain* const p, int size);
// Pre-compute the best matches for argb. pic and percent are for progress.
int VP8LHashChainFill(VP8LHashChain* const p, int quality,
                      const uint32_t* const argb, int xsize, int ysize,
                      int low_effort, const WebPPicture* const pic,
                      int percent_range, int* const percent);
void VP8LHashChainClear(VP8LHashChain* const p);  // release memory

static WEBP_INLINE int VP8LHashChainFindOffset(const VP8LHashChain* const p,
                                               const int base_position) {
  return p->offset_length_[base_position] >> MAX_LENGTH_BITS;
}

static WEBP_INLINE int VP8LHashChainFindLength(const VP8LHashChain* const p,
                                               const int base_position) {
  return p->offset_length_[base_position] & ((1U << MAX_LENGTH_BITS) - 1);
}

static WEBP_INLINE void VP8LHashChainFindCopy(const VP8LHashChain* const p,
                                              int base_position,
                                              int* const offset_ptr,
                                              int* const length_ptr) {
  *offset_ptr = VP8LHashChainFindOffset(p, base_position);
  *length_ptr = VP8LHashChainFindLength(p, base_position);
}

// -----------------------------------------------------------------------------
// VP8LBackwardRefs (block-based backward-references storage)

// maximum number of reference blocks the image will be segmented into
#define MAX_REFS_BLOCK_PER_IMAGE 16

typedef struct PixOrCopyBlock PixOrCopyBlock;   // forward declaration
typedef struct VP8LBackwardRefs VP8LBackwardRefs;

// Container for blocks chain
struct VP8LBackwardRefs {
  int block_size_;               // common block-size
  int error_;                    // set to true if some memory error occurred
  PixOrCopyBlock* refs_;         // list of currently used blocks
  PixOrCopyBlock** tail_;        // for list recycling
  PixOrCopyBlock* free_blocks_;  // free-list
  PixOrCopyBlock* last_block_;   // used for adding new refs (internal)
};

// Initialize the object. 'block_size' is the common block size to store
// references (typically, width * height / MAX_REFS_BLOCK_PER_IMAGE).
void VP8LBackwardRefsInit(VP8LBackwardRefs* const refs, int block_size);
// Release memory for backward references.
void VP8LBackwardRefsClear(VP8LBackwardRefs* const refs);

// Cursor for iterating on references content
typedef struct {
  // public:
  PixOrCopy* cur_pos;           // current position
  // private:
  PixOrCopyBlock* cur_block_;   // current block in the refs list
  const PixOrCopy* last_pos_;   // sentinel for switching to next block
} VP8LRefsCursor;

// Returns a cursor positioned at the beginning of the references list.
VP8LRefsCursor VP8LRefsCursorInit(const VP8LBackwardRefs* const refs);
// Returns true if cursor is pointing at a valid position.
static WEBP_INLINE int VP8LRefsCursorOk(const VP8LRefsCursor* const c) {
  return (c->cur_pos != NULL);
}
// Move to next block of references. Internal, not to be called directly.
void VP8LRefsCursorNextBlock(VP8LRefsCursor* const c);
// Move to next position, or NULL. Should not be called if !VP8LRefsCursorOk().
static WEBP_INLINE void VP8LRefsCursorNext(VP8LRefsCursor* const c) {
  assert(c != NULL);
  assert(VP8LRefsCursorOk(c));
  if (++c->cur_pos == c->last_pos_) VP8LRefsCursorNextBlock(c);
}

// -----------------------------------------------------------------------------
// Main entry points

enum VP8LLZ77Type {
  kLZ77Standard = 1,
  kLZ77RLE = 2,
  kLZ77Box = 4
};

// Evaluates best possible backward references for specified quality.
// The input cache_bits to 'VP8LGetBackwardReferences' sets the maximum cache
// bits to use (passing 0 implies disabling the local color cache).
// The optimal cache bits is evaluated and set for the *cache_bits_best
// parameter with the matching refs_best.
// If do_no_cache == 0, refs is an array of 2 values and the best
// VP8LBackwardRefs is put in the first element.
// If do_no_cache != 0, refs is an array of 3 values and the best
// VP8LBackwardRefs is put in the first element, the best value with no-cache in
// the second element.
// In both cases, the last element is used as temporary internally.
// pic and percent are for progress.
// Returns false in case of error (stored in pic->error_code).
int VP8LGetBackwardReferences(
    int width, int height, const uint32_t* const argb, int quality,
    int low_effort, int lz77_types_to_try, int cache_bits_max, int do_no_cache,
    const VP8LHashChain* const hash_chain, VP8LBackwardRefs* const refs,
    int* const cache_bits_best, const WebPPicture* const pic, int percent_range,
    int* const percent);

#ifdef __cplusplus
}
#endif

#endif  // WEBP_ENC_BACKWARD_REFERENCES_ENC_H_
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

- **VP8LHashChain**: A class/struct defined in this file
- **VP8LBackwardRefs**: A class/struct defined in this file
- **PixOrCopyBlock**: A class/struct defined in this file

### Functions and Methods

- **struct()**: A function/method defined in this file
- **WEBP_ENC_BACKWARD_REFERENCES_ENC_H_()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/webp/encode.h`
- `stdlib.h`
- `src/webp/format_constants.h`
- `assert.h`
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

