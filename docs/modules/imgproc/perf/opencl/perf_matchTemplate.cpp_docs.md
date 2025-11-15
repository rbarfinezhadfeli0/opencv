# Documentation for `modules/imgproc/perf/opencl/perf_matchTemplate.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/perf/opencl/perf_matchTemplate.cpp`
- **File Name**: `perf_matchTemplate.cpp`
- **File Size**: 2,887 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/perf/opencl/perf_matchTemplate.cpp](../../../../modules/imgproc/perf/opencl/perf_matchTemplate.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/perf/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "../perf_precomp.hpp"
#include "opencv2/ts/ocl_perf.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

CV_ENUM(MethodType, TM_SQDIFF, TM_SQDIFF_NORMED, TM_CCORR, TM_CCORR_NORMED, TM_CCOEFF, TM_CCOEFF_NORMED)

typedef tuple<Size, Size, MethodType, MatType> ImgSize_TmplSize_Method_MatType_t;
typedef TestBaseWithParam<ImgSize_TmplSize_Method_MatType_t> ImgSize_TmplSize_Method_MatType;

OCL_PERF_TEST_P(ImgSize_TmplSize_Method_MatType, MatchTemplate,
        ::testing::Combine(
            testing::Values(cv::Size(640, 480), cv::Size(1280, 1024)),
            testing::Values(cv::Size(11, 11), cv::Size(16, 16), cv::Size(41, 41)),
            MethodType::all(),
            testing::Values(CV_8UC1, CV_8UC3, CV_32FC1, CV_32FC3)
            )
        )
{
    const ImgSize_TmplSize_Method_MatType_t params = GetParam();
    const Size imgSz = get<0>(params), tmplSz = get<1>(params);
    const int method = get<2>(params);
    int type = get<3>(GetParam());

    UMat img(imgSz, type), tmpl(tmplSz, type);
    UMat result(imgSz - tmplSz + Size(1, 1), CV_32F);

    declare.in(img, tmpl, WARMUP_RNG).out(result);

    OCL_TEST_CYCLE() matchTemplate(img, tmpl, result, method);

    bool isNormed =
        method == TM_CCORR_NORMED ||
        method == TM_SQDIFF_NORMED ||
        method == TM_CCOEFF_NORMED;
    double eps = isNormed ? 3e-2
        : 255 * 255 * tmpl.total() * 1e-4;

    SANITY_CHECK(result, eps, ERROR_RELATIVE);
}

/////////// matchTemplate (performance tests from 2.4) ////////////////////////

typedef Size_MatType CV_TM_CCORRFixture;

OCL_PERF_TEST_P(CV_TM_CCORRFixture, matchTemplate,
                ::testing::Combine(::testing::Values(Size(1000, 1000), Size(2000, 2000)),
                               OCL_PERF_ENUM(CV_32FC1, CV_32FC4)))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params), templSize(5, 5);
    const int type = get<1>(params);

    UMat src(srcSize, type), templ(templSize, type);
    const Size dstSize(src.cols - templ.cols + 1, src.rows - templ.rows + 1);
    UMat dst(dstSize, CV_32F);

    declare.in(src, templ, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::matchTemplate(src, templ, dst, cv::TM_CCORR);

    SANITY_CHECK(dst, 1e-4);
}

typedef TestBaseWithParam<Size> CV_TM_CCORR_NORMEDFixture;

OCL_PERF_TEST_P(CV_TM_CCORR_NORMEDFixture, matchTemplate,
                ::testing::Values(Size(1000, 1000), Size(2000, 2000), Size(4000, 4000)))
{
    const Size srcSize = GetParam(), templSize(5, 5);

    UMat src(srcSize, CV_8UC1), templ(templSize, CV_8UC1);
    const Size dstSize(src.cols - templ.cols + 1, src.rows - templ.rows + 1);
    UMat dst(dstSize, CV_8UC1);

    declare.in(src, templ, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() cv::matchTemplate(src, templ, dst, cv::TM_CCORR_NORMED);

    SANITY_CHECK(dst, 3e-2);
}

} } // namespace

#endif // HAVE_OPENCL
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

- **Size_MatType()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file
- **TestBaseWithParam()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_perf.hpp`
- `../perf_precomp.hpp`

**Python Imports:**
- `2.4`


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

