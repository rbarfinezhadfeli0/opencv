# Documentation for `modules/core/perf/perf_norm.cpp`

## File Metadata

- **Full Path**: `modules/core/perf/perf_norm.cpp`
- **File Name**: `perf_norm.cpp`
- **File Size**: 8,494 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/perf/perf_norm.cpp](../../../modules/core/perf/perf_norm.cpp)

## Purpose and Role

This file is located in the `modules/core/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "perf_precomp.hpp"

namespace opencv_test
{
using namespace perf;

#define HAMMING_NORM_SIZES cv::Size(640, 480), cv::Size(1920, 1080)
#define HAMMING_NORM_TYPES CV_8UC1

CV_FLAGS(NormType, NORM_HAMMING2, NORM_HAMMING, NORM_INF, NORM_L1, NORM_L2, NORM_TYPE_MASK, NORM_RELATIVE, NORM_MINMAX)
typedef tuple<Size, MatType, NormType> Size_MatType_NormType_t;
typedef perf::TestBaseWithParam<Size_MatType_NormType_t> Size_MatType_NormType;

PERF_TEST_P(Size_MatType_NormType, norm,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(CV_8UC1, CV_8UC4, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src(sz, matType);
    double n;

    declare.in(src, WARMUP_RNG);

    TEST_CYCLE() n = cv::norm(src, normType);

    SANITY_CHECK(n, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType_NormType, norm_mask,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(CV_8UC1, CV_8UC4, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src(sz, matType);
    Mat mask = Mat::ones(sz, CV_8U);
    double n;

    declare.in(src, WARMUP_RNG).in(mask);

    TEST_CYCLE() n = cv::norm(src, normType, mask);

    SANITY_CHECK(n, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType_NormType, norm2,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(CV_8UC1, CV_8UC4, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2, (int)(NORM_RELATIVE+NORM_INF), (int)(NORM_RELATIVE+NORM_L1), (int)(NORM_RELATIVE+NORM_L2))
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src1(sz, matType);
    Mat src2(sz, matType);
    double n;

    declare.in(src1, src2, WARMUP_RNG);

    TEST_CYCLE() n = cv::norm(src1, src2, normType);

    SANITY_CHECK(n, 1e-5, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType_NormType, norm2_mask,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(CV_8UC1, CV_8UC4, CV_8SC1, CV_16UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_64FC1),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2, (int)(NORM_RELATIVE|NORM_INF), (int)(NORM_RELATIVE|NORM_L1), (int)(NORM_RELATIVE|NORM_L2))
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src1(sz, matType);
    Mat src2(sz, matType);
    Mat mask = Mat::ones(sz, CV_8U);
    double n;

    declare.in(src1, src2, WARMUP_RNG).in(mask);

    TEST_CYCLE() n = cv::norm(src1, src2, normType, mask);

    SANITY_CHECK(n, 1e-5, ERROR_RELATIVE);
}

namespace {
typedef tuple<NormType, MatType, Size> PerfHamming_t;
typedef perf::TestBaseWithParam<PerfHamming_t> PerfHamming;

PERF_TEST_P(PerfHamming, norm,
            testing::Combine(
                testing::Values(NORM_HAMMING, NORM_HAMMING2),
                testing::Values(HAMMING_NORM_TYPES),
                testing::Values(HAMMING_NORM_SIZES)
                )
            )
{
    Size sz = get<2>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<0>(GetParam());

    Mat src(sz, matType);
    double n;

    declare.in(src, WARMUP_RNG);

    TEST_CYCLE() n = cv::norm(src, normType);

    CV_UNUSED(n);
    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(PerfHamming, norm2,
            testing::Combine(
                testing::Values(NORM_HAMMING, NORM_HAMMING2),
                testing::Values(HAMMING_NORM_TYPES),
                testing::Values(HAMMING_NORM_SIZES)
                )
            )
{
    Size sz = get<2>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<0>(GetParam());

    Mat src1(sz, matType);
    Mat src2(sz, matType);
    double n;

    declare.in(src1, src2, WARMUP_RNG);

    TEST_CYCLE() n = cv::norm(src1, src2, normType);

    CV_UNUSED(n);
    SANITY_CHECK_NOTHING();
}

}


PERF_TEST_P(Size_MatType_NormType, normalize,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(TYPICAL_MAT_TYPES),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src(sz, matType);
    Mat dst(sz, matType);

    double alpha = 100.;
    if(normType==NORM_L1) alpha = (double)src.total() * src.channels();
    if(normType==NORM_L2) alpha = (double)src.total()/10;

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() cv::normalize(src, dst, alpha, 0., normType);

    SANITY_CHECK(dst, 1e-6);
}

PERF_TEST_P(Size_MatType_NormType, normalize_mask,
            testing::Combine(
                testing::Values(::perf::szVGA, ::perf::sz1080p),
                testing::Values(TYPICAL_MAT_TYPES),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src(sz, matType);
    Mat dst(sz, matType);
    Mat mask = Mat::ones(sz, CV_8U);

    double alpha = 100.;
    if(normType==NORM_L1) alpha = (double)src.total() * src.channels();
    if(normType==NORM_L2) alpha = (double)src.total()/10;

    declare.in(src, WARMUP_RNG).in(mask).out(dst);
    declare.time(100);

    TEST_CYCLE() cv::normalize(src, dst, alpha, 0., normType, -1, mask);

    SANITY_CHECK(dst, 1e-6);
}

PERF_TEST_P(Size_MatType_NormType, normalize_32f,
            testing::Combine(
                testing::Values(TYPICAL_MAT_SIZES),
                testing::Values(TYPICAL_MAT_TYPES),
                testing::Values((int)NORM_INF, (int)NORM_L1, (int)NORM_L2)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int normType = get<2>(GetParam());

    Mat src(sz, matType);
    Mat dst(sz, CV_32F);

    double alpha = 100.;
    if(normType==NORM_L1) alpha = (double)src.total() * src.channels();
    if(normType==NORM_L2) alpha = (double)src.total()/10;

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() cv::normalize(src, dst, alpha, 0., normType, CV_32F);

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P( Size_MatType, normalize_minmax, TYPICAL_MATS )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());

    Mat src(sz, matType);
    Mat dst(sz, matType);

    declare.in(src, WARMUP_RNG).out(dst);
    declare.time(30);

    TEST_CYCLE() cv::normalize(src, dst, 20., 100., NORM_MINMAX);

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
}

typedef TestBaseWithParam< int > test_len;
PERF_TEST_P(test_len, hal_normL1_u8,
            testing::Values(300000, 2000000)
           )
{
    int len = GetParam();

    Mat src1(1, len, CV_8UC1);
    Mat src2(1, len, CV_8UC1);

    declare.in(src1, src2, WARMUP_RNG);
    double n;
    TEST_CYCLE() n = hal::normL1_(src1.ptr<uchar>(0), src2.ptr<uchar>(0), len);
    CV_UNUSED(n);
    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(test_len, hal_normL1_f32,
            testing::Values(300000, 2000000)
           )
{
    int len = GetParam();

    Mat src1(1, len, CV_32FC1);
    Mat src2(1, len, CV_32FC1);

    declare.in(src1, src2, WARMUP_RNG);
    double n;
    TEST_CYCLE() n = hal::normL1_(src1.ptr<float>(0), src2.ptr<float>(0), len);
    CV_UNUSED(n);
    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(test_len, hal_normL2Sqr,
            testing::Values(300000, 2000000)
           )
{
    int len = GetParam();

    Mat src1(1, len, CV_32FC1);
    Mat src2(1, len, CV_32FC1);

    declare.in(src1, src2, WARMUP_RNG);
    double n;
    TEST_CYCLE() n = hal::normL2Sqr_(src1.ptr<float>(0), src2.ptr<float>(0), len);
    CV_UNUSED(n);
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

### Functions and Methods

- **perf()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file
- **TestBaseWithParam()**: A function/method defined in this file


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

