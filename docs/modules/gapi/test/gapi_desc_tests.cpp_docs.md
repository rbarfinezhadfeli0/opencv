# Documentation for `modules/gapi/test/gapi_desc_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_desc_tests.cpp`
- **File Name**: `gapi_desc_tests.cpp`
- **File Size**: 11,379 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_desc_tests.cpp](../../../modules/gapi/test/gapi_desc_tests.cpp)

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

#include <opencv2/gapi/cpu/gcpukernel.hpp>

namespace opencv_test
{

namespace
{
    G_TYPED_KERNEL(KTest, <cv::GScalar(cv::GScalar)>, "org.opencv.test.scalar_kernel") {
        static cv::GScalarDesc outMeta(cv::GScalarDesc in) { return in; }
    };
    GAPI_OCV_KERNEL(GOCVScalarTest, KTest)
    {
        static void run(const cv::Scalar &in, cv::Scalar &out) { out = in+cv::Scalar(1); }
    };
}

TEST(GAPI_MetaDesc, MatDescOneCh)
{
    cv::Mat mat(240, 320, CV_8U);

    const auto desc = cv::descr_of(mat);

    EXPECT_EQ(CV_8U, desc.depth);
    EXPECT_EQ(1,     desc.chan);
    EXPECT_EQ(320,   desc.size.width);
    EXPECT_EQ(240,   desc.size.height);
    EXPECT_FALSE(desc.isND());
}

TEST(GAPI_MetaDesc, MatDescThreeCh)
{
    cv::Mat mat(480, 640, CV_8UC3);

    const auto desc = cv::descr_of(mat);

    EXPECT_EQ(CV_8U,   desc.depth);
    EXPECT_EQ(3,       desc.chan);
    EXPECT_EQ(640,     desc.size.width);
    EXPECT_EQ(480,     desc.size.height);
    EXPECT_FALSE(desc.isND());
}

TEST(GAPI_MetaDesc, MatDescND)
{
    std::vector<int> dims = {1,3,299,299};
    cv::Mat m(dims, CV_32F);
    const auto desc = cv::descr_of(m);
    EXPECT_EQ(CV_32F, desc.depth);
    EXPECT_EQ(-1,     desc.chan);
    EXPECT_EQ(1,      desc.dims[0]);
    EXPECT_EQ(3,      desc.dims[1]);
    EXPECT_EQ(299,    desc.dims[2]);
    EXPECT_EQ(299,    desc.dims[3]);
    EXPECT_TRUE(desc.isND());
}

TEST(GAPI_MetaDesc, VecMatDesc)
{
    std::vector<cv::Mat> vec1 = {
    cv::Mat(240, 320, CV_8U)};

    const auto desc1 = cv::descrs_of(vec1);
    EXPECT_EQ((GMatDesc{CV_8U, 1, {320, 240}}), get<GMatDesc>(desc1[0]));

    std::vector<cv::UMat> vec2 = {
    cv::UMat(480, 640, CV_8UC3)};

    const auto desc2 = cv::descrs_of(vec2);
    EXPECT_EQ((GMatDesc{CV_8U, 3, {640, 480}}), get<GMatDesc>(desc2[0]));
}

TEST(GAPI_MetaDesc, CanDescribe)
{
    constexpr int w = 15;
    constexpr int h = 7;
    cv::Mat m0(h, w, CV_8UC3);
    cv::GMatDesc md0{CV_8U,3,{w,h},false};

    cv::Mat m1(h*3, w, CV_8UC1);
    cv::GMatDesc md10{CV_8U,3,{w,h},true};
    cv::GMatDesc md11{CV_8U,1,{w,h*3},false};

    EXPECT_TRUE (md0 .canDescribe(m0));
    EXPECT_FALSE(md0 .canDescribe(m1));
    EXPECT_TRUE (md10.canDescribe(m1));
    EXPECT_TRUE (md11.canDescribe(m1));
}

TEST(GAPI_MetaDesc, OwnMatDescOneCh)
{
    cv::gapi::own::Mat mat(240, 320, CV_8U, nullptr);

    const auto desc = cv::gapi::own::descr_of(mat);

    EXPECT_EQ(CV_8U, desc.depth);
    EXPECT_EQ(1,     desc.chan);
    EXPECT_EQ(320,   desc.size.width);
    EXPECT_EQ(240,   desc.size.height);
    EXPECT_FALSE(desc.isND());
}

TEST(GAPI_MetaDesc, OwnMatDescThreeCh)
{
    cv::gapi::own::Mat mat(480, 640, CV_8UC3, nullptr);

    const auto desc = cv::gapi::own::descr_of(mat);

    EXPECT_EQ(CV_8U,   desc.depth);
    EXPECT_EQ(3,       desc.chan);
    EXPECT_EQ(640,     desc.size.width);
    EXPECT_EQ(480,     desc.size.height);
    EXPECT_FALSE(desc.isND());
}

TEST(GAPI_MetaDesc, OwnMatDescND)
{
    std::vector<int> dims = {1,3,224,224};
    cv::gapi::own::Mat m(dims, CV_32F, nullptr);

    const auto desc = cv::gapi::own::descr_of(m);

    EXPECT_EQ(CV_32F, desc.depth);
    EXPECT_EQ(-1,     desc.chan);
    EXPECT_EQ(1,      desc.dims[0]);
    EXPECT_EQ(3,      desc.dims[1]);
    EXPECT_EQ(224,    desc.dims[2]);
    EXPECT_EQ(224,    desc.dims[3]);
    EXPECT_TRUE(desc.isND());
}

TEST(GAPI_MetaDesc, VecOwnMatDesc)
{
    std::vector<cv::gapi::own::Mat> vec = {
    cv::gapi::own::Mat(240, 320, CV_8U, nullptr),
    cv::gapi::own::Mat(480, 640, CV_8UC3, nullptr)};

    const auto desc = cv::gapi::own::descrs_of(vec);

    EXPECT_EQ((GMatDesc{CV_8U, 1, {320, 240}}), get<GMatDesc>(desc[0]));
    EXPECT_EQ((GMatDesc{CV_8U, 3, {640, 480}}), get<GMatDesc>(desc[1]));
}

TEST(GAPI_MetaDesc, AdlVecOwnMatDesc)
{
    std::vector<cv::gapi::own::Mat> vec = {
    cv::gapi::own::Mat(240, 320, CV_8U, nullptr),
    cv::gapi::own::Mat(480, 640, CV_8UC3, nullptr)};

    const auto desc = descrs_of(vec);

    EXPECT_EQ((GMatDesc{CV_8U, 1, {320, 240}}), get<GMatDesc>(desc[0]));
    EXPECT_EQ((GMatDesc{CV_8U, 3, {640, 480}}), get<GMatDesc>(desc[1]));
}

TEST(GAPI_MetaDesc, Compare_Equal_MatDesc)
{
    const auto desc1 = cv::GMatDesc{CV_8U, 1, {64, 64}};
    const auto desc2 = cv::GMatDesc{CV_8U, 1, {64, 64}};

    EXPECT_TRUE(desc1 == desc2);
}

TEST(GAPI_MetaDesc, Compare_Not_Equal_MatDesc)
{
    const auto desc1 = cv::GMatDesc{CV_8U,  1, {64, 64}};
    const auto desc2 = cv::GMatDesc{CV_32F, 1, {64, 64}};

    EXPECT_TRUE(desc1 != desc2);
}

TEST(GAPI_MetaDesc, Compare_Equal_MatDesc_ND)
{
    const auto desc1 = cv::GMatDesc{CV_8U, {1,3,224,224}};
    const auto desc2 = cv::GMatDesc{CV_8U, {1,3,224,224}};

    EXPECT_TRUE(desc1 == desc2);
}

TEST(GAPI_MetaDesc, Compare_Not_Equal_MatDesc_ND_1)
{
    const auto desc1 = cv::GMatDesc{CV_8U,  {1,1000}};
    const auto desc2 = cv::GMatDesc{CV_32F, {1,1000}};

    EXPECT_TRUE(desc1 != desc2);
}

TEST(GAPI_MetaDesc, Compare_Not_Equal_MatDesc_ND_2)
{
    const auto desc1 = cv::GMatDesc{CV_8U, {1,1000}};
    const auto desc2 = cv::GMatDesc{CV_8U, {1,1400}};

    EXPECT_TRUE(desc1 != desc2);
}

TEST(GAPI_MetaDesc, Compare_Not_Equal_MatDesc_ND_3)
{
    const auto desc1 = cv::GMatDesc{CV_8U, {1,1000}};
    const auto desc2 = cv::GMatDesc{CV_8U, 1, {32,32}};

    EXPECT_TRUE(desc1 != desc2);
}

TEST(GAPI_MetaDesc, Compile_MatchMetaNumber_1)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    const auto desc2 = cv::GMatDesc{CV_32F,1,{128,128}};

