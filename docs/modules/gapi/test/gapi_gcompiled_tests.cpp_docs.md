# Documentation for `modules/gapi/test/gapi_gcompiled_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_gcompiled_tests.cpp`
- **File Name**: `gapi_gcompiled_tests.cpp`
- **File Size**: 7,194 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_gcompiled_tests.cpp](../../../modules/gapi/test/gapi_gcompiled_tests.cpp)

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

namespace opencv_test
{

namespace
{
    static cv::GMat DemoCC(cv::GMat in, cv::GScalar scale)
    {
        return cv::gapi::medianBlur(in + in*scale, 3);
    }

    struct GCompiledValidateMetaTyped: public ::testing::Test
    {
        cv::GComputationT<cv::GMat(cv::GMat,cv::GScalar)> m_cc;

        GCompiledValidateMetaTyped() : m_cc(DemoCC)
        {
        }
    };

    struct GCompiledValidateMetaUntyped: public ::testing::Test
    {
        cv::GMat in;
        cv::GScalar scale;
        cv::GComputation m_ucc;

        GCompiledValidateMetaUntyped() : m_ucc(cv::GIn(in, scale),
                                               cv::GOut(DemoCC(in, scale)))
        {
        }
    };

    struct GCompiledValidateMetaEmpty: public ::testing::Test
    {
        cv::GMat in;
        cv::GScalar scale;
        cv::GComputation m_ucc;

        G_API_OP(GReturn42, <cv::GOpaque<int>(cv::GMat)>, "org.opencv.test.return_42")
        {
            static GOpaqueDesc outMeta(cv::GMatDesc /* in */) { return cv::empty_gopaque_desc(); }
        };

        GAPI_OCV_KERNEL(GOCVReturn42, GReturn42)
        {
            static void run(const cv::Mat &/* in */, int &out)
            {
                out = 42;
            }
        };

