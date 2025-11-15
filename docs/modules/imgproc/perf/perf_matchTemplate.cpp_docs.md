# Documentation for `modules/imgproc/perf/perf_matchTemplate.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/perf_matchTemplate.cpp`
- **File Name**: `perf_matchTemplate.cpp`
- **File Size**: 2,590 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/perf_matchTemplate.cpp](../../../modules/imgproc/perf/perf_matchTemplate.cpp)

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

CV_ENUM(MethodType, TM_SQDIFF, TM_SQDIFF_NORMED, TM_CCORR, TM_CCORR_NORMED, TM_CCOEFF, TM_CCOEFF_NORMED)

typedef tuple<Size, Size, MethodType> ImgSize_TmplSize_Method_t;
typedef perf::TestBaseWithParam<ImgSize_TmplSize_Method_t> ImgSize_TmplSize_Method;

PERF_TEST_P(ImgSize_TmplSize_Method, matchTemplateSmall,
            testing::Combine(
                testing::Values(szSmall128, cv::Size(320, 240),
                                cv::Size(640, 480), cv::Size(800, 600),
                                cv::Size(1024, 768), cv::Size(1280, 1024)),
                testing::Values(cv::Size(12, 12), cv::Size(28, 9),
                                cv::Size(8, 30), cv::Size(16, 16)),
                MethodType::all()
                )
            )
{
    Size imgSz = get<0>(GetParam());
    Size tmplSz = get<1>(GetParam());
    int method = get<2>(GetParam());

    Mat img(imgSz, CV_8UC1);
    Mat tmpl(tmplSz, CV_8UC1);
    Mat result(imgSz - tmplSz + Size(1,1), CV_32F);

    declare
        .in(img, WARMUP_RNG)
        .in(tmpl, WARMUP_RNG)
        .out(result)
        .time(30);

    TEST_CYCLE() matchTemplate(img, tmpl, result, method);

    bool isNormed =
        method == TM_CCORR_NORMED ||
        method == TM_SQDIFF_NORMED ||
        method == TM_CCOEFF_NORMED;
    double eps = isNormed ? 1e-5
        : 255 * 255 * tmpl.total() * 1e-6;

    SANITY_CHECK(result, eps);
}

PERF_TEST_P(ImgSize_TmplSize_Method, matchTemplateBig,
            testing::Combine(
                testing::Values(cv::Size(1280, 1024)),
                testing::Values(cv::Size(1260, 1000), cv::Size(1261, 1013)),
                MethodType::all()
                )
    )
{
    Size imgSz = get<0>(GetParam());
    Size tmplSz = get<1>(GetParam());
    int method = get<2>(GetParam());

    Mat img(imgSz, CV_8UC1);
    Mat tmpl(tmplSz, CV_8UC1);
    Mat result(imgSz - tmplSz + Size(1,1), CV_32F);

    declare
        .in(img, WARMUP_RNG)
        .in(tmpl, WARMUP_RNG)
        .out(result)
        .time(30);

    TEST_CYCLE() matchTemplate(img, tmpl, result, method);

    bool isNormed =
        method == TM_CCORR_NORMED ||
        method == TM_SQDIFF_NORMED ||
        method == TM_CCOEFF_NORMED;
    double eps = isNormed ? 1e-6
        : 255.0 * 255.0 * (double)tmpl.total() * 1e-6;

    SANITY_CHECK(result, eps);
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

