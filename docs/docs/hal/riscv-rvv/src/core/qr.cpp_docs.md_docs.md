# Documentation for `docs/hal/riscv-rvv/src/core/qr.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/riscv-rvv/src/core/qr.cpp_docs.md`
- **File Name**: `qr.cpp_docs.md`
- **File Size**: 9,948 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/riscv-rvv/src/core/qr.cpp_docs.md](../../../../../docs/hal/riscv-rvv/src/core/qr.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/riscv-rvv/src/core/qr.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/qr.cpp`
- **File Name**: `qr.cpp`
- **File Size**: 6,900 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/qr.cpp](../../../../hal/riscv-rvv/src/core/qr.cpp)

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
#include <cfloat>
#include <cmath>
#include <typeinfo>
#include <vector>

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

// the algorithm is copied from core/src/matrix_decomp.cpp,
// in the function template static int cv::QRImpl
template<typename RVV_T, typename T = typename RVV_T::ElemType>
inline int QR(T* src1, size_t src1_step, int m, int n, int k, T* src2, size_t src2_step, T* dst, int* info)
{
    T eps;
    if (typeid(T) == typeid(float))
        eps = FLT_EPSILON*10;
    else if (typeid(T) == typeid(double))
        eps = DBL_EPSILON*400;
    else
        return CV_HAL_ERROR_NOT_IMPLEMENTED;

    src1_step /= sizeof(T);
    src2_step /= sizeof(T);

    size_t buf_size = m ? m + n : dst != NULL;
    std::vector<T> buffer(buf_size);
    T* val = buffer.data();
    if (dst == NULL)
        dst = val + m;

    int vlmax = RVV_T::setvlmax(), vl;
    for (int l = 0; l < n; l++)
    {
        //generate val
        int vlSize = m - l;
        auto vec_sum = RVV_T::vmv(0, vlmax);
        for (int i = 0; i < vlSize; i += vl)
        {
            vl = RVV_T::setvl(vlSize - i);
            auto vec_src = RVV_T::vload_stride(src1 + (l + i) * src1_step + l, sizeof(T) * src1_step, vl);
            RVV_T::vstore(val + i, vec_src, vl);
            vec_sum = __riscv_vfmacc_tu(vec_sum, vec_src, vec_src, vl);
        }
        T vlNorm = __riscv_vfmv_f(__riscv_vfredosum(vec_sum, RVV_BaseType<RVV_T>::vmv_s(0, vlmax), vlmax));
        T tmpV = val[0];
        val[0] = val[0] + (val[0] >= 0 ? 1 : -1) * std::sqrt(vlNorm);
        vlNorm = std::sqrt(vlNorm + val[0] * val[0] - tmpV*tmpV);
        for (int i = 0; i < vlSize; i += vl)
        {
            vl = RVV_T::setvl(vlSize - i);
            auto vec_src = RVV_T::vload(val + i, vl);
            vec_src = __riscv_vfdiv(vec_src, vlNorm, vl);
            RVV_T::vstore(val + i, vec_src, vl);
        }
        //multiply A_l*val
        for (int j = l; j < n; j++)
        {
            vec_sum = RVV_T::vmv(0, vlmax);
            for (int i = l; i < m; i += vl)
            {
                vl = RVV_T::setvl(m - i);
                auto vec_src1 = RVV_T::vload(val + i - l, vl);
                auto vec_src2 = RVV_T::vload_stride(src1 + i * src1_step + j, sizeof(T) * src1_step, vl);
                vec_sum = __riscv_vfmacc_tu(vec_sum, vec_src1, vec_src2, vl);
            }
            T v_lA = 2 * __riscv_vfmv_f(__riscv_vfredosum(vec_sum, RVV_BaseType<RVV_T>::vmv_s(0, vlmax), vlmax));

            for (int i = l; i < m; i += vl)
            {
                vl = RVV_T::setvl(m - i);
                auto vec_src1 = RVV_T::vload(val + i - l, vl);
                auto vec_src2 = RVV_T::vload_stride(src1 + i * src1_step + j, sizeof(T) * src1_step, vl);
                vec_src2 = __riscv_vfnmsac(vec_src2, v_lA, vec_src1, vl);
                RVV_T::vstore_stride(src1 + i * src1_step + j, sizeof(T) * src1_step, vec_src2, vl);
            }
        }

        //save val and factors
        dst[l] = val[0] * val[0];
        for (int i = 1; i < vlSize; i += vl)
        {
            vl = RVV_T::setvl(vlSize - i);
            auto vec_src = RVV_T::vload(val + i, vl);
            vec_src = __riscv_vfdiv(vec_src, val[0], vl);
            RVV_T::vstore_stride(src1 + (l + i) * src1_step + l, sizeof(T) * src1_step, vec_src, vl);
        }
    }

    if (src2)
    {
        //generate new rhs
        for (int l = 0; l < n; l++)
        {
            //unpack val
            val[0] = (T)1;
            for (int j = 1; j < m - l; j += vl)
            {
                vl = RVV_T::setvl(m - l - j);
                auto vec_src = RVV_T::vload_stride(src1 + (j + l) * src1_step + l, sizeof(T) * src1_step, vl);
                RVV_T::vstore(val + j, vec_src, vl);
            }

            //h_l*x
            for (int j = 0; j < k; j++)
            {
                auto vec_sum = RVV_T::vmv(0, vlmax);
                for (int i = l; i < m; i += vl)
                {
                    vl = RVV_T::setvl(m - i);
                    auto vec_src1 = RVV_T::vload(val + i - l, vl);
                    auto vec_src2 = RVV_T::vload_stride(src2 + i * src2_step + j, sizeof(T) * src2_step, vl);
                    vec_sum = __riscv_vfmacc_tu(vec_sum, vec_src1, vec_src2, vl);
                }
                T v_lB = 2 * dst[l] * __riscv_vfmv_f(__riscv_vfredosum(vec_sum, RVV_BaseType<RVV_T>::vmv_s(0, vlmax), vlmax));

                for (int i = l; i < m; i += vl)
                {
                    vl = RVV_T::setvl(m - i);
                    auto vec_src1 = RVV_T::vload(val + i - l, vl);
                    auto vec_src2 = RVV_T::vload_stride(src2 + i * src2_step + j, sizeof(T) * src2_step, vl);
                    vec_src2 = __riscv_vfnmsac(vec_src2, v_lB, vec_src1, vl);
                    RVV_T::vstore_stride(src2 + i * src2_step + j, sizeof(T) * src2_step, vec_src2, vl);
                }
            }
        }
        //do back substitution
        for (int i = n - 1; i >= 0; i--)
        {
            for (int j = n - 1; j > i; j--)
            {
                for (int p = 0; p < k; p += vl)
                {
                    vl = RVV_T::setvl(k - p);
                    auto vec_src1 = RVV_T::vload(src2 + i * src2_step + p, vl);
                    auto vec_src2 = RVV_T::vload(src2 + j * src2_step + p, vl);
                    vec_src1 = __riscv_vfnmsac(vec_src1, src1[i*src1_step + j], vec_src2, vl);
                    RVV_T::vstore(src2 + i * src2_step + p, vec_src1, vl);
                }
            }
            if (std::abs(src1[i*src1_step + i]) < eps)
            {
                *info = 0;
                return CV_HAL_ERROR_OK;
            }
            for (int p = 0; p < k; p += vl)
            {
                vl = RVV_T::setvl(k - p);
                auto vec_src = RVV_T::vload(src2 + i * src2_step + p, vl);
                vec_src = __riscv_vfdiv(vec_src, src1[i*src1_step + i], vl);
                RVV_T::vstore(src2 + i * src2_step + p, vec_src, vl);
            }
        }
    }

    *info = 1;
    return CV_HAL_ERROR_OK;
}

} // anonymous

int QR32f(float* src1, size_t src1_step, int m, int n, int k, float* src2, size_t src2_step, float* dst, int* info) {
    return QR<RVV_F32M4>(src1, src1_step, m, n, k, src2, src2_step, dst, info);
}
int QR64f(double* src1, size_t src1_step, int m, int n, int k, double* src2, size_t src2_step, double* dst, int* info) {
    return QR<RVV_F64M4>(src1, src1_step, m, n, k, src2, src2_step, dst, info);
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

### Functions and Methods

- **template()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cmath`
- `cfloat`
- `rvv_hal.hpp`
- `vector`
- `typeinfo`

**Python Imports:**
- `core`


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

