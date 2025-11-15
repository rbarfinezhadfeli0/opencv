# Documentation for `modules/photo/src/hdr_common.cpp`

## File Metadata

- **Full Path**: `modules/photo/src/hdr_common.cpp`
- **File Name**: `hdr_common.cpp`
- **File Size**: 3,961 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/src/hdr_common.cpp](../../../modules/photo/src/hdr_common.cpp)

## Purpose and Role

This file is located in the `modules/photo/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "precomp.hpp"
#include "opencv2/photo.hpp"
#include "hdr_common.hpp"

namespace cv
{

void checkImageDimensions(const std::vector<Mat>& images)
{
    CV_Assert(!images.empty());
    int width = images[0].cols;
    int height = images[0].rows;
    int type = images[0].type();

    for(size_t i = 0; i < images.size(); i++) {
        CV_Assert(images[i].cols == width && images[i].rows == height);
        CV_Assert(images[i].type() == type);
    }
}

Mat triangleWeights()
{
    // hat function
    Mat w(LDR_SIZE, 1, CV_32F);
    int half   = LDR_SIZE / 2;
    int maxVal = LDR_SIZE - 1;
    float epsilon = 1e-6f;
    w.at<float>(0) = epsilon;
    w.at<float>(LDR_SIZE-1) = epsilon;
    for (int i = 1; i < LDR_SIZE-1; i++){
        w.at<float>(i) = (i < half)
            ? static_cast<float>(i)
            : static_cast<float>(maxVal - i);
    }
    return w;
}

Mat RobertsonWeights()
{
    Mat weight(LDR_SIZE, 1, CV_32FC3);
    float q = (LDR_SIZE - 1) / 4.0f;
    float e4 = exp(4.f);
    float scale = e4/(e4 - 1.f);
    float shift = 1 / (1.f - e4);

    for(int i = 0; i < LDR_SIZE; i++) {
        float value = i / q - 2.0f;
        value = scale*exp(-value * value) + shift;
        weight.at<Vec3f>(i) = Vec3f::all(value);
    }
    return weight;
}

void mapLuminance(Mat src, Mat dst, Mat lum, Mat new_lum, float saturation)
{
    std::vector<Mat> channels(3);
    split(src, channels);
    for(int i = 0; i < 3; i++) {
        channels[i] = channels[i].mul(1.0f / lum);
        pow(channels[i], saturation, channels[i]);
        channels[i] = channels[i].mul(new_lum);
    }
    merge(channels, dst);
}

Mat linearResponse(int channels)
{
    Mat response = Mat(LDR_SIZE, 1, CV_MAKETYPE(CV_32F, channels));
    for(int i = 0; i < LDR_SIZE; i++) {
        response.at<Vec3f>(i) = Vec3f::all(static_cast<float>(i));
    }
    return response;
}

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

### Functions and Methods

- **Mat()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `hdr_common.hpp`
- `opencv2/photo.hpp`

**Python Imports:**
- `this`


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