        GCompiledValidateMetaEmpty() : m_ucc(cv::GIn(in),
                                             cv::GOut(GReturn42::on(in)))
        {
        }
    };
} // anonymous namespace

TEST_F(GCompiledValidateMetaTyped, ValidMeta)
{
    cv::Mat in = cv::Mat::eye(cv::Size(128, 32), CV_8UC1);
    cv::Scalar sc(127);

    auto f = m_cc.compile(cv::descr_of(in),
                          cv::descr_of(sc));

    // Correct operation when meta is exactly the same
    cv::Mat out;
    EXPECT_NO_THROW(f(in, sc, out));

    // Correct operation on next invocation with same meta
    // taken from different input objects
    cv::Mat in2 = cv::Mat::zeros(cv::Size(128, 32), CV_8UC1);
    cv::Scalar sc2(64);
    cv::Mat out2;
    EXPECT_NO_THROW(f(in2, sc2, out2));
}

TEST_F(GCompiledValidateMetaTyped, InvalidMeta)
{
    auto f = m_cc.compile(cv::GMatDesc{CV_8U,1,cv::Size(64,32)},
                          cv::empty_scalar_desc());

    cv::Scalar sc(33);
    cv::Mat out;

    // 3 channels instead 1
    cv::Mat in1 = cv::Mat::eye(cv::Size(64,32), CV_8UC3);
    EXPECT_THROW(f(in1, sc, out), std::logic_error);

    // 32f instead 8u
    cv::Mat in2 = cv::Mat::eye(cv::Size(64,32), CV_32F);
    EXPECT_THROW(f(in2, sc, out), std::logic_error);

    // 32x32 instead of 64x32
    cv::Mat in3 = cv::Mat::eye(cv::Size(32,32), CV_8UC1);
    EXPECT_THROW(f(in3, sc, out), std::logic_error);

    // All is wrong
    cv::Mat in4 = cv::Mat::eye(cv::Size(128,64), CV_32FC3);
    EXPECT_THROW(f(in4, sc, out), std::logic_error);
}

TEST_F(GCompiledValidateMetaUntyped, ValidMeta)
{
    cv::Mat in1 = cv::Mat::eye(cv::Size(128, 32), CV_8UC1);
    cv::Scalar sc(127);

    auto f = m_ucc.compile(cv::descr_of(in1),
                           cv::descr_of(sc));

    // Correct operation when meta is exactly the same
    cv::Mat out1;
    EXPECT_NO_THROW(f(cv::gin(in1, sc), cv::gout(out1)));

    // Correct operation on next invocation with same meta
    // taken from different input objects
    cv::Mat in2 = cv::Mat::zeros(cv::Size(128, 32), CV_8UC1);
    cv::Scalar sc2(64);
    cv::Mat out2;
    EXPECT_NO_THROW(f(cv::gin(in2, sc2), cv::gout(out2)));
}

TEST_F(GCompiledValidateMetaUntyped, InvalidMetaValues)
{
    auto f = m_ucc.compile(cv::GMatDesc{CV_8U,1,cv::Size(64,32)},
                           cv::empty_scalar_desc());

    cv::Scalar sc(33);
    cv::Mat out;

    // 3 channels instead 1
    cv::Mat in1 = cv::Mat::eye(cv::Size(64,32), CV_8UC3);
    EXPECT_THROW(f(cv::gin(in1, sc), cv::gout(out)), std::logic_error);

    // 32f instead 8u
    cv::Mat in2 = cv::Mat::eye(cv::Size(64,32), CV_32F);
    EXPECT_THROW(f(cv::gin(in2, sc), cv::gout(out)), std::logic_error);

    // 32x32 instead of 64x32
    cv::Mat in3 = cv::Mat::eye(cv::Size(32,32), CV_8UC1);
    EXPECT_THROW(f(cv::gin(in3, sc), cv::gout(out)), std::logic_error);

    // All is wrong
    cv::Mat in4 = cv::Mat::eye(cv::Size(128,64), CV_32FC3);
    EXPECT_THROW(f(cv::gin(in4, sc), cv::gout(out)), std::logic_error);
}

TEST_F(GCompiledValidateMetaUntyped, InvalidMetaShape)
{
    auto f = m_ucc.compile(cv::GMatDesc{CV_8U,1,cv::Size(64,32)},
                           cv::empty_scalar_desc());

    cv::Mat in1 = cv::Mat::eye(cv::Size(64,32), CV_8UC1);
    cv::Scalar sc(33);
    cv::Mat out1;

    // call as f(Mat,Mat) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(in1, in1), cv::gout(out1)), std::logic_error);

    // call as f(Scalar,Mat) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(sc, in1), cv::gout(out1)), std::logic_error);

    // call as f(Scalar,Scalar) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(sc, sc), cv::gout(out1)), std::logic_error);
}

TEST_F(GCompiledValidateMetaUntyped, InvalidMetaNumber)
{
    auto f = m_ucc.compile(cv::GMatDesc{CV_8U,1,cv::Size(64,32)},
                           cv::empty_scalar_desc());

    cv::Mat in1 = cv::Mat::eye(cv::Size(64,32), CV_8UC1);
    cv::Scalar sc(33);
    cv::Mat out1, out2;

    // call as f(Mat,Scalar,Scalar) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(in1, sc, sc), cv::gout(out1)), std::logic_error);

    // call as f(Scalar,Mat,Scalar) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(sc, in1, sc), cv::gout(out1)), std::logic_error);

    // call as f(Scalar) while f(Mat,Scalar) is expected
    EXPECT_THROW(f(cv::gin(sc), cv::gout(out1)), std::logic_error);

    // call as f(Mat,Scalar,[out1],[out2]) while f(Mat,Scalar,[out]) is expected
    EXPECT_THROW(f(cv::gin(in1, sc), cv::gout(out1, out2)), std::logic_error);
}

TEST_F(GCompiledValidateMetaEmpty, InvalidMatMetaCompile)
{
    EXPECT_THROW(m_ucc.compile(cv::empty_gmat_desc(),
                               cv::empty_scalar_desc()),
                 std::logic_error);
}

TEST_F(GCompiledValidateMetaEmpty, InvalidMatMetaApply)
{
    cv::Mat emptyIn;
    int out {};
    const auto pkg = cv::gapi::kernels<GCompiledValidateMetaEmpty::GOCVReturn42>();

    EXPECT_THROW(m_ucc.apply(cv::gin(emptyIn), cv::gout(out), cv::compile_args(pkg)),
                 std::logic_error);
}

TEST_F(GCompiledValidateMetaEmpty, ValidInvalidMatMetasApply)
{
    int out {};
    const auto pkg = cv::gapi::kernels<GCompiledValidateMetaEmpty::GOCVReturn42>();

    cv::Mat nonEmptyMat = cv::Mat::eye(cv::Size(64,32), CV_8UC1);
    m_ucc.apply(cv::gin(nonEmptyMat), cv::gout(out), cv::compile_args(pkg));
    EXPECT_EQ(out, 42);

    cv::Mat emptyIn;
    EXPECT_THROW(m_ucc.apply(cv::gin(emptyIn), cv::gout(out), cv::compile_args(pkg)),
                 std::logic_error);

    out = 0;
    m_ucc.apply(cv::gin(nonEmptyMat), cv::gout(out), cv::compile_args(pkg));
    EXPECT_EQ(out, 42);
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

- **GCompiledValidateMetaUntyped**: A class/struct defined in this file
- **GCompiledValidateMetaTyped**: A class/struct defined in this file
- **GCompiledValidateMetaEmpty**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `test_precomp.hpp`

**Python Imports:**
- `different`


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

