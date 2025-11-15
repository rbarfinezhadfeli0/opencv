# Documentation for `docs/hal/riscv-rvv/src/core/transpose.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/riscv-rvv/src/core/transpose.cpp_docs.md`
- **File Name**: `transpose.cpp_docs.md`
- **File Size**: 12,181 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/riscv-rvv/src/core/transpose.cpp_docs.md](../../../../../docs/hal/riscv-rvv/src/core/transpose.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/riscv-rvv/src/core/transpose.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/transpose.cpp`
- **File Name**: `transpose.cpp`
- **File Size**: 9,258 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/transpose.cpp](../../../../hal/riscv-rvv/src/core/transpose.cpp)

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

#if defined (__clang__) && __clang_major__ < 18
#define OPENCV_HAL_IMPL_RVV_VCREATE_x4(suffix, width, v0, v1, v2, v3) \
    __riscv_vset_v_##suffix##m##width##_##suffix##m##width##x4(v, 0, v0); \
    v = __riscv_vset(v, 1, v1); \
    v = __riscv_vset(v, 2, v2); \
    v = __riscv_vset(v, 3, v3);

#define OPENCV_HAL_IMPL_RVV_VCREATE_x8(suffix, width, v0, v1, v2, v3, v4, v5, v6, v7) \
    __riscv_vset_v_##suffix##m##width##_##suffix##m##width##x8(v, 0, v0); \
    v = __riscv_vset(v, 1, v1); \
    v = __riscv_vset(v, 2, v2); \
    v = __riscv_vset(v, 3, v3); \
    v = __riscv_vset(v, 4, v4); \
    v = __riscv_vset(v, 5, v5); \
    v = __riscv_vset(v, 6, v6); \
    v = __riscv_vset(v, 7, v7);

#define __riscv_vcreate_v_u8m1x8(v0, v1, v2, v3, v4, v5, v6, v7) OPENCV_HAL_IMPL_RVV_VCREATE_x8(u8, 1, v0, v1, v2, v3, v4, v5, v6, v7)
#define __riscv_vcreate_v_u16m1x8(v0, v1, v2, v3, v4, v5, v6, v7) OPENCV_HAL_IMPL_RVV_VCREATE_x8(u16, 1, v0, v1, v2, v3, v4, v5, v6, v7)
#define __riscv_vcreate_v_i32m1x4(v0, v1, v2, v3) OPENCV_HAL_IMPL_RVV_VCREATE_x4(i32, 1, v0, v1, v2, v3)
#define __riscv_vcreate_v_i64m1x8(v0, v1, v2, v3, v4, v5, v6, v7) OPENCV_HAL_IMPL_RVV_VCREATE_x8(i64, 1, v0, v1, v2, v3, v4, v5, v6, v7)
#endif

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

static void transpose2d_8u(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step, int src_width, int src_height) {
    auto transpose_8u_8xVl = [](const uchar *src, size_t sstep, uchar *dst, size_t dstep, const int vl) {
        auto v0 = __riscv_vle8_v_u8m1(src, vl);
        auto v1 = __riscv_vle8_v_u8m1(src + sstep, vl);
        auto v2 = __riscv_vle8_v_u8m1(src + 2 * sstep, vl);
        auto v3 = __riscv_vle8_v_u8m1(src + 3 * sstep, vl);
        auto v4 = __riscv_vle8_v_u8m1(src + 4 * sstep, vl);
        auto v5 = __riscv_vle8_v_u8m1(src + 5 * sstep, vl);
        auto v6 = __riscv_vle8_v_u8m1(src + 6 * sstep, vl);
        auto v7 = __riscv_vle8_v_u8m1(src + 7 * sstep, vl);
        vuint8m1x8_t v = __riscv_vcreate_v_u8m1x8(v0, v1, v2, v3, v4, v5, v6, v7);
        __riscv_vssseg8e8(dst, dstep, v, vl);
    };

    int h = 0, w = 0;
    for (; h <= src_height - 8; h += 8) {
        const uchar *src = src_data + h * src_step;
        uchar *dst = dst_data + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e8m1(src_width - w);
            transpose_8u_8xVl(src + w, src_step, dst + w * dst_step, dst_step, vl);
        }
    }
    for (; h < src_height; h++) {
        const uchar *src = src_data + h * src_step;
        uchar *dst = dst_data + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e8m8(src_width - w);
            auto v = __riscv_vle8_v_u8m8(src + w, vl);
            __riscv_vsse8(dst + w * dst_step, dst_step, v, vl);
        }
    }
}

static void transpose2d_16u(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step, int src_width, int src_height) {
    auto transpose_16u_8xVl = [](const ushort *src, size_t sstep, ushort *dst, size_t dstep, const int vl) {
        auto v0 = __riscv_vle16_v_u16m1(src, vl);
        auto v1 = __riscv_vle16_v_u16m1(src + sstep, vl);
        auto v2 = __riscv_vle16_v_u16m1(src + 2 * sstep, vl);
        auto v3 = __riscv_vle16_v_u16m1(src + 3 * sstep, vl);
        auto v4 = __riscv_vle16_v_u16m1(src + 4 * sstep, vl);
        auto v5 = __riscv_vle16_v_u16m1(src + 5 * sstep, vl);
        auto v6 = __riscv_vle16_v_u16m1(src + 6 * sstep, vl);
        auto v7 = __riscv_vle16_v_u16m1(src + 7 * sstep, vl);
        vuint16m1x8_t v = __riscv_vcreate_v_u16m1x8(v0, v1, v2, v3, v4, v5, v6, v7);
        __riscv_vssseg8e16(dst, dstep, v, vl);
    };

    size_t src_step_base = src_step / sizeof(ushort);
    size_t dst_step_base = dst_step / sizeof(ushort);

    int h = 0, w = 0;
    for (; h <= src_height - 8; h += 8) {
        const ushort *src = (const ushort*)(src_data) + h * src_step_base;
        ushort *dst = (ushort*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e16m1(src_width - w);
            transpose_16u_8xVl(src + w, src_step_base, dst + w * dst_step_base, dst_step, vl);
        }
    }
    for (; h < src_height; h++) {
        const ushort *src = (const ushort*)(src_data) + h * src_step_base;
        ushort *dst = (ushort*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e16m8(src_width - w);
            auto v = __riscv_vle16_v_u16m8(src + w, vl);
            __riscv_vsse16(dst + w * dst_step_base, dst_step, v, vl);
        }
    }
}

static void transpose2d_32s(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step, int src_width, int src_height) {
    auto transpose_32s_4xVl = [](const int *src, size_t sstep, int *dst, size_t dstep, const int vl) {
        auto v0 = __riscv_vle32_v_i32m1(src, vl);
        auto v1 = __riscv_vle32_v_i32m1(src + sstep, vl);
        auto v2 = __riscv_vle32_v_i32m1(src + 2 * sstep, vl);
        auto v3 = __riscv_vle32_v_i32m1(src + 3 * sstep, vl);
        vint32m1x4_t v = __riscv_vcreate_v_i32m1x4(v0, v1, v2, v3);
        __riscv_vssseg4e32(dst, dstep, v, vl);
    };

    size_t src_step_base = src_step / sizeof(int);
    size_t dst_step_base = dst_step / sizeof(int);

    int h = 0, w = 0;
    for (; h <= src_height - 4; h += 4) {
        const int *src = (const int*)(src_data) + h * src_step_base;
        int *dst = (int*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e32m1(src_width - w);
            transpose_32s_4xVl(src + w, src_step_base, dst + w * dst_step_base, dst_step, vl);
        }
    }
    for (; h < src_height; h++) {
        const int *src = (const int*)(src_data) + h * src_step_base;
        int *dst = (int*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e32m8(src_width - w);
            auto v = __riscv_vle32_v_i32m8(src + w, vl);
            __riscv_vsse32(dst + w * dst_step_base, dst_step, v, vl);
        }
    }
}

static void transpose2d_32sC2(const uchar *src_data, size_t src_step, uchar *dst_data, size_t dst_step, int src_width, int src_height) {
    auto transpose_64s_8xVl = [](const int64_t *src, size_t sstep, int64_t *dst, size_t dstep, const int vl) {
        auto v0 = __riscv_vle64_v_i64m1(src, vl);
        auto v1 = __riscv_vle64_v_i64m1(src + sstep, vl);
        auto v2 = __riscv_vle64_v_i64m1(src + 2 * sstep, vl);
        auto v3 = __riscv_vle64_v_i64m1(src + 3 * sstep, vl);
        auto v4 = __riscv_vle64_v_i64m1(src + 4 * sstep, vl);
        auto v5 = __riscv_vle64_v_i64m1(src + 5 * sstep, vl);
        auto v6 = __riscv_vle64_v_i64m1(src + 6 * sstep, vl);
        auto v7 = __riscv_vle64_v_i64m1(src + 7 * sstep, vl);
        vint64m1x8_t v = __riscv_vcreate_v_i64m1x8(v0, v1, v2, v3, v4, v5, v6, v7);
        __riscv_vssseg8e64(dst, dstep, v, vl);
    };

    size_t src_step_base = src_step / sizeof(int64_t);
    size_t dst_step_base = dst_step / sizeof(int64_t);

    int h = 0, w = 0;
    for (; h <= src_height - 8; h += 8) {
        const int64_t *src = (const int64_t*)(src_data) + h * src_step_base;
        int64_t *dst = (int64_t*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e64m1(src_width - w);
            transpose_64s_8xVl(src + w, src_step_base, dst + w * dst_step_base, dst_step, vl);
        }
    }
    for (; h < src_height; h++) {
        const int64_t *src = (const int64_t*)(src_data) + h * src_step_base;
        int64_t *dst = (int64_t*)(dst_data) + h;
        int vl;
        for (w = 0; w < src_width; w += vl) {
            vl = __riscv_vsetvl_e64m8(src_width - w);
            auto v = __riscv_vle64_v_i64m8(src + w, vl);
            __riscv_vsse64(dst + w * dst_step_base, dst_step, v, vl);
        }
    }
}

using Transpose2dFunc = void (*)(const uchar*, size_t, uchar*, size_t, int, int);
int transpose2d(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step,
                int src_width, int src_height, int element_size) {
    if (src_data == dst_data) {
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    }

    static Transpose2dFunc tab[] = {
        0, transpose2d_8u, transpose2d_16u, 0,
        transpose2d_32s, 0, 0, 0,
        transpose2d_32sC2, 0, 0, 0,
        0, 0, 0, 0,
        0, 0, 0, 0,
        0, 0, 0, 0,
        0, 0, 0, 0,
        0, 0, 0, 0,
        0
    };
    Transpose2dFunc func = element_size <= 32 ? tab[element_size] : nullptr;
    if (!func) {
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    }

    func(src_data, src_step, dst_data, dst_step, src_width, src_height);

    return CV_HAL_ERROR_OK;
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

