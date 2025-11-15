# Documentation for `modules/videoio/src/cap_obsensor_capture.cpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_obsensor_capture.cpp`
- **File Name**: `cap_obsensor_capture.cpp`
- **File Size**: 11,223 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/videoio/src/cap_obsensor_capture.cpp](../../../modules/videoio/src/cap_obsensor_capture.cpp)

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

#include "precomp.hpp"

#include "cap_obsensor_capture.hpp"
#include "cap_obsensor/obsensor_stream_channel_interface.hpp"
#include <cstdint>

#define OB_WIDTH_ANY 0
#define OB_HEIGHT_ANY 0
#define OB_FPS_ANY 0

#if defined(HAVE_OBSENSOR) && !defined(HAVE_OBSENSOR_ORBBEC_SDK)
namespace cv {
Ptr<IVideoCapture> create_obsensor_capture(int index, const cv::VideoCaptureParameters& params)
{
    return makePtr<VideoCapture_obsensor>(index, params);
}

VideoCapture_obsensor::VideoCapture_obsensor(int index, const cv::VideoCaptureParameters& params) : isOpened_(false)
{
    static const obsensor::StreamProfile colorProfile = { 640, 480, 30, obsensor::FRAME_FORMAT_MJPG };
    static const obsensor::StreamProfile depthProfile = { 640, 480, 30, obsensor::FRAME_FORMAT_Y16 };
    static const obsensor::StreamProfile gemini2DepthProfile = { 1280, 800, 30, obsensor::FRAME_FORMAT_Y16 };
    static const obsensor::StreamProfile astra2ColorProfile = { 800, 600, 30, obsensor::FRAME_FORMAT_MJPG };
    static const obsensor::StreamProfile astra2DepthProfile = { 800, 600, 30, obsensor::FRAME_FORMAT_Y14 };
    static const obsensor::StreamProfile megaColorProfile = { 1280, 720, 30, obsensor::FRAME_FORMAT_MJPG };
    static const obsensor::StreamProfile megaDepthProfile = { 640, 576, 30, obsensor::FRAME_FORMAT_Y16 };
    static const obsensor::StreamProfile gemini2lColorProfile = { 1280, 720, 30, obsensor::FRAME_FORMAT_MJPG };
    static const obsensor::StreamProfile gemini2lDepthProfile = { 1280, 800, 30, obsensor::FRAME_FORMAT_Y16 };
    static const obsensor::StreamProfile gemini2XlColorProfile = { 1280, 800, 10, obsensor::FRAME_FORMAT_MJPG };
    static const obsensor::StreamProfile gemini2XlDepthProfile = { 1280, 800, 10, obsensor::FRAME_FORMAT_Y16 };

    streamChannelGroup_ = obsensor::getStreamChannelGroup(index);
    if (!streamChannelGroup_.empty())
    {
        for (auto& channel : streamChannelGroup_)
        {
            auto streamType = channel->streamType();
            switch (streamType)
            {
            case obsensor::OBSENSOR_STREAM_COLOR:
            {
                uint32_t color_width = params.get<uint32_t>(CAP_PROP_FRAME_WIDTH, OB_WIDTH_ANY);
                uint32_t color_height = params.get<uint32_t>(CAP_PROP_FRAME_HEIGHT, OB_HEIGHT_ANY);
                uint32_t color_fps = params.get<uint32_t>(CAP_PROP_FPS, OB_FPS_ANY);

                obsensor::StreamProfile profile = colorProfile;
                if (color_width != OB_WIDTH_ANY || color_height != OB_HEIGHT_ANY || color_fps != OB_FPS_ANY) {
                    profile = { color_width, color_height, color_fps, obsensor::FRAME_FORMAT_MJPG };
                } else {
                    if(OBSENSOR_FEMTO_MEGA_PID == channel->getPid()){
                        profile = megaColorProfile;
                    }else if(OBSENSOR_GEMINI2L_PID == channel->getPid()){
                        profile = gemini2lColorProfile;
                    }else if(OBSENSOR_ASTRA2_PID == channel->getPid()){
                        profile = astra2ColorProfile;
                    }else if(OBSENSOR_GEMINI2XL_PID == channel->getPid()){
                        profile = gemini2XlColorProfile;
                    }
                }
                channel->start(profile, [&](obsensor::Frame* frame) {
                    std::unique_lock<std::mutex> lk(frameMutex_);
                    colorFrame_ = Mat(1, frame->dataSize, CV_8UC1, frame->data).clone();
                    frameCv_.notify_all();
                });
            }
                break;
            case obsensor::OBSENSOR_STREAM_DEPTH:
            {
                uint8_t data = 1;
                channel->setProperty(obsensor::DEPTH_TO_COLOR_ALIGN, &data, 1);

                uint32_t depth_width = params.get<uint32_t>(CAP_PROP_OBSENSOR_DEPTH_WIDTH, OB_WIDTH_ANY);
                uint32_t depth_height = params.get<uint32_t>(CAP_PROP_OBSENSOR_DEPTH_HEIGHT, OB_HEIGHT_ANY);
                uint32_t depth_fps = params.get<uint32_t>(CAP_PROP_OBSENSOR_DEPTH_FPS, OB_FPS_ANY);

                obsensor::StreamProfile profile = depthProfile;
                if (depth_width != OB_WIDTH_ANY || depth_height != OB_HEIGHT_ANY || depth_fps != OB_FPS_ANY) {
                    profile = { depth_width, depth_height, depth_fps, obsensor::FRAME_FORMAT_Y16 };
                } else {
                    if(OBSENSOR_GEMINI2_PID == channel->getPid()){
                        profile = gemini2DepthProfile;
                    }else if(OBSENSOR_ASTRA2_PID == channel->getPid()){
                        profile = astra2DepthProfile;
                    }else if(OBSENSOR_FEMTO_MEGA_PID == channel->getPid()){
                        profile = megaDepthProfile;
                    }else if(OBSENSOR_GEMINI2L_PID == channel->getPid()){
                        profile = gemini2lDepthProfile;
                    }else if(OBSENSOR_GEMINI2XL_PID == channel->getPid()){
                        profile = gemini2XlDepthProfile;
                    }
                }
                channel->start(profile, [&](obsensor::Frame* frame) {
                    std::unique_lock<std::mutex> lk(frameMutex_);
                    depthFrame_ = Mat(frame->height, frame->width, CV_16UC1, frame->data, frame->width * 2).clone();
                    frameCv_.notify_all();
                });

                uint32_t len;
                memset(&camParam_, 0, sizeof(camParam_));
                channel->getProperty(obsensor::CAMERA_PARAM, (uint8_t*)&camParam_, &len);
                camParamScale_ = (int)(camParam_.p1[2] * 2 / 640 + 0.5);
            }
                break;
            default:
                break;
            }
        }
        isOpened_ = true;
    }
}

VideoCapture_obsensor::~VideoCapture_obsensor(){
    for (auto& channel : streamChannelGroup_)
    {
        channel->stop();
    }
    streamChannelGroup_.clear();
}

bool VideoCapture_obsensor::grabFrame()
{
    std::unique_lock<std::mutex> lk(frameMutex_);

    // Try waiting for 33 milliseconds to ensure that both depth and color frame have been received!
    frameCv_.wait_for(lk, std::chrono::milliseconds(33), [&](){ return !depthFrame_.empty() && !colorFrame_.empty(); });

    grabbedDepthFrame_ = depthFrame_;
    grabbedColorFrame_ = colorFrame_;

    depthFrame_.release();
    colorFrame_.release();

    return !grabbedDepthFrame_.empty() || !grabbedColorFrame_.empty();
}

bool VideoCapture_obsensor::retrieveFrame(int outputType, OutputArray frame)
{
    std::unique_lock<std::mutex> lk(frameMutex_);
    switch (outputType)
    {
    case CAP_OBSENSOR_DEPTH_MAP:
        if (!grabbedDepthFrame_.empty())
        {
            if(OBSENSOR_GEMINI2_PID == streamChannelGroup_.front()->getPid()){
                const double DepthValueScaleGemini2 = 0.2;
                grabbedDepthFrame_ = grabbedDepthFrame_*DepthValueScaleGemini2;
                Rect rect(320, 160, 640, 480);
                grabbedDepthFrame_(rect).copyTo(frame);
            }
            else if(OBSENSOR_ASTRA2_PID == streamChannelGroup_.front()->getPid()){
                const double DepthValueScaleAstra2 = 0.8;
                grabbedDepthFrame_ = grabbedDepthFrame_*DepthValueScaleAstra2;
                grabbedDepthFrame_.copyTo(frame);
            }
            else if(OBSENSOR_FEMTO_MEGA_PID == streamChannelGroup_.front()->getPid()){
                Rect rect(0, 0, 640, 360);
                grabbedDepthFrame_(rect).copyTo(frame);
            }else if(OBSENSOR_GEMINI2L_PID == streamChannelGroup_.front()->getPid()){
                const double DepthValueScaleGemini2L = 0.2;
                grabbedDepthFrame_ = grabbedDepthFrame_*DepthValueScaleGemini2L;
                Rect rect(0, 40, 1280, 720);
                grabbedDepthFrame_(rect).copyTo(frame);
            }else if(OBSENSOR_GEMINI2XL_PID == streamChannelGroup_.front()->getPid()){
                grabbedDepthFrame_.copyTo(frame);
            }else if(IS_OBSENSOR_GEMINI330_PID(streamChannelGroup_.front()->getPid())){
                const double DepthValueScaleG300 = 1.0;
                grabbedDepthFrame_ = grabbedDepthFrame_*DepthValueScaleG300;
                Rect rect(0, 0, 640, 480);
                grabbedDepthFrame_(rect).copyTo(frame);
            }else{
                grabbedDepthFrame_.copyTo(frame);
            }
            grabbedDepthFrame_.release();
            return true;
        }
        break;
    case CAP_OBSENSOR_BGR_IMAGE:
        if (!grabbedColorFrame_.empty())
        {
            auto mat = imdecode(grabbedColorFrame_, IMREAD_COLOR);
            grabbedColorFrame_.release();

            if (!mat.empty())
            {
                mat.copyTo(frame);
                return true;
            }
        }
        break;
    default:
        break;
    }

    return false;
}

double VideoCapture_obsensor::getProperty(int propIdx) const {
    double rst = 0.0;
    propIdx = propIdx & (~CAP_OBSENSOR_GENERATORS_MASK);
    // int gen = propIdx & CAP_OBSENSOR_GENERATORS_MASK;
    switch (propIdx)
    {
    case CAP_PROP_OBSENSOR_INTRINSIC_FX:
        rst = camParam_.p1[0] / camParamScale_;
        break;
    case CAP_PROP_OBSENSOR_INTRINSIC_FY:
        rst = camParam_.p1[1] / camParamScale_;
        break;
    case CAP_PROP_OBSENSOR_INTRINSIC_CX:
        rst = camParam_.p1[2] / camParamScale_;
        break;
    case CAP_PROP_OBSENSOR_INTRINSIC_CY:
        rst = camParam_.p1[3] / camParamScale_;
        break;
    }
    return rst;
}

bool VideoCapture_obsensor::setProperty(int propIdx, double /*propVal*/)
{
    switch(propIdx)
    {
        case CAP_PROP_OBSENSOR_DEPTH_WIDTH:
        case CAP_PROP_OBSENSOR_DEPTH_HEIGHT:
        case CAP_PROP_OBSENSOR_DEPTH_FPS:
            CV_LOG_WARNING(NULL, "CAP_PROP_OBSENSOR_DEPTH_WIDTH, CAP_PROP_OBSENSOR_DEPTH_HEIGHT, CAP_PROP_OBSENSOR_DEPTH_FPS options are supported during camera initialization only");
            break;
        case CAP_PROP_FRAME_WIDTH:
        case CAP_PROP_FRAME_HEIGHT:
        case CAP_PROP_FPS:
            CV_LOG_WARNING(NULL, "CAP_PROP_FRAME_WIDTH, CAP_PROP_FRAME_HEIGHT, CAP_PROP_FPS options are supported during camera initialization only");
            break;
        default:
            CV_LOG_WARNING(NULL, "Unsupported or read only property, id=" << propIdx);
    }

    return false;
}

} // namespace cv::
#endif // HAVE_OBSENSOR
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
- `cstdint`
- `cap_obsensor_capture.hpp`
- `precomp.hpp`
- `cap_obsensor/obsensor_stream_channel_interface.hpp`


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

