# Documentation for `modules/gapi/include/opencv2/gapi/oak/oak.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/oak/oak.hpp`
- **File Name**: `oak.hpp`
- **File Size**: 4,484 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/oak/oak.hpp](../../../../../../modules/gapi/include/opencv2/gapi/oak/oak.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/oak` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_OAK_HPP
#define OPENCV_GAPI_OAK_HPP

#include <opencv2/gapi/garg.hpp>       // IStreamSource
#include <opencv2/gapi/gkernel.hpp>    // GKernelPackage
#include <opencv2/gapi/gstreaming.hpp> // GOptRunArgsP

namespace cv {
namespace gapi {
namespace oak {

// FIXME: copypasted from dai library
struct EncoderConfig {
    /**
     * Rate control mode specifies if constant or variable bitrate should be used (H264 / H265)
     */
    enum class RateControlMode: int { CBR, VBR };

    /**
     * Encoding profile, H264, H265 or MJPEG
     */
    enum class Profile: int { H264_BASELINE, H264_HIGH, H264_MAIN, H265_MAIN, MJPEG };
    /**
     * Specifies preferred bitrate (kb) of compressed output bitstream
     */
    std::int32_t bitrate = 8000;
    /**
     * Every x number of frames a keyframe will be inserted
     */
    std::int32_t keyframeFrequency = 30;
    /**
     * Specifies maximum bitrate (kb) of compressed output bitstream
     */
    std::int32_t maxBitrate = 8000;
    /**
     * Specifies number of B frames to be inserted
     */
    std::int32_t numBFrames = 0;
    /**
     * This options specifies how many frames are available in this nodes pool (can help if
     * receiver node is slow at consuming
     */
    std::uint32_t numFramesPool = 4;
    /**
     * Encoding profile, H264, H265 or MJPEG
     */
    Profile profile = Profile::H265_MAIN;
    /**
     * Value between 0-100% (approximates quality)
     */
    std::int32_t quality = 80;
    /**
     * Lossless mode ([M]JPEG only)
     */
    bool lossless = false;
    /**
     * Rate control mode specifies if constant or variable bitrate should be used (H264 / H265)
     */
    RateControlMode rateCtrlMode = RateControlMode::CBR;
    /**
     * Input and compressed output frame width
     */
    std::int32_t width = 1920;
    /**
     * Input and compressed output frame height
     */
    std::int32_t height = 1080;
    /**
     * Frame rate
     */
    float frameRate = 30.0f;
};

G_API_OP(GEncFrame, <GArray<uint8_t>(GFrame, EncoderConfig)>, "org.opencv.oak.enc_frame") {
    static GArrayDesc outMeta(const GFrameDesc&, const EncoderConfig&) {
        return cv::empty_array_desc();
    }
};

G_API_OP(GSobelXY, <GFrame(GFrame, const cv::Mat&, const cv::Mat&)>, "org.opencv.oak.sobelxy") {
    static GFrameDesc outMeta(const GFrameDesc& in, const cv::Mat&, const cv::Mat&) {
        return in;
    }
};

G_API_OP(GCopy, <GFrame(GFrame)>, "org.opencv.oak.copy") {
    static GFrameDesc outMeta(const GFrameDesc& in) {
        return in;
    }
};

// FIXME: add documentation on operations below

GAPI_EXPORTS GArray<uint8_t> encode(const GFrame& in, const EncoderConfig&);

GAPI_EXPORTS GFrame sobelXY(const GFrame& in,
                            const cv::Mat& hk,
                            const cv::Mat& vk);

GAPI_EXPORTS GFrame copy(const GFrame& in);

// OAK backend & kernels ////////////////////////////////////////////////////////
GAPI_EXPORTS cv::gapi::GBackend backend();
GAPI_EXPORTS cv::gapi::GKernelPackage kernels();

// Camera object ///////////////////////////////////////////////////////////////

struct GAPI_EXPORTS ColorCameraParams {
    /**
     * Format of the frame one gets from the camera
     */
    bool interleaved = false;

    // FIXME: extend
    enum class BoardSocket: int { RGB, BGR };

    BoardSocket board_socket = BoardSocket::RGB;

    // FIXME: extend
    enum class Resolution: int { THE_1080_P };

    Resolution resolution = Resolution::THE_1080_P;
};

class GAPI_EXPORTS ColorCamera: public cv::gapi::wip::IStreamSource {
    cv::MediaFrame m_dummy;
    ColorCameraParams m_params;

    virtual bool pull(cv::gapi::wip::Data &data) override;
    virtual GMetaArg descr_of() const override;

public:
    ColorCamera();
    explicit ColorCamera(const ColorCameraParams& params);
};

} // namespace oak
} // namespace gapi

namespace detail {
template<> struct CompileArgTag<gapi::oak::ColorCameraParams> {
    static const char* tag() { return "gapi.oak.colorCameraParams"; }
};

template<> struct CompileArgTag<gapi::oak::EncoderConfig> {
    static const char* tag() { return "gapi.oak.encoderConfig"; }
};
} // namespace detail

} // namespace cv

#endif // OPENCV_GAPI_OAK_HPP
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

- **GAPI_EXPORTS**: A class/struct defined in this file
- **RateControlMode**: A class/struct defined in this file
- **CompileArgTag**: A class/struct defined in this file
- **EncoderConfig**: A class/struct defined in this file
- **BoardSocket**: A class/struct defined in this file
- **Resolution**: A class/struct defined in this file
- **Profile**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_OAK_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/gstreaming.hpp`
- `opencv2/gapi/garg.hpp`

**Python Imports:**
- `dai`
- `the`


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

