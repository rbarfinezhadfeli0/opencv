# Documentation for `modules/gapi/src/api/kernels_video.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/kernels_video.cpp`
- **File Name**: `kernels_video.cpp`
- **File Size**: 5,314 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/kernels_video.cpp](../../../../modules/gapi/src/api/kernels_video.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation


#include "precomp.hpp"

#include <opencv2/gapi/video.hpp>

namespace cv { namespace gapi {
using namespace video;

GBuildPyrOutput buildOpticalFlowPyramid(const GMat    &img,
                                        const Size    &winSize,
                                        const GScalar &maxLevel,
                                              bool     withDerivatives,
                                              int      pyrBorder,
                                              int      derivBorder,
                                              bool     tryReuseInputImage)
{
    return GBuildOptFlowPyramid::on(img, winSize, maxLevel, withDerivatives, pyrBorder,
                                    derivBorder, tryReuseInputImage);
}

GOptFlowLKOutput calcOpticalFlowPyrLK(const GMat                    &prevImg,
                                      const GMat                    &nextImg,
                                      const cv::GArray<cv::Point2f> &prevPts,
                                      const cv::GArray<cv::Point2f> &predPts,
                                      const Size                    &winSize,
                                      const GScalar                 &maxLevel,
                                      const TermCriteria            &criteria,
                                            int                      flags,
                                            double                   minEigThresh)
{
    return GCalcOptFlowLK::on(prevImg, nextImg, prevPts, predPts, winSize, maxLevel,
                              criteria, flags, minEigThresh);
}

GOptFlowLKOutput calcOpticalFlowPyrLK(const cv::GArray<cv::GMat>    &prevPyr,
                                      const cv::GArray<cv::GMat>    &nextPyr,
                                      const cv::GArray<cv::Point2f> &prevPts,
                                      const cv::GArray<cv::Point2f> &predPts,
                                      const Size                    &winSize,
                                      const GScalar                 &maxLevel,
                                      const TermCriteria            &criteria,
                                            int                      flags,
                                            double                   minEigThresh)
{
    return GCalcOptFlowLKForPyr::on(prevPyr, nextPyr, prevPts, predPts, winSize, maxLevel,
                                    criteria, flags, minEigThresh);
}

GMat BackgroundSubtractor(const GMat& src, const BackgroundSubtractorParams& bsp)
{
    return GBackgroundSubtractor::on(src, bsp);
}

GMat KalmanFilter(const GMat& m, const cv::GOpaque<bool>& have_m, const GMat& c, const KalmanParams& kp)
{
    return GKalmanFilter::on(m, have_m, c, kp);
}

GMat KalmanFilter(const GMat& m, const cv::GOpaque<bool>& have_m, const KalmanParams& kp)
{
    return GKalmanFilterNoControl::on(m, have_m, kp);
}

namespace video {
void checkParams(const cv::gapi::KalmanParams& kfParams,
                 const cv::GMatDesc& measurement, const cv::GMatDesc& control)
{
    int type = kfParams.transitionMatrix.type();
    GAPI_Assert(type == CV_32FC1 || type == CV_64FC1);
    int depth = CV_MAT_DEPTH(type);

    bool controlCapable = !(control == GMatDesc{});

    if (controlCapable)
    {
        GAPI_Assert(!kfParams.controlMatrix.empty());
        GAPI_Assert(control.depth == depth && control.chan == 1 &&
                    control.size.height == kfParams.controlMatrix.cols &&
                    control.size.width == 1);
    }
    else
        GAPI_Assert(kfParams.controlMatrix.empty());

    GAPI_Assert(!kfParams.state.empty() && kfParams.state.type() == type);
    GAPI_Assert(!kfParams.errorCov.empty() && kfParams.errorCov.type() == type);
    GAPI_Assert(!kfParams.transitionMatrix.empty() && kfParams.transitionMatrix.type() == type);
    GAPI_Assert(!kfParams.processNoiseCov.empty() && kfParams.processNoiseCov.type() == type);
    GAPI_Assert(!kfParams.measurementNoiseCov.empty() && kfParams.measurementNoiseCov.type() == type);
    GAPI_Assert(!kfParams.measurementMatrix.empty() && kfParams.measurementMatrix.type() == type);
    GAPI_Assert(measurement.depth == depth && measurement.chan == 1);

    int dDim = kfParams.transitionMatrix.cols;
    GAPI_Assert(kfParams.transitionMatrix.rows == dDim);

    GAPI_Assert(kfParams.processNoiseCov.cols == dDim &&
                kfParams.processNoiseCov.rows == dDim);
    GAPI_Assert(kfParams.errorCov.cols == dDim && kfParams.errorCov.rows == dDim);
    GAPI_Assert(kfParams.state.rows == dDim && kfParams.state.cols == 1);
    GAPI_Assert(kfParams.measurementMatrix.cols == dDim);

    int mDim = kfParams.measurementMatrix.rows;
    GAPI_Assert(kfParams.measurementNoiseCov.cols == mDim &&
                kfParams.measurementNoiseCov.rows == mDim);

    if (controlCapable)
        GAPI_Assert(kfParams.controlMatrix.rows == dDim);

    GAPI_Assert(measurement.size.height == mDim &&
                measurement.size.width == 1);
}
}  // namespace video
} //namespace gapi
} //namespace cv
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
- `precomp.hpp`
- `opencv2/gapi/video.hpp`


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

