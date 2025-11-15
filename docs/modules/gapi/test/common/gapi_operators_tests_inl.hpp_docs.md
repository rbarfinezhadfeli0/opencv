# Documentation for `modules/gapi/test/common/gapi_operators_tests_inl.hpp`

## File Metadata

- **Full Path**: `modules/gapi/test/common/gapi_operators_tests_inl.hpp`
- **File Name**: `gapi_operators_tests_inl.hpp`
- **File Size**: 5,441 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/test/common/gapi_operators_tests_inl.hpp](../../../../modules/gapi/test/common/gapi_operators_tests_inl.hpp)

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


#ifndef OPENCV_GAPI_OPERATOR_TESTS_INL_COMMON_HPP
#define OPENCV_GAPI_OPERATOR_TESTS_INL_COMMON_HPP

#include "gapi_operators_tests.hpp"

namespace opencv_test
{
TEST_P(MathOperatorMatScalarTest, OperatorAccuracyTest )
{
    g_api_ocv_pair_mat_scalar funcs(op);
    auto fun_gapi = funcs.g_api_function;
    auto fun_ocv  = funcs.ocv_function;

    if (op == DIVR)
        in_mat1.setTo(1, in_mat1 == 0);                               // avoiding zeros in divide input data
    if (op == DIV)
        sc += Scalar(sc[0] == 0, sc[1] == 0, sc[2] == 0, sc[3] == 0); // avoiding zeros in divide input data

    // G-API code & corresponding OpenCV code ////////////////////////////////

    cv::GMat in1;
    cv::GScalar in2;
    auto out = fun_gapi(in1, in2);
    cv::GComputation c(GIn(in1, in2), GOut(out));

    c.apply(gin(in_mat1, sc), gout(out_mat_gapi), getCompileArgs());

    fun_ocv(in_mat1, sc, out_mat_ocv);

    // Comparison //////////////////////////////////////////////////////////////
    {
        ASSERT_EQ(sz, out_mat_gapi.size());
        EXPECT_TRUE(cmpF(out_mat_gapi, out_mat_ocv));
    }
}

TEST_P(MathOperatorMatMatTest, OperatorAccuracyTest )
{
    g_api_ocv_pair_mat_mat funcs(op);
    auto fun_gapi = funcs.g_api_function;
    auto fun_ocv  = funcs.ocv_function;

    if (op == DIV)
        in_mat2.setTo(1, in_mat2 == 0); // avoiding zeros in divide input data

    // G-API code & corresponding OpenCV code ////////////////////////////////

    cv::GMat in1;
    cv::GMat in2;
    auto out = fun_gapi(in1, in2);
    cv::GComputation c(GIn(in1, in2), GOut(out));

    c.apply(gin(in_mat1, in_mat2), gout(out_mat_gapi), getCompileArgs());

    fun_ocv(in_mat1, in_mat2, out_mat_ocv);

    // Comparison //////////////////////////////////////////////////////////////
    {
        ASSERT_EQ(sz, out_mat_gapi.size());
        EXPECT_TRUE(cmpF(out_mat_gapi, out_mat_ocv));
    }
}

TEST_P(NotOperatorTest, OperatorAccuracyTest)
{
    // G-API code //////////////////////////////////////////////////////////////
    cv::GMat in;
    auto out = ~in;
    cv::GComputation c(in, out);

    c.apply(in_mat1, out_mat_gapi, getCompileArgs());

    // OpenCV code /////////////////////////////////////////////////////////////
    {
        out_mat_ocv =~in_mat1;
    }
    // Comparison //////////////////////////////////////////////////////////////
    {
        ASSERT_EQ(sz, out_mat_gapi.size());
        EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
    }
}

namespace for_test
{
class Foo {};

inline int operator&(Foo, int) { return 1; }
inline int operator|(Foo, int) { return 1; }
inline int operator^(Foo, int) { return 1; }
inline int operator~(Foo)      { return 1; }

inline int operator+(Foo, int) { return 1; }
inline int operator-(Foo, int) { return 1; }
inline int operator*(Foo, int) { return 1; }
inline int operator/(Foo, int) { return 1; }

inline int operator> (Foo, int) { return 1; }
inline int operator>=(Foo, int) { return 1; }
inline int operator< (Foo, int) { return 1; }
inline int operator<=(Foo, int) { return 1; }
inline int operator==(Foo, int) { return 1; }
inline int operator!=(Foo, int) { return 1; }

TEST(CVNamespaceOperatorsTest, OperatorCompilationTest)
{
    cv::GScalar sc;
    cv::GMat mat_in1, mat_in2;

    cv::GMat op_not = ~ mat_in1;

    cv::GMat op_mat_mat1  = mat_in1 &  mat_in2;
    cv::GMat op_mat_mat2  = mat_in1 |  mat_in2;
    cv::GMat op_mat_mat3  = mat_in1 ^  mat_in2;
    cv::GMat op_mat_mat4  = mat_in1 +  mat_in2;
    cv::GMat op_mat_mat5  = mat_in1 -  mat_in2;
    cv::GMat op_mat_mat6  = mat_in1 /  mat_in2;
    cv::GMat op_mat_mat7  = mat_in1 >  mat_in2;
    cv::GMat op_mat_mat8  = mat_in1 >= mat_in2;
    cv::GMat op_mat_mat9  = mat_in1 <  mat_in2;
    cv::GMat op_mat_mat10 = mat_in1 <= mat_in2;
    cv::GMat op_mat_mat11 = mat_in1 == mat_in2;
    cv::GMat op_mat_mat12 = mat_in1 != mat_in2;

    cv::GMat op_mat_sc1  = mat_in1 &  sc;
    cv::GMat op_mat_sc2  = mat_in1 |  sc;
    cv::GMat op_mat_sc3  = mat_in1 ^  sc;
    cv::GMat op_mat_sc4  = mat_in1 +  sc;
    cv::GMat op_mat_sc5  = mat_in1 -  sc;
    cv::GMat op_mat_sc6  = mat_in1 *  sc;
    cv::GMat op_mat_sc7  = mat_in1 /  sc;
    cv::GMat op_mat_sc8  = mat_in1 >  sc;
    cv::GMat op_mat_sc9  = mat_in1 >= sc;
    cv::GMat op_mat_sc10 = mat_in1 <  sc;
    cv::GMat op_mat_sc11 = mat_in1 <= sc;
    cv::GMat op_mat_sc12 = mat_in1 == sc;
    cv::GMat op_mat_sc13 = mat_in1 != sc;

    cv::GMat op_sc_mat1  = sc &  mat_in2;
    cv::GMat op_sc_mat2  = sc |  mat_in2;
    cv::GMat op_sc_mat3  = sc ^  mat_in2;
    cv::GMat op_sc_mat4  = sc +  mat_in2;
    cv::GMat op_sc_mat5  = sc -  mat_in2;
    cv::GMat op_sc_mat6  = sc *  mat_in2;
    cv::GMat op_sc_mat7  = sc /  mat_in2;
    cv::GMat op_sc_mat8  = sc >  mat_in2;
    cv::GMat op_sc_mat9  = sc >= mat_in2;
    cv::GMat op_sc_mat10 = sc <  mat_in2;
    cv::GMat op_sc_mat11 = sc <= mat_in2;
    cv::GMat op_sc_mat12 = sc == mat_in2;
    cv::GMat op_sc_mat13 = sc != mat_in2;

    cv::GMat mul_mat_float1 = mat_in1 * 1.0f;
    cv::GMat mul_mat_float2 = 1.0f * mat_in2;
    // No compilation errors expected
}
} // for_test
} // opencv_test

#endif // OPENCV_GAPI_OPERATOR_TESTS_INL_COMMON_HPP
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

- **Foo**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_OPERATOR_TESTS_INL_COMMON_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `gapi_operators_tests.hpp`


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

