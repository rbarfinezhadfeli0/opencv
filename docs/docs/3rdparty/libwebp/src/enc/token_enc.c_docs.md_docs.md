# Documentation for `docs/3rdparty/libwebp/src/enc/token_enc.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libwebp/src/enc/token_enc.c_docs.md`
- **File Name**: `token_enc.c_docs.md`
- **File Size**: 11,689 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libwebp/src/enc/token_enc.c_docs.md](../../../../../docs/3rdparty/libwebp/src/enc/token_enc.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libwebp/src/enc` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libwebp/src/enc/token_enc.c`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/enc/token_enc.c`
- **File Name**: `token_enc.c`
- **File Size**: 8,398 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/libwebp/src/enc/token_enc.c](../../../../3rdparty/libwebp/src/enc/token_enc.c)

## Purpose and Role

This file is located in the `3rdparty/libwebp/src/enc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Copyright 2011 Google Inc. All Rights Reserved.
//
// Use of this source code is governed by a BSD-style license
// that can be found in the COPYING file in the root of the source
// tree. An additional intellectual property rights grant can be found
// in the file PATENTS. All contributing project authors may
// be found in the AUTHORS file in the root of the source tree.
// -----------------------------------------------------------------------------
//
// Paginated token buffer
//
//  A 'token' is a bit value associated with a probability, either fixed
// or a later-to-be-determined after statistics have been collected.
// For dynamic probability, we just record the slot id (idx) for the probability
// value in the final probability array (uint8_t* probas in VP8EmitTokens).
//
// Author: Skal (pascal.massimino@gmail.com)

#include <assert.h>
#include <stdlib.h>
#include <string.h>

#include "src/enc/cost_enc.h"
#include "src/enc/vp8i_enc.h"
#include "src/utils/utils.h"

#if !defined(DISABLE_TOKEN_BUFFER)

// we use pages to reduce the number of memcpy()
#define MIN_PAGE_SIZE 8192          // minimum number of token per page
#define FIXED_PROBA_BIT (1u << 14)

typedef uint16_t token_t;  // bit #15: bit value
                           // bit #14: flags for constant proba or idx
                           // bits #0..13: slot or constant proba
struct VP8Tokens {
  VP8Tokens* next_;        // pointer to next page
};
// Token data is located in memory just after the next_ field.
// This macro is used to return their address and hide the trick.
#define TOKEN_DATA(p) ((const token_t*)&(p)[1])

//------------------------------------------------------------------------------

void VP8TBufferInit(VP8TBuffer* const b, int page_size) {
  b->tokens_ = NULL;
  b->pages_ = NULL;
  b->last_page_ = &b->pages_;
  b->left_ = 0;
  b->page_size_ = (page_size < MIN_PAGE_SIZE) ? MIN_PAGE_SIZE : page_size;
  b->error_ = 0;
}

void VP8TBufferClear(VP8TBuffer* const b) {
  if (b != NULL) {
    VP8Tokens* p = b->pages_;
    while (p != NULL) {
      VP8Tokens* const next = p->next_;
      WebPSafeFree(p);
      p = next;
    }
    VP8TBufferInit(b, b->page_size_);
  }
}

static int TBufferNewPage(VP8TBuffer* const b) {
  VP8Tokens* page = NULL;
  if (!b->error_) {
    const size_t size = sizeof(*page) + b->page_size_ * sizeof(token_t);
    page = (VP8Tokens*)WebPSafeMalloc(1ULL, size);
  }
  if (page == NULL) {
    b->error_ = 1;
    return 0;
  }
  page->next_ = NULL;

  *b->last_page_ = page;
  b->last_page_ = &page->next_;
  b->left_ = b->page_size_;
  b->tokens_ = (token_t*)TOKEN_DATA(page);
  return 1;
}

//------------------------------------------------------------------------------

#define TOKEN_ID(t, b, ctx) \
    (NUM_PROBAS * ((ctx) + NUM_CTX * ((b) + NUM_BANDS * (t))))

static WEBP_INLINE uint32_t AddToken(VP8TBuffer* const b, uint32_t bit,
                                     uint32_t proba_idx,
                                     proba_t* const stats) {
  assert(proba_idx < FIXED_PROBA_BIT);
  assert(bit <= 1);
  if (b->left_ > 0 || TBufferNewPage(b)) {
    const int slot = --b->left_;
    b->tokens_[slot] = (bit << 15) | proba_idx;
  }
  VP8RecordStats(bit, stats);
  return bit;
}

static WEBP_INLINE void AddConstantToken(VP8TBuffer* const b,
                                         uint32_t bit, uint32_t proba) {
  assert(proba < 256);
  assert(bit <= 1);
  if (b->left_ > 0 || TBufferNewPage(b)) {
    const int slot = --b->left_;
    b->tokens_[slot] = (bit << 15) | FIXED_PROBA_BIT | proba;
  }
}

int VP8RecordCoeffTokens(int ctx, const struct VP8Residual* const res,
                         VP8TBuffer* const tokens) {
  const int16_t* const coeffs = res->coeffs;
  const int coeff_type = res->coeff_type;
  const int last = res->last;
  int n = res->first;
  uint32_t base_id = TOKEN_ID(coeff_type, n, ctx);
  // should be stats[VP8EncBands[n]], but it's equivalent for n=0 or 1
  proba_t* s = res->stats[n][ctx];
  if (!AddToken(tokens, last >= 0, base_id + 0, s + 0)) {
    return 0;
  }

  while (n < 16) {
    const int c = coeffs[n++];
    const int sign = c < 0;
    const uint32_t v = sign ? -c : c;
    if (!AddToken(tokens, v != 0, base_id + 1, s + 1)) {
      base_id = TOKEN_ID(coeff_type, VP8EncBands[n], 0);  // ctx=0
      s = res->stats[VP8EncBands[n]][0];
      continue;
    }
    if (!AddToken(tokens, v > 1, base_id + 2, s + 2)) {
      base_id = TOKEN_ID(coeff_type, VP8EncBands[n], 1);  // ctx=1
      s = res->stats[VP8EncBands[n]][1];
    } else {
      if (!AddToken(tokens, v > 4, base_id + 3, s + 3)) {
        if (AddToken(tokens, v != 2, base_id + 4, s + 4)) {
          AddToken(tokens, v == 4, base_id + 5, s + 5);
        }
      } else if (!AddToken(tokens, v > 10, base_id + 6, s + 6)) {
        if (!AddToken(tokens, v > 6, base_id + 7, s + 7)) {
          AddConstantToken(tokens, v == 6, 159);
        } else {
          AddConstantToken(tokens, v >= 9, 165);
          AddConstantToken(tokens, !(v & 1), 145);
        }
      } else {
        int mask;
        const uint8_t* tab;
        uint32_t residue = v - 3;
        if (residue < (8 << 1)) {          // VP8Cat3  (3b)
          AddToken(tokens, 0, base_id + 8, s + 8);
          AddToken(tokens, 0, base_id + 9, s + 9);
          residue -= (8 << 0);
          mask = 1 << 2;
          tab = VP8Cat3;
        } else if (residue < (8 << 2)) {   // VP8Cat4  (4b)
          AddToken(tokens, 0, base_id + 8, s + 8);
          AddToken(tokens, 1, base_id + 9, s + 9);
          residue -= (8 << 1);
          mask = 1 << 3;
          tab = VP8Cat4;
        } else if (residue < (8 << 3)) {   // VP8Cat5  (5b)
          AddToken(tokens, 1, base_id + 8, s + 8);
          AddToken(tokens, 0, base_id + 10, s + 9);
          residue -= (8 << 2);
          mask = 1 << 4;
          tab = VP8Cat5;
        } else {                         // VP8Cat6 (11b)
          AddToken(tokens, 1, base_id + 8, s + 8);
          AddToken(tokens, 1, base_id + 10, s + 9);
          residue -= (8 << 3);
          mask = 1 << 10;
          tab = VP8Cat6;
        }
        while (mask) {
          AddConstantToken(tokens, !!(residue & mask), *tab++);
          mask >>= 1;
        }
      }
      base_id = TOKEN_ID(coeff_type, VP8EncBands[n], 2);  // ctx=2
      s = res->stats[VP8EncBands[n]][2];
    }
    AddConstantToken(tokens, sign, 128);
    if (n == 16 || !AddToken(tokens, n <= last, base_id + 0, s + 0)) {
      return 1;   // EOB
    }
  }
  return 1;
}

#undef TOKEN_ID

//------------------------------------------------------------------------------
// Final coding pass, with known probabilities

int VP8EmitTokens(VP8TBuffer* const b, VP8BitWriter* const bw,
                  const uint8_t* const probas, int final_pass) {
  const VP8Tokens* p = b->pages_;
  assert(!b->error_);
  while (p != NULL) {
    const VP8Tokens* const next = p->next_;
    const int N = (next == NULL) ? b->left_ : 0;
    int n = b->page_size_;
    const token_t* const tokens = TOKEN_DATA(p);
    while (n-- > N) {
      const token_t token = tokens[n];
      const int bit = (token >> 15) & 1;
      if (token & FIXED_PROBA_BIT) {
        VP8PutBit(bw, bit, token & 0xffu);  // constant proba
      } else {
        VP8PutBit(bw, bit, probas[token & 0x3fffu]);
      }
    }
    if (final_pass) WebPSafeFree((void*)p);
    p = next;
  }
  if (final_pass) b->pages_ = NULL;
  return 1;
}

// Size estimation
size_t VP8EstimateTokenSize(VP8TBuffer* const b, const uint8_t* const probas) {
  size_t size = 0;
  const VP8Tokens* p = b->pages_;
  assert(!b->error_);
  while (p != NULL) {
    const VP8Tokens* const next = p->next_;
    const int N = (next == NULL) ? b->left_ : 0;
    int n = b->page_size_;
    const token_t* const tokens = TOKEN_DATA(p);
    while (n-- > N) {
      const token_t token = tokens[n];
      const int bit = token & (1 << 15);
      if (token & FIXED_PROBA_BIT) {
        size += VP8BitCost(bit, token & 0xffu);
      } else {
        size += VP8BitCost(bit, probas[token & 0x3fffu]);
      }
    }
    p = next;
  }
  return size;
}

//------------------------------------------------------------------------------

#else     // DISABLE_TOKEN_BUFFER

void VP8TBufferInit(VP8TBuffer* const b, int page_size) {
  (void)b;
  (void)page_size;
}
void VP8TBufferClear(VP8TBuffer* const b) {
  (void)b;
}

#endif    // !DISABLE_TOKEN_BUFFER

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

### Classes and Structures

- **VP8Tokens**: A class/struct defined in this file
- **VP8Residual**: A class/struct defined in this file

### Functions and Methods

- **TOKEN_ID()**: A function/method defined in this file
- **uint16_t()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/enc/cost_enc.h`
- `src/enc/vp8i_enc.h`
- `stdlib.h`
- `string.h`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

