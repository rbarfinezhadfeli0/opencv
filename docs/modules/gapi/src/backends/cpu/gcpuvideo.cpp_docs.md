# Documentation for `modules/gapi/src/backends/cpu/gcpuvideo.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/cpu/gcpuvideo.cpp`
- **File Name**: `gcpuvideo.cpp`
- **File Size**: 8,137 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/cpu/gcpuvideo.cpp](../../../../../modules/gapi/src/backends/cpu/gcpuvideo.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/cpu` directory and serves as part of the OpenCV library infrastructure.

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
#include <opencv2/gapi/cpu/video.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>

#ifdef HAVE_OPENCV_VIDEO
#include <opencv2/video.hpp>
#endif // HAVE_OPENCV_VIDEO

#ifdef HAVE_OPENCV_VIDEO

GAPI_OCV_KERNEL(GCPUBuildOptFlowPyramid, cv::gapi::video::GBuildOptFlowPyramid)
{
    static void run(const cv::Mat              &img,
                    const cv::Size             &winSize,
                    const cv::Scalar           &maxLevel,
                          bool                  withDerivatives,
                          int                   pyrBorder,
                          int                   derivBorder,
                          bool                  tryReuseInputImage,
                          std::vector<cv::Mat> &outPyr,
                          cv::Scalar           &outMaxLevel)
    {
        outMaxLevel = cv::buildOpticalFlowPyramid(img, outPyr, winSize,
                                                  static_cast<int>(maxLevel[0]),
                                                  withDerivatives, pyrBorder,
                                                  derivBorder, tryReuseInputImage);
    }
};

GAPI_OCV_KERNEL(GCPUCalcOptFlowLK, cv::gapi::video::GCalcOptFlowLK)
{
    static void run(const cv::Mat                  &prevImg,
                    const cv::Mat                  &nextImg,
                    const std::vector<cv::Point2f> &prevPts,
                    const std::vector<cv::Point2f> &predPts,
                    const cv::Size                 &winSize,
                    const cv::Scalar               &maxLevel,
                    const cv::TermCriteria         &criteria,
                          int                       flags,
                          double                    minEigThresh,
                          std::vector<cv::Point2f> &outPts,
                          std::vector<uchar>       &status,
                          std::vector<float>       &err)
    {
        if (flags & cv::OPTFLOW_USE_INITIAL_FLOW)
            outPts = predPts;
        cv::calcOpticalFlowPyrLK(prevImg, nextImg, prevPts, outPts, status, err, winSize,
                                 static_cast<int>(maxLevel[0]), criteria, flags, minEigThresh);
    }
};

GAPI_OCV_KERNEL(GCPUCalcOptFlowLKForPyr, cv::gapi::video::GCalcOptFlowLKForPyr)
{
    static void run(const std::vector<cv::Mat>     &prevPyr,
                    const std::vector<cv::Mat>     &nextPyr,
                    const std::vector<cv::Point2f> &prevPts,
                    const std::vector<cv::Point2f> &predPts,
                    const cv::Size                 &winSize,
                    const cv::Scalar               &maxLevel,
                    const cv::TermCriteria         &criteria,
                          int                       flags,
                          double                    minEigThresh,
                          std::vector<cv::Point2f> &outPts,
                          std::vector<uchar>       &status,
                          std::vector<float>       &err)
    {
        if (flags & cv::OPTFLOW_USE_INITIAL_FLOW)
            outPts = predPts;
        cv::calcOpticalFlowPyrLK(prevPyr, nextPyr, prevPts, outPts, status, err, winSize,
                                 static_cast<int>(maxLevel[0]), criteria, flags, minEigThresh);
    }
};

GAPI_OCV_KERNEL_ST(GCPUBackgroundSubtractor,
                   cv::gapi::video::GBackgroundSubtractor,
                   cv::BackgroundSubtractor)
{
    static void setup(const cv::GMatDesc&, const cv::gapi::video::BackgroundSubtractorParams& bsParams,
                      std::shared_ptr<cv::BackgroundSubtractor>& state,
                      const cv::GCompileArgs&)
    {
        if (bsParams.operation == cv::gapi::video::TYPE_BS_MOG2)
            state = cv::createBackgroundSubtractorMOG2(bsParams.history,
                                                       bsParams.threshold,
                                                       bsParams.detectShadows);
        else if (bsParams.operation == cv::gapi::video::TYPE_BS_KNN)
            state = cv::createBackgroundSubtractorKNN(bsParams.history,
                                                      bsParams.threshold,
                                                      bsParams.detectShadows);

        GAPI_Assert(state);
    }

    static void run(const cv::Mat& in, const cv::gapi::video::BackgroundSubtractorParams& bsParams,
                    cv::Mat &out, cv::BackgroundSubtractor& state)
    {
        state.apply(in, out, bsParams.learningRate);
    }
};

GAPI_OCV_KERNEL_ST(GCPUKalmanFilter, cv::gapi::video::GKalmanFilter, cv::KalmanFilter)
{
    static void setup(const cv::GMatDesc&, const cv::GOpaqueDesc&,
                      const cv::GMatDesc&, const cv::gapi::KalmanParams& kfParams,
                      std::shared_ptr<cv::KalmanFilter> &state, const cv::GCompileArgs&)
    {
        state = std::make_shared<cv::KalmanFilter>(kfParams.transitionMatrix.rows, kfParams.measurementMatrix.rows,
                                                   kfParams.controlMatrix.cols, kfParams.transitionMatrix.type());

        // initial state
        kfParams.state.copyTo(state->statePost);
        kfParams.errorCov.copyTo(state->errorCovPost);

        // dynamic system initialization
        kfParams.controlMatrix.copyTo(state->controlMatrix);
        kfParams.measurementMatrix.copyTo(state->measurementMatrix);
        kfParams.transitionMatrix.copyTo(state->transitionMatrix);
        kfParams.processNoiseCov.copyTo(state->processNoiseCov);
        kfParams.measurementNoiseCov.copyTo(state->measurementNoiseCov);
    }

    static void run(const cv::Mat& measurements, bool haveMeasurement,
                    const cv::Mat& control, const cv::gapi::KalmanParams&,
                    cv::Mat &out, cv::KalmanFilter& state)
    {
        cv::Mat pre = state.predict(control);

        if (haveMeasurement)
            state.correct(measurements).copyTo(out);
        else
            pre.copyTo(out);
    }
};

GAPI_OCV_KERNEL_ST(GCPUKalmanFilterNoControl, cv::gapi::video::GKalmanFilterNoControl, cv::KalmanFilter)
{
    static void setup(const cv::GMatDesc&, const cv::GOpaqueDesc&,
                      const cv::gapi::KalmanParams& kfParams,
                      std::shared_ptr<cv::KalmanFilter> &state,
                      const cv::GCompileArgs&)
    {
        state = std::make_shared<cv::KalmanFilter>(kfParams.transitionMatrix.rows, kfParams.measurementMatrix.rows,
                                                   0, kfParams.transitionMatrix.type());
        // initial state
        kfParams.state.copyTo(state->statePost);
        kfParams.errorCov.copyTo(state->errorCovPost);

        // dynamic system initialization
        kfParams.measurementMatrix.copyTo(state->measurementMatrix);
        kfParams.transitionMatrix.copyTo(state->transitionMatrix);
        kfParams.processNoiseCov.copyTo(state->processNoiseCov);
        kfParams.measurementNoiseCov.copyTo(state->measurementNoiseCov);
    }

    static void run(const cv::Mat& measurements, bool haveMeasurement,
                    const cv::gapi::KalmanParams&, cv::Mat &out,
                    cv::KalmanFilter& state)
    {
        cv::Mat pre = state.predict();

        if (haveMeasurement)
            state.correct(measurements).copyTo(out);
        else
            pre.copyTo(out);
    }
};

cv::GKernelPackage cv::gapi::video::cpu::kernels()
{
    static auto pkg = cv::gapi::kernels
        < GCPUBuildOptFlowPyramid
        , GCPUCalcOptFlowLK
        , GCPUCalcOptFlowLKForPyr
        , GCPUBackgroundSubtractor
        , GCPUKalmanFilter
        , GCPUKalmanFilterNoControl
        >();
    return pkg;
}

#else

cv::GKernelPackage cv::gapi::video::cpu::kernels()
{
    return GKernelPackage();
}

#endif // HAVE_OPENCV_VIDEO
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

- **HAVE_OPENCV_VIDEO()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/video.hpp`
- `opencv2/gapi/cpu/video.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `opencv2/gapi/video.hpp`
- `precomp.hpp`


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

