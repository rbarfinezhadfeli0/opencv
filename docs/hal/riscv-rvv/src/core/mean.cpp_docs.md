# Documentation for `hal/riscv-rvv/src/core/mean.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/mean.cpp`
- **File Name**: `mean.cpp`
- **File Size**: 10,263 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/mean.cpp](../../../../hal/riscv-rvv/src/core/mean.cpp)

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

inline int meanStdDev_8UC1(const uchar* src_data, size_t src_step, int width, int height,
                            double* mean_val, double* stddev_val, uchar* mask, size_t mask_step);
inline int meanStdDev_8UC4(const uchar* src_data, size_t src_step, int width, int height,
                            double* mean_val, double* stddev_val, uchar* mask, size_t mask_step);
inline int meanStdDev_32FC1(const uchar* src_data, size_t src_step, int width, int height,
                            double* mean_val, double* stddev_val, uchar* mask, size_t mask_step);

int meanStdDev(const uchar* src_data, size_t src_step, int width, int height, int src_type,
               double* mean_val, double* stddev_val, uchar* mask, size_t mask_step) {
    switch (src_type)
    {
    case CV_8UC1:
        return meanStdDev_8UC1(src_data, src_step, width, height, mean_val, stddev_val, mask, mask_step);
    case CV_8UC4:
        return meanStdDev_8UC4(src_data, src_step, width, height, mean_val, stddev_val, mask, mask_step);
    case CV_32FC1:
        return meanStdDev_32FC1(src_data, src_step, width, height, mean_val, stddev_val, mask, mask_step);
    default:
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    }
}

