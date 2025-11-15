# Documentation for `modules/gapi/test/rmat/rmat_view_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/rmat/rmat_view_tests.cpp`
- **File Name**: `rmat_view_tests.cpp`
- **File Size**: 8,874 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/rmat/rmat_view_tests.cpp](../../../../modules/gapi/test/rmat/rmat_view_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/rmat` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation

#include "../test_precomp.hpp"
#include <opencv2/gapi/rmat.hpp>

#include <opencv2/gapi/util/compiler_hints.hpp>
#include "../src/backends/common/gbackend.hpp"

namespace opencv_test
{
using cv::GMatDesc;
using View = cv::RMat::View;
using cv::Mat;
using cv::gimpl::asMat;
using cv::gimpl::asView;
using namespace ::testing;

static void expect_eq_desc(const GMatDesc& desc, const View& view) {
    EXPECT_EQ(desc.size, view.size());
    EXPECT_EQ(desc.dims, view.dims());
    EXPECT_EQ(desc.size.width, view.cols());
    EXPECT_EQ(desc.size.height, view.rows());
    EXPECT_EQ(desc.depth, view.depth());
    EXPECT_EQ(desc.chan, view.chan());
    EXPECT_EQ(desc.depth, view.depth());
    EXPECT_EQ(desc.chan, view.chan());
}

TEST(RMatView, TestDefaultConstruction) {
    View view;
    GMatDesc desc{};
    expect_eq_desc(desc, view);
    EXPECT_EQ(nullptr, view.ptr());
    EXPECT_EQ(0u, view.step());
}

struct RMatViewTest : public TestWithParam<int /*dataType*/>{};
TEST_P(RMatViewTest, ConstructionFromMat) {
    auto type = GetParam();
    Mat mat(8,8,type);
    const auto desc = cv::descr_of(mat);
    View view = asView(mat);
    expect_eq_desc(desc, view);
    EXPECT_EQ(mat.ptr(), view.ptr());
    EXPECT_EQ(mat.step, view.step());
}

TEST(RMatView, TestConstructionFromMatND) {
    std::vector<int> dims(4, 8);
    Mat mat(dims, CV_8UC1);
    const auto desc = cv::descr_of(mat);
    View view(cv::descr_of(mat), mat.ptr());
    expect_eq_desc(desc, view);
    EXPECT_EQ(mat.ptr(), view.ptr());
}

TEST_P(RMatViewTest, DefaultStep) {
    auto type = GetParam();
    GMatDesc desc;
    desc.chan = CV_MAT_CN(type);
    desc.depth = CV_MAT_DEPTH(type);
    desc.size = {8,8};
    std::vector<unsigned char> data(desc.size.width*desc.size.height*CV_ELEM_SIZE(type));
    View view(desc, data.data());
    EXPECT_EQ(static_cast<size_t>(desc.size.width)*CV_ELEM_SIZE(type), view.step());
}

struct RMatViewNDTest : public TestWithParam<
    std::tuple<int /*depth*/, int /*ndims*/>>{};
TEST_P(RMatViewNDTest, DefaultStep) {
    int depth = 0, ndims = 0;
    std::tie(depth, ndims) = GetParam();
    std::vector<int> dims(ndims, 12);
    GMatDesc desc;
    desc.dims = dims;
    desc.depth = depth;
    GAPI_Assert(desc.chan == -1);
    auto elemSize = CV_ELEM_SIZE(depth);
    auto total = std::accumulate(dims.begin(), dims.end(), elemSize, std::multiplies<int>());
    std::vector<unsigned char> data(total);
    View view(desc, data.data());
    auto step = static_cast<size_t>(total/dims[0]);
    EXPECT_EQ(step, view.step(0));
    for (int i = 1; i < ndims; i++) {
        step /= dims[i];
        EXPECT_EQ(step, view.step(i));
    }
}

TEST_P(RMatViewNDTest, StepFromMat) {
    int depth = 0, ndims = 0;
    std::tie(depth, ndims) = GetParam();
    std::vector<int> dims(ndims, 12);
    cv::Mat mat(dims, depth);
    auto view = asView(mat);
    EXPECT_EQ(mat.ptr(), view.ptr());
    for (int i = 0; i < ndims; i++) {
        EXPECT_EQ(mat.step[i], view.step(i));
    }
}

TEST_P(RMatViewNDTest, StepFromView) {
    int depth = 0, ndims = 0;
    std::tie(depth, ndims) = GetParam();
    std::vector<int> dims(ndims, 12);
    std::vector<int> aligned(ndims, 16);
    GMatDesc desc;
    desc.dims = dims;
    desc.depth = depth;
    GAPI_Assert(desc.chan == -1);
    auto elemSize = CV_ELEM_SIZE(depth);
    auto total = std::accumulate(aligned.begin(), aligned.end(), elemSize, std::multiplies<int>());
    std::vector<unsigned char> data(total);
    View::stepsT steps(ndims);
    auto step = static_cast<size_t>(total/aligned[0]);
    steps[0] = step;
    for (int i = 1; i < ndims; i++) {
        step /= aligned[i];
        steps[i] = step;
    }
    View view(desc, data.data(), steps);
    auto mat = asMat(view);
    EXPECT_EQ(mat.ptr(), view.ptr());
    for (int i = 0; i < ndims; i++) {
        EXPECT_EQ(mat.step[i], view.step(i));
    }
}

INSTANTIATE_TEST_CASE_P(Test, RMatViewNDTest,
                        Combine(Values(CV_8U, CV_32F), // depth
                                Values(1,2,3,4,5)));   // ndims

struct RMatViewNDTestNegative : public TestWithParam<
    std::tuple<int /*depth*/, int /*chan*/, int /*ndims*/>>{};
TEST_P(RMatViewNDTestNegative, DefaultStep) {
    int depth = 0, chan = 0, ndims = 0;
    std::tie(depth, chan, ndims) = GetParam();
    std::vector<int> dims(ndims, 12);
    GMatDesc desc;
    desc.dims = dims;
    desc.depth = depth;
    desc.chan = chan;
    auto elemSize = CV_ELEM_SIZE(depth);
    auto total = std::accumulate(dims.begin(), dims.end(), elemSize, std::multiplies<int>());
    std::vector<unsigned char> data(total);
    EXPECT_ANY_THROW(View view(desc, data.data()));
}

INSTANTIATE_TEST_CASE_P(Test, RMatViewNDTestNegative,
                        Combine(Values(CV_8U, CV_32F), // depth
                                Values(1,2,3,4),       // chan
                                Values(2,4,5)));       // ndims

TEST_P(RMatViewTest, NonDefaultStepInput) {
    auto type = GetParam();
    Mat bigMat(16,16,type);
    cv::randn(bigMat, cv::Scalar::all(127), cv::Scalar::all(40));
    Mat mat = bigMat(cv::Rect{4,4,8,8});
    View view = asView(mat);
    const auto viewMat = asMat(view);
    Mat ref, out;
    cv::Size ksize{1,1};
    cv::blur(viewMat, out, ksize);
    cv::blur(    mat, ref, ksize);
    EXPECT_EQ(0, cvtest::norm(ref, out, NORM_INF));
}

TEST_P(RMatViewTest, NonDefaultStepOutput) {
    auto type = GetParam();
    Mat mat(8,8,type);
    cv::randn(mat, cv::Scalar::all(127), cv::Scalar::all(40));
    Mat bigMat = Mat::zeros(16,16,type);
    Mat out = bigMat(cv::Rect{4,4,8,8});
    View view = asView(out);
    auto viewMat = asMat(view);
    Mat ref;
    cv::Size ksize{1,1};
    cv::blur(mat, viewMat, ksize);
    cv::blur(mat, ref,     ksize);
    EXPECT_EQ(0, cvtest::norm(ref, out, NORM_INF));
}

TEST_P(RMatViewTest, NonDefaultStep2DInput) {
    auto type = GetParam();
    Mat bigMat(16,16,type);
    cv::randn(bigMat, cv::Scalar::all(127), cv::Scalar::all(40));
    Mat mat = bigMat(cv::Rect{4,4,8,8});
    View view(cv::descr_of(mat), mat.data, mat.step);
    const auto viewMat = asMat(view);
    Mat ref, out;
    cv::Size ksize{1,1};
    cv::blur(viewMat, out, ksize);
    cv::blur(    mat, ref, ksize);
    EXPECT_EQ(0, cvtest::norm(ref, out, NORM_INF));
}

TEST_P(RMatViewTest, NonDefaultStep2DOutput) {
    auto type = GetParam();
    Mat mat(8,8,type);
    cv::randn(mat, cv::Scalar::all(127), cv::Scalar::all(40));
    Mat bigMat = Mat::zeros(16,16,type);
    Mat out = bigMat(cv::Rect{4,4,8,8});
    View view(cv::descr_of(out), out.data, out.step);
    auto viewMat = asMat(view);
    Mat ref;
    cv::Size ksize{1,1};
    cv::blur(mat, viewMat, ksize);
    cv::blur(mat, ref,     ksize);
    EXPECT_EQ(0, cvtest::norm(ref, out, NORM_INF));
}

INSTANTIATE_TEST_CASE_P(Test, RMatViewTest,
                        Values(CV_8UC1, CV_8UC3, CV_32FC1));

struct RMatViewCallbackTest : public ::testing::Test {
    RMatViewCallbackTest()
        : mat(8,8,CV_8UC1) {
        cv::randn(mat, cv::Scalar::all(127), cv::Scalar::all(40));
    }
    View getView() { return asView(mat, [this](){ callbackCalls++; }); }
    int callbackCalls = 0;
    Mat mat;
};

TEST_F(RMatViewCallbackTest, MoveCtor) {
    {
        View copy(getView());
        cv::util::suppress_unused_warning(copy);
        EXPECT_EQ(0, callbackCalls);
    }
    EXPECT_EQ(1, callbackCalls);
}

TEST_F(RMatViewCallbackTest, MoveCopy) {
    {
        View copy;
        copy = getView();
        cv::util::suppress_unused_warning(copy);
        EXPECT_EQ(0, callbackCalls);
    }
    EXPECT_EQ(1, callbackCalls);
}

static int firstElement(const View& view) { return *view.ptr(); }
static void setFirstElement(View& view, uchar value) { *view.ptr() = value; }

TEST_F(RMatViewCallbackTest, MagazineInteraction) {
    cv::gimpl::magazine::Class<View> mag;
    constexpr int rc = 1;
    constexpr uchar value = 11;
    mag.slot<View>()[rc] = getView();
    {
        auto& mag_view = mag.slot<View>()[rc];
        setFirstElement(mag_view, value);
        auto mag_el = firstElement(mag_view);
        EXPECT_EQ(value, mag_el);
    }
    {
        const auto& mag_view = mag.slot<View>()[rc];
        auto mag_el = firstElement(mag_view);
        EXPECT_EQ(value, mag_el);
    }
    EXPECT_EQ(0, callbackCalls);
    mag.slot<View>().erase(rc);
    EXPECT_EQ(1, callbackCalls);
}

TEST(RMatView, Access1DMat) {
    cv::Mat m({1}, CV_32FC1);
    m.dims = 1;
    auto rmat = cv::make_rmat<cv::gimpl::RMatOnMat>(m);
    auto view = rmat.access(cv::RMat::Access::R);
    auto out = cv::gimpl::asMat(view);
    EXPECT_EQ(1, out.dims);
}
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

- **RMatViewTest**: A class/struct defined in this file
- **RMatViewNDTest**: A class/struct defined in this file
- **RMatViewNDTestNegative**: A class/struct defined in this file
- **RMatViewCallbackTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/rmat.hpp`
- `../src/backends/common/gbackend.hpp`
- `opencv2/gapi/util/compiler_hints.hpp`
- `../test_precomp.hpp`


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

