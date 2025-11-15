# Documentation for `docs/hal/riscv-rvv/src/core/convert_scale.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/riscv-rvv/src/core/convert_scale.cpp_docs.md`
- **File Name**: `convert_scale.cpp_docs.md`
- **File Size**: 7,025 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/riscv-rvv/src/core/convert_scale.cpp_docs.md](../../../../../docs/hal/riscv-rvv/src/core/convert_scale.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/riscv-rvv/src/core/convert_scale.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/convert_scale.cpp`
- **File Name**: `convert_scale.cpp`
- **File Size**: 4,082 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/convert_scale.cpp](../../../../hal/riscv-rvv/src/core/convert_scale.cpp)

## Purpose and Role

This file is located in the `hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2025, Institute of Software, Chinese Academy of Sciences.

#include "rvv_hal.hpp"

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

inline int convertScale_8U8U(const uchar* src, size_t src_step, uchar* dst, size_t dst_step, int width, int height, double alpha, double beta)
{
    int vlmax = __riscv_vsetvlmax_e32m8();
    auto vec_b = __riscv_vfmv_v_f_f32m8(beta, vlmax);
    float a = alpha;

    for (int i = 0; i < height; i++)
    {
        const uchar* src_row = src + i * src_step;
        uchar* dst_row = dst + i * dst_step;
        int vl;
        for (int j = 0; j < width; j += vl)
        {
            vl = __riscv_vsetvl_e8m2(width - j);
            auto vec_src = __riscv_vle8_v_u8m2(src_row + j, vl);
            auto vec_src_u16 = __riscv_vzext_vf2(vec_src, vl);
            auto vec_src_f32 = __riscv_vfwcvt_f(vec_src_u16, vl);
            auto vec_fma = __riscv_vfmadd(vec_src_f32, a, vec_b, vl);
            auto vec_dst_u16 = __riscv_vfncvt_xu(vec_fma, vl);
            auto vec_dst = __riscv_vnclipu(vec_dst_u16, 0, __RISCV_VXRM_RNU, vl);
            __riscv_vse8_v_u8m2(dst_row + j, vec_dst, vl);
        }
    }

    return CV_HAL_ERROR_OK;
}

inline int convertScale_8U32F(const uchar* src, size_t src_step, uchar* dst, size_t dst_step, int width, int height, double alpha, double beta)
{
    int vlmax = __riscv_vsetvlmax_e32m8();
    auto vec_b = __riscv_vfmv_v_f_f32m8(beta, vlmax);
    float a = alpha;

    for (int i = 0; i < height; i++)
    {
        const uchar* src_row = src + i * src_step;
        float* dst_row = reinterpret_cast<float*>(dst + i * dst_step);
        int vl;
        for (int j = 0; j < width; j += vl)
        {
            vl = __riscv_vsetvl_e8m2(width - j);
            auto vec_src = __riscv_vle8_v_u8m2(src_row + j, vl);
            auto vec_src_u16 = __riscv_vzext_vf2(vec_src, vl);
            auto vec_src_f32 = __riscv_vfwcvt_f(vec_src_u16, vl);
            auto vec_fma = __riscv_vfmadd(vec_src_f32, a, vec_b, vl);
            __riscv_vse32_v_f32m8(dst_row + j, vec_fma, vl);
        }
    }

    return CV_HAL_ERROR_OK;
}

inline int convertScale_32F32F(const uchar* src, size_t src_step, uchar* dst, size_t dst_step, int width, int height, double alpha, double beta)
{
    int vlmax = __riscv_vsetvlmax_e32m8();
    auto vec_b = __riscv_vfmv_v_f_f32m8(beta, vlmax);
    float a = alpha;

    for (int i = 0; i < height; i++)
    {
        const float* src_row = reinterpret_cast<const float*>(src + i * src_step);
        float* dst_row = reinterpret_cast<float*>(dst + i * dst_step);
        int vl;
        for (int j = 0; j < width; j += vl)
        {
            vl = __riscv_vsetvl_e32m8(width - j);
            auto vec_src = __riscv_vle32_v_f32m8(src_row + j, vl);
            auto vec_fma = __riscv_vfmadd(vec_src, a, vec_b, vl);
            __riscv_vse32_v_f32m8(dst_row + j, vec_fma, vl);
        }
    }

    return CV_HAL_ERROR_OK;
}

int convertScale(const uchar* src, size_t src_step, uchar* dst, size_t dst_step,
                 int width, int height, int sdepth, int ddepth, double alpha, double beta)
{
    if (!dst)
        return CV_HAL_ERROR_OK;

    switch (sdepth)
    {
    case CV_8U:
        switch (ddepth)
        {
        case CV_8U:
            return convertScale_8U8U(src, src_step, dst, dst_step, width, height, alpha, beta);
        case CV_32F:
            return convertScale_8U32F(src, src_step, dst, dst_step, width, height, alpha, beta);
        }
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    case CV_32F:
        switch (ddepth)
        {
        case CV_32F:
            return convertScale_32F32F(src, src_step, dst, dst_step, width, height, alpha, beta);
        }
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    }

    return CV_HAL_ERROR_NOT_IMPLEMENTED;
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

