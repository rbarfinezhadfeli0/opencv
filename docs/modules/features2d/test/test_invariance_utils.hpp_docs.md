# Documentation for `modules/features2d/test/test_invariance_utils.hpp`

## File Metadata

- **Full Path**: `modules/features2d/test/test_invariance_utils.hpp`
- **File Name**: `test_invariance_utils.hpp`
- **File Size**: 2,887 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/features2d/test/test_invariance_utils.hpp](../../../modules/features2d/test/test_invariance_utils.hpp)

## Purpose and Role

This file is located in the `modules/features2d/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#ifndef __OPENCV_TEST_INVARIANCE_UTILS_HPP__
#define __OPENCV_TEST_INVARIANCE_UTILS_HPP__

namespace opencv_test { namespace {

Mat generateHomography(float angle)
{
    // angle - rotation around Oz in degrees
    float angleRadian = static_cast<float>(angle * CV_PI / 180);
    Mat H = Mat::eye(3, 3, CV_32FC1);
    H.at<float>(0,0) = H.at<float>(1,1) = std::cos(angleRadian);
    H.at<float>(0,1) = -std::sin(angleRadian);
    H.at<float>(1,0) =  std::sin(angleRadian);

    return H;
}

Mat rotateImage(const Mat& srcImage, const Mat& srcMask, float angle, Mat& dstImage, Mat& dstMask)
{
    // angle - rotation around Oz in degrees
    float diag = std::sqrt(static_cast<float>(srcImage.cols * srcImage.cols + srcImage.rows * srcImage.rows));
    Mat LUShift = Mat::eye(3, 3, CV_32FC1); // left up
    LUShift.at<float>(0,2) = static_cast<float>(-srcImage.cols/2);
    LUShift.at<float>(1,2) = static_cast<float>(-srcImage.rows/2);
    Mat RDShift = Mat::eye(3, 3, CV_32FC1); // right down
    RDShift.at<float>(0,2) = diag/2;
    RDShift.at<float>(1,2) = diag/2;
    Size sz(cvRound(diag), cvRound(diag));

    Mat H = RDShift * generateHomography(angle) * LUShift;
    warpPerspective(srcImage, dstImage, H, sz);
    warpPerspective(srcMask, dstMask, H, sz);

    return H;
}

float calcCirclesIntersectArea(const Point2f& p0, float r0, const Point2f& p1, float r1)
{
    float c = static_cast<float>(cv::norm(p0 - p1)), sqr_c = c * c;

    float sqr_r0 = r0 * r0;
    float sqr_r1 = r1 * r1;

    if(r0 + r1 <= c)
       return 0;

    float minR = std::min(r0, r1);
    float maxR = std::max(r0, r1);
    if(c + minR <= maxR)
        return static_cast<float>(CV_PI * minR * minR);

    float cos_halfA0 = (sqr_r0 + sqr_c - sqr_r1) / (2 * r0 * c);
    float cos_halfA1 = (sqr_r1 + sqr_c - sqr_r0) / (2 * r1 * c);

    float A0 = 2 * acos(cos_halfA0);
    float A1 = 2 * acos(cos_halfA1);

    return  0.5f * sqr_r0 * (A0 - sin(A0)) +
            0.5f * sqr_r1 * (A1 - sin(A1));
}

float calcIntersectRatio(const Point2f& p0, float r0, const Point2f& p1, float r1)
{
    float intersectArea = calcCirclesIntersectArea(p0, r0, p1, r1);
    float unionArea = static_cast<float>(CV_PI) * (r0 * r0 + r1 * r1) - intersectArea;
    return intersectArea / unionArea;
}

void scaleKeyPoints(const vector<KeyPoint>& src, vector<KeyPoint>& dst, float scale)
{
    dst.resize(src.size());
    for (size_t i = 0; i < src.size(); i++) {
        dst[i] = src[i];
        dst[i].pt.x = dst[i].pt.x * scale + (scale - 1.0f) / 2.0f;
        dst[i].pt.y = dst[i].pt.y * scale + (scale - 1.0f) / 2.0f;
        dst[i].size *= scale;
    }
}

}} // namespace
#endif // __OPENCV_TEST_INVARIANCE_UTILS_HPP__
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

### Functions and Methods

- **__OPENCV_TEST_INVARIANCE_UTILS_HPP__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

