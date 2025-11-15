# Documentation for `modules/imgproc/test/ocl/test_gftt.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/test/ocl/test_gftt.cpp`
- **File Name**: `test_gftt.cpp`
- **File Size**: 5,284 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/test/ocl/test_gftt.cpp](../../../../modules/imgproc/test/ocl/test_gftt.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/test/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////////////////
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
// Copyright (C) 2010-2012, Multicoreware, Inc., all rights reserved.
// Copyright (C) 2010-2012, Institute Of Software Chinese Academy Of Science, all rights reserved.
// Copyright (C) 2010-2012, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
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

#include "../test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

//////////////////////////// GoodFeaturesToTrack //////////////////////////


PARAM_TEST_CASE(GoodFeaturesToTrack, double, bool)
{
    double minDistance;
    bool useRoi;

    static const int maxCorners;
    static const double qualityLevel;

    TEST_DECLARE_INPUT_PARAMETER(src);
    UMat points, upoints;
    std::vector<float> quality, uquality;

    virtual void SetUp()
    {
        minDistance = GET_PARAM(0);
        useRoi = GET_PARAM(1);
    }

    void generateTestData()
    {
        Mat frame = readImage("../gpu/opticalflow/rubberwhale1.png", IMREAD_GRAYSCALE);
        ASSERT_FALSE(frame.empty()) << "could not load gpu/opticalflow/rubberwhale1.png";

        Size roiSize = frame.size();
        Border srcBorder = randomBorder(0, useRoi ? 2 : 0);
        randomSubMat(src, src_roi, roiSize, srcBorder, frame.type(), 5, 256);
        src_roi.copyTo(frame);

        UMAT_UPLOAD_INPUT_PARAMETER(src);
    }

    void UMatToVector(const UMat & um, std::vector<Point2f> & v) const
    {
        v.resize(um.size().area());
        um.copyTo(Mat(um.size(), CV_32FC2, &v[0]));
    }
};

const int GoodFeaturesToTrack::maxCorners = 1000;
const double GoodFeaturesToTrack::qualityLevel = 0.01;

OCL_TEST_P(GoodFeaturesToTrack, Accuracy)
{
    for (int j = 0; j < test_loop_times; ++j)
    {
        generateTestData();

        std::vector<Point2f> upts, pts;

        OCL_OFF(cv::goodFeaturesToTrack(src_roi, points, maxCorners, qualityLevel, minDistance, noArray(), quality));
        ASSERT_FALSE(points.empty());
        UMatToVector(points, pts);

        OCL_ON(cv::goodFeaturesToTrack(usrc_roi, upoints, maxCorners, qualityLevel, minDistance, noArray(), uquality));
        ASSERT_FALSE(upoints.empty());
        UMatToVector(upoints, upts);

        ASSERT_EQ(pts.size(), quality.size());
        ASSERT_EQ(upts.size(), uquality.size());
        ASSERT_EQ(upts.size(), pts.size());

        int mistmatch = 0;
        for (size_t i = 0; i < pts.size(); ++i)
        {
            Point2i a = upts[i], b = pts[i];

            bool eq = std::abs(a.x - b.x) < 1 && std::abs(a.y - b.y) < 1 &&
                    std::abs(quality[i] - uquality[i]) <= 3.f * FLT_EPSILON * std::max(quality[i], uquality[i]);

            if (!eq)
                ++mistmatch;
        }

        double bad_ratio = static_cast<double>(mistmatch) / pts.size();
        ASSERT_GE(1e-2, bad_ratio);
    }
}

OCL_TEST_P(GoodFeaturesToTrack, EmptyCorners)
{
    generateTestData();
    usrc_roi.setTo(Scalar::all(0));

    OCL_ON(cv::goodFeaturesToTrack(usrc_roi, upoints, maxCorners, qualityLevel, minDistance, noArray(), uquality));

    ASSERT_TRUE(upoints.empty());
    ASSERT_TRUE(uquality.empty());
}

OCL_INSTANTIATE_TEST_CASE_P(Imgproc, GoodFeaturesToTrack,
                            ::testing::Combine(testing::Values(0.0, 3.0), Bool()));

} } // namespace opencv_test::ocl

#endif
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

### Functions and Methods

- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../test_precomp.hpp`
- `opencv2/ts/ocl_test.hpp`

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

