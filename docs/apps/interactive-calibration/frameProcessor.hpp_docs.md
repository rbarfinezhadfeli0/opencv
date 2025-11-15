# Documentation for `apps/interactive-calibration/frameProcessor.hpp`

## File Metadata

- **Full Path**: `apps/interactive-calibration/frameProcessor.hpp`
- **File Name**: `frameProcessor.hpp`
- **File Size**: 3,066 bytes
- **File Type**: .hpp
- **Link to Source**: [apps/interactive-calibration/frameProcessor.hpp](../../apps/interactive-calibration/frameProcessor.hpp)

## Purpose and Role

This file is located in the `apps/interactive-calibration` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef FRAME_PROCESSOR_HPP
#define FRAME_PROCESSOR_HPP

#include <opencv2/core.hpp>
#include <opencv2/calib3d.hpp>
#include <opencv2/objdetect.hpp>

#include "calibCommon.hpp"
#include "calibController.hpp"

namespace calib
{
class FrameProcessor
{
protected:

public:
    virtual ~FrameProcessor();
    virtual cv::Mat processFrame(const cv::Mat& frame) = 0;
    virtual bool isProcessed() const = 0;
    virtual void resetState() = 0;
};

class CalibProcessor : public FrameProcessor
{
protected:
    cv::Ptr<calibrationData> mCalibData;
    TemplateType mBoardType;
    cv::Size mBoardSizeUnits;
    cv::Size mBoardSizeInnerCorners;
    std::vector<cv::Point2f> mTemplateLocations;
    std::vector<cv::Point2f> mCurrentImagePoints;
    cv::Mat mCurrentCharucoCorners;
    cv::Mat mCurrentCharucoIds;

    cv::Ptr<cv::SimpleBlobDetector> mBlobDetectorPtr;
    cv::aruco::Dictionary mArucoDictionary;
    cv::Ptr<cv::aruco::CharucoBoard> mCharucoBoard;
    cv::Ptr<cv::aruco::CharucoDetector> detector;

    int mNeededFramesNum;
    unsigned mDelayBetweenCaptures;
    int mCapuredFrames;
    double mMaxTemplateOffset;
    float mSquareSize;
    float mTemplDist;
    bool mSaveFrames;
    float mZoom;

    bool detectAndParseChessboard(const cv::Mat& frame);
    bool detectAndParseChAruco(const cv::Mat& frame);
    bool detectAndParseCircles(const cv::Mat& frame);
    bool detectAndParseACircles(const cv::Mat& frame);
    bool detectAndParseDualACircles(const cv::Mat& frame);
    void saveFrameData();
    void showCaptureMessage(const cv::Mat &frame, const std::string& message);
    bool checkLastFrame();

public:
    CalibProcessor(cv::Ptr<calibrationData> data, captureParameters& capParams);
    virtual cv::Mat processFrame(const cv::Mat& frame) CV_OVERRIDE;
    virtual bool isProcessed() const CV_OVERRIDE;
    virtual void resetState() CV_OVERRIDE;
    ~CalibProcessor() CV_OVERRIDE;
};

enum visualisationMode {Grid, Window};

class ShowProcessor : public FrameProcessor
{
protected:
    cv::Ptr<calibrationData> mCalibdata;
    cv::Ptr<calibController> mController;
    TemplateType mBoardType;
    visualisationMode mVisMode;
    bool mNeedUndistort;
    double mGridViewScale;
    double mTextSize;

    void drawBoard(cv::Mat& img, cv::InputArray points);
    void drawGridPoints(const cv::Mat& frame);
public:
    ShowProcessor(cv::Ptr<calibrationData> data, cv::Ptr<calibController> controller, TemplateType board);
    virtual cv::Mat processFrame(const cv::Mat& frame) CV_OVERRIDE;
    virtual bool isProcessed() const CV_OVERRIDE;
    virtual void resetState() CV_OVERRIDE;

    void setVisualizationMode(visualisationMode mode);
    void switchVisualizationMode();
    void clearBoardsView();
    void updateBoardsView();

    void switchUndistort();
    void setUndistort(bool isEnabled);
    ~ShowProcessor() CV_OVERRIDE;
};

}


#endif
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

- **CalibProcessor**: A class/struct defined in this file
- **FrameProcessor**: A class/struct defined in this file
- **ShowProcessor**: A class/struct defined in this file

### Functions and Methods

- **FRAME_PROCESSOR_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/objdetect.hpp`
- `opencv2/core.hpp`
- `calibCommon.hpp`
- `opencv2/calib3d.hpp`
- `calibController.hpp`


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