inline int meanStdDev_8UC1(const uchar* src_data, size_t src_step, int width, int height,
                             double* mean_val, double* stddev_val, uchar* mask, size_t mask_step) {
    int nz = 0;
    int vlmax = __riscv_vsetvlmax_e64m8();
    vuint64m8_t vec_sum = __riscv_vmv_v_x_u64m8(0, vlmax);
    vuint64m8_t vec_sqsum = __riscv_vmv_v_x_u64m8(0, vlmax);
    if (mask) {
        for (int i = 0; i < height; ++i) {
            const uchar* src_row = src_data + i * src_step;
            const uchar* mask_row = mask + i * mask_step;
            int j = 0, vl;
            for ( ; j < width; j += vl) {
                vl = __riscv_vsetvl_e8m1(width - j);
                auto vec_pixel_u8 = __riscv_vle8_v_u8m1(src_row + j, vl);
                auto vmask_u8 = __riscv_vle8_v_u8m1(mask_row+j, vl);
                auto vec_pixel = __riscv_vzext_vf4(vec_pixel_u8, vl);
                auto vmask = __riscv_vmseq_vx_u8m1_b8(vmask_u8, 1, vl);
                vec_sum = __riscv_vwaddu_wv_u64m8_tumu(vmask, vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vwmaccu_vv_u64m8_tumu(vmask, vec_sqsum, vec_pixel, vec_pixel, vl);
                nz += __riscv_vcpop_m_b8(vmask, vl);
            }
        }
    } else {
        for (int i = 0; i < height; i++) {
            const uchar*  src_row = src_data + i * src_step;
            int j = 0, vl;
            for ( ; j < width; j += vl) {
                vl = __riscv_vsetvl_e8m1(width - j);
                auto vec_pixel_u8 = __riscv_vle8_v_u8m1(src_row + j, vl);
                auto vec_pixel = __riscv_vzext_vf4(vec_pixel_u8, vl);
                vec_sum = __riscv_vwaddu_wv_u64m8_tu(vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vwmaccu_vv_u64m8_tu(vec_sqsum, vec_pixel, vec_pixel, vl);
            }
        }
        nz = height * width;
    }
    if (nz == 0) {
        if (mean_val) *mean_val = 0.0;
        if (stddev_val) *stddev_val = 0.0;
        return CV_HAL_ERROR_OK;
    }
    auto zero = __riscv_vmv_s_x_u64m1(0, vlmax);
    auto vec_red = __riscv_vmv_v_x_u64m1(0, vlmax);
    auto vec_reddev = __riscv_vmv_v_x_u64m1(0, vlmax);
    vec_red = __riscv_vredsum(vec_sum, zero, vlmax);
    vec_reddev = __riscv_vredsum(vec_sqsum, zero, vlmax);
    double sum = __riscv_vmv_x(vec_red);
    double mean = sum / nz;
    if (mean_val) {
        *mean_val = mean;
    }
    if (stddev_val) {
        double sqsum = __riscv_vmv_x(vec_reddev);
        double variance = std::max((sqsum / nz) - (mean * mean), 0.0);
        double stddev = std::sqrt(variance);
        *stddev_val = stddev;
    }
    return CV_HAL_ERROR_OK;
}

inline int meanStdDev_8UC4(const uchar* src_data, size_t src_step, int width, int height,
                             double* mean_val, double* stddev_val, uchar* mask, size_t mask_step) {
    int nz = 0;
    int vlmax = __riscv_vsetvlmax_e64m8();
    vuint64m8_t vec_sum = __riscv_vmv_v_x_u64m8(0, vlmax);
    vuint64m8_t vec_sqsum = __riscv_vmv_v_x_u64m8(0, vlmax);
    if (mask) {
        for (int i = 0; i < height; ++i) {
            const uchar* src_row = src_data + i * src_step;
            const uchar* mask_row = mask + i * mask_step;
            int j = 0, jm = 0, vl, vlm;
            for ( ; j < width*4; j += vl, jm += vlm) {
                vl = __riscv_vsetvl_e8m1(width*4 - j);
                vlm = __riscv_vsetvl_e8mf4(width - jm);
                auto vec_pixel_u8 = __riscv_vle8_v_u8m1(src_row + j, vl);
                auto vmask_u8mf4 = __riscv_vle8_v_u8mf4(mask_row + jm, vlm);
                auto vmask_u32 = __riscv_vzext_vf4(vmask_u8mf4, vlm);
                // 0 -> 0000; 1 -> 1111
                vmask_u32 = __riscv_vmul(vmask_u32, 0b00000001000000010000000100000001, vlm);
                auto vmask_u8 = __riscv_vreinterpret_u8m1(vmask_u32);
                auto vec_pixel = __riscv_vzext_vf4(vec_pixel_u8, vl);
                auto vmask = __riscv_vmseq_vx_u8m1_b8(vmask_u8, 1, vl);
                vec_sum = __riscv_vwaddu_wv_u64m8_tumu(vmask, vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vwmaccu_vv_u64m8_tumu(vmask, vec_sqsum, vec_pixel, vec_pixel, vl);
                nz += __riscv_vcpop_m_b8(vmask, vl);
            }
        }
        nz /= 4;
    } else {
        for (int i = 0; i < height; i++) {
            const uchar*  src_row = src_data + i * src_step;
            int j = 0, vl;
            for ( ; j <  width*4; j += vl) {
                vl = __riscv_vsetvl_e8m1(width*4 - j);
                auto vec_pixel_u8 = __riscv_vle8_v_u8m1(src_row + j, vl);
                auto vec_pixel = __riscv_vzext_vf4(vec_pixel_u8, vl);
                vec_sum = __riscv_vwaddu_wv_u64m8_tu(vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vwmaccu_vv_u64m8_tu(vec_sqsum, vec_pixel, vec_pixel, vl);
            }
        }
        nz = height * width;
    }
    if (nz == 0) {
        if (mean_val) *mean_val = 0.0;
        if (stddev_val) *stddev_val = 0.0;
        return CV_HAL_ERROR_OK;
    }
    uint64_t s[256], sq[256], sum[4] = {0}, sqsum[4] = {0};
    __riscv_vse64(s, vec_sum, vlmax);
    __riscv_vse64(sq, vec_sqsum, vlmax);
    for (int i = 0; i < vlmax; ++i)
    {
        sum[i % 4] += s[i];
        sqsum[i % 4] += sq[i];
    }
    if (mean_val) {
        mean_val[0] = (double)sum[0] / nz;
        mean_val[1] = (double)sum[1] / nz;
        mean_val[2] = (double)sum[2] / nz;
        mean_val[3] = (double)sum[3] / nz;
    }
    if (stddev_val) {
        stddev_val[0] = std::sqrt(std::max(((double)sqsum[0] / nz) - (mean_val[0] * mean_val[0]), 0.0));
        stddev_val[1] = std::sqrt(std::max(((double)sqsum[1] / nz) - (mean_val[1] * mean_val[1]), 0.0));
        stddev_val[2] = std::sqrt(std::max(((double)sqsum[2] / nz) - (mean_val[2] * mean_val[2]), 0.0));
        stddev_val[3] = std::sqrt(std::max(((double)sqsum[3] / nz) - (mean_val[3] * mean_val[3]), 0.0));
    }
    return CV_HAL_ERROR_OK;
}

inline int meanStdDev_32FC1(const uchar* src_data, size_t src_step, int width, int height,
                             double* mean_val, double* stddev_val, uchar* mask, size_t mask_step) {
    int nz = 0;
    int vlmax = __riscv_vsetvlmax_e64m4();
    vfloat64m4_t vec_sum = __riscv_vfmv_v_f_f64m4(0, vlmax);
    vfloat64m4_t vec_sqsum = __riscv_vfmv_v_f_f64m4(0, vlmax);
    src_step /= sizeof(float);
    if (mask) {
        for (int i = 0; i < height; ++i) {
            const float* src_row0 = reinterpret_cast<const float*>(src_data) + i * src_step;
            const uchar* mask_row = mask + i * mask_step;
            int j = 0, vl;
            for ( ; j < width; j += vl) {
                vl = __riscv_vsetvl_e32m2(width - j);
                auto vec_pixel = __riscv_vle32_v_f32m2(src_row0 + j, vl);
                auto vmask_u8 = __riscv_vle8_v_u8mf2(mask_row + j, vl);
                auto vmask_u32 = __riscv_vzext_vf4(vmask_u8, vl);
                auto vmask = __riscv_vmseq_vx_u32m2_b16(vmask_u32, 1, vl);
                vec_sum = __riscv_vfwadd_wv_f64m4_tumu(vmask, vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vfwmacc_vv_f64m4_tumu(vmask, vec_sqsum, vec_pixel, vec_pixel, vl);
                nz += __riscv_vcpop_m_b16(vmask, vl);
            }
        }
    } else {
        for (int i = 0; i < height; i++) {
            const float* src_row0 = reinterpret_cast<const float*>(src_data) + i * src_step;
            int j = 0, vl;
            for ( ; j < width; j += vl) {
                vl = __riscv_vsetvl_e32m2(width - j);
                auto vec_pixel = __riscv_vle32_v_f32m2(src_row0 + j, vl);
                vec_sum = __riscv_vfwadd_wv_f64m4_tu(vec_sum, vec_sum, vec_pixel, vl);
                vec_sqsum = __riscv_vfwmacc_vv_f64m4_tu(vec_sqsum, vec_pixel, vec_pixel, vl);
            }
        }
        nz = height * width;
    }
    if (nz == 0) {
        if (mean_val) *mean_val = 0.0;
        if (stddev_val) *stddev_val = 0.0;
        return CV_HAL_ERROR_OK;
    }
    auto zero = __riscv_vfmv_v_f_f64m1(0, vlmax);
    auto vec_red = __riscv_vfmv_v_f_f64m1(0, vlmax);
    auto vec_reddev = __riscv_vfmv_v_f_f64m1(0, vlmax);
    vec_red = __riscv_vfredusum(vec_sum, zero, vlmax);
    vec_reddev = __riscv_vfredusum(vec_sqsum, zero, vlmax);
    double sum = __riscv_vfmv_f(vec_red);
    double mean = sum / nz;
    if (mean_val) {
        *mean_val = mean;
    }
    if (stddev_val) {
        double sqsum = __riscv_vfmv_f(vec_reddev);
        double variance = std::max((sqsum / nz) - (mean * mean), 0.0);
        double stddev = std::sqrt(variance);
        *stddev_val = stddev;
    }
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

