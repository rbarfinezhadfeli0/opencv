# Documentation for `modules/imgproc/perf/perf_sepfilters.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_sepfilters.cpp`
- **File Name**: `perf_sepfilters.cpp`
- **File Size**: 8,066 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_sepfilters.cpp](../../../modules/imgproc/perf/perf_sepfilters.cpp)

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

#define FILTER_SRC_SIZES szODD, szQVGA, szVGA

CV_ENUM(BorderType3x3, BORDER_REPLICATE, BORDER_CONSTANT)
CV_ENUM(BorderType3x3ROI, BORDER_DEFAULT, BORDER_REPLICATE|BORDER_ISOLATED, BORDER_CONSTANT|BORDER_ISOLATED)

CV_ENUM(BorderType, BORDER_REPLICATE, BORDER_CONSTANT, BORDER_REFLECT, BORDER_REFLECT101)
CV_ENUM(BorderTypeROI, BORDER_DEFAULT, BORDER_REPLICATE|BORDER_ISOLATED, BORDER_CONSTANT|BORDER_ISOLATED, BORDER_REFLECT|BORDER_ISOLATED, BORDER_REFLECT101|BORDER_ISOLATED)

typedef tuple<Size, MatType, tuple<int, int>, BorderType3x3> Size_MatType_dx_dy_Border3x3_t;
typedef perf::TestBaseWithParam<Size_MatType_dx_dy_Border3x3_t> Size_MatType_dx_dy_Border3x3;

typedef tuple<Size, MatType, tuple<int, int>, BorderType3x3ROI> Size_MatType_dx_dy_Border3x3ROI_t;
typedef perf::TestBaseWithParam<Size_MatType_dx_dy_Border3x3ROI_t> Size_MatType_dx_dy_Border3x3ROI;

typedef tuple<Size, MatType, tuple<int, int>, BorderType> Size_MatType_dx_dy_Border5x5_t;
typedef perf::TestBaseWithParam<Size_MatType_dx_dy_Border5x5_t> Size_MatType_dx_dy_Border5x5;

typedef tuple<Size, MatType, tuple<int, int>, BorderTypeROI> Size_MatType_dx_dy_Border5x5ROI_t;
typedef perf::TestBaseWithParam<Size_MatType_dx_dy_Border5x5ROI_t> Size_MatType_dx_dy_Border5x5ROI;


/**************** Sobel ********************/

PERF_TEST_P(Size_MatType_dx_dy_Border3x3, sobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0), make_tuple(1, 1), make_tuple(0, 2), make_tuple(2, 0), make_tuple(2, 2)),
                BorderType3x3::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3 border = get<3>(GetParam());

    Mat src(size, CV_8U);
    Mat dst(size, ddepth);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, 3, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border3x3ROI, sobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0), make_tuple(1, 1), make_tuple(0, 2), make_tuple(2, 0), make_tuple(2, 2)),
                BorderType3x3ROI::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3ROI border = get<3>(GetParam());

    Mat src(size.height + 10, size.width + 10, CV_8U);
    Mat dst(size, ddepth);

    warmup(src, WARMUP_RNG);
    src = src(Range(5, 5 + size.height), Range(5, 5 + size.width));

    declare.in(src).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, 3, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border5x5, sobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0), make_tuple(1, 1), make_tuple(0, 2), make_tuple(2, 0)),
                BorderType::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType border = get<3>(GetParam());

    Mat src(size, CV_8U);
    Mat dst(size, ddepth);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, 5, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border5x5ROI, sobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0), make_tuple(1, 1), make_tuple(0, 2), make_tuple(2, 0)),
                BorderTypeROI::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderTypeROI border = get<3>(GetParam());

    Mat src(size.height + 10, size.width + 10, CV_8U);
    Mat dst(size, ddepth);

    warmup(src, WARMUP_RNG);
    src = src(Range(5, 5 + size.height), Range(5, 5 + size.width));

    declare.in(src).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, 5, 1, 0, border);

    SANITY_CHECK(dst);
}

/**************** Scharr ********************/

PERF_TEST_P(Size_MatType_dx_dy_Border3x3, scharrFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0)),
                BorderType3x3::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3 border = get<3>(GetParam());

    Mat src(size, CV_8U);
    Mat dst(size, ddepth);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() Scharr(src, dst, ddepth, dx, dy, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border3x3ROI, scharrFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0)),
                BorderType3x3ROI::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3ROI border = get<3>(GetParam());

    Mat src(size.height + 10, size.width + 10, CV_8U);
    Mat dst(size, ddepth);

    warmup(src, WARMUP_RNG);
    src = src(Range(5, 5 + size.height), Range(5, 5 + size.width));

    declare.in(src).out(dst);

    TEST_CYCLE() Scharr(src, dst, ddepth, dx, dy, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border3x3, scharrViaSobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0)),
                BorderType3x3::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3 border = get<3>(GetParam());

    Mat src(size, CV_8U);
    Mat dst(size, ddepth);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, -1, 1, 0, border);

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType_dx_dy_Border3x3ROI, scharrViaSobelFilter,
            testing::Combine(
                testing::Values(FILTER_SRC_SIZES),
                testing::Values(CV_16S, CV_32F),
                testing::Values(make_tuple(0, 1), make_tuple(1, 0)),
                BorderType3x3ROI::all()
            )
          )
{
    Size size = get<0>(GetParam());
    int ddepth = get<1>(GetParam());
    int dx = get<0>(get<2>(GetParam()));
    int dy = get<1>(get<2>(GetParam()));
    BorderType3x3ROI border = get<3>(GetParam());

    Mat src(size.height + 10, size.width + 10, CV_8U);
    Mat dst(size, ddepth);

    warmup(src, WARMUP_RNG);
    src = src(Range(5, 5 + size.height), Range(5, 5 + size.width));

    declare.in(src).out(dst);

    TEST_CYCLE() Sobel(src, dst, ddepth, dx, dy, -1, 1, 0, border);

    SANITY_CHECK(dst);
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

