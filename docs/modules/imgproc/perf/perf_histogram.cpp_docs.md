# Documentation for `modules/imgproc/perf/perf_histogram.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_histogram.cpp`
- **File Name**: `perf_histogram.cpp`
- **File Size**: 4,525 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_histogram.cpp](../../../modules/imgproc/perf/perf_histogram.cpp)

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

typedef tuple<Size, MatType> Size_Source_t;
typedef TestBaseWithParam<Size_Source_t> Size_Source;
typedef TestBaseWithParam<Size> TestMatSize;

static const float rangeHight = 256.0f;
static const float rangeLow = 0.0f;

PERF_TEST_P(Size_Source, calcHist1d,
            testing::Combine(testing::Values(sz3MP, sz5MP),
                             testing::Values(CV_8U, CV_16U, CV_32F) )
            )
{
    Size size = get<0>(GetParam());
    MatType type = get<1>(GetParam());
    Mat source(size.height, size.width, type);
    Mat hist;
    int channels [] = {0};
    int histSize [] = {256};
    int dims = 1;
    int numberOfImages = 1;

    const float range[] = {rangeLow, rangeHight};
    const float* ranges[] = {range};

    randu(source, rangeLow, rangeHight);

    declare.in(source);

    TEST_CYCLE_MULTIRUN(3)
    {
        calcHist(&source, numberOfImages, channels, Mat(), hist, dims, histSize, ranges);
    }

    SANITY_CHECK(hist);
}

PERF_TEST_P(Size_Source, calcHist2d,
            testing::Combine(testing::Values(sz3MP, sz5MP),
                             testing::Values(CV_8UC2, CV_16UC2, CV_32FC2) )
            )
{
    Size size = get<0>(GetParam());
    MatType type = get<1>(GetParam());
    Mat source(size.height, size.width, type);
    Mat hist;
    int channels [] = {0, 1};
    int histSize [] = {256, 256};
    int dims = 2;
    int numberOfImages = 1;

    const float r[] = {rangeLow, rangeHight};
    const float* ranges[] = {r, r};

    randu(source, rangeLow, rangeHight);

    declare.in(source);
    TEST_CYCLE()
    {
        calcHist(&source, numberOfImages, channels, Mat(), hist, dims, histSize, ranges);
    }

    SANITY_CHECK(hist);
}

PERF_TEST_P(Size_Source, calcHist3d,
            testing::Combine(testing::Values(sz3MP, sz5MP),
                             testing::Values(CV_8UC3, CV_16UC3, CV_32FC3) )
            )
{
    Size size = get<0>(GetParam());
    MatType type = get<1>(GetParam());
    Mat hist;
    int channels [] = {0, 1, 2};
    int histSize [] = {32, 32, 32};
    int dims = 3;
    int numberOfImages = 1;
    Mat source(size.height, size.width, type);

    const float r[] = {rangeLow, rangeHight};
    const float* ranges[] = {r, r, r};

    randu(source, rangeLow, rangeHight);

    declare.in(source);
    TEST_CYCLE()
    {
        calcHist(&source, numberOfImages, channels, Mat(), hist, dims, histSize, ranges);
    }

    SANITY_CHECK(hist);
}

#define MatSize TestMatSize
PERF_TEST_P(MatSize, equalizeHist,
            testing::Values(TYPICAL_MAT_SIZES)
            )
{
    Size size = GetParam();
    Mat source(size.height, size.width, CV_8U);
    Mat destination;
    declare.in(source, WARMUP_RNG);

    TEST_CYCLE()
    {
        equalizeHist(source, destination);
    }

    SANITY_CHECK(destination);
}
#undef MatSize

typedef TestBaseWithParam< tuple<int, int> > Dim_Cmpmethod;
PERF_TEST_P(Dim_Cmpmethod, compareHist,
            testing::Combine(testing::Values(1, 3),
                             testing::Values(HISTCMP_CORREL, HISTCMP_CHISQR, HISTCMP_INTERSECT, HISTCMP_BHATTACHARYYA, HISTCMP_CHISQR_ALT, HISTCMP_KL_DIV))
            )
{
    int dims = get<0>(GetParam());
    int method = get<1>(GetParam());
    int histSize[] = { 2048, 128, 64 };

    Mat hist1(dims, histSize, CV_32FC1);
    Mat hist2(dims, histSize, CV_32FC1);
    randu(hist1, 0, 256);
    randu(hist2, 0, 256);

    declare.in(hist1.reshape(1, 256), hist2.reshape(1, 256));

    TEST_CYCLE()
    {
        compareHist(hist1, hist2, method);
    }

    SANITY_CHECK_NOTHING();
}

typedef tuple<Size, double, MatType> Sz_ClipLimit_t;
typedef TestBaseWithParam<Sz_ClipLimit_t> Sz_ClipLimit;

PERF_TEST_P(Sz_ClipLimit, CLAHE,
            testing::Combine(testing::Values(::perf::szVGA, ::perf::sz720p, ::perf::sz1080p),
                             testing::Values(0.0, 40.0),
                             testing::Values(MatType(CV_8UC1), MatType(CV_16UC1)))
            )
{
    const Size size = get<0>(GetParam());
    const double clipLimit = get<1>(GetParam());
    const int type = get<2>(GetParam());

    Mat src(size, type);
    declare.in(src, WARMUP_RNG);

    Ptr<CLAHE> clahe = createCLAHE(clipLimit);
    Mat dst;

    TEST_CYCLE() clahe->apply(src, dst);

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

- **MatSize()**: A function/method defined in this file
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

