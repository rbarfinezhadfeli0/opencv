# Documentation for `hal/riscv-rvv/src/imgproc/sep_filter.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/imgproc/sep_filter.cpp`
- **File Name**: `sep_filter.cpp`
- **File Size**: 10,734 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/imgproc/sep_filter.cpp](../../../../hal/riscv-rvv/src/imgproc/sep_filter.cpp)

## Purpose and Role

This file is located in the `hal/riscv-rvv/src/imgproc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2025, Institute of Software, Chinese Academy of Sciences.

#include "rvv_hal.hpp"
#include "common.hpp"

namespace cv { namespace rvv_hal { namespace imgproc {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

struct sepFilter2D
{
    int src_type;
    int dst_type;
    int kernel_type;
    const uchar* kernelx_data;
    int kernelx_length;
    const uchar* kernely_data;
    int kernely_length;
    int anchor_x;
    int anchor_y;
    double delta;
    int borderType;
};

// the algorithm is copied from 3rdparty/carotene/src/separable_filter.hpp,
// in the functor RowFilter3x3S16Generic and ColFilter3x3S16Generic
template<int ksize, typename T>
static inline int sepFilter(int start, int end, sepFilter2D* data, const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int width, int height, int full_width, int full_height, int offset_x, int offset_y)
{
    constexpr int noval = std::numeric_limits<int>::max();
    auto accessX = [&](int x) {
        int pi;
        if (data->borderType & BORDER_ISOLATED)
        {
            pi = common::borderInterpolate(x - data->anchor_y, height, data->borderType & ~BORDER_ISOLATED);
            pi = pi < 0 ? noval : pi;
        }
        else
        {
            pi = common::borderInterpolate(offset_y + x - data->anchor_y, full_height, data->borderType);
            pi = pi < 0 ? noval : pi - offset_y;
        }
        return pi;
    };
    auto accessY = [&](int y) {
        int pj;
        if (data->borderType & BORDER_ISOLATED)
        {
            pj = common::borderInterpolate(y - data->anchor_x, width, data->borderType & ~BORDER_ISOLATED);
            pj = pj < 0 ? noval : pj;
        }
        else
        {
            pj = common::borderInterpolate(offset_x + y - data->anchor_x, full_width, data->borderType);
            pj = pj < 0 ? noval : pj - offset_x;
        }
        return pj;
    };
    auto p2idx = [&](int x, int y){ return (x + ksize) % ksize * width + y; };

    const float* kx = reinterpret_cast<const float*>(data->kernelx_data);
    const float* ky = reinterpret_cast<const float*>(data->kernely_data);
    std::vector<float> res(width * ksize);
    auto process = [&](int x, int y) {
        float sum = 0;
        for (int i = 0; i < ksize; i++)
        {
            int p = accessY(y + i);
            if (p != noval)
            {
                sum += kx[i] * reinterpret_cast<const T*>(src_data + x * src_step)[p];
            }
        }
        res[p2idx(x, y)] = sum;
    };

    const int left = data->anchor_x, right = width - (ksize - 1 - data->anchor_x);
    for (int i = start - data->anchor_y; i < end + (ksize - 1 - data->anchor_y); i++)
    {
        if (i + offset_y >= 0 && i + offset_y < full_height)
        {
            if (left >= right)
            {
                for (int j = 0; j < width; j++)
                    process(i, j);
            }
            else
            {
                for (int j = 0; j < left; j++)
                    process(i, j);
                for (int j = right; j < width; j++)
                    process(i, j);

                int vl;
                for (int j = left; j < right; j += vl)
                {
                    vl = __riscv_vsetvl_e8m2(right - j);
                    const T* extra = reinterpret_cast<const T*>(src_data + i * src_step) + j - data->anchor_x;
                    vfloat32m8_t src;
                    if (std::is_same<T, uchar>::value)
                    {
                        src = __riscv_vfwcvt_f(__riscv_vwcvtu_x(__riscv_vle8_v_u8m2(reinterpret_cast<const uchar*>(extra), vl), vl), vl);
                    }
                    else if (std::is_same<T, short>::value)
                    {
                        src = __riscv_vfwcvt_f(__riscv_vle16_v_i16m4(reinterpret_cast<const short*>(extra), vl), vl);
                    }
                    else
                    {
                        src = __riscv_vle32_v_f32m8(reinterpret_cast<const float*>(extra), vl);
                    }

                    extra += vl;
                    auto sum = __riscv_vfmul(src, kx[0], vl);
                    src = __riscv_vfslide1down(src, extra[0], vl);
                    sum = __riscv_vfmacc(sum, kx[1], src, vl);
                    src = __riscv_vfslide1down(src, extra[1], vl);
                    sum = __riscv_vfmacc(sum, kx[2], src, vl);
                    if (ksize == 5)
                    {
                        src = __riscv_vfslide1down(src, extra[2], vl);
                        sum = __riscv_vfmacc(sum, kx[3], src, vl);
                        src = __riscv_vfslide1down(src, extra[3], vl);
                        sum = __riscv_vfmacc(sum, kx[4], src, vl);
                    }
                    __riscv_vse32(res.data() + p2idx(i, j), sum, vl);
                }
            }
        }

        int cur = i - (ksize - 1 - data->anchor_y);
        if (cur >= start)
        {
            const float* row0 = accessX(cur    ) == noval ? nullptr : res.data() + p2idx(accessX(cur    ), 0);
            const float* row1 = accessX(cur + 1) == noval ? nullptr : res.data() + p2idx(accessX(cur + 1), 0);
            const float* row2 = accessX(cur + 2) == noval ? nullptr : res.data() + p2idx(accessX(cur + 2), 0);
            const float* row3 = nullptr, *row4 = nullptr;
            if (ksize == 5)
            {
                row3 = accessX(cur + 3) == noval ? nullptr : res.data() + p2idx(accessX(cur + 3), 0);
                row4 = accessX(cur + 4) == noval ? nullptr : res.data() + p2idx(accessX(cur + 4), 0);
            }

            int vl;
            for (int j = 0; j < width; j += vl)
            {
                vl = __riscv_vsetvl_e32m4(width - j);
                auto v0 = row0 ? __riscv_vle32_v_f32m4(row0 + j, vl) : __riscv_vfmv_v_f_f32m4(0, vl);
                auto v1 = row1 ? __riscv_vle32_v_f32m4(row1 + j, vl) : __riscv_vfmv_v_f_f32m4(0, vl);
                auto v2 = row2 ? __riscv_vle32_v_f32m4(row2 + j, vl) : __riscv_vfmv_v_f_f32m4(0, vl);
                auto sum = __riscv_vfmacc(__riscv_vfmacc(__riscv_vfmacc(__riscv_vfmv_v_f_f32m4(data->delta, vl), ky[0], v0, vl), ky[1], v1, vl), ky[2], v2, vl);

                if (ksize == 5)
                {
                    auto v3 = row3 ? __riscv_vle32_v_f32m4(row3 + j, vl) : __riscv_vfmv_v_f_f32m4(0, vl);
                    auto v4 = row4 ? __riscv_vle32_v_f32m4(row4 + j, vl) : __riscv_vfmv_v_f_f32m4(0, vl);
                    sum = __riscv_vfmacc(__riscv_vfmacc(sum, ky[3], v3, vl), ky[4], v4, vl);
                }

                if (data->dst_type == CV_16SC1)
                {
                    __riscv_vse16(reinterpret_cast<short*>(dst_data + cur * dst_step) + j, __riscv_vfncvt_x(sum, vl), vl);
                }
                else
                {
                    __riscv_vse32(reinterpret_cast<float*>(dst_data + cur * dst_step) + j, sum, vl);
                }
            }
        }
    }

    return CV_HAL_ERROR_OK;
}

} // anonymous

int sepFilterInit(cvhalFilter2D **context, int src_type, int dst_type, int kernel_type, uchar* kernelx_data, int kernelx_length, uchar* kernely_data, int kernely_length, int anchor_x, int anchor_y, double delta, int borderType)
{
    if (kernel_type != CV_32FC1)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    if (src_type != CV_8UC1 && src_type != CV_16SC1 && src_type != CV_32FC1)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    if (dst_type != CV_16SC1 && dst_type != CV_32FC1)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    if ((kernelx_length != 3 && kernelx_length != 5) || kernelx_length != kernely_length)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    if ((borderType & ~BORDER_ISOLATED) == BORDER_WRAP)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;

    anchor_x = anchor_x < 0 ? kernelx_length / 2 : anchor_x;
    anchor_y = anchor_y < 0 ? kernely_length / 2 : anchor_y;
    *context = reinterpret_cast<cvhalFilter2D*>(new sepFilter2D{src_type, dst_type, kernel_type, kernelx_data, kernelx_length, kernely_data, kernely_length, anchor_x, anchor_y, delta, borderType & ~BORDER_ISOLATED});
    return CV_HAL_ERROR_OK;
}

int sepFilter(cvhalFilter2D *context, uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int width, int height, int full_width, int full_height, int offset_x, int offset_y)
{
    sepFilter2D* data = reinterpret_cast<sepFilter2D*>(context);

    uchar* _dst_data = dst_data;
    size_t _dst_step = dst_step;
    const size_t size = CV_ELEM_SIZE(data->dst_type);
    std::vector<uchar> dst;
    if (src_data == _dst_data)
    {
        dst = std::vector<uchar>(width * height * size);
        dst_data = dst.data();
        dst_step = width * size;
    }

    int res = CV_HAL_ERROR_NOT_IMPLEMENTED;
    switch (data->kernelx_length*100 + data->src_type)
    {
    case 300 + CV_8UC1:
        res = common::invoke(height, {sepFilter<3, uchar>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    case 500 + CV_8UC1:
        res = common::invoke(height, {sepFilter<5, uchar>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    case 300 + CV_16SC1:
        res = common::invoke(height, {sepFilter<3, short>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    case 500 + CV_16SC1:
        res = common::invoke(height, {sepFilter<5, short>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    case 300 + CV_32FC1:
        res = common::invoke(height, {sepFilter<3, float>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    case 500 + CV_32FC1:
        res = common::invoke(height, {sepFilter<5, float>}, data, src_data, src_step, dst_data, dst_step, width, height, full_width, full_height, offset_x, offset_y);
        break;
    }
    if (res == CV_HAL_ERROR_NOT_IMPLEMENTED)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;

    if (src_data == _dst_data)
    {
        for (int i = 0; i < height; i++)
            memcpy(_dst_data + i * _dst_step, dst.data() + i * dst_step, dst_step);
    }

    return res;
}

int sepFilterFree(cvhalFilter2D* context)
{
    delete reinterpret_cast<sepFilter2D*>(context);
    return CV_HAL_ERROR_OK;
}

#endif // CV_HAL_RVV_1P0_ENABLED

}}} // cv::rvv_hal::imgproc
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

- **sepFilter2D**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `rvv_hal.hpp`
- `common.hpp`

**Python Imports:**
- `3rdparty`


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

