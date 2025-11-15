# Documentation for `hal/riscv-rvv/src/core/compare.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/compare.cpp`
- **File Name**: `compare.cpp`
- **File Size**: 8,103 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/compare.cpp](../../../../hal/riscv-rvv/src/core/compare.cpp)

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

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

constexpr RVV_LMUL getLMUL(size_t sz) {
    // c++11 only allows exactly one return statement inside the function body modified by constexpr
    return sz == 1 ? LMUL_8 : (sz == 2 ? LMUL_4 : (sz == 4 ? LMUL_2 : LMUL_1));
}

static inline vbool1_t vlt(const vuint8m8_t   &a, const vuint8m8_t   &b, const int vl) { return __riscv_vmsltu(a, b, vl); }
static inline vbool1_t vlt(const vint8m8_t    &a, const vint8m8_t    &b, const int vl) { return __riscv_vmslt(a, b, vl); }
static inline vbool2_t vlt(const vuint16m8_t  &a, const vuint16m8_t  &b, const int vl) { return __riscv_vmsltu(a, b, vl); }
static inline vbool2_t vlt(const vint16m8_t   &a, const vint16m8_t   &b, const int vl) { return __riscv_vmslt(a, b, vl); }
static inline vbool4_t vlt(const vint32m8_t   &a, const vint32m8_t   &b, const int vl) { return __riscv_vmslt(a, b, vl); }
static inline vbool4_t vlt(const vfloat32m8_t &a, const vfloat32m8_t &b, const int vl) { return __riscv_vmflt(a, b, vl); }

static inline vbool1_t vle(const vuint8m8_t   &a, const vuint8m8_t   &b, const int vl) { return __riscv_vmsleu(a, b, vl); }
static inline vbool1_t vle(const vint8m8_t    &a, const vint8m8_t    &b, const int vl) { return __riscv_vmsle(a, b, vl); }
static inline vbool2_t vle(const vuint16m8_t  &a, const vuint16m8_t  &b, const int vl) { return __riscv_vmsleu(a, b, vl); }
static inline vbool2_t vle(const vint16m8_t   &a, const vint16m8_t   &b, const int vl) { return __riscv_vmsle(a, b, vl); }
static inline vbool4_t vle(const vint32m8_t   &a, const vint32m8_t   &b, const int vl) { return __riscv_vmsle(a, b, vl); }
static inline vbool4_t vle(const vfloat32m8_t &a, const vfloat32m8_t &b, const int vl) { return __riscv_vmfle(a, b, vl); }

static inline vbool1_t veq(const vuint8m8_t   &a, const vuint8m8_t   &b, const int vl) { return __riscv_vmseq(a, b, vl); }
static inline vbool1_t veq(const vint8m8_t    &a, const vint8m8_t    &b, const int vl) { return __riscv_vmseq(a, b, vl); }
static inline vbool2_t veq(const vuint16m8_t  &a, const vuint16m8_t  &b, const int vl) { return __riscv_vmseq(a, b, vl); }
static inline vbool2_t veq(const vint16m8_t   &a, const vint16m8_t   &b, const int vl) { return __riscv_vmseq(a, b, vl); }
static inline vbool4_t veq(const vint32m8_t   &a, const vint32m8_t   &b, const int vl) { return __riscv_vmseq(a, b, vl); }
static inline vbool4_t veq(const vfloat32m8_t &a, const vfloat32m8_t &b, const int vl) { return __riscv_vmfeq(a, b, vl); }

static inline vbool1_t vne(const vuint8m8_t   &a, const vuint8m8_t   &b, const int vl) { return __riscv_vmsne(a, b, vl); }
static inline vbool1_t vne(const vint8m8_t    &a, const vint8m8_t    &b, const int vl) { return __riscv_vmsne(a, b, vl); }
static inline vbool2_t vne(const vuint16m8_t  &a, const vuint16m8_t  &b, const int vl) { return __riscv_vmsne(a, b, vl); }
static inline vbool2_t vne(const vint16m8_t   &a, const vint16m8_t   &b, const int vl) { return __riscv_vmsne(a, b, vl); }
static inline vbool4_t vne(const vint32m8_t   &a, const vint32m8_t   &b, const int vl) { return __riscv_vmsne(a, b, vl); }
static inline vbool4_t vne(const vfloat32m8_t &a, const vfloat32m8_t &b, const int vl) { return __riscv_vmfne(a, b, vl); }

