# Documentation for `modules/imgproc/perf/perf_pyramids.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_pyramids.cpp`
- **File Name**: `perf_pyramids.cpp`
- **File Size**: 3,373 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_pyramids.cpp](../../../modules/imgproc/perf/perf_pyramids.cpp)

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

PERF_TEST_P(Size_MatType, pyrDown, testing::Combine(
                testing::Values(sz1080p, sz720p, szVGA, szQVGA, szODD),
                testing::Values(CV_8UC1, CV_8UC3, CV_8UC4, CV_16SC1, CV_16SC3, CV_16SC4, CV_32FC1, CV_32FC3, CV_32FC4)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    const double eps = CV_MAT_DEPTH(matType) <= CV_32S ? 1 : 1e-5;
    perf::ERROR_TYPE error_type = CV_MAT_DEPTH(matType) <= CV_32S ? ERROR_ABSOLUTE : ERROR_RELATIVE;

    Mat src(sz, matType);
    Mat dst((sz.height + 1)/2, (sz.width + 1)/2, matType);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() pyrDown(src, dst);

    SANITY_CHECK(dst, eps, error_type);
}

PERF_TEST_P(Size_MatType, DISABLED_pyrDown_ovx, testing::Combine(
    testing::Values(sz1080p, sz720p, szVGA, szQVGA, szODD),
    testing::Values(CV_8UC1, CV_8UC3, CV_8UC4, CV_16SC1, CV_16SC3, CV_16SC4, CV_32FC1, CV_32FC3, CV_32FC4)
)
)
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    const double eps = CV_MAT_DEPTH(matType) <= CV_32S ? 1 : 1e-5;
    perf::ERROR_TYPE error_type = CV_MAT_DEPTH(matType) <= CV_32S ? ERROR_ABSOLUTE : ERROR_RELATIVE;

    Mat src(sz, matType);
    Mat dst((sz.height + 1) / 2, (sz.width + 1) / 2, matType);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() pyrDown(src, dst, cv::Size(), BORDER_REPLICATE);

    SANITY_CHECK(dst, eps, error_type);
}

PERF_TEST_P(Size_MatType, pyrUp, testing::Combine(
                testing::Values(sz720p, szVGA, szQVGA, szODD),
                testing::Values(CV_8UC1, CV_8UC3, CV_8UC4, CV_16SC1, CV_16SC3, CV_16SC4, CV_32FC1, CV_32FC3, CV_32FC4)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    const double eps = CV_MAT_DEPTH(matType) <= CV_32S ? 1 : 1e-5;
    perf::ERROR_TYPE error_type = CV_MAT_DEPTH(matType) <= CV_32S ? ERROR_ABSOLUTE : ERROR_RELATIVE;

    Mat src(sz, matType);
    Mat dst(sz.height*2, sz.width*2, matType);

    declare.in(src, WARMUP_RNG).out(dst);

    TEST_CYCLE() pyrUp(src, dst);

    SANITY_CHECK(dst, eps, error_type);
}

PERF_TEST_P(Size_MatType, buildPyramid, testing::Combine(
                testing::Values(sz1080p, sz720p, szVGA, szQVGA, szODD),
                testing::Values(CV_8UC1, CV_8UC3, CV_8UC4, CV_32FC1, CV_32FC3, CV_32FC4)
                )
            )
{
    Size sz = get<0>(GetParam());
    int matType = get<1>(GetParam());
    int maxLevel = 5;
    const double eps = CV_MAT_DEPTH(matType) <= CV_32S ? 1 : 1e-5;
    perf::ERROR_TYPE error_type = CV_MAT_DEPTH(matType) <= CV_32S ? ERROR_ABSOLUTE : ERROR_RELATIVE;
    Mat src(sz, matType);
    std::vector<Mat> dst(maxLevel);

    declare.in(src, WARMUP_RNG);

    TEST_CYCLE() buildPyramid(src, dst, maxLevel);

    Mat dst0 = dst[0], dst1 = dst[1], dst2 = dst[2], dst3 = dst[3], dst4 = dst[4];

    SANITY_CHECK(dst0, eps, error_type);
    SANITY_CHECK(dst1, eps, error_type);
    SANITY_CHECK(dst2, eps, error_type);
    SANITY_CHECK(dst3, eps, error_type);
    SANITY_CHECK(dst4, eps, error_type);
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

