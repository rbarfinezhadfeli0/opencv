# Documentation for `apps/interactive-calibration/calibPipeline.cpp`

## File Metadata

- **Full Path**: `apps/interactive-calibration/calibPipeline.cpp`
- **File Name**: `calibPipeline.cpp`
- **File Size**: 5,209 bytes
- **File Type**: .cpp
- **Link to Source**: [apps/interactive-calibration/calibPipeline.cpp](../../apps/interactive-calibration/calibPipeline.cpp)

## Purpose and Role

This file is located in the `apps/interactive-calibration` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "calibPipeline.hpp"

#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/core/utils/logger.hpp>

#include <stdexcept>

using namespace calib;

#define CAP_DELAY 10

cv::Size CalibPipeline::getCameraResolution()
{
    mCapture.set(cv::CAP_PROP_FRAME_WIDTH, 10000);
    mCapture.set(cv::CAP_PROP_FRAME_HEIGHT, 10000);
    int w = (int)mCapture.get(cv::CAP_PROP_FRAME_WIDTH);
    int h = (int)mCapture.get(cv::CAP_PROP_FRAME_HEIGHT);
    return cv::Size(w,h);
}

CalibPipeline::CalibPipeline(captureParameters params) :
    mCaptureParams(params)
{

}

PipelineExitStatus CalibPipeline::start(std::vector<cv::Ptr<FrameProcessor> > processors)
{
    const int allowedEmptyFrames = 5;
    int emptyFrames = 0;

    auto open_camera = [this] () {
        if(mCaptureParams.source == Camera)
        {
            mCapture.open(mCaptureParams.camID, mCaptureParams.camBackend);
            cv::Size maxRes = getCameraResolution();
            cv::Size neededRes = mCaptureParams.cameraResolution;

            if(maxRes.width < neededRes.width) {
                double aR = (double)maxRes.width / maxRes.height;
                mCapture.set(cv::CAP_PROP_FRAME_WIDTH, neededRes.width);
                mCapture.set(cv::CAP_PROP_FRAME_HEIGHT, neededRes.width/aR);
            }
            else if(maxRes.height < neededRes.height) {
                double aR = (double)maxRes.width / maxRes.height;
                mCapture.set(cv::CAP_PROP_FRAME_HEIGHT, neededRes.height);
                mCapture.set(cv::CAP_PROP_FRAME_WIDTH, neededRes.height*aR);
            }
            else {
                mCapture.set(cv::CAP_PROP_FRAME_HEIGHT, neededRes.height);
                mCapture.set(cv::CAP_PROP_FRAME_WIDTH, neededRes.width);
            }
            mCapture.set(cv::CAP_PROP_AUTOFOCUS, 0);
        }
        else if (mCaptureParams.source == File)
            mCapture.open(mCaptureParams.videoFileName, mCaptureParams.camBackend);
    };

    if(!mCapture.isOpened()) {
        open_camera();
    }
    mImageSize = cv::Size((int)mCapture.get(cv::CAP_PROP_FRAME_WIDTH), (int)mCapture.get(cv::CAP_PROP_FRAME_HEIGHT));

    if(!mCapture.isOpened())
        throw std::runtime_error("Unable to open video source");

    cv::Mat frame, processedFrame, resizedFrame;
    while (true) {
        if (!mCapture.grab())
        {
            if (!mCaptureParams.forceReopen)
            {
                CV_LOG_ERROR(NULL, "VideoCapture error: could not grab the frame.");
                break;
            }

            CV_LOG_INFO(NULL, "VideoCapture error: trying to reopen...");
            do
            {
                open_camera();
            } while (!mCapture.isOpened() || !mCapture.grab());

            CV_LOG_INFO(NULL, "VideoCapture error: reopened successfully.");
            auto newSize = cv::Size((int)mCapture.get(cv::CAP_PROP_FRAME_WIDTH), (int)mCapture.get(cv::CAP_PROP_FRAME_HEIGHT));
            CV_CheckEQ(mImageSize, newSize, "Camera image size changed after reopening.");
        }
        mCapture.retrieve(frame);

        if (frame.empty()) {
            emptyFrames++;
            if (emptyFrames >= allowedEmptyFrames) {
                CV_LOG_ERROR(NULL, "VideoCapture error: grabbed sequence of empty frames. VideoCapture is not ready or broken.");
                return Finished;
            }

            continue;
        } else {
            emptyFrames = 0;
            if (mImageSize.width == 0 || mImageSize.height == 0) { // looks like VideoCapture does not support required properties
                mImageSize = frame.size();
            }
        }

        if(mCaptureParams.flipVertical)
            cv::flip(frame, frame, -1);

        frame.copyTo(processedFrame);
        for (std::vector<cv::Ptr<FrameProcessor> >::iterator it = processors.begin(); it != processors.end(); ++it)
            processedFrame = (*it)->processFrame(processedFrame);
        if (std::fabs(mCaptureParams.zoom - 1.) > 0.001f)
        {
            cv::resize(processedFrame, resizedFrame, cv::Size(), mCaptureParams.zoom, mCaptureParams.zoom);
        }
        else
        {
            resizedFrame = std::move(processedFrame);
        }
        cv::imshow(mainWindowName, resizedFrame);
        char key = (char)cv::waitKey(CAP_DELAY);

        if(key == 27) // esc
            return Finished;
        else if (key == 114) // r
            return DeleteLastFrame;
        else if (key == 100) // d
            return DeleteAllFrames;
        else if (key == 115) // s
            return SaveCurrentData;
        else if (key == 117) // u
            return SwitchUndistort;
        else if (key == 118) // v
            return SwitchVisualisation;

        for (std::vector<cv::Ptr<FrameProcessor> >::iterator it = processors.begin(); it != processors.end(); ++it)
            if((*it)->isProcessed())
                return Calibrate;
    }

    return Finished;
}

cv::Size CalibPipeline::getImageSize() const
{
    return mImageSize;
}
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
- `opencv2/core/utils/logger.hpp`
- `calibPipeline.hpp`
- `opencv2/imgproc.hpp`
- `stdexcept`
- `opencv2/highgui.hpp`


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

