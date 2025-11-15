# Documentation for `modules/core/src/stat.simd.hpp`

## File Metadata

- **Full Path**: `modules/core/src/stat.simd.hpp`
- **File Name**: `stat.simd.hpp`
- **File Size**: 3,311 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/src/stat.simd.hpp](../../../modules/core/src/stat.simd.hpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "opencv2/core/hal/intrin.hpp"

namespace cv { namespace hal {

extern const uchar popCountTable[256];

CV_CPU_OPTIMIZATION_NAMESPACE_BEGIN

// forward declarations
int normHamming(const uchar* a, int n);
int normHamming(const uchar* a, const uchar* b, int n);

#ifndef CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY

#if CV_AVX2
static inline int _mm256_extract_epi32_(__m256i reg, const int i)
{
    CV_DECL_ALIGNED(32) int reg_data[8];
    CV_DbgAssert(0 <= i && i < 8);
    _mm256_store_si256((__m256i*)reg_data, reg);
    return reg_data[i];
}
#endif

int normHamming(const uchar* a, int n)
{
    CV_AVX_GUARD;

    int i = 0;
    int result = 0;

#if (CV_SIMD || CV_SIMD_SCALABLE)
    {
        v_uint64 t = vx_setzero_u64();
        for (; i <= n - VTraits<v_uint8>::vlanes(); i += VTraits<v_uint8>::vlanes())
            t = v_add(t, v_popcount(v_reinterpret_as_u64(vx_load(a + i))));
        result = (int)v_reduce_sum(t);
        vx_cleanup();
    }
#endif

#if CV_POPCNT
    {
#  if defined CV_POPCNT_U64
        for(; i <= n - 8; i += 8)
        {
            uint64_t val;
            std::memcpy(&val, a + i, sizeof(val));
            result += (int)CV_POPCNT_U64(val);
        }
#  endif
        for(; i <= n - 4; i += 4)
        {
            uint32_t val;
            std::memcpy(&val, a + i, sizeof(val));
            result += CV_POPCNT_U32(val);
        }
    }
#endif
#if CV_ENABLE_UNROLLED
    for(; i <= n - 4; i += 4)
    {
        result += popCountTable[a[i]] + popCountTable[a[i+1]] +
        popCountTable[a[i+2]] + popCountTable[a[i+3]];
    }
#endif
    for(; i < n; i++)
    {
        result += popCountTable[a[i]];
    }
    return result;
}

int normHamming(const uchar* a, const uchar* b, int n)
{
    CV_AVX_GUARD;

    int i = 0;
    int result = 0;

#if (CV_SIMD || CV_SIMD_SCALABLE)
    {
        v_uint64 t = vx_setzero_u64();
        for (; i <= n - VTraits<v_uint8>::vlanes(); i += VTraits<v_uint8>::vlanes())
            t = v_add(t, v_popcount(v_reinterpret_as_u64(v_xor(vx_load(a + i), vx_load(b + i)))));
        result += (int)v_reduce_sum(t);
    }
#endif

#if CV_POPCNT
    {
#  if defined CV_POPCNT_U64
        for(; i <= n - 8; i += 8)
        {
            uint64_t val_a, val_b;
            std::memcpy(&val_a, a + i, sizeof(val_a));
            std::memcpy(&val_b, b + i, sizeof(val_b));
            result += (int)CV_POPCNT_U64(val_a ^ val_b);
        }
#  endif
        for(; i <= n - 4; i += 4)
        {
            uint32_t val_a, val_b;
            std::memcpy(&val_a, a + i, sizeof(val_a));
            std::memcpy(&val_b, b + i, sizeof(val_b));
            result += (int)CV_POPCNT_U32(val_a ^ val_b);
        }
    }
#endif
#if CV_ENABLE_UNROLLED
    for(; i <= n - 4; i += 4)
    {
        result += popCountTable[a[i] ^ b[i]] + popCountTable[a[i+1] ^ b[i+1]] +
                popCountTable[a[i+2] ^ b[i+2]] + popCountTable[a[i+3] ^ b[i+3]];
    }
#endif
    for(; i < n; i++)
    {
        result += popCountTable[a[i] ^ b[i]];
    }
    return result;
}

#endif // CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY

CV_CPU_OPTIMIZATION_NAMESPACE_END
}} //cv::hal
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **CV_CPU_OPTIMIZATION_DECLARATIONS_ONLY()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/intrin.hpp`


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

