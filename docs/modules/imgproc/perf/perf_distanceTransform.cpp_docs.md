# Documentation for `modules/imgproc/perf/perf_distanceTransform.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_distanceTransform.cpp`
- **File Name**: `perf_distanceTransform.cpp`
- **File Size**: 2,527 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_distanceTransform.cpp](../../../modules/imgproc/perf/perf_distanceTransform.cpp)

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

CV_ENUM(DistanceType, DIST_L1, DIST_L2 , DIST_C)
CV_ENUM(MaskSize, DIST_MASK_3, DIST_MASK_5, DIST_MASK_PRECISE)
CV_ENUM(DstType, CV_8U, CV_32F)
CV_ENUM(LabelType, DIST_LABEL_CCOMP, DIST_LABEL_PIXEL)

typedef tuple<Size, DistanceType, MaskSize, DstType> SrcSize_DistType_MaskSize_DstType;
typedef tuple<Size, DistanceType, MaskSize, LabelType> SrcSize_DistType_MaskSize_LabelType;
typedef perf::TestBaseWithParam<SrcSize_DistType_MaskSize_DstType> DistanceTransform_Test;
typedef perf::TestBaseWithParam<SrcSize_DistType_MaskSize_LabelType> DistanceTransform_NeedLabels_Test;

PERF_TEST_P(DistanceTransform_Test, distanceTransform,
            testing::Combine(
                testing::Values(cv::Size(640, 480), cv::Size(800, 600), cv::Size(1024, 768), cv::Size(1280, 1024)),
                DistanceType::all(),
                MaskSize::all(),
                DstType::all()
                )
            )
{
    Size srcSize = get<0>(GetParam());
    int distanceType = get<1>(GetParam());
    int maskSize = get<2>(GetParam());
    int dstType = get<3>(GetParam());

    Mat src(srcSize, CV_8U);
    Mat dst(srcSize, dstType);

    declare
        .in(src, WARMUP_RNG)
        .out(dst, WARMUP_RNG)
        .time(30);

    TEST_CYCLE() distanceTransform( src, dst, distanceType, maskSize, dstType);

    double eps = 2e-4;

    SANITY_CHECK(dst, eps);
}

PERF_TEST_P(DistanceTransform_NeedLabels_Test, distanceTransform_NeedLabels,
            testing::Combine(
                testing::Values(cv::Size(640, 480), cv::Size(800, 600), cv::Size(1024, 768), cv::Size(1280, 1024)),
                DistanceType::all(),
                MaskSize::all(),
                LabelType::all()
                )
    )
{
    Size srcSize = get<0>(GetParam());
    int distanceType = get<1>(GetParam());
    int maskSize = get<2>(GetParam());
    int labelType = get<3>(GetParam());

    Mat src(srcSize, CV_8U);
    Mat label(srcSize, CV_32S);
    Mat dst(srcSize, CV_32F);

    declare
        .in(src, WARMUP_RNG)
        .out(label, WARMUP_RNG)
        .out(dst, WARMUP_RNG)
        .time(30);

    TEST_CYCLE() distanceTransform( src, dst, label, distanceType, maskSize, labelType);

    double eps = 2e-4;

    SANITY_CHECK(label, eps);
    SANITY_CHECK(dst, eps);
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

