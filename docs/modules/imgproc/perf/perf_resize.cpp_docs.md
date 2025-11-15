# Documentation for `modules/imgproc/perf/perf_resize.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_resize.cpp`
- **File Name**: `perf_resize.cpp`
- **File Size**: 9,435 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_resize.cpp](../../../modules/imgproc/perf/perf_resize.cpp)

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

typedef tuple<MatType, Size, Size> MatInfo_Size_Size_t;
typedef TestBaseWithParam<MatInfo_Size_Size_t> MatInfo_Size_Size;
typedef tuple<Size,Size> Size_Size_t;
typedef tuple<MatType, Size_Size_t> MatInfo_SizePair_t;
typedef TestBaseWithParam<MatInfo_SizePair_t> MatInfo_SizePair;

#define MATTYPE_NE_VALUES CV_8UC1, CV_8UC2, CV_8UC3, CV_8UC4,     \
                          CV_16UC1, CV_16UC2, CV_16UC3, CV_16UC4, \
                          CV_32FC1, CV_32FC2, CV_32FC3, CV_32FC4

// For gradient-ish testing of the other matrix formats
template<typename T>
static void fillFPGradient(Mat& img)
{
    const int ch = img.channels();

    int r, c, i;
    for(r=0; r<img.rows; r++)
    {
        for(c=0; c<img.cols; c++)
        {
            T vals[] = {(T)r, (T)c, (T)(r*c), (T)(r*c/(r+c+1))};
            T *p = (T*)img.ptr(r, c);
            for(i=0; i<ch; i++) p[i] = (T)vals[i];
        }
    }
}

PERF_TEST_P(MatInfo_Size_Size, resizeUpLinear,
            testing::Values(
                MatInfo_Size_Size_t(CV_8UC1, szVGA, szqHD),
                MatInfo_Size_Size_t(CV_8UC2, szVGA, szqHD),
                MatInfo_Size_Size_t(CV_8UC3, szVGA, szqHD),
                MatInfo_Size_Size_t(CV_8UC4, szVGA, szqHD),
                MatInfo_Size_Size_t(CV_8UC1, szVGA, sz720p),
                MatInfo_Size_Size_t(CV_8UC2, szVGA, sz720p),
                MatInfo_Size_Size_t(CV_8UC3, szVGA, sz720p),
                MatInfo_Size_Size_t(CV_8UC4, szVGA, sz720p)
                )
            )
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    Size to = get<2>(GetParam());

    cv::Mat src(from, matType), dst(to, matType);
    cvtest::fillGradient(src);
    declare.in(src).out(dst);

    TEST_CYCLE_MULTIRUN(10) resize(src, dst, to, 0, 0, INTER_LINEAR_EXACT);

#ifdef __ANDROID__
    SANITY_CHECK(dst, 5);
#else
    SANITY_CHECK(dst, 1 + 1e-6);
#endif
}

