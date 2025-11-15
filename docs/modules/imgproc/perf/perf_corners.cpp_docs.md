# Documentation for `modules/imgproc/perf/perf_corners.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_corners.cpp`
- **File Name**: `perf_corners.cpp`
- **File Size**: 3,245 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_corners.cpp](../../../modules/imgproc/perf/perf_corners.cpp)

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

CV_ENUM(BorderType, BORDER_REPLICATE, BORDER_CONSTANT, BORDER_REFLECT, BORDER_REFLECT_101)

typedef tuple<string, int, int, double, BorderType> Img_BlockSize_ApertureSize_k_BorderType_t;
typedef perf::TestBaseWithParam<Img_BlockSize_ApertureSize_k_BorderType_t> Img_BlockSize_ApertureSize_k_BorderType;

PERF_TEST_P(Img_BlockSize_ApertureSize_k_BorderType, cornerHarris,
            testing::Combine(
                testing::Values( "stitching/a1.png", "cv/shared/pic5.png"),
                testing::Values( 3, 5 ),
                testing::Values( 3, 5 ),
                testing::Values( 0.04, 0.1 ),
                BorderType::all()
                )
          )
{
    string filename = getDataPath(get<0>(GetParam()));
    int blockSize = get<1>(GetParam());
    int apertureSize = get<2>(GetParam());
    double k = get<3>(GetParam());
    BorderType borderType = get<4>(GetParam());

    Mat src = imread(filename, IMREAD_GRAYSCALE);
    ASSERT_FALSE(src.empty()) << "Unable to load source image: " << filename;

    Mat dst;

    TEST_CYCLE() cornerHarris(src, dst, blockSize, apertureSize, k, borderType);

    SANITY_CHECK(dst, 2e-5, ERROR_RELATIVE);
}

typedef tuple<string, int, int, BorderType> Img_BlockSize_ApertureSize_BorderType_t;
typedef perf::TestBaseWithParam<Img_BlockSize_ApertureSize_BorderType_t> Img_BlockSize_ApertureSize_BorderType;

PERF_TEST_P(Img_BlockSize_ApertureSize_BorderType, cornerEigenValsAndVecs,
            testing::Combine(
                testing::Values( "stitching/a1.png", "cv/shared/pic5.png"),
                testing::Values( 3, 5 ),
                testing::Values( 3, 5 ),
                BorderType::all()
            )
          )
{
    string filename = getDataPath(get<0>(GetParam()));
    int blockSize = get<1>(GetParam());
    int apertureSize = get<2>(GetParam());
    BorderType borderType = get<3>(GetParam());

    Mat src = imread(filename, IMREAD_GRAYSCALE);
    ASSERT_FALSE(src.empty()) << "Unable to load source image: " << filename;

    Mat dst;

    TEST_CYCLE() cornerEigenValsAndVecs(src, dst, blockSize, apertureSize, borderType);

    Mat l1;
    extractChannel(dst, l1, 0);

    SANITY_CHECK(l1, 2e-5, ERROR_RELATIVE);
}

PERF_TEST_P(Img_BlockSize_ApertureSize_BorderType, cornerMinEigenVal,
            testing::Combine(
                testing::Values( "stitching/a1.png", "cv/shared/pic5.png"),
                testing::Values( 3, 5 ),
                testing::Values( 3, 5 ),
                BorderType::all()
            )
          )
{
    string filename = getDataPath(get<0>(GetParam()));
    int blockSize = get<1>(GetParam());
    int apertureSize = get<2>(GetParam());
    BorderType borderType = get<3>(GetParam());

    Mat src = imread(filename, IMREAD_GRAYSCALE);
    ASSERT_FALSE(src.empty()) << "Unable to load source image: " << filename;

    Mat dst;

    TEST_CYCLE() cornerMinEigenVal(src, dst, blockSize, apertureSize, borderType);

    SANITY_CHECK(dst, 2e-5, ERROR_RELATIVE);
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