    EXPECT_NO_THROW(cc.compile(desc1));
    EXPECT_NO_THROW(cc.compile(desc2));

    // FIXME: custom exception type?
    // It is worth checking if compilation fails with different number
    // of meta parameters
    EXPECT_THROW(cc.compile(desc1, desc1),        std::logic_error);
    EXPECT_THROW(cc.compile(desc1, desc2, desc2), std::logic_error);
}

TEST(GAPI_MetaDesc, Compile_MatchMetaNumber_2)
{
    cv::GMat a, b;
    cv::GComputation cc(cv::GIn(a, b), cv::GOut(a+b));

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    EXPECT_NO_THROW(cc.compile(desc1, desc1));

    const auto desc2 = cv::GMatDesc{CV_32F,1,{128,128}};
    EXPECT_NO_THROW(cc.compile(desc2, desc2));

    // FIXME: custom exception type?
    EXPECT_THROW(cc.compile(desc1),               std::logic_error);
    EXPECT_THROW(cc.compile(desc2),               std::logic_error);
    EXPECT_THROW(cc.compile(desc2, desc2, desc2), std::logic_error);
}

TEST(GAPI_MetaDesc, Compile_MatchMetaType_Mat)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    EXPECT_NO_THROW(cc.compile(cv::GMatDesc{CV_8U,1,{64,64}}));

    // FIXME: custom exception type?
    EXPECT_THROW(cc.compile(cv::empty_scalar_desc()), std::logic_error);
}

