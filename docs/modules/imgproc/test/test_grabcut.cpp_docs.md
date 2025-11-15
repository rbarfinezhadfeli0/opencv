# Documentation for `modules/imgproc/test/test_grabcut.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/test/test_grabcut.cpp`
- **File Name**: `test_grabcut.cpp`
- **File Size**: 6,722 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/test/test_grabcut.cpp](../../../modules/imgproc/test/test_grabcut.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/test` directory and serves as part of the OpenCV library infrastructure.

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
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
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
//   * The name of the copyright holders may not be used to endorse or promote products
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

namespace opencv_test { namespace {

class CV_GrabcutTest : public cvtest::BaseTest
{
public:
    CV_GrabcutTest();
    ~CV_GrabcutTest();
protected:
    bool verify(const Mat& mask, const Mat& exp);
    void run(int);
};

CV_GrabcutTest::CV_GrabcutTest() {}
CV_GrabcutTest::~CV_GrabcutTest() {}

bool CV_GrabcutTest::verify(const Mat& mask, const Mat& exp)
{
    const float maxDiffRatio = 0.005f;
    int expArea = countNonZero( exp );
    int nonIntersectArea = countNonZero( mask != exp );

    float curRatio = (float)nonIntersectArea / (float)expArea;
    ts->printf( cvtest::TS::LOG, "nonIntersectArea/expArea = %f\n", curRatio );
    return curRatio < maxDiffRatio;
}

void CV_GrabcutTest::run( int /* start_from */)
{
    cvtest::DefaultRngAuto defRng;

    Mat img = imread(string(ts->get_data_path()) + "shared/airplane.png");
    Mat mask_prob = imread(string(ts->get_data_path()) + "grabcut/mask_prob.png", 0);
    Mat exp_mask1 = imread(string(ts->get_data_path()) + "grabcut/exp_mask1.png", 0);
    Mat exp_mask2 = imread(string(ts->get_data_path()) + "grabcut/exp_mask2.png", 0);

    if (img.empty() || (!mask_prob.empty() && img.size() != mask_prob.size()) ||
                       (!exp_mask1.empty() && img.size() != exp_mask1.size()) ||
                       (!exp_mask2.empty() && img.size() != exp_mask2.size()) )
    {
         ts->set_failed_test_info(cvtest::TS::FAIL_MISSING_TEST_DATA);
         return;
    }

    Rect rect(Point(24, 126), Point(483, 294));
    Mat exp_bgdModel, exp_fgdModel;

    Mat mask;
    Mat bgdModel, fgdModel;
    grabCut( img, mask, rect, bgdModel, fgdModel, 0, GC_INIT_WITH_RECT );
    bgdModel.copyTo(exp_bgdModel);
    fgdModel.copyTo(exp_fgdModel);
    grabCut( img, mask, rect, bgdModel, fgdModel, 2, GC_EVAL_FREEZE_MODEL );

    // Multiply images by 255 for more visuality of test data.
    if( mask_prob.empty() )
    {
        mask.copyTo( mask_prob );
        imwrite(string(ts->get_data_path()) + "grabcut/mask_prob.png", mask_prob);
    }
    if( exp_mask1.empty() )
    {
        exp_mask1 = (mask & 1) * 255;
        imwrite(string(ts->get_data_path()) + "grabcut/exp_mask1.png", exp_mask1);
    }
    if (!verify((mask & 1) * 255, exp_mask1))
    {
        ts->set_failed_test_info(cvtest::TS::FAIL_MISMATCH);
        return;
    }
    // The model should not be changed after calling with GC_EVAL_FREEZE_MODEL
    double sumBgdModel = cv::sum(cv::abs(bgdModel) - cv::abs(exp_bgdModel))[0];
    double sumFgdModel = cv::sum(cv::abs(fgdModel) - cv::abs(exp_fgdModel))[0];
    if (sumBgdModel >= 0.1 || sumFgdModel >= 0.1)
    {
        ts->printf(cvtest::TS::LOG, "sumBgdModel = %f, sumFgdModel = %f\n", sumBgdModel, sumFgdModel);
        ts->set_failed_test_info(cvtest::TS::FAIL_MISMATCH);
        return;
    }

    mask = mask_prob;
    bgdModel.release();
    fgdModel.release();
    rect = Rect();
    grabCut( img, mask, rect, bgdModel, fgdModel, 0, GC_INIT_WITH_MASK );
    grabCut( img, mask, rect, bgdModel, fgdModel, 1, GC_EVAL );

    if( exp_mask2.empty() )
    {
        exp_mask2 = (mask & 1) * 255;
        imwrite(string(ts->get_data_path()) + "grabcut/exp_mask2.png", exp_mask2);
    }
    if (!verify((mask & 1) * 255, exp_mask2))
    {
        ts->set_failed_test_info(cvtest::TS::FAIL_MISMATCH);
        return;
    }
    ts->set_failed_test_info(cvtest::TS::OK);
}

TEST(Imgproc_GrabCut, regression) { CV_GrabcutTest test; test.safe_run(); }

TEST(Imgproc_GrabCut, repeatability)
{
    cvtest::TS& ts = *cvtest::TS::ptr();

    Mat image_1 = imread(string(ts.get_data_path()) + "grabcut/image1652.ppm", IMREAD_COLOR);
    Mat mask_1 = imread(string(ts.get_data_path()) + "grabcut/mask1652.ppm", IMREAD_GRAYSCALE);
    Rect roi_1(0, 0, 150, 150);

    Mat image_2 = image_1.clone();
    Mat mask_2 = mask_1.clone();
    Rect roi_2 = roi_1;

    Mat image_3 = image_1.clone();
    Mat mask_3 = mask_1.clone();
    Rect roi_3 = roi_1;

    Mat bgdModel_1, fgdModel_1;
    Mat bgdModel_2, fgdModel_2;
    Mat bgdModel_3, fgdModel_3;

    theRNG().state = 12378213;
    grabCut(image_1, mask_1, roi_1, bgdModel_1, fgdModel_1, 1, GC_INIT_WITH_MASK);
    theRNG().state = 12378213;
    grabCut(image_2, mask_2, roi_2, bgdModel_2, fgdModel_2, 1, GC_INIT_WITH_MASK);
    theRNG().state = 12378213;
    grabCut(image_3, mask_3, roi_3, bgdModel_3, fgdModel_3, 1, GC_INIT_WITH_MASK);

    EXPECT_EQ(0, countNonZero(mask_1 != mask_2));
    EXPECT_EQ(0, countNonZero(mask_1 != mask_3));
    EXPECT_EQ(0, countNonZero(mask_2 != mask_3));
}

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

- **CV_GrabcutTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
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

