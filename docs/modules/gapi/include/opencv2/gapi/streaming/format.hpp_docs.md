# Documentation for `modules/gapi/include/opencv2/gapi/streaming/format.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/format.hpp`
- **File Name**: `format.hpp`
- **File Size**: 2,721 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/format.hpp](../../../../../../modules/gapi/include/opencv2/gapi/streaming/format.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation

#ifndef OPENCV_GAPI_GSTREAMING_FORMAT_HPP
#define OPENCV_GAPI_GSTREAMING_FORMAT_HPP

#include <opencv2/gapi/gkernel.hpp> // GKernelPackage

namespace cv {
namespace gapi {
namespace streaming {

GAPI_EXPORTS cv::GKernelPackage kernels();

G_API_OP(GBGR, <GMat(GFrame)>, "org.opencv.streaming.BGR")
{
    static GMatDesc outMeta(const GFrameDesc& in) { return GMatDesc{CV_8U, 3, in.size}; }
};

G_API_OP(GY, <GMat(GFrame)>, "org.opencv.streaming.Y") {
    static GMatDesc outMeta(const GFrameDesc& frameDesc) {
        return GMatDesc { CV_8U, 1, frameDesc.size , false };
    }
};

G_API_OP(GUV, <GMat(GFrame)>, "org.opencv.streaming.UV") {
    static GMatDesc outMeta(const GFrameDesc& frameDesc) {
        return GMatDesc { CV_8U, 2, cv::Size(frameDesc.size.width / 2, frameDesc.size.height / 2),
                          false };
    }
};

/** @brief Gets bgr plane from input frame

@note Function textual ID is "org.opencv.streaming.BGR"

@param in Input frame
@return Image in BGR format
*/
GAPI_EXPORTS cv::GMat BGR(const cv::GFrame& in);

/** @brief Extracts Y plane from media frame.

Output image is 8-bit 1-channel image of @ref CV_8UC1.

@note Function textual ID is "org.opencv.streaming.Y"

@param frame input media frame.
*/
GAPI_EXPORTS GMat Y(const cv::GFrame& frame);

/** @brief Extracts UV plane from media frame.

Output image is 8-bit 2-channel image of @ref CV_8UC2.

@note Function textual ID is "org.opencv.streaming.UV"

@param frame input media frame.
*/
GAPI_EXPORTS GMat UV(const cv::GFrame& frame);
} // namespace streaming

//! @addtogroup gapi_transform
//! @{
/** @brief Makes a copy of the input image. Note that this copy may be not real
(no actual data copied). Use this function to maintain graph contracts,
e.g when graph's input needs to be passed directly to output, like in Streaming mode.

@note Function textual ID is "org.opencv.streaming.copy"

@param in Input image
@return Copy of the input
*/
GAPI_EXPORTS_W GMat copy(const GMat& in);

/** @brief Makes a copy of the input frame. Note that this copy may be not real
(no actual data copied). Use this function to maintain graph contracts,
e.g when graph's input needs to be passed directly to output, like in Streaming mode.

@note Function textual ID is "org.opencv.streaming.copy"

@param in Input frame
@return Copy of the input
*/
GAPI_EXPORTS GFrame copy(const GFrame& in);
//! @} gapi_transform

} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_GSTREAMING_FORMAT_HPP
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

- **to()**: A function/method defined in this file
- **OPENCV_GAPI_GSTREAMING_FORMAT_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gkernel.hpp`

**Python Imports:**
- `input`
- `media`


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

