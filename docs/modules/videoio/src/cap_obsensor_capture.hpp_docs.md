# Documentation for `modules/videoio/src/cap_obsensor_capture.hpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_obsensor_capture.hpp`
- **File Name**: `cap_obsensor_capture.hpp`
- **File Size**: 2,172 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/videoio/src/cap_obsensor_capture.hpp](../../../modules/videoio/src/cap_obsensor_capture.hpp)

## Purpose and Role

This file is located in the `modules/videoio/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

/*
* Copyright(C) 2022 by ORBBEC Technology., Inc.
* Authors:
*   Huang Zhenchang <yufeng@orbbec.com>
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*     http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/

#ifndef OPENCV_VIDEOIO_CAP_OBSENSOR_CAPTURE_HPP
#define OPENCV_VIDEOIO_CAP_OBSENSOR_CAPTURE_HPP

#include <map>
#include <mutex>
#include <condition_variable>

#include "cap_obsensor/obsensor_stream_channel_interface.hpp"

#if defined(HAVE_OBSENSOR) && !defined(HAVE_OBSENSOR_ORBBEC_SDK)

namespace cv {
class VideoCapture_obsensor : public IVideoCapture
{
public:
    VideoCapture_obsensor(int index, const cv::VideoCaptureParameters& params);
    virtual ~VideoCapture_obsensor();

    virtual double getProperty(int propIdx) const CV_OVERRIDE;
    virtual bool setProperty(int propIdx, double propVal) CV_OVERRIDE;
    virtual bool grabFrame() CV_OVERRIDE;
    virtual bool retrieveFrame(int outputType, OutputArray frame) CV_OVERRIDE;
    virtual int getCaptureDomain() CV_OVERRIDE {
        return CAP_OBSENSOR;
    }
    virtual bool isOpened() const CV_OVERRIDE {
        return isOpened_;
    }

private:
    bool isOpened_;
    std::vector<Ptr<obsensor::IStreamChannel>> streamChannelGroup_;

    std::mutex frameMutex_;
    std::condition_variable frameCv_;

    Mat depthFrame_;
    Mat colorFrame_;

    Mat grabbedDepthFrame_;
    Mat grabbedColorFrame_;

    obsensor::CameraParam camParam_;
    int camParamScale_;
};
} // namespace cv::
#endif // HAVE_OBSENSOR
#endif // OPENCV_VIDEOIO_CAP_OBSENSOR_CAPTURE_HPP
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

- **VideoCapture_obsensor**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_VIDEOIO_CAP_OBSENSOR_CAPTURE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `mutex`
- `condition_variable`
- `cap_obsensor/obsensor_stream_channel_interface.hpp`
- `map`


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