PERF_TEST_P(MatInfo_SizePair, resizeUpLinearNonExact,
            testing::Combine
                (
                testing::Values( MATTYPE_NE_VALUES ),
                testing::Values( Size_Size_t(szVGA, szqHD), Size_Size_t(szVGA, sz720p) )
                )
             )
{
    int matType = get<0>(GetParam());
    Size_Size_t sizes = get<1>(GetParam());
    Size from = get<0>(sizes);
    Size to = get<1>(sizes);

    cv::Mat src(from, matType), dst(to, matType);
    switch(src.depth())
    {
        case CV_8U: cvtest::fillGradient(src); break;
        case CV_16U: fillFPGradient<ushort>(src); break;
        case CV_32F: fillFPGradient<float>(src); break;
    }
    declare.in(src).out(dst);

    TEST_CYCLE_MULTIRUN(10) resize(src, dst, to, 0, 0, INTER_LINEAR);

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(MatInfo_Size_Size, resizeDownLinear,
            testing::Values(
                MatInfo_Size_Size_t(CV_8UC1, szVGA, szQVGA),
                MatInfo_Size_Size_t(CV_8UC2, szVGA, szQVGA),
                MatInfo_Size_Size_t(CV_8UC3, szVGA, szQVGA),
                MatInfo_Size_Size_t(CV_8UC4, szVGA, szQVGA),
                MatInfo_Size_Size_t(CV_8UC1, szqHD, szVGA),
                MatInfo_Size_Size_t(CV_8UC2, szqHD, szVGA),
                MatInfo_Size_Size_t(CV_8UC3, szqHD, szVGA),
                MatInfo_Size_Size_t(CV_8UC4, szqHD, szVGA),
                MatInfo_Size_Size_t(CV_8UC1, sz720p, Size(120 * sz720p.width / sz720p.height, 120)),//face detection min_face_size = 20%
                MatInfo_Size_Size_t(CV_8UC2, sz720p, Size(120 * sz720p.width / sz720p.height, 120)),//face detection min_face_size = 20%
                MatInfo_Size_Size_t(CV_8UC3, sz720p, Size(120 * sz720p.width / sz720p.height, 120)),//face detection min_face_size = 20%
                MatInfo_Size_Size_t(CV_8UC4, sz720p, Size(120 * sz720p.width / sz720p.height, 120)),//face detection min_face_size = 20%
                MatInfo_Size_Size_t(CV_8UC1, sz720p, szVGA),
                MatInfo_Size_Size_t(CV_8UC2, sz720p, szVGA),
                MatInfo_Size_Size_t(CV_8UC3, sz720p, szVGA),
                MatInfo_Size_Size_t(CV_8UC4, sz720p, szVGA),
                MatInfo_Size_Size_t(CV_8UC1, sz720p, szQVGA),
                MatInfo_Size_Size_t(CV_8UC2, sz720p, szQVGA),
                MatInfo_Size_Size_t(CV_8UC3, sz720p, szQVGA),
                MatInfo_Size_Size_t(CV_8UC4, sz720p, szQVGA)
                )
            )
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    Size to = get<2>(GetParam());

    cv::Mat src(from, matType), dst(to, matType);
    cvtest::fillGradient(src);
    declare.in(src).out(dst);

    TEST_CYCLE_MULTIRUN(10) resize(src, dst, to, 0, 0, INTER_LINEAR_EXACT);

#ifdef __ANDROID__
    SANITY_CHECK(dst, 5);
#else
    SANITY_CHECK(dst, 1 + 1e-6);
#endif
}

PERF_TEST_P(MatInfo_SizePair, resizeDownLinearNonExact,
            testing::Combine
                (
                testing::Values( MATTYPE_NE_VALUES ),
                testing::Values
                    (
                    Size_Size_t(szVGA, szQVGA),
                    Size_Size_t(szqHD, szVGA),
                    Size_Size_t(sz720p, Size(120 * sz720p.width / sz720p.height, 120)),
                    Size_Size_t(sz720p, szVGA),
                    Size_Size_t(sz720p, szQVGA)
                    )
                )
            )
{
    int matType = get<0>(GetParam());
    Size_Size_t sizes = get<1>(GetParam());
    Size from = get<0>(sizes);
    Size to = get<1>(sizes);

    cv::Mat src(from, matType), dst(to, matType);
    switch(src.depth())
    {
        case CV_8U: cvtest::fillGradient(src); break;
        case CV_16U: fillFPGradient<ushort>(src); break;
        case CV_32F: fillFPGradient<float>(src); break;
    }
    declare.in(src).out(dst);

    TEST_CYCLE_MULTIRUN(10) resize(src, dst, to, 0, 0, INTER_LINEAR);

    SANITY_CHECK_NOTHING();
}


typedef tuple<MatType, Size, int> MatInfo_Size_Scale_t;
typedef TestBaseWithParam<MatInfo_Size_Scale_t> MatInfo_Size_Scale;

