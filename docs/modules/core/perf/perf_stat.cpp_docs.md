# Documentation for `modules/core/perf/perf_stat.cpp`

## File Metadata

- **Full Path**: `modules/core/perf/perf_stat.cpp`
- **File Name**: `perf_stat.cpp`
- **File Size**: 2,665 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/perf/perf_stat.cpp](../../../modules/core/perf/perf_stat.cpp)

## Purpose and Role

This file is located in the `modules/core/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "perf_precomp.hpp"

namespace opencv_test
{
using namespace perf;

PERF_TEST_P(Size_MatType, sum, TYPICAL_MATS)
{
    Size sz = get<0>(GetParam());
    int type = get<1>(GetParam());

    Mat arr(sz, type);
    Scalar s;

    declare.in(arr, WARMUP_RNG).out(s);

    TEST_CYCLE() s = sum(arr);

    SANITY_CHECK(s, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType, mean, TYPICAL_MATS)
{
    Size sz = get<0>(GetParam());
    int type = get<1>(GetParam());

    Mat src(sz, type);
    Scalar s;

    declare.in(src, WARMUP_RNG).out(s);

    TEST_CYCLE() s = mean(src);

    SANITY_CHECK(s, 1e-5);
}

PERF_TEST_P(Size_MatType, mean_mask, TYPICAL_MATS)
{
    Size sz = get<0>(GetParam());
    int type = get<1>(GetParam());

    Mat src(sz, type);
    Mat mask = Mat::ones(src.size(), CV_8U);
    Scalar s;

    declare.in(src, WARMUP_RNG).in(mask).out(s);

    TEST_CYCLE() s = mean(src, mask);

    SANITY_CHECK(s, 5e-5);
}

PERF_TEST_P(Size_MatType, meanStdDev, TYPICAL_MATS)
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());

    Mat src(sz, matType);
    Scalar mean;
    Scalar dev;

    declare.in(src, WARMUP_RNG).out(mean, dev);

    TEST_CYCLE() meanStdDev(src, mean, dev);

    SANITY_CHECK(mean, 1e-5, ERROR_RELATIVE);
    SANITY_CHECK(dev, 1e-5, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType, meanStdDev_mask, TYPICAL_MATS)
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());

    Mat src(sz, matType);
    Mat mask = Mat::ones(sz, CV_8U);
    Scalar mean;
    Scalar dev;

    declare.in(src, WARMUP_RNG).in(mask).out(mean, dev);

    TEST_CYCLE() meanStdDev(src, mean, dev, mask);

    SANITY_CHECK(mean, 1e-5);
    SANITY_CHECK(dev, 1e-5);
}

PERF_TEST_P(Size_MatType, countNonZero, testing::Combine( testing::Values( TYPICAL_MAT_SIZES ), testing::Values( CV_8UC1, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1 ) ))
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());

    Mat src(sz, matType);
    int cnt = 0;

    declare.in(src, WARMUP_RNG);

    int runs = (sz.width <= 640) ? 8 : 1;
    TEST_CYCLE_MULTIRUN(runs) cnt = countNonZero(src);

    SANITY_CHECK(cnt);
}

PERF_TEST_P(Size_MatType, hasNonZero, testing::Combine( testing::Values( TYPICAL_MAT_SIZES ), testing::Values( CV_8UC1, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1 ) ))
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());

    Mat src(sz, matType);
    /*bool hnz = false;*/

    declare.in(src, WARMUP_RNG);

    int runs = (sz.width <= 640) ? 8 : 1;
    TEST_CYCLE_MULTIRUN(runs) /*hnz =*/ hasNonZero(src);

    SANITY_CHECK_NOTHING();
}

} // namespace
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
- `perf_precomp.hpp`


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

