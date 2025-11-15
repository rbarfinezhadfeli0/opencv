# Documentation for `modules/gapi/test/gapi_scalar_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_scalar_tests.cpp`
- **File Name**: `gapi_scalar_tests.cpp`
- **File Size**: 2,937 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_scalar_tests.cpp](../../../modules/gapi/test/gapi_scalar_tests.cpp)

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

#include <iostream>

namespace opencv_test
{

TEST(GAPI_Scalar, Argument)
{
    cv::Size sz(2, 2);
    cv::Mat in_mat(sz, CV_8U);
    cv::randn(in_mat, cv::Scalar::all(127), cv::Scalar::all(40.f));

    cv::GComputationT<cv::GMat (cv::GMat, cv::GScalar)> mulS([](cv::GMat in, cv::GScalar c)
    {
        return in*c;
    });

    cv::Mat out_mat(sz, CV_8U);
    mulS.apply(in_mat, cv::Scalar(2), out_mat);

    cv::Mat reference = in_mat*2;
    EXPECT_EQ(0, cvtest::norm(out_mat, reference, NORM_INF));
}

TEST(GAPI_Scalar, ReturnValue)
{
    const cv::Size sz(2, 2);
    cv::Mat in_mat(sz, CV_8U, cv::Scalar(1));

    cv::GComputationT<cv::GScalar (cv::GMat)> sum_of_sum([](cv::GMat in)
    {
        return cv::gapi::sum(in + in);
    });

    cv::Scalar out;
    sum_of_sum.apply(in_mat, out);

    EXPECT_EQ(8, out[0]);
}

TEST(GAPI_Scalar, TmpScalar)
{
    const cv::Size sz(2, 2);
    cv::Mat in_mat(sz, CV_8U, cv::Scalar(1));

    cv::GComputationT<cv::GMat (cv::GMat)> mul_by_sum([](cv::GMat in)
    {
        return in * cv::gapi::sum(in);
    });

    cv::Mat out_mat(sz, CV_8U);
    mul_by_sum.apply(in_mat, out_mat);

    cv::Mat reference = cv::Mat(sz, CV_8U, cv::Scalar(4));
    EXPECT_EQ(0, cvtest::norm(out_mat, reference, NORM_INF));
}

TEST(GAPI_ScalarWithValue, Simple_Arithmetic_Pipeline)
{
    GMat in;
    GMat out = (in + 1) * 2;
    cv::GComputation comp(in, out);

    cv::Mat in_mat  = cv::Mat::eye(3, 3, CV_8UC1);
    cv::Mat ref_mat, out_mat;

    ref_mat = (in_mat + 1) * 2;
    comp.apply(in_mat, out_mat);

    EXPECT_EQ(0, cvtest::norm(out_mat, ref_mat, NORM_INF));
}

TEST(GAPI_ScalarWithValue, GScalar_Initilization)
{
    cv::Scalar sc(2);
    cv::GMat in;
    cv::GScalar s(sc);
    cv::GComputation comp(in, cv::gapi::mulC(in, s));

    cv::Mat in_mat = cv::Mat::eye(3, 3, CV_8UC1);
    cv::Mat ref_mat, out_mat;
    cv::multiply(in_mat, sc, ref_mat, 1, CV_8UC1);
    comp.apply(cv::gin(in_mat), cv::gout(out_mat));

    EXPECT_EQ(0, cvtest::norm(out_mat, ref_mat, NORM_INF));
}

TEST(GAPI_ScalarWithValue, Constant_GScalar_In_Middle_Graph)
{
    cv::Scalar  sc(5);
    cv::GMat    in1;
    cv::GScalar in2;
    cv::GScalar s(sc);

    auto add_out = cv::gapi::addC(in1, in2);
    cv::GComputation comp(cv::GIn(in1, in2), cv::GOut(cv::gapi::mulC(add_out, s)));

    cv::Mat    in_mat = cv::Mat::eye(3, 3, CV_8UC1);
    cv::Scalar in_scalar(3);

    cv::Mat ref_mat, out_mat, add_mat;
    cv::add(in_mat, in_scalar, add_mat);
    cv::multiply(add_mat, sc, ref_mat, 1, CV_8UC1);
    comp.apply(cv::gin(in_mat, in_scalar), cv::gout(out_mat));

    EXPECT_EQ(0, cvtest::norm(out_mat, ref_mat, NORM_INF));
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
- `iostream`
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

