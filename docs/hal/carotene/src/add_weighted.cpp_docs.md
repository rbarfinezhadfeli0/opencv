# Documentation for `hal/carotene/src/add_weighted.cpp`

## File Metadata

- **Full Path**: `hal/carotene/src/add_weighted.cpp`
- **File Name**: `add_weighted.cpp`
- **File Size**: 9,951 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/carotene/src/add_weighted.cpp](../../../hal/carotene/src/add_weighted.cpp)

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
 * Copyright (C) 2012-2015, NVIDIA Corporation, all rights reserved.
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
#include "vtransform.hpp"
#include "vround_helper.hpp"

namespace CAROTENE_NS {

#ifdef CAROTENE_NEON

namespace {

using namespace internal;

template <typename T> struct TypeTraits;
template <> struct TypeTraits< u8> { typedef u16 wide;                     typedef  u8 unsign; typedef  uint8x16_t vec128; };
template <> struct TypeTraits< s8> { typedef s16 wide;                     typedef  u8 unsign; typedef   int8x16_t vec128; };
template <> struct TypeTraits<u16> { typedef u32 wide; typedef  u8 narrow; typedef u16 unsign; typedef  uint16x8_t vec128; };
template <> struct TypeTraits<s16> { typedef s32 wide; typedef  s8 narrow; typedef u16 unsign; typedef   int16x8_t vec128; };
template <> struct TypeTraits<u32> { typedef u64 wide; typedef u16 narrow; typedef u32 unsign; typedef  uint32x4_t vec128; };
template <> struct TypeTraits<s32> { typedef s64 wide; typedef s16 narrow; typedef u32 unsign; typedef   int32x4_t vec128; };
template <> struct TypeTraits<f32> { typedef f64 wide;                                         typedef float32x4_t vec128; };

template <typename T> struct wAdd
{
    typedef T type;

    f32 alpha, beta, gamma;
    typedef typename TypeTraits<T>::wide wtype;
    wAdd<wtype> wideAdd;
    wAdd(f32 _alpha, f32 _beta, f32 _gamma):
        alpha(_alpha), beta(_beta), gamma(_gamma),
        wideAdd(_alpha, _beta, _gamma) {}

    void operator() (const typename VecTraits<T>::vec128 & v_src0,
                     const typename VecTraits<T>::vec128 & v_src1,
                     typename VecTraits<T>::vec128 & v_dst) const
    {
        typename VecTraits<wtype>::vec128 vrl, vrh;
        wideAdd(vmovl( vget_low(v_src0)), vmovl( vget_low(v_src1)), vrl);
        wideAdd(vmovl(vget_high(v_src0)), vmovl(vget_high(v_src1)), vrh);

        v_dst = vcombine(vqmovn(vrl), vqmovn(vrh));
    }

    void operator() (const typename VecTraits<T>::vec64 & v_src0,
                     const typename VecTraits<T>::vec64 & v_src1,
                     typename VecTraits<T>::vec64 & v_dst) const
    {
        typename VecTraits<wtype>::vec128 vr;
        wideAdd(vmovl(v_src0), vmovl(v_src1), vr);

        v_dst = vqmovn(vr);
    }

    void operator() (const T * src0, const T * src1, T * dst) const
    {
        dst[0] = saturate_cast<T>(alpha*src0[0] + beta*src1[0] + gamma);
    }
};

template <> struct wAdd<s32>
{
    typedef s32 type;

    f32 alpha, beta, gamma;
    float32x4_t valpha, vbeta, vgamma;
    wAdd(f32 _alpha, f32 _beta, f32 _gamma):
        alpha(_alpha), beta(_beta), gamma(_gamma)
    {
        valpha = vdupq_n_f32(_alpha);
        vbeta = vdupq_n_f32(_beta);
        vgamma = vdupq_n_f32(_gamma);
    }

    void operator() (const VecTraits<s32>::vec128 & v_src0,
                     const VecTraits<s32>::vec128 & v_src1,
                     VecTraits<s32>::vec128 & v_dst) const
    {
        float32x4_t vs1 = vcvtq_f32_s32(v_src0);
        float32x4_t vs2 = vcvtq_f32_s32(v_src1);

        vs1 = vmlaq_f32(vgamma, vs1, valpha);
        vs1 = vmlaq_f32(vs1, vs2, vbeta);
        v_dst = vroundq_s32_f32(vs1);
    }

    void operator() (const VecTraits<s32>::vec64 & v_src0,
                     const VecTraits<s32>::vec64 & v_src1,
                     VecTraits<s32>::vec64 & v_dst) const
    {
        float32x2_t vs1 = vcvt_f32_s32(v_src0);
        float32x2_t vs2 = vcvt_f32_s32(v_src1);

        vs1 = vmla_f32(vget_low(vgamma), vs1, vget_low(valpha));
        vs1 = vmla_f32(vs1, vs2, vget_low(vbeta));
        v_dst = vround_s32_f32(vs1);
    }

    void operator() (const s32 * src0, const s32 * src1, s32 * dst) const
    {
        dst[0] = saturate_cast<s32>(alpha*src0[0] + beta*src1[0] + gamma);
    }
};

template <> struct wAdd<u32>
{
    typedef u32 type;

    f32 alpha, beta, gamma;
    float32x4_t valpha, vbeta, vgamma;
    wAdd(f32 _alpha, f32 _beta, f32 _gamma):
        alpha(_alpha), beta(_beta), gamma(_gamma)
    {
        valpha = vdupq_n_f32(_alpha);
        vbeta = vdupq_n_f32(_beta);
        vgamma = vdupq_n_f32(_gamma);
    }

    void operator() (const VecTraits<u32>::vec128 & v_src0,
                     const VecTraits<u32>::vec128 & v_src1,
                     VecTraits<u32>::vec128 & v_dst) const
    {
        float32x4_t vs1 = vcvtq_f32_u32(v_src0);
        float32x4_t vs2 = vcvtq_f32_u32(v_src1);

        vs1 = vmlaq_f32(vgamma, vs1, valpha);
        vs1 = vmlaq_f32(vs1, vs2, vbeta);
        v_dst = vroundq_u32_f32(vs1);
    }

    void operator() (const VecTraits<u32>::vec64 & v_src0,
                     const VecTraits<u32>::vec64 & v_src1,
                     VecTraits<u32>::vec64 & v_dst) const
    {
        float32x2_t vs1 = vcvt_f32_u32(v_src0);
        float32x2_t vs2 = vcvt_f32_u32(v_src1);

        vs1 = vmla_f32(vget_low(vgamma), vs1, vget_low(valpha));
        vs1 = vmla_f32(vs1, vs2, vget_low(vbeta));
        v_dst = vround_u32_f32(vs1);
    }

    void operator() (const u32 * src0, const u32 * src1, u32 * dst) const
    {
        dst[0] = saturate_cast<u32>(alpha*src0[0] + beta*src1[0] + gamma);
    }
};

template <> struct wAdd<f32>
{
    typedef f32 type;

    f32 alpha, beta, gamma;
    float32x4_t valpha, vbeta, vgamma;
    wAdd(f32 _alpha, f32 _beta, f32 _gamma):
        alpha(_alpha), beta(_beta), gamma(_gamma)
    {
        valpha = vdupq_n_f32(_alpha);
        vbeta = vdupq_n_f32(_beta);
        vgamma = vdupq_n_f32(_gamma + 0.5);
    }

    void operator() (const VecTraits<f32>::vec128 & v_src0,
                     const VecTraits<f32>::vec128 & v_src1,
                     VecTraits<f32>::vec128 & v_dst) const
    {
        float32x4_t vs1 = vmlaq_f32(vgamma, v_src0, valpha);
        v_dst = vmlaq_f32(vs1, v_src1, vbeta);
    }

    void operator() (const VecTraits<f32>::vec64 & v_src0,
                     const VecTraits<f32>::vec64 & v_src1,
                     VecTraits<f32>::vec64 & v_dst) const
    {
        float32x2_t vs1 = vmla_f32(vget_low(vgamma), v_src0, vget_low(valpha));
        v_dst = vmla_f32(vs1, v_src1, vget_low(vbeta));

    }

    void operator() (const f32 * src0, const f32 * src1, f32 * dst) const
    {
        dst[0] = alpha*src0[0] + beta*src1[0] + gamma;
    }
};

} // namespace

#define IMPL_ADDWEIGHTED(type)                                \
void addWeighted(const Size2D &size,                          \
                 const type * src0Base, ptrdiff_t src0Stride, \
                 const type * src1Base, ptrdiff_t src1Stride, \
                 type * dstBase, ptrdiff_t dstStride,         \
                 f32 alpha, f32 beta, f32 gamma)              \
{                                                             \
    internal::assertSupportedConfiguration();                 \
    wAdd<type> wgtAdd(alpha,                                  \
                      beta,                                   \
                      gamma);                                 \
    internal::vtransform(size,                                \
                         src0Base, src0Stride,                \
                         src1Base, src1Stride,                \
                         dstBase, dstStride,                  \
                         wgtAdd);                             \
}

#else

#define IMPL_ADDWEIGHTED(type)                                \
void addWeighted(const Size2D &,                              \
                 const type *, ptrdiff_t,                     \
                 const type *, ptrdiff_t,                     \
                 type *, ptrdiff_t,                           \
                 f32, f32, f32)                               \
{                                                             \
    internal::assertSupportedConfiguration();                 \
}

#endif

IMPL_ADDWEIGHTED(u8)
IMPL_ADDWEIGHTED(s8)
IMPL_ADDWEIGHTED(u16)
IMPL_ADDWEIGHTED(s16)
IMPL_ADDWEIGHTED(u32)
IMPL_ADDWEIGHTED(s32)
IMPL_ADDWEIGHTED(f32)

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

### Classes and Structures

- **wAdd**: A class/struct defined in this file
- **TypeTraits**: A class/struct defined in this file

### Functions and Methods

- **s64()**: A function/method defined in this file
- **CAROTENE_NEON()**: A function/method defined in this file
- **int16x8_t()**: A function/method defined in this file
- **int32x4_t()**: A function/method defined in this file
- **float32x4_t()**: A function/method defined in this file
- **uint16x8_t()**: A function/method defined in this file
- **T()**: A function/method defined in this file
- **uint8x16_t()**: A function/method defined in this file
- **f32()**: A function/method defined in this file
- **uint32x4_t()**: A function/method defined in this file
- **s16()**: A function/method defined in this file
- **int8x16_t()**: A function/method defined in this file
- **f64()**: A function/method defined in this file
- **u32()**: A function/method defined in this file
- **u64()**: A function/method defined in this file
- **u8()**: A function/method defined in this file
- **typename()**: A function/method defined in this file
- **u16()**: A function/method defined in this file
- **s8()**: A function/method defined in this file
- **s32()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `vtransform.hpp`
- `common.hpp`
- `vround_helper.hpp`

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

