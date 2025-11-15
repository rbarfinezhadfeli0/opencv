# Documentation for `docs/3rdparty/libwebp/src/dsp/lossless.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/libwebp/src/dsp/lossless.h_docs.md`
- **File Name**: `lossless.h_docs.md`
- **File Size**: 15,594 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/libwebp/src/dsp/lossless.h_docs.md](../../../../../docs/3rdparty/libwebp/src/dsp/lossless.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/libwebp/src/dsp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/libwebp/src/dsp/lossless.h`

## File Metadata

- **Full Path**: `3rdparty/libwebp/src/dsp/lossless.h`
- **File Name**: `lossless.h`
- **File Size**: 11,950 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/libwebp/src/dsp/lossless.h](../../../../3rdparty/libwebp/src/dsp/lossless.h)

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

#ifndef WEBP_DSP_LOSSLESS_H_
#define WEBP_DSP_LOSSLESS_H_

#include "src/webp/types.h"
#include "src/webp/decode.h"

#include "src/enc/histogram_enc.h"
#include "src/utils/utils.h"

#ifdef __cplusplus
extern "C" {
#endif

//------------------------------------------------------------------------------
// Decoding

typedef uint32_t (*VP8LPredictorFunc)(const uint32_t* const left,
                                      const uint32_t* const top);
extern VP8LPredictorFunc VP8LPredictors[16];

uint32_t VP8LPredictor0_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor1_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor2_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor3_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor4_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor5_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor6_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor7_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor8_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor9_C(const uint32_t* const left,
                          const uint32_t* const top);
uint32_t VP8LPredictor10_C(const uint32_t* const left,
                           const uint32_t* const top);
uint32_t VP8LPredictor11_C(const uint32_t* const left,
                           const uint32_t* const top);
uint32_t VP8LPredictor12_C(const uint32_t* const left,
                           const uint32_t* const top);
uint32_t VP8LPredictor13_C(const uint32_t* const left,
                           const uint32_t* const top);

// These Add/Sub function expects upper[-1] and out[-1] to be readable.
typedef void (*VP8LPredictorAddSubFunc)(const uint32_t* in,
                                        const uint32_t* upper, int num_pixels,
                                        uint32_t* out);
extern VP8LPredictorAddSubFunc VP8LPredictorsAdd[16];
extern VP8LPredictorAddSubFunc VP8LPredictorsAdd_C[16];

typedef void (*VP8LProcessDecBlueAndRedFunc)(const uint32_t* src,
                                             int num_pixels, uint32_t* dst);
extern VP8LProcessDecBlueAndRedFunc VP8LAddGreenToBlueAndRed;

typedef struct {
  // Note: the members are uint8_t, so that any negative values are
  // automatically converted to "mod 256" values.
  uint8_t green_to_red_;
  uint8_t green_to_blue_;
  uint8_t red_to_blue_;
} VP8LMultipliers;
typedef void (*VP8LTransformColorInverseFunc)(const VP8LMultipliers* const m,
                                              const uint32_t* src,
                                              int num_pixels, uint32_t* dst);
extern VP8LTransformColorInverseFunc VP8LTransformColorInverse;

struct VP8LTransform;  // Defined in dec/vp8li.h.

// Performs inverse transform of data given transform information, start and end
// rows. Transform will be applied to rows [row_start, row_end[.
// The *in and *out pointers refer to source and destination data respectively
// corresponding to the intermediate row (row_start).
void VP8LInverseTransform(const struct VP8LTransform* const transform,
                          int row_start, int row_end,
                          const uint32_t* const in, uint32_t* const out);

// Color space conversion.
typedef void (*VP8LConvertFunc)(const uint32_t* src, int num_pixels,
                                uint8_t* dst);
extern VP8LConvertFunc VP8LConvertBGRAToRGB;
extern VP8LConvertFunc VP8LConvertBGRAToRGBA;
extern VP8LConvertFunc VP8LConvertBGRAToRGBA4444;
extern VP8LConvertFunc VP8LConvertBGRAToRGB565;
extern VP8LConvertFunc VP8LConvertBGRAToBGR;

// Converts from BGRA to other color spaces.
void VP8LConvertFromBGRA(const uint32_t* const in_data, int num_pixels,
                         WEBP_CSP_MODE out_colorspace, uint8_t* const rgba);

typedef void (*VP8LMapARGBFunc)(const uint32_t* src,
                                const uint32_t* const color_map,
                                uint32_t* dst, int y_start,
                                int y_end, int width);
typedef void (*VP8LMapAlphaFunc)(const uint8_t* src,
                                 const uint32_t* const color_map,
                                 uint8_t* dst, int y_start,
                                 int y_end, int width);

extern VP8LMapARGBFunc VP8LMapColor32b;
extern VP8LMapAlphaFunc VP8LMapColor8b;

// Similar to the static method ColorIndexInverseTransform() that is part of
// lossless.c, but used only for alpha decoding. It takes uint8_t (rather than
// uint32_t) arguments for 'src' and 'dst'.
void VP8LColorIndexInverseTransformAlpha(
    const struct VP8LTransform* const transform, int y_start, int y_end,
    const uint8_t* src, uint8_t* dst);

// Expose some C-only fallback functions
void VP8LTransformColorInverse_C(const VP8LMultipliers* const m,
                                 const uint32_t* src, int num_pixels,
                                 uint32_t* dst);

void VP8LConvertBGRAToRGB_C(const uint32_t* src, int num_pixels, uint8_t* dst);
void VP8LConvertBGRAToRGBA_C(const uint32_t* src, int num_pixels, uint8_t* dst);
void VP8LConvertBGRAToRGBA4444_C(const uint32_t* src,
                                 int num_pixels, uint8_t* dst);
void VP8LConvertBGRAToRGB565_C(const uint32_t* src,
                               int num_pixels, uint8_t* dst);
void VP8LConvertBGRAToBGR_C(const uint32_t* src, int num_pixels, uint8_t* dst);
void VP8LAddGreenToBlueAndRed_C(const uint32_t* src, int num_pixels,
                                uint32_t* dst);

// Must be called before calling any of the above methods.
void VP8LDspInit(void);

//------------------------------------------------------------------------------
// Encoding

typedef void (*VP8LProcessEncBlueAndRedFunc)(uint32_t* dst, int num_pixels);
extern VP8LProcessEncBlueAndRedFunc VP8LSubtractGreenFromBlueAndRed;
typedef void (*VP8LTransformColorFunc)(const VP8LMultipliers* const m,
                                       uint32_t* dst, int num_pixels);
extern VP8LTransformColorFunc VP8LTransformColor;
typedef void (*VP8LCollectColorBlueTransformsFunc)(
    const uint32_t* argb, int stride,
    int tile_width, int tile_height,
    int green_to_blue, int red_to_blue, int histo[]);
extern VP8LCollectColorBlueTransformsFunc VP8LCollectColorBlueTransforms;

typedef void (*VP8LCollectColorRedTransformsFunc)(
    const uint32_t* argb, int stride,
    int tile_width, int tile_height,
    int green_to_red, int histo[]);
extern VP8LCollectColorRedTransformsFunc VP8LCollectColorRedTransforms;

// Expose some C-only fallback functions
void VP8LTransformColor_C(const VP8LMultipliers* const m,
                          uint32_t* data, int num_pixels);
void VP8LSubtractGreenFromBlueAndRed_C(uint32_t* argb_data, int num_pixels);
void VP8LCollectColorRedTransforms_C(const uint32_t* argb, int stride,
                                     int tile_width, int tile_height,
                                     int green_to_red, int histo[]);
void VP8LCollectColorBlueTransforms_C(const uint32_t* argb, int stride,
                                      int tile_width, int tile_height,
                                      int green_to_blue, int red_to_blue,
                                      int histo[]);

extern VP8LPredictorAddSubFunc VP8LPredictorsSub[16];
extern VP8LPredictorAddSubFunc VP8LPredictorsSub_C[16];

// -----------------------------------------------------------------------------
// Huffman-cost related functions.

typedef uint32_t (*VP8LCostFunc)(const uint32_t* population, int length);
typedef uint32_t (*VP8LCostCombinedFunc)(const uint32_t* X, const uint32_t* Y,
                                         int length);
typedef float (*VP8LCombinedShannonEntropyFunc)(const int X[256],
                                                const int Y[256]);

extern VP8LCostFunc VP8LExtraCost;
extern VP8LCostCombinedFunc VP8LExtraCostCombined;
extern VP8LCombinedShannonEntropyFunc VP8LCombinedShannonEntropy;

typedef struct {        // small struct to hold counters
  int counts[2];        // index: 0=zero streak, 1=non-zero streak
  int streaks[2][2];    // [zero/non-zero][streak<3 / streak>=3]
} VP8LStreaks;

typedef struct {            // small struct to hold bit entropy results
  float entropy;            // entropy
  uint32_t sum;             // sum of the population
  int nonzeros;             // number of non-zero elements in the population
  uint32_t max_val;         // maximum value in the population
  uint32_t nonzero_code;    // index of the last non-zero in the population
} VP8LBitEntropy;

void VP8LBitEntropyInit(VP8LBitEntropy* const entropy);

// Get the combined symbol bit entropy and Huffman cost stats for the
// distributions 'X' and 'Y'. Those results can then be refined according to
// codec specific heuristics.
typedef void (*VP8LGetCombinedEntropyUnrefinedFunc)(
    const uint32_t X[], const uint32_t Y[], int length,
    VP8LBitEntropy* const bit_entropy, VP8LStreaks* const stats);
extern VP8LGetCombinedEntropyUnrefinedFunc VP8LGetCombinedEntropyUnrefined;

// Get the entropy for the distribution 'X'.
typedef void (*VP8LGetEntropyUnrefinedFunc)(const uint32_t X[], int length,
                                            VP8LBitEntropy* const bit_entropy,
                                            VP8LStreaks* const stats);
extern VP8LGetEntropyUnrefinedFunc VP8LGetEntropyUnrefined;

void VP8LBitsEntropyUnrefined(const uint32_t* const array, int n,
                              VP8LBitEntropy* const entropy);

typedef void (*VP8LAddVectorFunc)(const uint32_t* a, const uint32_t* b,
                                  uint32_t* out, int size);
extern VP8LAddVectorFunc VP8LAddVector;
typedef void (*VP8LAddVectorEqFunc)(const uint32_t* a, uint32_t* out, int size);
extern VP8LAddVectorEqFunc VP8LAddVectorEq;
void VP8LHistogramAdd(const VP8LHistogram* const a,
                      const VP8LHistogram* const b,
                      VP8LHistogram* const out);

// -----------------------------------------------------------------------------
// PrefixEncode()

typedef int (*VP8LVectorMismatchFunc)(const uint32_t* const array1,
                                      const uint32_t* const array2, int length);
// Returns the first index where array1 and array2 are different.
extern VP8LVectorMismatchFunc VP8LVectorMismatch;

typedef void (*VP8LBundleColorMapFunc)(const uint8_t* const row, int width,
                                       int xbits, uint32_t* dst);
extern VP8LBundleColorMapFunc VP8LBundleColorMap;
void VP8LBundleColorMap_C(const uint8_t* const row, int width, int xbits,
                          uint32_t* dst);

// Must be called before calling any of the above methods.
void VP8LEncDspInit(void);

//------------------------------------------------------------------------------

#ifdef __cplusplus
}    // extern "C"
#endif

#endif  // WEBP_DSP_LOSSLESS_H_
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

- **to**: A class/struct defined in this file
- **VP8LTransform**: A class/struct defined in this file

### Functions and Methods

- **uint32_t()**: A function/method defined in this file
- **expects()**: A function/method defined in this file
- **WEBP_DSP_LOSSLESS_H_()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **struct()**: A function/method defined in this file
- **float()**: A function/method defined in this file
- **int()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `src/webp/decode.h`
- `src/enc/histogram_enc.h`
- `src/utils/utils.h`
- `src/webp/types.h`

**Python Imports:**
- `BGRA`


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

