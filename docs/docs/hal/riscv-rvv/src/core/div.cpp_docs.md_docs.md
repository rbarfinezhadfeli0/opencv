# Documentation for `docs/hal/riscv-rvv/src/core/div.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/riscv-rvv/src/core/div.cpp_docs.md`
- **File Name**: `div.cpp_docs.md`
- **File Size**: 16,124 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/riscv-rvv/src/core/div.cpp_docs.md](../../../../../docs/hal/riscv-rvv/src/core/div.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/riscv-rvv/src/core/div.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/div.cpp`
- **File Name**: `div.cpp`
- **File Size**: 13,204 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/div.cpp](../../../../hal/riscv-rvv/src/core/div.cpp)

## Purpose and Role

This file is located in the `hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2025, SpaceMIT Inc., all rights reserved.
// Third party copyrights are property of their respective owners.

#include "rvv_hal.hpp"
#include "common.hpp"
#include <limits>

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

inline size_t setvl(int l) { return __riscv_vsetvl_e8m1(l); }

inline   vuint8m1_t vle(const uint8_t  *p, int vl) { return __riscv_vle8_v_u8m1(p, vl); }
inline    vint8m1_t vle(const int8_t   *p, int vl) { return __riscv_vle8_v_i8m1(p, vl); }
inline  vuint16m2_t vle(const uint16_t *p, int vl) { return __riscv_vle16_v_u16m2(p, vl); }
inline   vint16m2_t vle(const int16_t  *p, int vl) { return __riscv_vle16_v_i16m2(p, vl); }
inline   vint32m4_t vle(const int      *p, int vl) { return __riscv_vle32_v_i32m4(p, vl); }
inline vfloat32m4_t vle(const float    *p, int vl) { return __riscv_vle32_v_f32m4(p, vl); }

inline void vse(uint8_t  *p, const   vuint8m1_t &v, int vl) { __riscv_vse8(p, v, vl); }
inline void vse(int8_t   *p, const    vint8m1_t &v, int vl) { __riscv_vse8(p, v, vl); }
inline void vse(uint16_t *p, const  vuint16m2_t &v, int vl) { __riscv_vse16(p, v, vl); }
inline void vse(int16_t  *p, const   vint16m2_t &v, int vl) { __riscv_vse16(p, v, vl); }
inline void vse(int      *p, const   vint32m4_t &v, int vl) { __riscv_vse32(p, v, vl); }
inline void vse(float    *p, const vfloat32m4_t &v, int vl) { __riscv_vse32(p, v, vl); }

inline vuint16m2_t ext(const  vuint8m1_t &v, const int vl) { return __riscv_vzext_vf2(v, vl); }
inline  vint16m2_t ext(const   vint8m1_t &v, const int vl) { return __riscv_vsext_vf2(v, vl); }

inline  vuint8m1_t nclip(const vuint16m2_t &v, const int vl) { return __riscv_vnclipu(v, 0, __RISCV_VXRM_RNU, vl); }
inline   vint8m1_t nclip(const  vint16m2_t &v, const int vl) { return __riscv_vnclip(v, 0, __RISCV_VXRM_RNU, vl); }

template <typename VT> inline
VT div_sat(const VT &v1, const VT &v2, const float scale, const int vl) {
    return nclip(div_sat(ext(v1, vl), ext(v2, vl), scale, vl), vl);
}
template <> inline
vint16m2_t div_sat(const vint16m2_t &v1, const vint16m2_t &v2, const float scale, const int vl) {
    auto f1 = __riscv_vfwcvt_f(v1, vl);
    auto f2 = __riscv_vfwcvt_f(v2, vl);
    auto res = __riscv_vfmul(f1, __riscv_vfmul(common::__riscv_vfrec(f2, vl), scale, vl), vl);
    return __riscv_vfncvt_x(res, vl);
}
template <> inline
vuint16m2_t div_sat(const vuint16m2_t &v1, const vuint16m2_t &v2, const float scale, const int vl) {
    auto f1 = __riscv_vfwcvt_f(v1, vl);
    auto f2 = __riscv_vfwcvt_f(v2, vl);
    auto res = __riscv_vfmul(f1, __riscv_vfmul(common::__riscv_vfrec(f2, vl), scale, vl), vl);
    return __riscv_vfncvt_xu(res, vl);
}
template <> inline
vint32m4_t div_sat(const vint32m4_t &v1, const vint32m4_t &v2, const float scale, const int vl) {
    auto f1 = __riscv_vfcvt_f(v1, vl);
    auto f2 = __riscv_vfcvt_f(v2, vl);
    auto res = __riscv_vfmul(f1, __riscv_vfmul(common::__riscv_vfrec(f2, vl), scale, vl), vl);
    return __riscv_vfcvt_x(res, vl);
}
template <> inline
vuint32m4_t div_sat(const vuint32m4_t &v1, const vuint32m4_t &v2, const float scale, const int vl) {
    auto f1 = __riscv_vfcvt_f(v1, vl);
    auto f2 = __riscv_vfcvt_f(v2, vl);
    auto res = __riscv_vfmul(f1, __riscv_vfmul(common::__riscv_vfrec(f2, vl), scale, vl), vl);
    return __riscv_vfcvt_xu(res, vl);
}

template <typename VT> inline
VT recip_sat(const VT &v, const float scale, const int vl) {
    return nclip(recip_sat(ext(v, vl), scale, vl), vl);
}
template <> inline
vint16m2_t recip_sat(const vint16m2_t &v, const float scale, const int vl) {
    auto f = __riscv_vfwcvt_f(v, vl);
    auto res = __riscv_vfmul(common::__riscv_vfrec(f, vl), scale, vl);
    return __riscv_vfncvt_x(res, vl);
}
template <> inline
vuint16m2_t recip_sat(const vuint16m2_t &v, const float scale, const int vl) {
    auto f = __riscv_vfwcvt_f(v, vl);
    auto res = __riscv_vfmul(common::__riscv_vfrec(f, vl), scale, vl);
    return __riscv_vfncvt_xu(res, vl);
}
template <> inline
vint32m4_t recip_sat(const vint32m4_t &v, const float scale, const int vl) {
    auto f = __riscv_vfcvt_f(v, vl);
    auto res = __riscv_vfmul(common::__riscv_vfrec(f, vl), scale, vl);
    return __riscv_vfcvt_x(res, vl);
}
template <> inline
vuint32m4_t recip_sat(const vuint32m4_t &v, const float scale, const int vl) {
    auto f = __riscv_vfcvt_f(v, vl);
    auto res = __riscv_vfmul(common::__riscv_vfrec(f, vl), scale, vl);
    return __riscv_vfcvt_xu(res, vl);
}

// Implementation

template <typename ST> inline
int div(const ST *src1, size_t step1, const ST *src2, size_t step2,
         ST *dst, size_t step, int width, int height, float scale) {
    float max_fval = static_cast<float>(std::numeric_limits<ST>::max());
    if (scale == 0.f || ((scale * max_fval) <  1.f && (scale * max_fval) > -1.f)) {
        for (int h = 0; h < height; h++) {
            ST *dst_h = reinterpret_cast<ST*>((uchar*)dst + h * step);
            std::memset(dst_h, 0, sizeof(ST) * width);
        }
        return CV_HAL_ERROR_OK;
    }

    for (int h = 0; h < height; h++) {
        const ST *src1_h = reinterpret_cast<const ST*>((const uchar*)src1 + h * step1);
        const ST *src2_h = reinterpret_cast<const ST*>((const uchar*)src2 + h * step2);
        ST *dst_h = reinterpret_cast<ST*>((uchar*)dst + h * step);

        int vl;
        for (int w = 0; w < width; w += vl) {
            vl = setvl(width - w);

            auto v1 = vle(src1_h + w, vl);
            auto v2 = vle(src2_h + w, vl);

            auto mask = __riscv_vmseq(v2, 0, vl);
            vse(dst_h + w, __riscv_vmerge(div_sat(v1, v2, scale, vl), 0, mask, vl), vl);
        }
    }

    return CV_HAL_ERROR_OK;
}

template <>
int div(const float *src1, size_t step1, const float *src2, size_t step2,
        float *dst, size_t step, int width, int height, float scale) {
    if (scale == 0.f) {
        for (int h = 0; h < height; h++) {
            float *dst_h = reinterpret_cast<float*>((uchar*)dst + h * step);
            std::memset(dst_h, 0, sizeof(float) * width);
        }
        return CV_HAL_ERROR_OK;
    }

    if (std::fabs(scale - 1.f) < FLT_EPSILON) {
        for (int h = 0; h < height; h++) {
            const float *src1_h = reinterpret_cast<const float*>((const uchar*)src1 + h * step1);
            const float *src2_h = reinterpret_cast<const float*>((const uchar*)src2 + h * step2);
            float *dst_h = reinterpret_cast<float*>((uchar*)dst + h * step);

            int vl;
            for (int w = 0; w < width; w += vl) {
                vl = setvl(width - w);

                auto v1 = vle(src1_h + w, vl);
                auto v2 = vle(src2_h + w, vl);

                vse(dst_h + w, __riscv_vfmul(v1, common::__riscv_vfrec(v2, vl), vl), vl);
            }
        }
    } else {
        for (int h = 0; h < height; h++) {
            const float *src1_h = reinterpret_cast<const float*>((const uchar*)src1 + h * step1);
            const float *src2_h = reinterpret_cast<const float*>((const uchar*)src2 + h * step2);
            float *dst_h = reinterpret_cast<float*>((uchar*)dst + h * step);

            int vl;
            for (int w = 0; w < width; w += vl) {
                vl = setvl(width - w);

                auto v1 = vle(src1_h + w, vl);
                auto v2 = vle(src2_h + w, vl);

                vse(dst_h + w, __riscv_vfmul(v1, __riscv_vfmul(common::__riscv_vfrec(v2, vl), scale, vl), vl), vl);
            }
        }
    }

    return CV_HAL_ERROR_OK;
}

template <typename ST> inline
int recip(const ST *src_data, size_t src_step, ST *dst_data, size_t dst_step,
          int width, int height, float scale) {
    if (scale == 0.f || (scale < 1.f && scale > -1.f)) {
        for (int h = 0; h < height; h++) {
            ST *dst_h = reinterpret_cast<ST*>((uchar*)dst_data + h * dst_step);
            std::memset(dst_h, 0, sizeof(ST) * width);
        }
        return CV_HAL_ERROR_OK;
    }

    for (int h = 0; h < height; h++) {
        const ST *src_h = reinterpret_cast<const ST*>((const uchar*)src_data + h * src_step);
        ST *dst_h = reinterpret_cast<ST*>((uchar*)dst_data + h * dst_step);

        int vl;
        for (int w = 0; w < width; w += vl) {
            vl = setvl(width - w);

            auto v = vle(src_h + w, vl);

            auto mask = __riscv_vmseq(v, 0, vl);
            vse(dst_h + w, __riscv_vmerge(recip_sat(v, scale, vl), 0, mask, vl), vl);
        }
    }

    return CV_HAL_ERROR_OK;
}

template <>
int recip(const float *src_data, size_t src_step, float *dst_data, size_t dst_step,
          int width, int height, float scale) {
    if (scale == 0.f) {
        for (int h = 0; h < height; h++) {
            float *dst_h = reinterpret_cast<float*>((uchar*)dst_data + h * dst_step);
            std::memset(dst_h, 0, sizeof(float) * width);
        }
        return CV_HAL_ERROR_OK;
    }

    if (std::fabs(scale - 1.f) < FLT_EPSILON) {
        for (int h = 0; h < height; h++) {
            const float *src_h = reinterpret_cast<const float*>((const uchar*)src_data + h * src_step);
            float *dst_h = reinterpret_cast<float*>((uchar*)dst_data + h * dst_step);

            int vl;
            for (int w = 0; w < width; w += vl) {
                vl = setvl(width - w);

                auto v = vle(src_h + w, vl);

                vse(dst_h + w, common::__riscv_vfrec(v, vl), vl);
            }
        }
    } else {
        for (int h = 0; h < height; h++) {
            const float *src_h = reinterpret_cast<const float*>((const uchar*)src_data + h * src_step);
            float *dst_h = reinterpret_cast<float*>((uchar*)dst_data + h * dst_step);

            int vl;
            for (int w = 0; w < width; w += vl) {
                vl = setvl(width - w);

                auto v = vle(src_h + w, vl);

                vse(dst_h + w, __riscv_vfmul(common::__riscv_vfrec(v, vl), scale, vl), vl);
            }
        }
    }

    return CV_HAL_ERROR_OK;
}

} // anonymous

