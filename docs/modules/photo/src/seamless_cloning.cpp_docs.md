# Documentation for `modules/photo/src/seamless_cloning.cpp`

## File Metadata

- **Full Path**: `modules/photo/src/seamless_cloning.cpp`
- **File Name**: `seamless_cloning.cpp`
- **File Size**: 5,309 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/src/seamless_cloning.cpp](../../../modules/photo/src/seamless_cloning.cpp)

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

#include "seamless_cloning.hpp"

using namespace std;
using namespace cv;

static Mat checkMask(InputArray mask, Size size)
{
    Mat gray;
    if (mask.channels() == 3 || mask.channels() == 4)
        cvtColor(mask, gray, COLOR_BGRA2GRAY);
    else
    {
        if (mask.empty())
            gray = Mat(size.height, size.width, CV_8UC1, Scalar(255));
        else
            return mask.getMat();
    }

    return gray;
}

void cv::seamlessClone(InputArray _src, InputArray _dst, InputArray _mask, Point p, OutputArray _blend, int flags)
{
    CV_INSTRUMENT_REGION();
    CV_Assert(!_src.empty());
    CV_Assert(!_dst.empty());

    const Mat src  = _src.getMat();
    const Mat dest = _dst.getMat();

    Mat mask = checkMask(_mask, src.size());
    dest.copyTo(_blend);
    Mat blend = _blend.getMat();

    Mat mask_inner = mask(Rect(1, 1, mask.cols - 2, mask.rows - 2));
    copyMakeBorder(mask_inner, mask, 1, 1, 1, 1, BORDER_ISOLATED | BORDER_CONSTANT, Scalar(0));

    Rect roi_s = boundingRect(mask);
    if (roi_s.empty()) return;

    int l_from_center = p.x - roi_s.width / 2;
    int t_from_center = p.y - roi_s.height / 2;

    if (flags >= NORMAL_CLONE_WIDE)
    {
        l_from_center = p.x - (mask.cols / 2 - roi_s.x);
        t_from_center = p.y - (mask.rows / 2 - roi_s.y);
    }

    Rect roi_d(l_from_center, t_from_center, roi_s.width, roi_s.height);
    Mat destinationROI = dest(roi_d);
    Mat sourceROI = Mat::zeros(roi_s.height, roi_s.width, src.type());
    src(roi_s).copyTo(sourceROI,mask(roi_s));

    Mat maskROI = mask(roi_s);
    Mat recoveredROI = blend(roi_d);

    Cloning obj;
    obj.normalClone(destinationROI,sourceROI,maskROI,recoveredROI,flags);
}

void cv::colorChange(InputArray _src, InputArray _mask, OutputArray _dst, float red, float green, float blue)
{
    CV_INSTRUMENT_REGION();

    Mat src  = _src.getMat();
    Mat mask = checkMask(_mask, src.size());
    _dst.create(src.size(), src.type());
    Mat blend = _dst.getMat();

    Mat cs_mask = Mat::zeros(src.size(), src.type());
    src.copyTo(cs_mask, mask);

    Cloning obj;
    obj.localColorChange(src, cs_mask, mask, blend, red, green, blue);
}

void cv::illuminationChange(InputArray _src, InputArray _mask, OutputArray _dst, float alpha, float beta)
{
    CV_INSTRUMENT_REGION();

    Mat src  = _src.getMat();
    Mat mask = checkMask(_mask, src.size());
    _dst.create(src.size(), src.type());
    Mat blend = _dst.getMat();

    Mat cs_mask = Mat::zeros(src.size(), src.type());
    src.copyTo(cs_mask, mask);

    Cloning obj;
    obj.illuminationChange(src, cs_mask, mask, blend, alpha, beta);

}

void cv::textureFlattening(InputArray _src, InputArray _mask, OutputArray _dst,
                           float low_threshold, float high_threshold, int kernel_size)
{
    CV_INSTRUMENT_REGION();

    Mat src  = _src.getMat();
    Mat mask = checkMask(_mask, src.size());
    _dst.create(src.size(), src.type());
    Mat blend = _dst.getMat();

    Mat cs_mask = Mat::zeros(src.size(), src.type());
    src.copyTo(cs_mask, mask);

    Cloning obj;
    obj.textureFlatten(src, cs_mask, mask, low_threshold, high_threshold, kernel_size, blend);
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
- `seamless_cloning.hpp`
- `precomp.hpp`
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

