# Documentation for `docs/hal/riscv-rvv/src/core/minmax.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/riscv-rvv/src/core/minmax.cpp_docs.md`
- **File Name**: `minmax.cpp_docs.md`
- **File Size**: 14,279 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/riscv-rvv/src/core/minmax.cpp_docs.md](../../../../../docs/hal/riscv-rvv/src/core/minmax.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/riscv-rvv/src/core/minmax.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/minmax.cpp`
- **File Name**: `minmax.cpp`
- **File Size**: 11,370 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/minmax.cpp](../../../../hal/riscv-rvv/src/core/minmax.cpp)

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

template<typename VEC_T, typename BOOL_T, typename T = typename VEC_T::ElemType>
inline int minMaxIdxReadTwice(const uchar* src_data, size_t src_step, int width, int height, double* minVal, double* maxVal,
                              int* minIdx, int* maxIdx, uchar* mask, size_t mask_step)
{
    int vlmax = VEC_T::setvlmax();
    auto vec_min = VEC_T::vmv(std::numeric_limits<T>::max(), vlmax);
    auto vec_max = VEC_T::vmv(std::numeric_limits<T>::lowest(), vlmax);
    T val_min, val_max;

    if (mask)
    {
        for (int i = 0; i < height; i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            const uchar* mask_row = mask + i * mask_step;
            int vl;
            for (int j = 0; j < width; j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);
                auto vec_mask = BOOL_T::vload(mask_row + j, vl);
                auto bool_mask = __riscv_vmsne(vec_mask, 0, vl);
                vec_min = VEC_T::vmin_tumu(bool_mask, vec_min, vec_min, vec_src, vl);
                vec_max = VEC_T::vmax_tumu(bool_mask, vec_max, vec_max, vec_src, vl);
            }
        }

        auto sc_minval = VEC_T::vmv(std::numeric_limits<T>::max(), vlmax);
        auto sc_maxval = VEC_T::vmv(std::numeric_limits<T>::lowest(), vlmax);
        sc_minval = VEC_T::vredmin(vec_min, sc_minval, vlmax);
        sc_maxval = VEC_T::vredmax(vec_max, sc_maxval, vlmax);
        val_min = __riscv_vmv_x(sc_minval);
        val_max = __riscv_vmv_x(sc_maxval);

        bool found_min = !minIdx, found_max = !maxIdx;
        for (int i = 0; i < height && (!found_min || !found_max); i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            const uchar* mask_row = mask + i * mask_step;
            int vl;
            for (int j = 0; j < width && (!found_min || !found_max); j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);
                auto vec_mask = BOOL_T::vload(mask_row + j, vl);
                auto bool_mask = __riscv_vmsne(vec_mask, 0, vl);
                auto bool_zero = __riscv_vmxor(bool_mask, bool_mask, vl);
                if (!found_min)
                {
                    auto bool_minpos = __riscv_vmseq_mu(bool_mask, bool_zero, vec_src, val_min, vl);
                    int index = __riscv_vfirst(bool_minpos, vl);
                    if (index != -1)
                    {
                        found_min = true;
                        minIdx[0] = i;
                        minIdx[1] = j + index;
                    }
                }
                if (!found_max)
                {
                    auto bool_maxpos = __riscv_vmseq_mu(bool_mask, bool_zero, vec_src, val_max, vl);
                    int index = __riscv_vfirst(bool_maxpos, vl);
                    if (index != -1)
                    {
                        found_max = true;
                        maxIdx[0] = i;
                        maxIdx[1] = j + index;
                    }
                }
            }
        }
    }
    else
    {
        for (int i = 0; i < height; i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            int vl;
            for (int j = 0; j < width; j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);
                vec_min = VEC_T::vmin_tu(vec_min, vec_min, vec_src, vl);
                vec_max = VEC_T::vmax_tu(vec_max, vec_max, vec_src, vl);
            }
        }

        auto sc_minval = VEC_T::vmv(std::numeric_limits<T>::max(), vlmax);
        auto sc_maxval = VEC_T::vmv(std::numeric_limits<T>::lowest(), vlmax);
        sc_minval = VEC_T::vredmin(vec_min, sc_minval, vlmax);
        sc_maxval = VEC_T::vredmax(vec_max, sc_maxval, vlmax);
        val_min = __riscv_vmv_x(sc_minval);
        val_max = __riscv_vmv_x(sc_maxval);

        bool found_min = !minIdx, found_max = !maxIdx;
        for (int i = 0; i < height && (!found_min || !found_max); i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            int vl;
            for (int j = 0; j < width && (!found_min || !found_max); j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);
                if (!found_min)
                {
                    auto bool_minpos = __riscv_vmseq(vec_src, val_min, vl);
                    int index = __riscv_vfirst(bool_minpos, vl);
                    if (index != -1)
                    {
                        found_min = true;
                        minIdx[0] = i;
                        minIdx[1] = j + index;
                    }
                }
                if (!found_max)
                {
                    auto bool_maxpos = __riscv_vmseq(vec_src, val_max, vl);
                    int index = __riscv_vfirst(bool_maxpos, vl);
                    if (index != -1)
                    {
                        found_max = true;
                        maxIdx[0] = i;
                        maxIdx[1] = j + index;
                    }
                }
            }
        }
    }
    if (minVal)
    {
        *minVal = val_min;
    }
    if (maxVal)
    {
        *maxVal = val_max;
    }

    return CV_HAL_ERROR_OK;
}

template<typename VEC_T, typename BOOL_T, typename IDX_T, typename T = typename VEC_T::ElemType>
inline int minMaxIdxReadOnce(const uchar* src_data, size_t src_step, int width, int height, double* minVal, double* maxVal,
                             int* minIdx, int* maxIdx, uchar* mask, size_t mask_step)
{
    int vlmax = VEC_T::setvlmax();
    auto vec_min = VEC_T::vmv(std::numeric_limits<T>::max(), vlmax);
    auto vec_max = VEC_T::vmv(std::numeric_limits<T>::lowest(), vlmax);
    auto vec_pos = IDX_T::vid(vlmax);
    auto vec_minpos = IDX_T::vundefined(), vec_maxpos = IDX_T::vundefined();
    T val_min, val_max;

    if (mask)
    {
        for (int i = 0; i < height; i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            const uchar* mask_row = mask + i * mask_step;
            int vl;
            for (int j = 0; j < width; j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);
                auto vec_mask = BOOL_T::vload(mask_row + j, vl);
                auto bool_mask = __riscv_vmsne(vec_mask, 0, vl);
                auto bool_zero = __riscv_vmxor(bool_mask, bool_mask, vl);

                auto bool_minpos = VEC_T::vmlt_mu(bool_mask, bool_zero, vec_src, vec_min, vl);
                auto bool_maxpos = VEC_T::vmgt_mu(bool_mask, bool_zero, vec_src, vec_max, vl);
                vec_minpos = __riscv_vmerge_tu(vec_minpos, vec_minpos, vec_pos, bool_minpos, vl);
                vec_maxpos = __riscv_vmerge_tu(vec_maxpos, vec_maxpos, vec_pos, bool_maxpos, vl);

                vec_min = __riscv_vmerge_tu(vec_min, vec_min, vec_src, bool_minpos, vl);
                vec_max = __riscv_vmerge_tu(vec_max, vec_max, vec_src, bool_maxpos, vl);
                vec_pos = __riscv_vadd(vec_pos, vl, vlmax);
            }
        }
    }
    else
    {
        for (int i = 0; i < height; i++)
        {
            const T* src_row = reinterpret_cast<const T*>(src_data + i * src_step);
            int vl;
            for (int j = 0; j < width; j += vl)
            {
                vl = VEC_T::setvl(width - j);
                auto vec_src = VEC_T::vload(src_row + j, vl);

                auto bool_minpos = VEC_T::vmlt(vec_src, vec_min, vl);
                auto bool_maxpos = VEC_T::vmgt(vec_src, vec_max, vl);
                vec_minpos = __riscv_vmerge_tu(vec_minpos, vec_minpos, vec_pos, bool_minpos, vl);
                vec_maxpos = __riscv_vmerge_tu(vec_maxpos, vec_maxpos, vec_pos, bool_maxpos, vl);

                vec_min = __riscv_vmerge_tu(vec_min, vec_min, vec_src, bool_minpos, vl);
                vec_max = __riscv_vmerge_tu(vec_max, vec_max, vec_src, bool_maxpos, vl);
                vec_pos = __riscv_vadd(vec_pos, vl, vlmax);
            }
        }
    }

    val_min = std::numeric_limits<T>::max();
    val_max = std::numeric_limits<T>::lowest();
    for (int i = 0; i < vlmax; i++)
    {
        if (val_min > VEC_T::vmv_x(vec_min))
        {
            val_min = VEC_T::vmv_x(vec_min);
            if (minIdx)
            {
                minIdx[0] = __riscv_vmv_x(vec_minpos) / width;
                minIdx[1] = __riscv_vmv_x(vec_minpos) % width;
            }
        }
        if (val_max < VEC_T::vmv_x(vec_max))
        {
            val_max = VEC_T::vmv_x(vec_max);
            if (maxIdx)
            {
                maxIdx[0] = __riscv_vmv_x(vec_maxpos) / width;
                maxIdx[1] = __riscv_vmv_x(vec_maxpos) % width;
            }
        }
        vec_min = __riscv_vslidedown(vec_min, 1, vlmax);
        vec_max = __riscv_vslidedown(vec_max, 1, vlmax);
        vec_minpos = __riscv_vslidedown(vec_minpos, 1, vlmax);
        vec_maxpos = __riscv_vslidedown(vec_maxpos, 1, vlmax);
    }
    if (minVal)
    {
        *minVal = val_min;
    }
    if (maxVal)
    {
        *maxVal = val_max;
    }

    return CV_HAL_ERROR_OK;
}

int minMaxIdx(const uchar* src_data, size_t src_step, int width, int height, int depth,
              double* minVal, double* maxVal, int* minIdx, int* maxIdx, uchar* mask, size_t mask_step)
{
    if (!mask_step)
        mask_step = src_step;

    switch (depth)
    {
    case CV_8UC1:
        return minMaxIdxReadTwice<RVV_U8M1, RVV_U8M1>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_8SC1:
        return minMaxIdxReadTwice<RVV_I8M1, RVV_U8M1>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_16UC1:
        return minMaxIdxReadTwice<RVV_U16M1, RVV_U8MF2>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_16SC1:
        return minMaxIdxReadTwice<RVV_I16M1, RVV_U8MF2>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_32SC1:
        return minMaxIdxReadOnce<RVV_I32M4, RVV_U8M1, RVV_U32M4>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_32FC1:
        return minMaxIdxReadOnce<RVV_F32M4, RVV_U8M1, RVV_U32M4>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
    case CV_64FC1:
        return minMaxIdxReadOnce<RVV_F64M4, RVV_U8MF2, RVV_U32M2>(src_data, src_step, width, height, minVal, maxVal, minIdx, maxIdx, mask, mask_step);
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

