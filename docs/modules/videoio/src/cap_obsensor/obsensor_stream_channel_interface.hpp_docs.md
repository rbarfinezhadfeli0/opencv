# Documentation for `modules/videoio/src/cap_obsensor/obsensor_stream_channel_interface.hpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_obsensor/obsensor_stream_channel_interface.hpp`
- **File Name**: `obsensor_stream_channel_interface.hpp`
- **File Size**: 4,007 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/videoio/src/cap_obsensor/obsensor_stream_channel_interface.hpp](../../../../modules/videoio/src/cap_obsensor/obsensor_stream_channel_interface.hpp)

## Purpose and Role

This file is located in the `modules/videoio/src/cap_obsensor` directory and serves as part of the OpenCV library infrastructure.

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

#ifndef OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_INTERFACE_HPP
#define OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_INTERFACE_HPP

#ifdef HAVE_OBSENSOR

#include "../precomp.hpp" // #include "precomp.hpp : compile error on linux

#include <functional>
#include <vector>
#include <memory>

namespace cv {
namespace obsensor {

#define OBSENSOR_CAM_VID 0x2bc5 // usb vid
#define OBSENSOR_ASTRA2_PID 0x0660 // pid of Orbbec Astra 2 Camera
#define OBSENSOR_GEMINI2_PID 0x0670 // pid of Orbbec Gemini 2 Camera
#define OBSENSOR_FEMTO_MEGA_PID 0x0669 // pid of Orbbec Femto Mega Camera
#define OBSENSOR_GEMINI2L_PID 0x0673 // pid of Orbbec Gemini 2 L Camera
#define OBSENSOR_GEMINI2XL_PID 0x0671 // pid of Orbbec Gemini 2 XL Camera
#define OBSENSOR_GEMINI335_PID 0x0800 // pid of Orbbec Gemini 335 Camera
#define OBSENSOR_GEMINI330_PID 0x0801 // pid of Orbbec Gemini 330 Camera
#define OBSENSOR_GEMINI336_PID 0x0803 // pid of Orbbec Gemini 336 Camera
#define OBSENSOR_GEMINI335L_PID 0x0804 // pid of Orbbec Gemini 335L Camera
#define OBSENSOR_GEMINI330L_PID 0x0805 // pid of Orbbec Gemini 330L Camera
#define OBSENSOR_GEMINI336L_PID 0x0807 // pid of Orbbec Gemini 336L Camera

#define IS_OBSENSOR_GEMINI330_SHORT_PID(pid) \
    ((pid) == OBSENSOR_GEMINI335_PID || (pid) == OBSENSOR_GEMINI330_PID || (pid) == OBSENSOR_GEMINI336_PID)

#define IS_OBSENSOR_GEMINI330_LONG_PID(pid) \
    ((pid) == OBSENSOR_GEMINI335L_PID || (pid) == OBSENSOR_GEMINI330L_PID || (pid) == OBSENSOR_GEMINI336L_PID)

#define IS_OBSENSOR_GEMINI330_PID(pid) \
    (IS_OBSENSOR_GEMINI330_SHORT_PID(pid) || IS_OBSENSOR_GEMINI330_LONG_PID(pid))

enum StreamType
{
    OBSENSOR_STREAM_IR = 1,
    OBSENSOR_STREAM_COLOR = 2,
    OBSENSOR_STREAM_DEPTH = 3,
};

enum FrameFormat
{
    FRAME_FORMAT_UNKNOWN = -1,
    FRAME_FORMAT_YUYV = 0,
    FRAME_FORMAT_MJPG = 5,
    FRAME_FORMAT_Y16 = 8,
    FRAME_FORMAT_Y14 = 9,
};

enum PropertyId
{
    DEPTH_TO_COLOR_ALIGN = 42,
    CAMERA_PARAM = 1001,
};

struct Frame
{
    FrameFormat format;
    uint32_t width;
    uint32_t height;
    uint32_t dataSize;
    uint8_t* data;
};

struct StreamProfile
{
    uint32_t width;
    uint32_t height;
    uint32_t fps;
    FrameFormat format;
};

struct CameraParam
{
    float    p0[4];
    float    p1[4];
    float    p2[9];
    float    p3[3];
    float    p4[5];
    float    p5[5];
    uint32_t p6[2];
    uint32_t p7[2];
};

typedef std::function<void(Frame*)> FrameCallback;
class IStreamChannel
{
public:
    virtual ~IStreamChannel() noexcept {}
    virtual void start(const StreamProfile& profile, FrameCallback frameCallback) = 0;
    virtual void stop() = 0;
    virtual bool setProperty(int propId, const uint8_t* data, uint32_t dataSize) = 0;
    virtual bool getProperty(int propId, uint8_t* recvData, uint32_t* recvDataSize) = 0;

    virtual StreamType streamType() const = 0;
    virtual uint16_t getPid() const =0;
};

// "StreamChannelGroup" mean a group of stream channels from same one physical device
std::vector<Ptr<IStreamChannel>> getStreamChannelGroup(uint32_t groupIdx = 0);

}} // namespace cv::obsensor::
#endif // HAVE_OBSENSOR
#endif // OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_INTERFACE_HPP
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

- **StreamProfile**: A class/struct defined in this file
- **IStreamChannel**: A class/struct defined in this file
- **Frame**: A class/struct defined in this file
- **CameraParam**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **HAVE_OBSENSOR()**: A function/method defined in this file
- **OPENCV_VIDEOIO_OBSENSOR_STREAM_CHANNEL_INTERFACE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp : compile error on linux

#include <functional`
- `memory`
- `../precomp.hpp`
- `vector`

**Python Imports:**
- `same`


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

