# Documentation for `modules/gapi/include/opencv2/gapi/stereo.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/stereo.hpp`
- **File Name**: `stereo.hpp`
- **File Size**: 3,402 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/stereo.hpp](../../../../../modules/gapi/include/opencv2/gapi/stereo.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distereoibution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STEREO_HPP
#define OPENCV_GAPI_STEREO_HPP

#include <opencv2/gapi/gmat.hpp>
#include <opencv2/gapi/gscalar.hpp>
#include <opencv2/gapi/gkernel.hpp>

namespace cv {
namespace gapi {

/**
 * The enum specified format of result that you get from @ref cv::gapi::stereo.
 */
enum class StereoOutputFormat {
    DEPTH_FLOAT16, ///< Floating point 16 bit value, CV_16FC1.
                   ///< This identifier is deprecated, use DEPTH_16F instead.
    DEPTH_FLOAT32, ///< Floating point 32 bit value, CV_32FC1
                   ///< This identifier is deprecated, use DEPTH_16F instead.
    DISPARITY_FIXED16_11_5, ///< 16 bit signed: first bit for sign,
                            ///< 10 bits for integer part,
                            ///< 5 bits for fractional part.
                            ///< This identifier is deprecated,
                            ///< use DISPARITY_16Q_10_5 instead.
    DISPARITY_FIXED16_12_4, ///< 16 bit signed: first bit for sign,
                            ///< 11 bits for integer part,
                            ///< 4 bits for fractional part.
                            ///< This identifier is deprecated,
                            ///< use DISPARITY_16Q_11_4 instead.
    DEPTH_16F = DEPTH_FLOAT16, ///< Same as DEPTH_FLOAT16
    DEPTH_32F = DEPTH_FLOAT32, ///< Same as DEPTH_FLOAT32
    DISPARITY_16Q_10_5 = DISPARITY_FIXED16_11_5, ///< Same as DISPARITY_FIXED16_11_5
    DISPARITY_16Q_11_4 = DISPARITY_FIXED16_12_4 ///< Same as DISPARITY_FIXED16_12_4
};


/**
 * @brief This namespace contains G-API Operation Types for Stereo and
 * related functionality.
 */
namespace calib3d {

G_TYPED_KERNEL(GStereo, <GMat(GMat, GMat, const StereoOutputFormat)>, "org.opencv.stereo") {
    static GMatDesc outMeta(const GMatDesc &left, const GMatDesc &right, const StereoOutputFormat of) {
        GAPI_Assert(left.chan == 1);
        GAPI_Assert(left.depth == CV_8U);

        GAPI_Assert(right.chan == 1);
        GAPI_Assert(right.depth == CV_8U);

        switch(of) {
            case StereoOutputFormat::DEPTH_FLOAT16:
                return left.withDepth(CV_16FC1);
            case StereoOutputFormat::DEPTH_FLOAT32:
                return left.withDepth(CV_32FC1);
            case StereoOutputFormat::DISPARITY_FIXED16_11_5:
            case StereoOutputFormat::DISPARITY_FIXED16_12_4:
                return left.withDepth(CV_16SC1);
            default:
                GAPI_Error("Unknown output format!");
        }
    }
};

} // namespace calib3d

/** @brief Computes disparity/depth map for the specified stereo-pair.
The function computes disparity or depth map depending on passed StereoOutputFormat argument.

@param left 8-bit single-channel left image of @ref CV_8UC1 type.
@param right 8-bit single-channel right image of @ref CV_8UC1 type.
@param of enum to specified output kind: depth or disparity and corresponding type
*/
GAPI_EXPORTS GMat stereo(const GMat& left,
                         const GMat& right,
                         const StereoOutputFormat of = StereoOutputFormat::DEPTH_FLOAT32);
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STEREO_HPP
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

- **StereoOutputFormat**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STEREO_HPP()**: A function/method defined in this file
- **computes()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/gscalar.hpp`
- `opencv2/gapi/gmat.hpp`


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

