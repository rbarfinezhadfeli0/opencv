# Documentation for `modules/video/test/test_accum.cpp`

## File Metadata

- **Full Path**: `modules/video/test/test_accum.cpp`
- **File Name**: `test_accum.cpp`
- **File Size**: 7,608 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/test/test_accum.cpp](../../../modules/video/test/test_accum.cpp)

## Purpose and Role

This file is located in the `modules/video/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of Intel Corporation may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "test_precomp.hpp"
#include "opencv2/imgproc/imgproc_c.h"

namespace opencv_test { namespace {

class CV_AccumBaseTest : public cvtest::ArrayTest
{
public:
    CV_AccumBaseTest();

protected:
    void get_test_array_types_and_sizes( int test_case_idx, vector<vector<Size> >& sizes, vector<vector<int> >& types );
    double get_success_error_level( int test_case_idx, int i, int j );
    double alpha;
};


CV_AccumBaseTest::CV_AccumBaseTest()
{
    test_array[INPUT].push_back(NULL);
    test_array[INPUT_OUTPUT].push_back(NULL);
    test_array[REF_INPUT_OUTPUT].push_back(NULL);
    test_array[MASK].push_back(NULL);
    optional_mask = true;
    element_wise_relative_error = false;
} // ctor


void CV_AccumBaseTest::get_test_array_types_and_sizes( int test_case_idx,
                        vector<vector<Size> >& sizes, vector<vector<int> >& types )
{
    RNG& rng = ts->get_rng();
    int depth = cvtest::randInt(rng) % 4, cn = cvtest::randInt(rng) & 1 ? 3 : 1;
    int accdepth = (int)(cvtest::randInt(rng) % 2 + 1);
    int i, input_count = (int)test_array[INPUT].size();
    cvtest::ArrayTest::get_test_array_types_and_sizes( test_case_idx, sizes, types );
    depth = depth == 0 ? CV_8U : depth == 1 ? CV_16U : depth == 2 ? CV_32F : CV_64F;
    accdepth = accdepth == 1 ? CV_32F : CV_64F;
    accdepth = MAX(accdepth, depth);

    for( i = 0; i < input_count; i++ )
        types[INPUT][i] = CV_MAKETYPE(depth,cn);

    types[INPUT_OUTPUT][0] = types[REF_INPUT_OUTPUT][0] = CV_MAKETYPE(accdepth,cn);

    alpha = cvtest::randReal(rng);
}


double CV_AccumBaseTest::get_success_error_level( int /*test_case_idx*/, int /*i*/, int /*j*/ )
{
    return test_mat[INPUT_OUTPUT][0].depth() < CV_64F ||
           test_mat[INPUT][0].depth() == CV_32F ? FLT_EPSILON*100 : DBL_EPSILON*1000;
}


/// acc
class CV_AccTest : public CV_AccumBaseTest
{
public:
    CV_AccTest() { }
protected:
    void run_func();
    void prepare_to_validation( int );
};


void CV_AccTest::run_func(void)
{
    cvAcc( test_array[INPUT][0], test_array[INPUT_OUTPUT][0], test_array[MASK][0] );
}


void CV_AccTest::prepare_to_validation( int )
{
    const Mat& src = test_mat[INPUT][0];
    Mat& dst = test_mat[REF_INPUT_OUTPUT][0];
    const Mat& mask = test_array[MASK][0] ? test_mat[MASK][0] : Mat();
    Mat temp;
    cvtest::add( src, 1, dst, 1, cvScalarAll(0.), temp, dst.type() );
    cvtest::copy( temp, dst, mask );
}


/// square acc
class CV_SquareAccTest : public CV_AccumBaseTest
{
public:
    CV_SquareAccTest();
protected:
    void run_func();
    void prepare_to_validation( int );
};


CV_SquareAccTest::CV_SquareAccTest()
{
}


void CV_SquareAccTest::run_func()
{
    cvSquareAcc( test_array[INPUT][0], test_array[INPUT_OUTPUT][0], test_array[MASK][0] );
}


void CV_SquareAccTest::prepare_to_validation( int )
{
    const Mat& src = test_mat[INPUT][0];
    Mat& dst = test_mat[REF_INPUT_OUTPUT][0];
    const Mat& mask = test_array[MASK][0] ? test_mat[MASK][0] : Mat();
    Mat temp;

    cvtest::convert( src, temp, dst.type() );
    cvtest::multiply( temp, temp, temp, 1 );
    cvtest::add( temp, 1, dst, 1, cvScalarAll(0.), temp, dst.depth() );
    cvtest::copy( temp, dst, mask );
}


/// multiply acc
class CV_MultiplyAccTest : public CV_AccumBaseTest
{
public:
    CV_MultiplyAccTest();
protected:
    void run_func();
    void prepare_to_validation( int );
};


CV_MultiplyAccTest::CV_MultiplyAccTest()
{
    test_array[INPUT].push_back(NULL);
}


void CV_MultiplyAccTest::run_func()
{
    cvMultiplyAcc( test_array[INPUT][0], test_array[INPUT][1],
                   test_array[INPUT_OUTPUT][0], test_array[MASK][0] );
}


void CV_MultiplyAccTest::prepare_to_validation( int )
{
    const Mat& src1 = test_mat[INPUT][0];
    const Mat& src2 = test_mat[INPUT][1];
    Mat& dst = test_mat[REF_INPUT_OUTPUT][0];
    const Mat& mask = test_array[MASK][0] ? test_mat[MASK][0] : Mat();
    Mat temp1, temp2;

    cvtest::convert( src1, temp1, dst.type() );
    cvtest::convert( src2, temp2, dst.type() );

    cvtest::multiply( temp1, temp2, temp1, 1 );
    cvtest::add( temp1, 1, dst, 1, cvScalarAll(0.), temp1, dst.depth() );
    cvtest::copy( temp1, dst, mask );
}


/// running average
class CV_RunningAvgTest : public CV_AccumBaseTest
{
public:
    CV_RunningAvgTest();
protected:
    void run_func();
    void prepare_to_validation( int );
};


CV_RunningAvgTest::CV_RunningAvgTest()
{
}


void CV_RunningAvgTest::run_func()
{
    cvRunningAvg( test_array[INPUT][0], test_array[INPUT_OUTPUT][0],
                  alpha, test_array[MASK][0] );
}


void CV_RunningAvgTest::prepare_to_validation( int )
{
    const Mat& src = test_mat[INPUT][0];
    Mat& dst = test_mat[REF_INPUT_OUTPUT][0];
    Mat temp;
    const Mat& mask = test_array[MASK][0] ? test_mat[MASK][0] : Mat();
    double a[1], b[1];
    int accdepth = test_mat[INPUT_OUTPUT][0].depth();
    CvMat A = cvMat(1,1,accdepth,a), B = cvMat(1,1,accdepth,b);
    cvSetReal1D( &A, 0, alpha);
    cvSetReal1D( &B, 0, 1 - cvGetReal1D(&A, 0));

    cvtest::convert( src, temp, dst.type() );
    cvtest::add( src, cvGetReal1D(&A, 0), dst, cvGetReal1D(&B, 0), cvScalarAll(0.), temp, temp.depth() );
    cvtest::copy( temp, dst, mask );
}


TEST(Video_Acc, accuracy) { CV_AccTest test; test.safe_run(); }
TEST(Video_AccSquared, accuracy) { CV_SquareAccTest test; test.safe_run(); }
TEST(Video_AccProduct, accuracy) { CV_MultiplyAccTest test; test.safe_run(); }
TEST(Video_RunningAvg, accuracy) { CV_RunningAvgTest test; test.safe_run(); }

}} // namespace
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

- **CV_RunningAvgTest**: A class/struct defined in this file
- **CV_AccumBaseTest**: A class/struct defined in this file
- **CV_MultiplyAccTest**: A class/struct defined in this file
- **CV_SquareAccTest**: A class/struct defined in this file
- **CV_AccTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/imgproc/imgproc_c.h`
- `test_precomp.hpp`

**Python Imports:**
- `this`


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