PERF_TEST_P(MatInfo_Size_Scale, ResizeAreaFast,
            testing::Combine(
                testing::Values(CV_8UC1, CV_8UC3, CV_8UC4, CV_16UC1, CV_16UC3, CV_16UC4),
                testing::Values(szVGA, szqHD, sz720p, sz1080p),
                testing::Values(2)
                )
            )
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    int scale = get<2>(GetParam());

    from.width = (from.width/scale)*scale;
    from.height = (from.height/scale)*scale;

    cv::Mat src(from, matType);
    cv::Mat dst(from.height / scale, from.width / scale, matType);

    declare.in(src, WARMUP_RNG).out(dst);

    int runs = 15;
    TEST_CYCLE_MULTIRUN(runs) resize(src, dst, dst.size(), 0, 0, INTER_AREA);

    //difference equal to 1 is allowed because of different possible rounding modes: round-to-nearest vs bankers' rounding
    SANITY_CHECK(dst, 1);
}


typedef TestBaseWithParam<tuple<MatType, Size, double> > MatInfo_Size_Scale_Area;

PERF_TEST_P(MatInfo_Size_Scale_Area, ResizeArea,
            testing::Combine(
                testing::Values(CV_8UC1, CV_8UC3, CV_8UC4),
                testing::Values(szVGA, szqHD, sz720p, sz1080p, sz2160p),
                testing::Values(0.1, 0.25, 0.81)
                )
            )
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    double scale = get<2>(GetParam());

    cv::Mat src(from, matType);

    Size to(cvRound(from.width * scale), cvRound(from.height * scale));
    cv::Mat dst(to, matType);

    declare.in(src, WARMUP_RNG).out(dst);
    declare.time(100);

    TEST_CYCLE() resize(src, dst, dst.size(), 0, 0, INTER_AREA);

    //difference equal to 1 is allowed because of different possible rounding modes: round-to-nearest vs bankers' rounding
    SANITY_CHECK(dst, 1);
}

typedef MatInfo_Size_Scale_Area MatInfo_Size_Scale_NN;

PERF_TEST_P(MatInfo_Size_Scale_NN, ResizeNN,
    testing::Combine(
        testing::Values(CV_8UC1, CV_8UC2, CV_8UC4),
        testing::Values(szVGA, szqHD, sz720p, sz1080p, sz2160p),
        testing::Values(2.4, 3.4, 1.3)
    )
)
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    double scale = get<2>(GetParam());

    cv::Mat src(from, matType);

    Size to(cvRound(from.width * scale), cvRound(from.height * scale));
    cv::Mat dst(to, matType);

    declare.in(src, WARMUP_RNG).out(dst);
    declare.time(100);

    TEST_CYCLE() resize(src, dst, dst.size(), 0, 0, INTER_NEAREST);

    EXPECT_GT(countNonZero(dst.reshape(1)), 0);
    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(MatInfo_Size_Scale_NN, ResizeNNExact,
    testing::Combine(
        testing::Values(CV_8UC1, CV_8UC3, CV_8UC4),
        testing::Values(sz720p, sz1080p),
        testing::Values(0.25, 0.5, 2.0)
    )
)
{
    int matType = get<0>(GetParam());
    Size from = get<1>(GetParam());
    double scale = get<2>(GetParam());

    cv::Mat src(from, matType);

    Size to(cvRound(from.width * scale), cvRound(from.height * scale));
    cv::Mat dst(to, matType);

    declare.in(src, WARMUP_RNG).out(dst);
    declare.time(100);

    TEST_CYCLE() resize(src, dst, dst.size(), 0, 0, INTER_NEAREST_EXACT);

    EXPECT_GT(countNonZero(dst.reshape(1)), 0);
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

- **__ANDROID__()**: A function/method defined in this file
- **MatInfo_Size_Scale_Area()**: A function/method defined in this file
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

