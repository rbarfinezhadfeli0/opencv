# Documentation for `modules/gapi/test/common/gapi_render_tests.hpp`

## File Metadata

- **Full Path**: `modules/gapi/test/common/gapi_render_tests.hpp`
- **File Name**: `gapi_render_tests.hpp`
- **File Size**: 5,755 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/test/common/gapi_render_tests.hpp](../../../../modules/gapi/test/common/gapi_render_tests.hpp)

## Purpose and Role

This file is located in the `modules/gapi/test/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_RENDER_TESTS_HPP
#define OPENCV_GAPI_RENDER_TESTS_HPP

#include "gapi_tests_common.hpp"

namespace opencv_test
{

template<typename ...SpecificParams>
struct RenderParams : public Params<SpecificParams...>
{
    using common_params_t = std::tuple<cv::Size>;
    using specific_params_t = std::tuple<SpecificParams...>;
    using params_t = std::tuple<cv::Size, SpecificParams...>;

    static constexpr const size_t common_params_size = std::tuple_size<common_params_t>::value;
    static constexpr const size_t specific_params_size = std::tuple_size<specific_params_t>::value;

    template<size_t I>
    static const typename std::tuple_element<I, common_params_t>::type&
    getCommon(const params_t& t)
    {
        static_assert(I < common_params_size, "Index out of range");
        return std::get<I>(t);
    }

    template<size_t I>
    static const typename std::tuple_element<I, specific_params_t>::type&
    getSpecific(const params_t& t)
    {
        static_assert(specific_params_size > 0,
            "Impossible to call this function: no specific parameters specified");
        static_assert(I < specific_params_size, "Index out of range");
        return std::get<common_params_size + I>(t);
    }
};

template<typename ...SpecificParams>
struct RenderTestBase : public TestWithParam<typename RenderParams<SpecificParams...>::params_t>
{
    using AllParams = RenderParams<SpecificParams...>;

    // Get common (pre-defined) parameter value by index
    template<size_t I>
    inline auto getCommonParam() const
        -> decltype(AllParams::template getCommon<I>(this->GetParam()))
    {
        return AllParams::template getCommon<I>(this->GetParam());
    }

    // Get specific (user-defined) parameter value by index
    template<size_t I>
    inline auto getSpecificParam() const
        -> decltype(AllParams::template getSpecific<I>(this->GetParam()))
    {
        return AllParams::template getSpecific<I>(this->GetParam());
    }

    cv::Size sz_ = getCommonParam<0>();
};

template <typename ...Args>
class RenderBGRTestBase : public RenderTestBase<Args...>
{
protected:
    void Init(const cv::Size& sz)
    {
        MatType type = CV_8UC3;

        ref_mat.create(sz, type);
        gapi_mat.create(sz, type);

        cv::randu(ref_mat, cv::Scalar::all(0), cv::Scalar::all(255));
        ref_mat.copyTo(gapi_mat);
    }

    cv::Mat gapi_mat, ref_mat;
};

template <typename ...Args>
class RenderNV12TestBase : public RenderTestBase<Args...>
{
protected:
    void Init(const cv::Size& sz)
    {
        auto create_rand_mats = [](const cv::Size& size, MatType type, cv::Mat& ref_mat, cv::Mat& gapi_mat) {
            ref_mat.create(size, type);
            cv::randu(ref_mat, cv::Scalar::all(0), cv::Scalar::all(255));
            ref_mat.copyTo(gapi_mat);
        };

        create_rand_mats(sz,     CV_8UC1, y_ref_mat  , y_gapi_mat);
        create_rand_mats(sz / 2, CV_8UC2, uv_ref_mat , uv_gapi_mat);
    }

    cv::Mat y_ref_mat, uv_ref_mat, y_gapi_mat, uv_gapi_mat;
};

cv::Scalar cvtBGRToYUVC(const cv::Scalar& bgr);
void drawMosaicRef(const cv::Mat& mat, const cv::Rect &rect, int cellSz);
void blendImageRef(cv::Mat& mat,
                   const cv::Point& org,
                   const cv::Mat& img,
                   const cv::Mat& alpha);

#define GAPI_RENDER_TEST_FIXTURE_NV12(Fixture, API, Number, ...)  \
struct Fixture : public RenderNV12TestBase API {                  \
    __WRAP_VAARGS(DEFINE_SPECIFIC_PARAMS_##Number(__VA_ARGS__))   \
    Fixture() {                                                   \
        Init(sz_);                                                \
    }                                                             \
};

#define GAPI_RENDER_TEST_FIXTURE_BGR(Fixture, API, Number, ...)  \
struct Fixture : public RenderBGRTestBase API {                  \
    __WRAP_VAARGS(DEFINE_SPECIFIC_PARAMS_##Number(__VA_ARGS__))   \
    Fixture() {                                                   \
        Init(sz_);                                                \
    }                                                             \
};

#define GET_VA_ARGS(...) __VA_ARGS__
#define GAPI_RENDER_TEST_FIXTURES(Fixture, API, Number, ...)                    \
    GAPI_RENDER_TEST_FIXTURE_BGR(RenderBGR##Fixture,   GET_VA_ARGS(API), Number, __VA_ARGS__) \
    GAPI_RENDER_TEST_FIXTURE_NV12(RenderNV12##Fixture, GET_VA_ARGS(API), Number, __VA_ARGS__) \
    GAPI_RENDER_TEST_FIXTURE_NV12(RenderMFrame##Fixture, GET_VA_ARGS(API), Number, __VA_ARGS__) \


using Points = std::vector<cv::Point>;
GAPI_RENDER_TEST_FIXTURES(TestTexts,     FIXTURE_API(std::string, cv::Point, double, cv::Scalar), 4, text, org, fs, color)
GAPI_RENDER_TEST_FIXTURES(TestRects,     FIXTURE_API(cv::Rect, cv::Scalar, int),                  3, rect, color, thick)
GAPI_RENDER_TEST_FIXTURES(TestCircles,   FIXTURE_API(cv::Point, int, cv::Scalar, int),            4, center, radius, color, thick)
GAPI_RENDER_TEST_FIXTURES(TestLines,     FIXTURE_API(cv::Point, cv::Point, cv::Scalar, int),      4, pt1, pt2, color, thick)
GAPI_RENDER_TEST_FIXTURES(TestMosaics,   FIXTURE_API(cv::Rect, int, int),                         3, mos, cellsz, decim)
GAPI_RENDER_TEST_FIXTURES(TestImages,    FIXTURE_API(cv::Rect, cv::Scalar, double),               3, rect, color, transparency)
GAPI_RENDER_TEST_FIXTURES(TestPolylines, FIXTURE_API(Points, cv::Scalar, int),                    3, points, color, thick)

} // opencv_test

#endif //OPENCV_GAPI_RENDER_TESTS_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **RenderTestBase**: A class/struct defined in this file
- **RenderBGRTestBase**: A class/struct defined in this file
- **RenderParams**: A class/struct defined in this file
- **Fixture**: A class/struct defined in this file
- **RenderNV12TestBase**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_RENDER_TESTS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `gapi_tests_common.hpp`


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