#define CV_HAL_RVV_COMPARE_OP(op_name) \
template <typename _Tps> \
struct op_name { \
    using in = RVV<_Tps, LMUL_8>; \
    using out = RVV<uint8_t, getLMUL(sizeof(_Tps))>; \
    constexpr static uint8_t one = 255; \
    static inline void run(const _Tps *src1, const _Tps *src2, uchar *dst, const int len) { \
        auto zero = out::vmv(0, out::setvlmax()); \
        int vl; \
        for (int i = 0; i < len; i += vl) { \
            vl = in::setvl(len - i); \
            auto v1 = in::vload(src1 + i, vl); \
            auto v2 = in::vload(src2 + i, vl); \
            auto m = v##op_name(v1, v2, vl); \
            out::vstore(dst + i, __riscv_vmerge(zero, one, m, vl), vl); \
        } \
    } \
};

CV_HAL_RVV_COMPARE_OP(lt)
CV_HAL_RVV_COMPARE_OP(le)
CV_HAL_RVV_COMPARE_OP(eq)
CV_HAL_RVV_COMPARE_OP(ne)

template <template<typename _Tps> class op, typename _Tps> static inline
int compare_impl(const _Tps *src1_data, size_t src1_step, const _Tps *src2_data, size_t src2_step,
                 uchar *dst_data, size_t dst_step, int width, int height) {
    if (src1_step == src2_step && src1_step == dst_step && src1_step == width * sizeof(_Tps)) {
        width *= height;
        height = 1;
    }

    for (int h = 0; h < height; h++) {
        const _Tps *src1 = reinterpret_cast<const _Tps*>((const uchar*)src1_data + h * src1_step);
        const _Tps *src2 = reinterpret_cast<const _Tps*>((const uchar*)src2_data + h * src2_step);
        uchar *dst = dst_data + h * dst_step;

        op<_Tps>::run(src1, src2, dst, width);
    }

    return CV_HAL_ERROR_OK;
}

template <typename _Tps> inline
int compare(const _Tps *src1_data, size_t src1_step, const _Tps *src2_data, size_t src2_step,
            uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    switch (operation) {
        case CMP_LT: return compare_impl<lt, _Tps>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height);
        case CMP_GT: return compare_impl<lt, _Tps>(src2_data, src2_step, src1_data, src1_step, dst_data, dst_step, width, height);
        case CMP_LE: return compare_impl<le, _Tps>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height);
        case CMP_GE: return compare_impl<le, _Tps>(src2_data, src2_step, src1_data, src1_step, dst_data, dst_step, width, height);
        case CMP_EQ: return compare_impl<eq, _Tps>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height);
        case CMP_NE: return compare_impl<ne, _Tps>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height);
        default: return CV_HAL_ERROR_NOT_IMPLEMENTED;
    }
}

} // namespace anonymous

int cmp8u(const uchar *src1_data, size_t src1_step, const uchar *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<uchar>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
}
int cmp8s(const schar *src1_data, size_t src1_step, const schar *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<schar>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
}
int cmp16u(const ushort *src1_data, size_t src1_step, const ushort *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<ushort>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
}
int cmp16s(const short *src1_data, size_t src1_step, const short *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<short>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
}
int cmp32s(const int *src1_data, size_t src1_step, const int *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<int>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
}
int cmp32f(const float *src1_data, size_t src1_step, const float *src2_data, size_t src2_step, uchar *dst_data, size_t dst_step, int width, int height, int operation) {
    return compare<float>(src1_data, src1_step, src2_data, src2_step, dst_data, dst_step, width, height, operation);
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

### Classes and Structures

- **op**: A class/struct defined in this file
- **op_name**: A class/struct defined in this file

### Functions and Methods

- **body()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `rvv_hal.hpp`


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

