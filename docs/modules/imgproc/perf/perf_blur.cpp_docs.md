# Documentation for `modules/imgproc/perf/perf_blur.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_blur.cpp`
- **File Name**: `perf_blur.cpp`
- **File Size**: 9,032 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_blur.cpp](../../../modules/imgproc/perf/perf_blur.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
#include "perf_precomp.hpp"

namespace opencv_test {

typedef tuple<Size, MatType, int> Size_MatType_kSize_t;
typedef perf::TestBaseWithParam<Size_MatType_kSize_t> Size_MatType_kSize;

PERF_TEST_P(Size_MatType_kSize, medianBlur,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1),
                testing::Values(3, 5)
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    int ksize = get<2>(GetParam());

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    if (CV_MAT_DEPTH(type) > CV_16S || CV_MAT_CN(type) > 1)
        declare.time(15);

    TEST_CYCLE() medianBlur(src, dst, ksize);

    SANITY_CHECK(dst);
}

CV_ENUM(BorderType3x3, BORDER_REPLICATE, BORDER_CONSTANT)
CV_ENUM(BorderType, BORDER_REPLICATE, BORDER_CONSTANT, BORDER_REFLECT, BORDER_REFLECT101)

typedef tuple<Size, MatType, BorderType3x3> Size_MatType_BorderType3x3_t;
typedef perf::TestBaseWithParam<Size_MatType_BorderType3x3_t> Size_MatType_BorderType3x3;

typedef tuple<Size, MatType, BorderType> Size_MatType_BorderType_t;
typedef perf::TestBaseWithParam<Size_MatType_BorderType_t> Size_MatType_BorderType;

typedef tuple<Size, int, BorderType3x3> Size_ksize_BorderType_t;
typedef perf::TestBaseWithParam<Size_ksize_BorderType_t> Size_ksize_BorderType;

typedef tuple<Size, MatType, BorderType, int> Size_MatType_BorderType_ksize_t;
typedef perf::TestBaseWithParam<Size_MatType_BorderType_ksize_t> Size_MatType_BorderType_ksize;


PERF_TEST_P(Size_MatType_BorderType3x3, gaussianBlur3x3,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1),
                BorderType3x3::all()
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    BorderType3x3 btype = get<2>(GetParam());

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() GaussianBlur(src, dst, Size(3,3), 0, 0, btype);

    SANITY_CHECK(dst, 1);
}

PERF_TEST_P(Size_MatType_BorderType3x3, blur3x3,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1),
                BorderType3x3::all()
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    BorderType3x3 btype = get<2>(GetParam());

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() blur(src, dst, Size(3,3), Point(-1,-1), btype);

    SANITY_CHECK(dst, 1);
}

PERF_TEST_P(Size_MatType_BorderType, blur16x16,
            testing::Combine(
                testing::Values(szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1),
                BorderType::all()
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    BorderType btype = get<2>(GetParam());
    double eps = 1.25e-3;

    eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : eps;

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() blur(src, dst, Size(16,16), Point(-1,-1), btype);

    SANITY_CHECK(dst, eps);
}

PERF_TEST_P(Size_MatType_BorderType_ksize, box,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_32FC3),
                BorderType::all(),
                testing::Values(3, 5)
                )
            )
{
    auto p = GetParam();
    Size       size  = get<0>(p);
    int        type  = get<1>(p);
    BorderType btype = get<2>(p);
    int        ksize = get<3>(p);

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() boxFilter(src, dst, -1, Size(ksize, ksize), Point(-1,-1), false, btype);

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_ksize_BorderType, box_CV8U_CV16U,
            testing::Combine(
                    testing::Values(szODD, szQVGA, szVGA, sz720p),
                    testing::Values(3, 5, 15),
                    BorderType3x3::all()
                    )
            )
{
    Size size = get<0>(GetParam());
    int ksize = get<1>(GetParam());
    BorderType3x3 btype = get<2>(GetParam());

    Mat src(size, CV_8UC1);
    Mat dst(size, CV_16UC1);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() boxFilter(src, dst, CV_16UC1, Size(ksize, ksize), Point(-1,-1), false, btype);

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType_BorderType_ksize, box_inplace,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_16SC1, CV_32SC1, CV_32FC1, CV_32FC3),
                BorderType::all(),
                testing::Values(3, 5)
                )
            )
{
    auto p = GetParam();
    Size       size  = get<0>(p);
    int        type  = get<1>(p);
    BorderType btype = get<2>(p);
    int        ksize = get<3>(p);

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    while(next())
    {
        src.copyTo(dst);
        startTimer();
        boxFilter(dst, dst, -1, Size(ksize, ksize), Point(-1,-1), false, btype);
        stopTimer();
    }

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
}

PERF_TEST_P(Size_MatType_BorderType, gaussianBlur5x5,
            testing::Combine(
                testing::Values(szODD, szQVGA, szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1),
                BorderType::all()
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    BorderType btype = get<2>(GetParam());

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() GaussianBlur(src, dst, Size(5,5), 0, 0, btype);

    SANITY_CHECK(dst, 1);
}

PERF_TEST_P(Size_MatType_BorderType, blur5x5,
            testing::Combine(
                testing::Values(szVGA, sz720p),
                testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1, CV_32FC3),
                BorderType::all()
                )
            )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    BorderType btype = get<2>(GetParam());

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() blur(src, dst, Size(5,5), Point(-1,-1), btype);

    SANITY_CHECK(dst, 1);
}

///////////// BlendLinear ////////////////////////
PERF_TEST_P(Size_MatType, BlendLinear,
            testing::Combine(
                testing::Values(szVGA, sz720p, sz1080p, sz2160p),
                testing::Values(CV_8UC1, CV_32FC1, CV_8UC3, CV_32FC3, CV_8UC4, CV_32FC4)
                )
           )
{
    const Size srcSize = get<0>(GetParam());
    const int srcType = get<1>(GetParam());

    Mat src1(srcSize, srcType), src2(srcSize, srcType), dst(srcSize, srcType);
    Mat weights1(srcSize, CV_32FC1), weights2(srcSize, CV_32FC1);

    declare.in(src1, src2, WARMUP_RNG).in(weights1, weights2, WARMUP_READ).out(dst);
    randu(weights1, 0, 1);
    randu(weights2, 0, 1);

    TEST_CYCLE() blendLinear(src1, src2, weights1, weights2, dst);

    SANITY_CHECK_NOTHING();
}

///////////// Stackblur ////////////////////////
PERF_TEST_P(Size_MatType, stackblur3x3,
            testing::Combine(
                    testing::Values(sz720p, sz1080p, sz2160p),
                    testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1)
            )
)
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    double eps = 1e-3;

    eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : eps;

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() stackBlur(src, dst, Size(3,3));

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Size_MatType, stackblur101x101,
            testing::Combine(
                    testing::Values(sz720p, sz1080p, sz2160p),
                    testing::Values(CV_8UC1, CV_8UC4, CV_16UC1, CV_16SC1, CV_32FC1)
            )
)
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    double eps = 1e-3;

    eps = CV_MAT_DEPTH(type) <= CV_32S ? 1 : eps;

    Mat src(size, type);
    Mat dst(size, type);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() stackBlur(src, dst, Size(101,101));

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

