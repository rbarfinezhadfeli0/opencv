# Documentation for `modules/gapi/test/gapi_array_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_array_tests.cpp`
- **File Name**: `gapi_array_tests.cpp`
- **File Size**: 10,503 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_array_tests.cpp](../../../modules/gapi/test/gapi_array_tests.cpp)

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

#include <vector>
#include <ade/util/algorithm.hpp>

namespace opencv_test
{

namespace ThisTest
{
using GPointArray = cv::GArray<cv::Point>;
G_TYPED_KERNEL(GeneratePoints, <GPointArray(GMat)>, "test.array.out_const")
{
    static GArrayDesc outMeta(const GMatDesc&) { return empty_array_desc(); }
};
G_TYPED_KERNEL(FindCorners,    <GPointArray(GMat)>, "test.array.out")
{
    static GArrayDesc outMeta(const GMatDesc&) { return empty_array_desc(); }
};
G_TYPED_KERNEL(CountCorners,   <GScalar(GPointArray)>,  "test.array.in")
{
    static GScalarDesc outMeta(const GArrayDesc &) { return empty_scalar_desc(); }
};
G_TYPED_KERNEL(PointIncrement, <GPointArray(GMat, GPointArray)>, "test.point_increment")
{
    static GArrayDesc outMeta(const GMatDesc&, const GArrayDesc&) { return empty_array_desc(); }
};
G_TYPED_KERNEL(CountContours, <GOpaque<size_t>(GArray<GPointArray>)>, "test.array.array.in")
{
    static GOpaqueDesc outMeta(const GArrayDesc&) { return empty_gopaque_desc(); }
};
} // namespace ThisTest

namespace
{
GAPI_OCV_KERNEL(OCVGeneratePoints, ThisTest::GeneratePoints)
{
    static void run(cv::Mat, std::vector<cv::Point> &out)
    {
        for (int i = 0; i < 10; i++)
            out.emplace_back(i, i);
    }
};

GAPI_OCV_KERNEL(OCVFindCorners, ThisTest::FindCorners)
{
    static void run(cv::Mat in, std::vector<cv::Point> &out)
    {
        cv::goodFeaturesToTrack(in, out, 1024, 0.01, 3);
    }
};

GAPI_OCV_KERNEL(OCVCountCorners, ThisTest::CountCorners)
{
    static void run(const std::vector<cv::Point> &in, cv::Scalar &out)
    {
        out[0] = static_cast<double>(in.size());
    }
};

GAPI_OCV_KERNEL(OCVPointIncrement, ThisTest::PointIncrement)
{
    static void run(const cv::Mat&, const std::vector<cv::Point>& in, std::vector<cv::Point>& out)
    {
        for (const auto& el : in)
            out.emplace_back(el + Point(1,1));
    }
};

GAPI_OCV_KERNEL(OCVCountContours, ThisTest::CountContours)
{
    static void run(const std::vector<std::vector<cv::Point>> &contours, size_t &out)
    {
        out = contours.size();
    }
};

cv::Mat cross(int w, int h)
{
    cv::Mat mat = cv::Mat::eye(h, w, CV_8UC1)*255;
    cv::Mat yee;
    cv::flip(mat, yee, 0); // X-axis
    mat |= yee;            // make an "X" matrix;
    return mat;
}
} // (anonymous namespace)

TEST(GArray, TestReturnValue)
{
    // FIXME: Make .apply() able to take compile arguments
    cv::GComputationT<ThisTest::GPointArray(cv::GMat)> c(ThisTest::FindCorners::on);
    auto cc = c.compile(cv::GMatDesc{CV_8U,1,{32,32}},
                        cv::compile_args(cv::gapi::kernels<OCVFindCorners>()));

    // Prepare input matrix
    cv::Mat input = cross(32, 32);

    std::vector<cv::Point> points;
    cc(input, points);

    // OCV goodFeaturesToTrack should find 5 points here (with these settings)
    EXPECT_EQ(5u, points.size());
    EXPECT_TRUE(ade::util::find(points, cv::Point(16,16)) != points.end());
    EXPECT_TRUE(ade::util::find(points, cv::Point(30,30)) != points.end());
    EXPECT_TRUE(ade::util::find(points, cv::Point( 1,30)) != points.end());
    EXPECT_TRUE(ade::util::find(points, cv::Point(30, 1)) != points.end());
    EXPECT_TRUE(ade::util::find(points, cv::Point( 1, 1)) != points.end());
}

TEST(GArray, TestInputArg)
{
    cv::GComputationT<cv::GScalar(ThisTest::GPointArray)> c(ThisTest::CountCorners::on);
    auto cc = c.compile(cv::empty_array_desc(),
                        cv::compile_args(cv::gapi::kernels<OCVCountCorners>()));

    const std::vector<cv::Point> arr = {cv::Point(1,1), cv::Point(2,2)};
    cv::Scalar out;
    cc(arr, out);
    EXPECT_EQ(2, out[0]);
}

TEST(GArray, TestPipeline)
{
    cv::GComputationT<cv::GScalar(cv::GMat)> c([](cv::GMat in)
    {
        return ThisTest::CountCorners::on(ThisTest::FindCorners::on(in));
    });
    auto cc = c.compile(cv::GMatDesc{CV_8U,1,{32,32}},
                        cv::compile_args(cv::gapi::kernels<OCVFindCorners, OCVCountCorners>()));

    cv::Mat input = cross(32, 32);
    cv::Scalar out;
    cc(input, out);
    EXPECT_EQ(5, out[0]);
}

TEST(GArray, NoAggregationBetweenRuns)
{
    cv::GComputationT<cv::GScalar(cv::GMat)> c([](cv::GMat in)
    {
        return ThisTest::CountCorners::on(ThisTest::GeneratePoints::on(in));
    });
    auto cc = c.compile(cv::GMatDesc{CV_8U,1,{32,32}},
                        cv::compile_args(cv::gapi::kernels<OCVGeneratePoints, OCVCountCorners>()));

    cv::Mat input = cv::Mat::eye(32, 32, CV_8UC1);
    cv::Scalar out;

    cc(input, out);
    EXPECT_EQ(10, out[0]);

    // Last kernel in the graph counts number of elements in array, returned by the previous kernel
    // (in this test, this variable is constant).
    // After 10 executions, this number MUST remain the same - 1st kernel is adding new values on every
    // run, but it is graph's responsibility to reset internal object state.
    cv::Scalar out2;
    for (int i = 0; i < 10; i++)
    {
        cc(input, out2);
    }
    EXPECT_EQ(10, out2[0]);
}

TEST(GArray, TestIntermediateOutput)
{
    using Result = std::tuple<ThisTest::GPointArray, cv::GScalar>;
    cv::GComputationT<Result(cv::GMat)> c([](cv::GMat in)
    {
        auto corners = ThisTest::GeneratePoints::on(in);
        return std::make_tuple(corners, ThisTest::CountCorners::on(corners));
    });

    cv::Mat in_mat = cv::Mat::eye(32, 32, CV_8UC1);
    std::vector<cv::Point> out_points;
    cv::Scalar out_count;

    auto cc = c.compile(cv::descr_of(in_mat),
                        cv::compile_args(cv::gapi::kernels<OCVGeneratePoints, OCVCountCorners>()));
    cc(in_mat, out_points, out_count);

    EXPECT_EQ(10u, out_points.size());
    EXPECT_EQ(10,  out_count[0]);
}

TEST(GArray, TestGArrayGArrayKernelInput)
{
    cv::GMat in;
    auto contours = cv::gapi::findContours(in, cv::RETR_EXTERNAL, cv::CHAIN_APPROX_NONE);
    auto out = ThisTest::CountContours::on(contours);
    cv::GComputation c(GIn(in), GOut(out));

    // Create input - two filled rectangles
    cv::Mat in_mat = cv::Mat::zeros(50, 50, CV_8UC1);
    cv::rectangle(in_mat, cv::Point{5,5},   cv::Point{20,20}, 255, cv::FILLED);
    cv::rectangle(in_mat, cv::Point{25,25}, cv::Point{40,40}, 255, cv::FILLED);

    size_t out_count = 0u;
    c.apply(gin(in_mat), gout(out_count), cv::compile_args(cv::gapi::kernels<OCVCountContours>()));

    EXPECT_EQ(2u, out_count) << "Two contours must be found";
}

TEST(GArray, GArrayConstValInitialization)
{
    std::vector<cv::Point> initial_vec {Point(0,0), Point(1,1), Point(2,2)};
    std::vector<cv::Point> ref_vec     {Point(1,1), Point(2,2), Point(3,3)};
    std::vector<cv::Point> out_vec;
    cv::Mat in_mat = cv::Mat::eye(32, 32, CV_8UC1);

    cv::GComputationT<ThisTest::GPointArray(cv::GMat)> c([&](cv::GMat in)
    {
        // Initialization
        ThisTest::GPointArray test_garray(initial_vec);
        return ThisTest::PointIncrement::on(in, test_garray);
    });
    auto cc = c.compile(cv::descr_of(in_mat),
                        cv::compile_args(cv::gapi::kernels<OCVPointIncrement>()));
    cc(in_mat, out_vec);

    EXPECT_EQ(ref_vec, out_vec);
}

TEST(GArray, GArrayRValInitialization)
{
    std::vector<cv::Point> ref_vec {Point(1,1), Point(2,2), Point(3,3)};
    std::vector<cv::Point> out_vec;
    cv::Mat in_mat = cv::Mat::eye(32, 32, CV_8UC1);

    cv::GComputationT<ThisTest::GPointArray(cv::GMat)> c([&](cv::GMat in)
    {
        // Rvalue initialization
        ThisTest::GPointArray test_garray({Point(0,0), Point(1,1), Point(2,2)});
        return ThisTest::PointIncrement::on(in, test_garray);
    });
    auto cc = c.compile(cv::descr_of(in_mat),
                        cv::compile_args(cv::gapi::kernels<OCVPointIncrement>()));
    cc(in_mat, out_vec);

    EXPECT_EQ(ref_vec, out_vec);
}

TEST(GArray_VectorRef, TestMov)
{
    // Warning: this test is testing some not-very-public APIs
    // Test how VectorRef's mov() (aka poor man's move()) is working.

    using I = int;
    using V = std::vector<I>;
    const V vgold = { 1, 2, 3};
    V vtest = vgold;
    const I* vptr = vtest.data();

    cv::detail::VectorRef vref(vtest);
    cv::detail::VectorRef vmov;
    vmov.reset<I>();

    EXPECT_EQ(vgold, vref.rref<I>());

    vmov.mov(vref);
    EXPECT_EQ(vgold, vmov.rref<I>());
    EXPECT_EQ(vptr,  vmov.rref<I>().data());
    EXPECT_EQ(V{}, vref.rref<I>());
    EXPECT_EQ(V{}, vtest);
}

// types from anonymous namespace doesn't work well with templates
inline namespace gapi_array_tests {
    struct MyTestStruct {
        int i;
        float f;
        std::string name;
    };
}

TEST(GArray_VectorRef, Kind)
{
    cv::detail::VectorRef v1(std::vector<cv::Rect>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_RECT, v1.getKind());

    cv::detail::VectorRef v2(std::vector<cv::Mat>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_MAT,  v2.getKind());

    cv::detail::VectorRef v3(std::vector<int>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_INT, v3.getKind());

    cv::detail::VectorRef v4(std::vector<double>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_DOUBLE, v4.getKind());

    cv::detail::VectorRef v5(std::vector<cv::Scalar>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_SCALAR, v5.getKind());

    cv::detail::VectorRef v6(std::vector<cv::Point>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_POINT, v6.getKind());

    cv::detail::VectorRef v7(std::vector<cv::Size>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_SIZE, v7.getKind());

    cv::detail::VectorRef v8(std::vector<std::string>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_STRING, v8.getKind());

    cv::detail::VectorRef v9(std::vector<MyTestStruct>{});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_UNKNOWN, v9.getKind());
}

TEST(GArray_VectorRef, TestRvalue)
{
    // Warning: this test is testing some not-very-public APIs
    cv::detail::VectorRef vref(std::vector<int>{3, 5, -4});
    auto v = std::vector<int>{3, 5, -4};
    EXPECT_EQ(vref.rref<int>(), v);
}

TEST(GArray_VectorRef, TestReset)
{
    // Warning: this test is testing some not-very-public APIs
    cv::detail::VectorRef vref(std::vector<int>{3, 5, -4});
    EXPECT_EQ(cv::detail::OpaqueKind::CV_INT, vref.getKind());
    vref.reset<int>();
    EXPECT_EQ(cv::detail::OpaqueKind::CV_INT, vref.getKind());
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

- **MyTestStruct**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `vector`
- `ade/util/algorithm.hpp`
- `test_precomp.hpp`

**Python Imports:**
- `anonymous`


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