int div8u(const uchar *src1_data, size_t src1_step, const uchar *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<uchar>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}
int div8s(const schar *src1_data, size_t src1_step, const schar *src2_data, size_t src2_step, schar *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<schar>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}
int div16u(const ushort *src1_data, size_t src1_step, const ushort *src2_data, size_t src2_step, ushort *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<ushort>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}
int div16s(const short *src1_data, size_t src1_step, const short *src2_data, size_t src2_step, short *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<short>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}
int div32s(const int *src1_data, size_t src1_step, const int *src2_data, size_t src2_step, int *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<int>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}
int div32f(const float *src1_data, size_t src1_step, const float *src2_data, size_t src2_step, float *dst_data, size_t dst_step, int width, int height, double scale) {
    return div<float>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, scale);
}

int recip8u(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<uchar>(src_data, src_step, dst_data, dst_step, width, height, scale);
}
int recip8s(const schar *src_data, size_t src_step, schar *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<schar>(src_data, src_step, dst_data, dst_step, width, height, scale);
}
int recip16u(const ushort *src_data, size_t src_step, ushort *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<ushort>(src_data, src_step, dst_data, dst_step, width, height, scale);
}
int recip16s(const short *src_data, size_t src_step, short *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<short>(src_data, src_step, dst_data, dst_step, width, height, scale);
}
int recip32s(const int *src_data, size_t src_step, int *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<int>(src_data, src_step, dst_data, dst_step, width, height, scale);
}
int recip32f(const float *src_data, size_t src_step, float *dst_data, size_t dst_step, int width, int height, double scale) {
    return recip<float>(src_data, src_step, dst_data, dst_step, width, height, scale);
}

#endif // CV_HAL_RVV_1P0_ENABLED

}}} // cv::rvv_hal::core
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
- `limits`
- `rvv_hal.hpp`
- `common.hpp`


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

