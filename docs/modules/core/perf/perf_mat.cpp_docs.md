# Documentation for `modules/core/perf/perf_mat.cpp`

## File Metadata

- **Full Path**: `modules/core/perf/perf_mat.cpp`
- **File Name**: `perf_mat.cpp`
- **File Size**: 4,492 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/perf/perf_mat.cpp](../../../modules/core/perf/perf_mat.cpp)

## Purpose and Role

This file is located in the `modules/core/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "perf_precomp.hpp"

namespace opencv_test
{
using namespace perf;

PERF_TEST_P(Size_MatType, Mat_Eye,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(TYPICAL_MAT_TYPES))

             )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    Mat diagonalMatrix(size.height, size.width, type);

    declare.out(diagonalMatrix);

    int runs = (size.width <= 640) ? 15 : 5;
    TEST_CYCLE_MULTIRUN(runs)
    {
        diagonalMatrix = Mat::eye(size, type);
    }

    SANITY_CHECK(diagonalMatrix, 1);
}

PERF_TEST_P(Size_MatType, Mat_Zeros,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(TYPICAL_MAT_TYPES, CV_32FC3))

             )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    Mat zeroMatrix(size.height, size.width, type);

    declare.out(zeroMatrix);

    int runs = (size.width <= 640) ? 15 : 5;
    TEST_CYCLE_MULTIRUN(runs)
    {
        zeroMatrix = Mat::zeros(size, type);
    }

    SANITY_CHECK(zeroMatrix, 1);
}

PERF_TEST_P(Size_MatType, Mat_Clone,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(TYPICAL_MAT_TYPES))

             )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());
    Mat source(size.height, size.width, type);
    Mat destination(size.height, size.width, type);

    declare.in(source, WARMUP_RNG).out(destination);

    TEST_CYCLE()
    {
        Mat tmp = source.clone();
        CV_UNUSED(tmp);
    }
    destination = source.clone();

    SANITY_CHECK(destination, 1);
}

PERF_TEST_P(Size_MatType, Mat_Clone_Roi,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(TYPICAL_MAT_TYPES))

             )
{
    Size size = get<0>(GetParam());
    int type = get<1>(GetParam());

    unsigned int width = size.width;
    unsigned int height = size.height;
    Mat source(height, width, type);
    Mat destination(size.height/2, size.width/2, type);

    declare.in(source, WARMUP_RNG).out(destination);

    Mat roi(source, Rect(width/4, height/4, 3*width/4, 3*height/4));

    TEST_CYCLE()
    {
        Mat tmp = roi.clone();
        CV_UNUSED(tmp);
    }
    destination = roi.clone();

    SANITY_CHECK(destination, 1);
}

PERF_TEST_P(Size_MatType, Mat_CopyToWithMask,
            testing::Combine(testing::Values(::perf::sz1080p, ::perf::szODD),
                             testing::Values(CV_8UC1, CV_8UC2, CV_8UC3, CV_16UC1, CV_16UC3, CV_32SC1, CV_32SC2, CV_32FC4))
            )
{
    const Size_MatType_t params = GetParam();
    const Size size = get<0>(params);
    const int type = get<1>(params);

    Mat src(size, type), dst(size, type), mask(size, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG).out(dst);

    TEST_CYCLE()
    {
        src.copyTo(dst, mask);
    }

    SANITY_CHECK(dst);
}

PERF_TEST_P(Size_MatType, Mat_SetToWithMask,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(CV_8UC1, CV_8UC2))
            )
{
    const Size_MatType_t params = GetParam();
    const Size size = get<0>(params);
    const int type = get<1>(params);
    const Scalar sc = Scalar::all(27);

    Mat src(size, type), mask(size, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG).out(src);

    TEST_CYCLE()
    {
        src.setTo(sc, mask);
    }

    SANITY_CHECK(src);
}

///////////// Transform ////////////////////////

PERF_TEST_P(Size_MatType, Mat_Transform,
            testing::Combine(testing::Values(TYPICAL_MAT_SIZES),
                             testing::Values(CV_8UC3, CV_8SC3, CV_16UC3, CV_16SC3, CV_32SC3, CV_32FC3, CV_64FC3))
            )
{
    const Size_MatType_t params = GetParam();
    const Size srcSize0 = get<0>(params);
    const Size srcSize = Size(1, srcSize0.width*srcSize0.height);
    const int type = get<1>(params);
    const float transform[] = { 0.5f,           0.f, 0.86602540378f, 128,
                                0.f,            1.f, 0.f,            -64,
                                0.86602540378f, 0.f, 0.5f,            32,};
    Mat mtx(Size(4, 3), CV_32FC1, (void*)transform);

    Mat src(srcSize, type), dst(srcSize, type);
    randu(src, 0, 30);
    declare.in(src).out(dst);

    TEST_CYCLE()
    {
        cv::transform(src, dst, mtx);
    }

    SANITY_CHECK(dst, 1e-6, ERROR_RELATIVE);
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