TEST(GAPI_MetaDesc, Compile_MatchMetaType_Scalar)
{
    cv::GScalar in;
    cv::GComputation cc(cv::GIn(in), cv::GOut(KTest::on(in)));

    const auto desc1 = cv::descr_of(cv::Scalar(128));
    const auto desc2 = cv::GMatDesc{CV_8U,1,{64,64}};
    const auto pkg   = cv::gapi::kernels<GOCVScalarTest>();
    EXPECT_NO_THROW(cc.compile(desc1, cv::compile_args(pkg)));

    // FIXME: custom exception type?
    EXPECT_THROW(cc.compile(desc2, cv::compile_args(pkg)), std::logic_error);
}

TEST(GAPI_MetaDesc, Compile_MatchMetaType_Mixed)
{
    cv::GMat a;
    cv::GScalar v;
    cv::GComputation cc(cv::GIn(a, v), cv::GOut(cv::gapi::addC(a, v)));

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    const auto desc2 = cv::descr_of(cv::Scalar(4));

    EXPECT_NO_THROW(cc.compile(desc1, desc2));

    // FIXME: custom exception type(s)?
    EXPECT_THROW(cc.compile(desc1),               std::logic_error);
    EXPECT_THROW(cc.compile(desc2),               std::logic_error);
    EXPECT_THROW(cc.compile(desc2, desc1),        std::logic_error);
    EXPECT_THROW(cc.compile(desc1, desc1, desc1), std::logic_error);
    EXPECT_THROW(cc.compile(desc1, desc2, desc1), std::logic_error);
}

TEST(GAPI_MetaDesc, Typed_Compile_MatchMetaNumber_1)
{
    cv::GComputationT<cv::GMat(cv::GMat)> cc([](cv::GMat in)
    {
        return in+in;
    });

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    const auto desc2 = cv::GMatDesc{CV_32F,1,{128,128}};

    EXPECT_NO_THROW(cc.compile(desc1));
    EXPECT_NO_THROW(cc.compile(desc2));
}

