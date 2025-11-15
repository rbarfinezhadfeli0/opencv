# Documentation for `hal/riscv-rvv/src/core/sqrt.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/core/sqrt.cpp`
- **File Name**: `sqrt.cpp`
- **File Size**: 2,259 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/core/sqrt.cpp](../../../../hal/riscv-rvv/src/core/sqrt.cpp)

## Purpose and Role

This file is located in the `hal/riscv-rvv/src/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level
// directory of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2025, Institute of Software, Chinese Academy of Sciences.

#include "rvv_hal.hpp"
#include "common.hpp"

namespace cv { namespace rvv_hal { namespace core {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

template <typename SQRT_T, typename Elem = typename SQRT_T::T::ElemType>
inline int sqrt(const Elem* src, Elem* dst, int _len)
{
    size_t vl;
    for (size_t len = _len; len > 0; len -= vl, src += vl, dst += vl)
    {
        vl = SQRT_T::T::setvl(len);
        auto x = SQRT_T::T::vload(src, vl);
        SQRT_T::T::vstore(dst, common::sqrt<SQRT_T::iter_times>(x, vl), vl);
    }

    return CV_HAL_ERROR_OK;
}

template <typename SQRT_T, typename Elem = typename SQRT_T::T::ElemType>
inline int invSqrt(const Elem* src, Elem* dst, int _len)
{
    size_t vl;
    for (size_t len = _len; len > 0; len -= vl, src += vl, dst += vl)
    {
        vl = SQRT_T::T::setvl(len);
        auto x = SQRT_T::T::vload(src, vl);
        SQRT_T::T::vstore(dst, common::invSqrt<SQRT_T::iter_times>(x, vl), vl);
    }

    return CV_HAL_ERROR_OK;
}

} // anonymous

int sqrt32f(const float* src, float* dst, int len) {
    return sqrt<common::Sqrt32f<RVV_F32M8>>(src, dst, len);
}
int sqrt64f(const double* src, double* dst, int len) {
    return sqrt<common::Sqrt64f<RVV_F64M8>>(src, dst, len);
}

int invSqrt32f(const float* src, float* dst, int len) {
#ifdef __clang__
// Strange bug in clang: invSqrt use 2 LMUL registers to store mask, which will cause memory access.
// So a smaller LMUL is used here.
    return invSqrt<common::Sqrt32f<RVV_F32M4>>(src, dst, len);
#else
    return invSqrt<common::Sqrt32f<RVV_F32M8>>(src, dst, len);
#endif
}
int invSqrt64f(const double* src, double* dst, int len) {
#ifdef __clang__
// Strange bug in clang: invSqrt use 2 LMUL registers to store mask, which will cause memory access.
// So a smaller LMUL is used here.
    return invSqrt<common::Sqrt64f<RVV_F64M4>>(src, dst, len);
#else
    return invSqrt<common::Sqrt64f<RVV_F64M8>>(src, dst, len);
#endif
}

#endif // CV_HAL_RVV_1P0_ENABLED

}}}  // cv::rvv_hal::core
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

- **__clang__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
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

