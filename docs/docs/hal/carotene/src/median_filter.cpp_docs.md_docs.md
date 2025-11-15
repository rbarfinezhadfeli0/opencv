# Documentation for `docs/hal/carotene/src/median_filter.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/carotene/src/median_filter.cpp_docs.md`
- **File Name**: `median_filter.cpp_docs.md`
- **File Size**: 11,862 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/carotene/src/median_filter.cpp_docs.md](../../../../docs/hal/carotene/src/median_filter.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/carotene/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/carotene/src/median_filter.cpp`

## File Metadata

- **Full Path**: `hal/carotene/src/median_filter.cpp`
- **File Name**: `median_filter.cpp`
- **File Size**: 8,721 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/carotene/src/median_filter.cpp](../../../hal/carotene/src/median_filter.cpp)

## Purpose and Role

This file is located in the `hal/carotene/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * By downloading, copying, installing or using the software you agree to this license.
 * If you do not agree to this license, do not download, install,
 * copy or use the software.
 *
 *
 *                           License Agreement
 *                For Open Source Computer Vision Library
 *                        (3-clause BSD License)
 *
 * Copyright (C) 2012-2014, NVIDIA Corporation, all rights reserved.
 * Third party copyrights are property of their respective owners.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 *   * Redistributions of source code must retain the above copyright notice,
 *     this list of conditions and the following disclaimer.
 *
 *   * Redistributions in binary form must reproduce the above copyright notice,
 *     this list of conditions and the following disclaimer in the documentation
 *     and/or other materials provided with the distribution.
 *
 *   * Neither the names of the copyright holders nor the names of the contributors
 *     may be used to endorse or promote products derived from this software
 *     without specific prior written permission.
 *
 * This software is provided by the copyright holders and contributors "as is" and
 * any express or implied warranties, including, but not limited to, the implied
 * warranties of merchantability and fitness for a particular purpose are disclaimed.
 * In no event shall copyright holders or contributors be liable for any direct,
 * indirect, incidental, special, exemplary, or consequential damages
 * (including, but not limited to, procurement of substitute goods or services;
 * loss of use, data, or profits; or business interruption) however caused
 * and on any theory of liability, whether in contract, strict liability,
 * or tort (including negligence or otherwise) arising in any way out of
 * the use of this software, even if advised of the possibility of such damage.
 */

#include "common.hpp"

/*
 * The code here is based on the code in
 * <http://ndevilla.free.fr/median/median/src/optmed.c>, which is in public domain.
 * See also <http://ndevilla.free.fr/median/median/index.html>.
 */

namespace CAROTENE_NS {

#ifdef CAROTENE_NEON
namespace {

    uint8x16_t getLeftReplicate(uint8x16_t r, u32 cn)
    {
        u8 buf[16+8];
        vst1q_u8(buf+cn, r);
        for (u32 i = 0; i < cn; ++i) buf[i] = buf[cn+i];
        return vld1q_u8(buf);
    }

    uint8x8_t getRightReplicate(uint8x8_t r, u32 cn)
    {
        u8 buf[8+8];
        vst1_u8(buf, r);
        for (u32 i = 0; i < cn; ++i) buf[8+i] = buf[8-cn+i];
        return vld1_u8(buf+cn);
    }

} // namespace

//o------^-------^-----------------------------o 0
//       |       |
//o--^---v---^---|-------^---------------------o 1
//   |       |   |       |
//o--v-------v---|-------|-^-------^-------^---o 2
//               |       | |       |       |
//o------^-------v-----^-|-|-------|-------|---o 3
//       |             | | |       |       |
//o--^---v---^-----^---|-v-|---^---v---^---v---o 4
//   |       |     |   |   |   |       |
//o--v-------v---^-|---|---v---|-------|-------o 5
//               | |   |       |       |
//o------^-------|-|---v-------|-------v-------o 6
//       |       | |           |
//o--^---v---^---|-v-----------v---------------o 7
//   |       |   |
//o--v-------v---v-----------------------------o 8

#define ELT(num, level) v ## num ## _lv ## level
#define PIX_SORT(a, alvl, b, blvl, newlvl) \
    PIX_MIN(a, alvl, b, blvl, newlvl); \
    PIX_MAX(a, alvl, b, blvl, newlvl);

#define SORT9 \
    PIX_SORT(1, 00, 2, 00, 01); \
    PIX_SORT(4, 00, 5, 00, 02); \
    PIX_SORT(7, 00, 8, 00, 03); \
    PIX_SORT(0, 00, 1, 01, 04); \
    PIX_SORT(3, 00, 4, 02, 05); \
    PIX_SORT(6, 00, 7, 03, 06); \
    PIX_SORT(1, 04, 2, 01, 07); \
    PIX_SORT(4, 05, 5, 02, 08); \
    PIX_SORT(7, 06, 8, 03, 09); \
    PIX_MAX (0, 04, 3, 05, 10); \
    PIX_MIN (5, 08, 8, 09, 11); \
    PIX_SORT(4, 08, 7, 09, 12); \
    PIX_MAX (3, 10, 6, 06, 13); \
    PIX_MAX (1, 07, 4, 12, 14); \
    PIX_MIN (2, 07, 5, 11, 15); \
    PIX_MIN (4, 14, 7, 12, 16); \
    PIX_SORT(4, 16, 2, 15, 17); \
    PIX_MAX (6, 13, 4, 17, 18); \
    PIX_MIN (4, 18, 2, 17, 19);

#endif

bool isMedianFilter3x3Supported(const Size2D &size, u32 numChannels)
{
    return isSupportedConfiguration() && size.width >= 16 + numChannels && numChannels <= 8;
}

void medianFilter3x3(const Size2D &size, u32 numChannels,
                     const u8 *srcBase, ptrdiff_t srcStride,
                     const Margin &srcMargin,
                     u8 *dstBase, ptrdiff_t dstStride)
{
    internal::assertSupportedConfiguration(isMedianFilter3x3Supported(size, numChannels));
#ifdef CAROTENE_NEON
    u32 cn = numChannels;
    size_t colsn = size.width * cn;

    for (size_t i = 0; i < size.height; ++i) {
        const u8* psrc1 = internal::getRowPtr(srcBase, srcStride, i);
        const u8* psrc0 = i == 0 && srcMargin.top == 0 ? psrc1 : psrc1 - srcStride;
        const u8* psrc2 = i + 1 == size.height && srcMargin.bottom == 0 ? psrc1 : psrc1 + srcStride;
        u8* pdst = internal::getRowPtr(dstBase, dstStride, i);
        size_t j = 0;

        {
            uint8x16_t v3_lv00 = vld1q_u8(psrc0);
            uint8x16_t v4_lv00 = vld1q_u8(psrc1);
            uint8x16_t v5_lv00 = vld1q_u8(psrc2);
            uint8x16_t v6_lv00 = vld1q_u8(psrc0 + cn);
            uint8x16_t v7_lv00 = vld1q_u8(psrc1 + cn);
            uint8x16_t v8_lv00 = vld1q_u8(psrc2 + cn);
            uint8x16_t v0_lv00 = srcMargin.left > 0 ? vld1q_u8(psrc0 - cn) : getLeftReplicate(v3_lv00, cn);
            uint8x16_t v1_lv00 = srcMargin.left > 0 ? vld1q_u8(psrc1 - cn) : getLeftReplicate(v4_lv00, cn);
            uint8x16_t v2_lv00 = srcMargin.left > 0 ? vld1q_u8(psrc2 - cn) : getLeftReplicate(v5_lv00, cn);

            goto medianBlur3x3_mainBody;

            for (; j < colsn - 16; j += 16) {
                internal::prefetch(psrc0 + j);
                internal::prefetch(psrc1 + j);
                internal::prefetch(psrc2 + j);

                v0_lv00 = vld1q_u8(psrc0 + j - cn);
                v1_lv00 = vld1q_u8(psrc1 + j - cn);
                v2_lv00 = vld1q_u8(psrc2 + j - cn);
                v3_lv00 = vld1q_u8(psrc0 + j);
                v4_lv00 = vld1q_u8(psrc1 + j);
                v5_lv00 = vld1q_u8(psrc2 + j);
                v6_lv00 = vld1q_u8(psrc0 + j + cn);
                v7_lv00 = vld1q_u8(psrc1 + j + cn);
                v8_lv00 = vld1q_u8(psrc2 + j + cn);

medianBlur3x3_mainBody:

#define PIX_MIN(a, alvl, b, blvl, newlvl) uint8x16_t ELT(a, newlvl) = vminq_u8(ELT(a, alvl), ELT(b, blvl))
#define PIX_MAX(a, alvl, b, blvl, newlvl) uint8x16_t ELT(b, newlvl) = vmaxq_u8(ELT(a, alvl), ELT(b, blvl))
                SORT9;
#undef PIX_MAX
#undef PIX_MIN

                vst1q_u8(pdst + j, v4_lv19);
            }
        }

        {
            size_t k = colsn - 8;
            uint8x8_t v0_lv00 = vld1_u8(psrc0 + k - cn);
            uint8x8_t v1_lv00 = vld1_u8(psrc1 + k - cn);
            uint8x8_t v2_lv00 = vld1_u8(psrc2 + k - cn);
            uint8x8_t v3_lv00 = vld1_u8(psrc0 + k);
            uint8x8_t v4_lv00 = vld1_u8(psrc1 + k);
            uint8x8_t v5_lv00 = vld1_u8(psrc2 + k);
            uint8x8_t v6_lv00 = srcMargin.right > 0 ? vld1_u8(psrc0 + k + cn) : getRightReplicate(v3_lv00, cn);
            uint8x8_t v7_lv00 = srcMargin.right > 0 ? vld1_u8(psrc1 + k + cn) : getRightReplicate(v4_lv00, cn);
            uint8x8_t v8_lv00 = srcMargin.right > 0 ? vld1_u8(psrc2 + k + cn) : getRightReplicate(v5_lv00, cn);

            goto medianBlur3x3_tailBody;

            for (; k >= j - 8; k -= 8) {
                v0_lv00 = vld1_u8(psrc0 + k - cn);
                v1_lv00 = vld1_u8(psrc1 + k - cn);
                v2_lv00 = vld1_u8(psrc2 + k - cn);
                v3_lv00 = vld1_u8(psrc0 + k);
                v4_lv00 = vld1_u8(psrc1 + k);
                v5_lv00 = vld1_u8(psrc2 + k);
                v6_lv00 = vld1_u8(psrc0 + k + cn);
                v7_lv00 = vld1_u8(psrc1 + k + cn);
                v8_lv00 = vld1_u8(psrc2 + k + cn);

medianBlur3x3_tailBody:

#define PIX_MIN(a, alvl, b, blvl, newlvl) uint8x8_t ELT(a, newlvl) = vmin_u8(ELT(a, alvl), ELT(b, blvl))
#define PIX_MAX(a, alvl, b, blvl, newlvl) uint8x8_t ELT(b, newlvl) = vmax_u8(ELT(a, alvl), ELT(b, blvl))
                SORT9;
#undef PIX_MAX
#undef PIX_MIN

                vst1_u8(pdst + k, v4_lv19);
            }
        }
    }
#else
    (void)size;
    (void)numChannels;
    (void)srcBase;
    (void)srcStride;
    (void)srcMargin;
    (void)dstBase;
    (void)dstStride;
#endif
}

} // namespace CAROTENE_NS
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

- **PIX_MIN()**: A function/method defined in this file
- **CAROTENE_NEON()**: A function/method defined in this file
- **PIX_MAX()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `common.hpp`

**Python Imports:**
- `this`


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

