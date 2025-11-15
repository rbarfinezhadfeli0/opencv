# Documentation for `modules/gapi/test/gapi_fluid_roi_test.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_fluid_roi_test.cpp`
- **File Name**: `gapi_fluid_roi_test.cpp`
- **File Size**: 7,093 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_fluid_roi_test.cpp](../../../modules/gapi/test/gapi_fluid_roi_test.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "test_precomp.hpp"

#include "gapi_fluid_test_kernels.hpp"

namespace opencv_test
{

using namespace cv::gapi_test_kernels;

struct PartialComputation : public TestWithParam <std::tuple<cv::Rect>> {};
TEST_P(PartialComputation, Test)
{
    cv::Rect roi;
    std::tie(roi) = GetParam();

    int borderType = BORDER_REPLICATE;
    int kernelSize = 3;
    cv::Point anchor = {-1, -1};

    cv::GMat in;
    cv::GMat out = TBlur3x3::on(in, borderType, {});
    cv::GComputation c(cv::GIn(in), cv::GOut(out));

    const auto sz = cv::Size(8, 10);
    cv::Mat in_mat(sz, CV_8UC1);
    cv::Scalar mean   = cv::Scalar(127.0f);
    cv::Scalar stddev = cv::Scalar(40.f);
    cv::randn(in_mat, mean, stddev);

    cv::Mat out_mat_gapi = cv::Mat::zeros(sz, CV_8UC1);
    cv::Mat out_mat_ocv = cv::Mat::zeros(sz, CV_8UC1);

    // Run G-API
    auto cc = c.compile(cv::descr_of(in_mat), cv::compile_args(fluidTestPackage, GFluidOutputRois{{roi}}));
    cc(cv::gin(in_mat), cv::gout(out_mat_gapi));

    // Check with OpenCV
    if (roi == cv::Rect{}) roi = cv::Rect{0,0,sz.width,sz.height};
    cv::blur(in_mat(roi), out_mat_ocv(roi), {kernelSize, kernelSize}, anchor, borderType);

    EXPECT_EQ(0, cvtest::norm(out_mat_gapi, out_mat_ocv, NORM_INF));
}

INSTANTIATE_TEST_CASE_P(Fluid, PartialComputation,
                        Values(cv::Rect{},        cv::Rect{0,0,8,6}, cv::Rect{0,1,8,3},
                               cv::Rect{0,2,8,3}, cv::Rect{0,3,8,5}, cv::Rect{0,4,8,6}));

struct PartialComputationAddC : public TestWithParam <std::tuple<cv::Rect>> {};
TEST_P(PartialComputationAddC, Test)
{
    cv::Rect roi;
    std::tie(roi) = GetParam();

    cv::GMat in;
    cv::GMat out = TAddCSimple::on(in, 1);
    cv::GComputation c(cv::GIn(in), cv::GOut(out));

    const auto sz = cv::Size(8, 10);
    cv::Mat in_mat(sz, CV_8UC1);
    cv::Scalar mean   = cv::Scalar(127.0f);
    cv::Scalar stddev = cv::Scalar(40.f);
    cv::randn(in_mat, mean, stddev);

    cv::Mat out_mat_gapi = cv::Mat::zeros(sz, CV_8UC1);
    cv::Mat out_mat_ocv = cv::Mat::zeros(sz, CV_8UC1);

    // Run G-API
    auto cc = c.compile(cv::descr_of(in_mat), cv::compile_args(fluidTestPackage, GFluidOutputRois{{roi}}));
    cc(cv::gin(in_mat), cv::gout(out_mat_gapi));

    // Check with OpenCV
    if (roi == cv::Rect{}) roi = cv::Rect{0,0,sz.width,sz.height};
    out_mat_ocv(roi) = in_mat(roi) + 1;

    EXPECT_EQ(0, cvtest::norm(out_mat_gapi, out_mat_ocv, NORM_INF));
}

INSTANTIATE_TEST_CASE_P(FluidRoi, PartialComputationAddC,
                        Values(cv::Rect{},        cv::Rect{0,0,8,6}, cv::Rect{0,1,8,3},
                               cv::Rect{0,2,8,3}, cv::Rect{0,3,8,5}, cv::Rect{0,4,8,6}));

struct SequenceOfBlursRoiTest : public TestWithParam <std::tuple<int, cv::Rect>> {};
TEST_P(SequenceOfBlursRoiTest, Test)
{
    cv::Size sz_in = { 320, 240 };

    int borderType = 0;
    cv::Rect roi;
    std::tie(borderType, roi) = GetParam();
    cv::Mat in_mat(sz_in, CV_8UC1);
    cv::Scalar mean   = cv::Scalar(127.0f);
    cv::Scalar stddev = cv::Scalar(40.f);

    cv::randn(in_mat, mean, stddev);

    cv::Point anchor = {-1, -1};
    cv::Scalar borderValue(0);

    GMat in;
    auto mid = TBlur3x3::on(in,  borderType, borderValue);
    auto out = TBlur5x5::on(mid, borderType, borderValue);

    Mat out_mat_gapi = Mat::zeros(sz_in, CV_8UC1);

    GComputation c(GIn(in), GOut(out));
    auto cc = c.compile(descr_of(in_mat), cv::compile_args(fluidTestPackage, GFluidOutputRois{{roi}}));
    cc(gin(in_mat), gout(out_mat_gapi));

    cv::Mat mid_mat_ocv = Mat::zeros(sz_in, CV_8UC1);
    cv::Mat out_mat_ocv = Mat::zeros(sz_in, CV_8UC1);

    cv::blur(in_mat, mid_mat_ocv, {3,3}, anchor, borderType);

    if (roi == cv::Rect{})
    {
        roi = cv::Rect{0, 0, sz_in.width, sz_in.height};
    }

    cv::blur(mid_mat_ocv(roi), out_mat_ocv(roi), {5,5}, anchor, borderType);

    EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
}

INSTANTIATE_TEST_CASE_P(FluidRoi, SequenceOfBlursRoiTest,
                        Combine(Values(BORDER_CONSTANT, BORDER_REPLICATE, BORDER_REFLECT_101),
                                Values(cv::Rect{0,0,320,240}, cv::Rect{0,64,320,128}, cv::Rect{0,128,320,112})));

struct TwoBlursRoiTest : public TestWithParam <std::tuple<int, int, int, int, int, int, bool, cv::Rect>> {};
TEST_P(TwoBlursRoiTest, Test)
{
    cv::Size sz_in = { 320, 240 };

    int kernelSize1 = 0, kernelSize2 = 0;
    int borderType1 = -1, borderType2 = -1;
    cv::Scalar borderValue1{}, borderValue2{};
    bool readFromInput = false;
    cv::Rect outRoi;
    std::tie(kernelSize1, borderType1, borderValue1, kernelSize2, borderType2, borderValue2, readFromInput, outRoi) = GetParam();
    cv::Mat in_mat(sz_in, CV_8UC1);
    cv::Scalar mean   = cv::Scalar(127.0f);
    cv::Scalar stddev = cv::Scalar(40.f);

    cv::randn(in_mat, mean, stddev);

    cv::Point anchor = {-1, -1};

    auto blur1 = kernelSize1 == 3 ? &TBlur3x3::on : TBlur5x5::on;
    auto blur2 = kernelSize2 == 3 ? &TBlur3x3::on : TBlur5x5::on;

    GMat in, out1, out2;
    if (readFromInput)
    {
        out1 = blur1(in, borderType1, borderValue1);
        out2 = blur2(in, borderType2, borderValue2);
    }
    else
    {
        auto mid = TAddCSimple::on(in, 0);
        out1 = blur1(mid, borderType1, borderValue1);
        out2 = blur2(mid, borderType2, borderValue2);
    }

    Mat out_mat_gapi1 = Mat::zeros(sz_in, CV_8UC1);
    Mat out_mat_gapi2 = Mat::zeros(sz_in, CV_8UC1);

    GComputation c(GIn(in), GOut(out1, out2));
    auto cc = c.compile(descr_of(in_mat), cv::compile_args(fluidTestPackage, GFluidOutputRois{{outRoi, outRoi}}));
    cc(gin(in_mat), gout(out_mat_gapi1, out_mat_gapi2));

    cv::Mat out_mat_ocv1 = Mat::zeros(sz_in, CV_8UC1);
    cv::Mat out_mat_ocv2 = Mat::zeros(sz_in, CV_8UC1);

    cv::blur(in_mat(outRoi), out_mat_ocv1(outRoi), {kernelSize1, kernelSize1}, anchor, borderType1);
    cv::blur(in_mat(outRoi), out_mat_ocv2(outRoi), {kernelSize2, kernelSize2}, anchor, borderType2);

    EXPECT_EQ(0, cvtest::norm(out_mat_ocv1, out_mat_gapi1, NORM_INF));
    EXPECT_EQ(0, cvtest::norm(out_mat_ocv2, out_mat_gapi2, NORM_INF));
}

INSTANTIATE_TEST_CASE_P(FluidRoi, TwoBlursRoiTest,
                        Combine(Values(3, 5),
                                Values(cv::BORDER_CONSTANT, cv::BORDER_REPLICATE, cv::BORDER_REFLECT_101),
                                Values(0),
                                Values(3, 5),
                                Values(cv::BORDER_CONSTANT, cv::BORDER_REPLICATE, cv::BORDER_REFLECT_101),
                                Values(0),
                                testing::Bool(), // Read from input directly or place a copy node at start
                                Values(cv::Rect{0,0,320,240}, cv::Rect{0,64,320,128}, cv::Rect{0,128,320,112})));

} // namespace opencv_test
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

### Classes and Structures

- **TwoBlursRoiTest**: A class/struct defined in this file
- **SequenceOfBlursRoiTest**: A class/struct defined in this file
- **PartialComputationAddC**: A class/struct defined in this file
- **PartialComputation**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `gapi_fluid_test_kernels.hpp`
- `test_precomp.hpp`

**Python Imports:**
- `input`


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

