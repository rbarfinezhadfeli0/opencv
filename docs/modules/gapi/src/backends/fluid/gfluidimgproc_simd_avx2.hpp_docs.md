# Documentation for `modules/gapi/src/backends/fluid/gfluidimgproc_simd_avx2.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/fluid/gfluidimgproc_simd_avx2.hpp`
- **File Name**: `gfluidimgproc_simd_avx2.hpp`
- **File Size**: 6,645 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/backends/fluid/gfluidimgproc_simd_avx2.hpp](../../../../../modules/gapi/src/backends/fluid/gfluidimgproc_simd_avx2.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/fluid` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#if !defined(GAPI_STANDALONE)

#include "opencv2/gapi/own/saturate.hpp"

#include <immintrin.h>

#include "opencv2/core.hpp"

#include <opencv2/core/hal/intrin.hpp>

#include <cstdint>
#include <cstring>

#include <algorithm>
#include <limits>
#include <vector>

namespace cv {
namespace gapi {
namespace fluid {
namespace avx2 {

CV_ALWAYS_INLINE void v_gather_pairs(const float src[], const int* mapsx,
                                     v_float32x8& low, v_float32x8& high)
{
    low.val = _mm256_castsi256_ps(_mm256_setr_epi64x(*reinterpret_cast<const int64_t*>(&src[mapsx[0]]),
                                                     *reinterpret_cast<const int64_t*>(&src[mapsx[1]]),
                                                     *reinterpret_cast<const int64_t*>(&src[mapsx[2]]),
                                                     *reinterpret_cast<const int64_t*>(&src[mapsx[3]])));
    high.val = _mm256_castsi256_ps(_mm256_setr_epi64x(*reinterpret_cast<const int64_t*>(&src[mapsx[4]]),
                                                      *reinterpret_cast<const int64_t*>(&src[mapsx[5]]),
                                                      *reinterpret_cast<const int64_t*>(&src[mapsx[6]]),
                                                      *reinterpret_cast<const int64_t*>(&src[mapsx[7]])));
}

CV_ALWAYS_INLINE void v_deinterleave(const v_float32x8& low, const v_float32x8& high,
                                     v_float32x8& even,      v_float32x8& odd)
{
    __m256 tmp0 = _mm256_unpacklo_ps(low.val, high.val);
    __m256 tmp1 = _mm256_unpackhi_ps(low.val, high.val);
    __m256 tmp2 = _mm256_unpacklo_ps(tmp0, tmp1);
    __m256 tmp3 = _mm256_unpackhi_ps(tmp0, tmp1);
    even.val = _mm256_castsi256_ps(_mm256_permute4x64_epi64(_mm256_castps_si256(tmp2), 216 /*11011000*/));
    odd.val = _mm256_castsi256_ps(_mm256_permute4x64_epi64(_mm256_castps_si256(tmp3), 216 /*11011000*/));
}

// Resize (bi-linear, 32FC1)
CV_ALWAYS_INLINE void calcRowLinear32FC1Impl(float *dst[],
                                             const float *src0[],
                                             const float *src1[],
                                             const float  alpha[],
                                             const int    mapsx[],
                                             const float  beta[],
                                             const Size& inSz,
                                             const Size& outSz,
                                             const int   lpi)
{
    bool xRatioEq1 = inSz.width == outSz.width;
    bool yRatioEq1 = inSz.height == outSz.height;

    const int nlanes = VTraits<v_float32x8>::vlanes();

    if (!xRatioEq1 && !yRatioEq1)
    {
        for (int line = 0; line < lpi; ++line) {
            float beta0 = beta[line];
            float beta1 = 1 - beta0;
            v_float32x8 v_beta0 = v256_setall_f32(beta0);
            int x = 0;

            v_float32x8 low1, high1, s00, s01;
            v_float32x8 low2, high2, s10, s11;
            for (; x <= outSz.width - nlanes; x += nlanes)
            {
                v_float32x8 alpha0 = v256_load(&alpha[x]);
                //  v_float32 alpha1 = 1.f - alpha0;

                v_gather_pairs(src0[line], &mapsx[x], low1, high1);
                v_deinterleave(low1, high1, s00, s01);

                //  v_float32 res0 = s00*alpha0 + s01*alpha1;
                v_float32x8 res0 = v_fma(v_sub(s00, s01), alpha0, s01);

                v_gather_pairs(src1[line], &mapsx[x], low2, high2);
                v_deinterleave(low2, high2, s10, s11);

                //  v_float32 res1 = s10*alpha0 + s11*alpha1;
                v_float32x8 res1 = v_fma(v_sub(s10, s11), alpha0, s11);
                //  v_float32 d = res0*beta0 + res1*beta1;
                v_float32x8 d = v_fma(v_sub(res0, res1), v_beta0, res1);

                v_store(&dst[line][x], d);
            }

            for (; x < outSz.width; ++x)
            {
                float alpha0 = alpha[x];
                float alpha1 = 1 - alpha0;
                int   sx0 = mapsx[x];
                int   sx1 = sx0 + 1;
                float res0 = src0[line][sx0] * alpha0 + src0[line][sx1] * alpha1;
                float res1 = src1[line][sx0] * alpha0 + src1[line][sx1] * alpha1;
                dst[line][x] = beta0 * res0 + beta1 * res1;
            }
        }
    }
    else if (!xRatioEq1)
    {

        for (int line = 0; line < lpi; ++line) {
            int x = 0;

            v_float32x8 low, high, s00, s01;
            for (; x <= outSz.width - nlanes; x += nlanes)
            {
                v_float32x8 alpha0 = v256_load(&alpha[x]);
                //  v_float32 alpha1 = 1.f - alpha0;

                v_gather_pairs(src0[line], &mapsx[x], low, high);
                v_deinterleave(low, high, s00, s01);

                //  v_float32 d = s00*alpha0 + s01*alpha1;
                v_float32x8 d = v_fma(v_sub(s00, s01), alpha0, s01);

                v_store(&dst[line][x], d);
            }

            for (; x < outSz.width; ++x) {
                float alpha0 = alpha[x];
                float alpha1 = 1 - alpha0;
                int   sx0 = mapsx[x];
                int   sx1 = sx0 + 1;
                dst[line][x] = src0[line][sx0] * alpha0 + src0[line][sx1] * alpha1;
            }
        }

    }
    else if (!yRatioEq1)
    {
        int length = inSz.width;  // == outSz.width

        for (int line = 0; line < lpi; ++line) {
            float beta0 = beta[line];
            float beta1 = 1 - beta0;
            v_float32x8 v_beta0 = v256_setall_f32(beta0);
            int x = 0;

            for (; x <= length - nlanes; x += nlanes)
            {
                v_float32x8 s0 = v256_load(&src0[line][x]);
                v_float32x8 s1 = v256_load(&src1[line][x]);

                //  v_float32 d = s0*beta0 + s1*beta1;
                v_float32x8 d = v_fma(v_sub(s0, s1), v_beta0, s1);

                v_store(&dst[line][x], d);
            }

            for (; x < length; ++x) {
                dst[line][x] = beta0 * src0[line][x] + beta1 * src1[line][x];
            }
        }

    }
    else
    {
        int length = inSz.width;  // == outSz.width
        memcpy(dst[0], src0[0], length * sizeof(float)*lpi);
    }
}
} // namespace avx2
} // namespace fliud
} // namespace gapi
} // namespace cv
#endif // !defined(GAPI_STANDALONE)
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cstdint`
- `limits`
- `immintrin.h`
- `cstring`
- `vector`
- `opencv2/core/hal/intrin.hpp`
- `opencv2/gapi/own/saturate.hpp`
- `algorithm`
- `opencv2/core.hpp`


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