TEST(GAPI_MetaDesc, Typed_Compile_MatchMetaNumber_2)
{
    cv::GComputationT<cv::GMat(cv::GMat,cv::GMat)> cc([](cv::GMat a, cv::GMat b)
    {
        return a + b;
    });

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    EXPECT_NO_THROW(cc.compile(desc1, desc1));

    const auto desc2 = cv::GMatDesc{CV_32F,1,{128,128}};
    EXPECT_NO_THROW(cc.compile(desc2, desc2));
}

TEST(GAPI_MetaDesc, Typed_Compile_MatchMetaType_Mat)
{
    cv::GComputationT<cv::GMat(cv::GMat)> cc([](cv::GMat in)
    {
        return in+in;
    });

    EXPECT_NO_THROW(cc.compile(cv::GMatDesc{CV_8U,1,{64,64}}));
}

TEST(GAPI_MetaDesc, Typed_Compile_MatchMetaType_Scalar)
{
    cv::GComputationT<cv::GScalar(cv::GScalar)> cc([](cv::GScalar in)
    {
        return KTest::on(in);
    });

    const auto desc1 = cv::descr_of(cv::Scalar(128));
    const auto pkg = cv::gapi::kernels<GOCVScalarTest>();
    //     EXPECT_NO_THROW(cc.compile(desc1, cv::compile_args(pkg)));
    cc.compile(desc1, cv::compile_args(pkg));
}

TEST(GAPI_MetaDesc, Typed_Compile_MatchMetaType_Mixed)
{
    cv::GComputationT<cv::GMat(cv::GMat,cv::GScalar)> cc([](cv::GMat a, cv::GScalar v)
    {
        return cv::gapi::addC(a, v);
    });

    const auto desc1 = cv::GMatDesc{CV_8U,1,{64,64}};
    const auto desc2 = cv::descr_of(cv::Scalar(4));

    EXPECT_NO_THROW(cc.compile(desc1, desc2));
}

TEST(GAPI_MetaDesc, Compare_Planar)
{
    const auto desc0 = cv::GMatDesc{CV_8U,3,{32,32},false};
    const auto desc1 = cv::GMatDesc{CV_8U,3,{32,32},false};
    const auto desc2 = cv::GMatDesc{CV_8U,3,{32,32},true};
    const auto desc3 = cv::GMatDesc{CV_8U,3,{64,64},true};

    EXPECT_TRUE(desc0 == desc1);
    EXPECT_TRUE(desc1 != desc2);
    EXPECT_TRUE(desc1 != desc3);
    EXPECT_TRUE(desc2 != desc3);
}

TEST(GAPI_MetaDesc, Sanity_asPlanar)
{
    constexpr int w = 32;
    constexpr int h = 16;
    const auto desc1 = cv::GMatDesc{CV_8U,3,{w,h},false};
    const auto desc2 = cv::GMatDesc{CV_8U,3,{w,h},true};

    EXPECT_NO_THROW(desc1.asPlanar());
    EXPECT_NO_THROW(desc2.asInterleaved());
    EXPECT_ANY_THROW(desc1.asInterleaved());
    EXPECT_ANY_THROW(desc2.asPlanar());
}

TEST(GAPI_MetaDesc, Compare_asPlanar)
{
    constexpr int w = 32;
    constexpr int h = 64;
    const auto desc0 = cv::GMatDesc{CV_8U,3,{w,h},false};
    const auto desc1 = cv::GMatDesc{CV_8U,3,{w,h},true};

    EXPECT_TRUE(desc0.asPlanar()      == desc1);
    EXPECT_TRUE(desc1.asInterleaved() == desc0);
}

TEST(GAPI_MetaDesc, Compare_asPlanarTransform)
{
    constexpr int w = 64;
    constexpr int h = 32;
    const auto desc0 = cv::GMatDesc{CV_8U,3,{w,h},true};
    const auto desc1 = cv::GMatDesc{CV_8U,1,{w,h*3},false};

    EXPECT_ANY_THROW(desc0.asPlanar(3));
    EXPECT_NO_THROW(desc1.asPlanar(3));
    EXPECT_TRUE(desc1.asPlanar(3) == desc0);
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `test_precomp.hpp`


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

