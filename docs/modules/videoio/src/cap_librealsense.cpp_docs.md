# Documentation for `modules/videoio/src/cap_librealsense.cpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_librealsense.cpp`
- **File Name**: `cap_librealsense.cpp`
- **File Size**: 6,841 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/videoio/src/cap_librealsense.cpp](../../../modules/videoio/src/cap_librealsense.cpp)

## Purpose and Role

This file is located in the `modules/videoio/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#ifdef HAVE_LIBREALSENSE
#include "cap_librealsense.hpp"

namespace cv
{

VideoCapture_LibRealsense::VideoCapture_LibRealsense(int) : mAlign(RS2_STREAM_COLOR)
{
    try
    {
        rs2::config config;
        // Configure all streams to run at VGA resolution at default fps
        config.enable_stream(RS2_STREAM_DEPTH, 640, 480, RS2_FORMAT_Z16);
        config.enable_stream(RS2_STREAM_COLOR, 640, 480, RS2_FORMAT_BGR8);
        config.enable_stream(RS2_STREAM_INFRARED, 640, 480, RS2_FORMAT_Y8);
        mPipe.start(config);
    }
    catch (const rs2::error&)
    {
    }
}
VideoCapture_LibRealsense::~VideoCapture_LibRealsense(){}

double VideoCapture_LibRealsense::getProperty(int propIdx) const
{
    double propValue = 0.0;

    const int purePropIdx = propIdx & ~CAP_INTELPERC_GENERATORS_MASK;
    if((propIdx & CAP_INTELPERC_GENERATORS_MASK) == CAP_INTELPERC_IMAGE_GENERATOR)
    {
        propValue = getImageGeneratorProperty(purePropIdx);
    }
    else if((propIdx & CAP_INTELPERC_GENERATORS_MASK) == CAP_INTELPERC_DEPTH_GENERATOR)
    {
        propValue = getDepthGeneratorProperty(purePropIdx);
    }
    else if((propIdx & CAP_INTELPERC_GENERATORS_MASK) == CAP_INTELPERC_IR_GENERATOR)
    {
        propValue = getIrGeneratorProperty(purePropIdx);
    }
    else
    {
        propValue = getCommonProperty(purePropIdx);
    }

    return propValue;
}

double VideoCapture_LibRealsense::getImageGeneratorProperty(int propIdx) const
{
    double propValue = 0.0;
    const rs2::video_stream_profile profile = mPipe.get_active_profile().get_stream(RS2_STREAM_COLOR).as<rs2::video_stream_profile>();
    if(!profile)
    {
        return propValue;
    }

    switch(propIdx)
    {
    case CAP_PROP_FRAME_WIDTH:
        propValue = static_cast<double>(profile.width());
        break;
    case CAP_PROP_FRAME_HEIGHT:
        propValue = static_cast<double>(profile.height());
        break;
    case CAP_PROP_FPS:
        propValue = static_cast<double>(profile.fps());
        break;
    }

    return propValue;
}

double VideoCapture_LibRealsense::getDepthGeneratorProperty(int propIdx) const
{
    double propValue = 0.0;
    const rs2::video_stream_profile profile = mPipe.get_active_profile().get_stream(RS2_STREAM_DEPTH).as<rs2::video_stream_profile>();
    const rs2::depth_sensor sensor = mPipe.get_active_profile().get_device().first<rs2::depth_sensor>();
    if(!profile || !sensor)
    {
        return propValue;
    }

    switch(propIdx)
    {
    case CAP_PROP_FRAME_WIDTH:
        propValue = static_cast<double>(profile.width());
        break;
    case CAP_PROP_FRAME_HEIGHT:
        propValue = static_cast<double>(profile.height());
        break;
    case CAP_PROP_FPS:
        propValue = static_cast<double>(profile.fps());
        break;
    case CAP_PROP_INTELPERC_DEPTH_SATURATION_VALUE:
        propValue = static_cast<double>(sensor.get_depth_scale());
        break;
    case CAP_PROP_INTELPERC_DEPTH_FOCAL_LENGTH_HORZ:
        propValue = static_cast<double>(profile.get_intrinsics().fx);
        break;
    case CAP_PROP_INTELPERC_DEPTH_FOCAL_LENGTH_VERT:
        propValue = static_cast<double>(profile.get_intrinsics().fy);
        break;
    }

    return propValue;
}

double VideoCapture_LibRealsense::getIrGeneratorProperty(int propIdx) const
{
    double propValue = 0.0;
    const rs2::video_stream_profile profile = mPipe.get_active_profile().get_stream(RS2_STREAM_INFRARED).as<rs2::video_stream_profile>();
    if(!profile)
    {
        return propValue;
    }

    switch(propIdx)
    {
    case CAP_PROP_FRAME_WIDTH:
        propValue = static_cast<double>(profile.width());
        break;
    case CAP_PROP_FRAME_HEIGHT:
        propValue = static_cast<double>(profile.height());
        break;
    case CAP_PROP_FPS:
        propValue = static_cast<double>(profile.fps());
        break;
    }

    return propValue;
}

double VideoCapture_LibRealsense::getCommonProperty(int propIdx) const
{
    double propValue = 0.0;
    const rs2::video_stream_profile profile = mPipe.get_active_profile().get_stream(RS2_STREAM_DEPTH).as<rs2::video_stream_profile>();
    const rs2::depth_sensor sensor = mPipe.get_active_profile().get_device().first<rs2::depth_sensor>();
    if(!profile || !sensor)
    {
        return propValue;
    }

    switch(propIdx)
    {
    case CAP_PROP_FRAME_WIDTH:
    case CAP_PROP_FRAME_HEIGHT:
    case CAP_PROP_FPS:
        propValue = getDepthGeneratorProperty(propIdx);
        break;
    case CAP_PROP_INTELPERC_DEPTH_SATURATION_VALUE:
        propValue = static_cast<double>(sensor.get_depth_scale());
        break;
    case CAP_PROP_INTELPERC_DEPTH_FOCAL_LENGTH_HORZ:
        propValue = static_cast<double>(profile.get_intrinsics().fx);
        break;
    case CAP_PROP_INTELPERC_DEPTH_FOCAL_LENGTH_VERT:
        propValue = static_cast<double>(profile.get_intrinsics().fy);
        break;
    }

    return propValue;
}

bool VideoCapture_LibRealsense::setProperty(int, double)
{
    bool isSet = false;
    return isSet;
}

bool VideoCapture_LibRealsense::grabFrame()
{
    if (!isOpened())
        return false;

    try
    {
        mData = mAlign.process(mPipe.wait_for_frames());
    }
    catch (const rs2::error&)
    {
        return false;
    }

    return true;
}
bool VideoCapture_LibRealsense::retrieveFrame(int outputType, cv::OutputArray frame)
{
    rs2::video_frame _frame(nullptr);
    int type;
    switch (outputType)
    {
    case CAP_INTELPERC_DEPTH_MAP:
        _frame = mData.get_depth_frame().as<rs2::video_frame>();
        type = CV_16UC1;
        break;
    case CAP_INTELPERC_IR_MAP:
        _frame = mData.get_infrared_frame();
        type = CV_8UC1;
        break;
    case CAP_INTELPERC_IMAGE:
        _frame = mData.get_color_frame();
        type = CV_8UC3;
        break;
    default:
        return false;
    }

    try
    {
        // we copy the data straight away, so const_cast should be fine
        void* data = const_cast<void*>(_frame.get_data());
        Mat(_frame.get_height(), _frame.get_width(), type, data, _frame.get_stride_in_bytes()).copyTo(frame);

        if(_frame.get_profile().format() == RS2_FORMAT_RGB8)
            cvtColor(frame, frame, COLOR_RGB2BGR);
    }
    catch (const rs2::error&)
    {
        return false;
    }

    return true;
}
int VideoCapture_LibRealsense::getCaptureDomain()
{
    return CAP_INTELPERC;
}

bool VideoCapture_LibRealsense::isOpened() const
{
    return bool(std::shared_ptr<rs2_pipeline>(mPipe));
}

Ptr<IVideoCapture> create_RealSense_capture(int index)
{
    return makePtr<VideoCapture_LibRealsense>(index);
}


}

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

- **HAVE_LIBREALSENSE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `cap_librealsense.hpp`


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

