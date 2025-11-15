# Documentation for `modules/videoio/include/opencv2/videoio/videoio_c.h`

## File Metadata

- **Full Path**: `modules/videoio/include/opencv2/videoio/videoio_c.h`
- **File Name**: `videoio_c.h`
- **File Size**: 5,647 bytes
- **File Type**: .h
- **Link to Source**: [modules/videoio/include/opencv2/videoio/videoio_c.h](../../../../../modules/videoio/include/opencv2/videoio/videoio_c.h)

## Purpose and Role

This file is located in the `modules/videoio/include/opencv2/videoio` directory and serves as part of the OpenCV library infrastructure.

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
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
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
//   * The name of Intel Corporation may not be used to endorse or promote products
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

#ifndef OPENCV_VIDEOIO_H
#define OPENCV_VIDEOIO_H

#include "opencv2/core/core_c.h"

#include "opencv2/videoio/legacy/constants_c.h"

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/**
  @addtogroup videoio_c
  @{
*/

/****************************************************************************************\
*                         Working with Video Files and Cameras                           *
\****************************************************************************************/

/** @brief "black box" capture structure

In C++ use cv::VideoCapture
*/
typedef struct CvCapture CvCapture;

/** @brief start capturing frames from video file
*/
CVAPI(CvCapture*) cvCreateFileCapture( const char* filename );

/** @brief start capturing frames from video file. allows specifying a preferred API to use
*/
CVAPI(CvCapture*) cvCreateFileCaptureWithPreference( const char* filename , int apiPreference);

/** @brief start capturing frames from camera: index = camera_index + domain_offset (CV_CAP_*)
*/
CVAPI(CvCapture*) cvCreateCameraCapture( int index );

/** @brief grab a frame, return 1 on success, 0 on fail.

  this function is thought to be fast
*/
CVAPI(int) cvGrabFrame( CvCapture* capture );

/** @brief get the frame grabbed with cvGrabFrame(..)

  This function may apply some frame processing like
  frame decompression, flipping etc.
  @warning !!!DO NOT RELEASE or MODIFY the retrieved frame!!!
*/
CVAPI(IplImage*) cvRetrieveFrame( CvCapture* capture, int streamIdx CV_DEFAULT(0) );

/** @brief Just a combination of cvGrabFrame and cvRetrieveFrame

  @warning !!!DO NOT RELEASE or MODIFY the retrieved frame!!!
*/
CVAPI(IplImage*) cvQueryFrame( CvCapture* capture );

/** @brief stop capturing/reading and free resources
*/
CVAPI(void) cvReleaseCapture( CvCapture** capture );

/** @brief retrieve capture properties
*/
CVAPI(double) cvGetCaptureProperty( CvCapture* capture, int property_id );
/** @brief set capture properties
*/
CVAPI(int)    cvSetCaptureProperty( CvCapture* capture, int property_id, double value );

/** @brief Return the type of the capturer (eg, ::CV_CAP_VFW, ::CV_CAP_UNICAP)

It is unknown if created with ::CV_CAP_ANY
*/
CVAPI(int)    cvGetCaptureDomain( CvCapture* capture);

/** @brief "black box" video file writer structure

In C++ use cv::VideoWriter
*/
typedef struct CvVideoWriter CvVideoWriter;

/** @brief initialize video file writer
*/
CVAPI(CvVideoWriter*) cvCreateVideoWriter( const char* filename, int fourcc,
                                           double fps, CvSize frame_size,
                                           int is_color CV_DEFAULT(1));

/** @brief write frame to video file
*/
CVAPI(int) cvWriteFrame( CvVideoWriter* writer, const IplImage* image );

/** @brief close video file writer
*/
CVAPI(void) cvReleaseVideoWriter( CvVideoWriter** writer );

// ***************************************************************************************
//! @name Obsolete functions/synonyms
//! @{
#define cvCaptureFromCAM cvCreateCameraCapture //!< @deprecated use cvCreateCameraCapture() instead
#define cvCaptureFromFile cvCreateFileCapture  //!< @deprecated use cvCreateFileCapture() instead
#define cvCaptureFromAVI cvCaptureFromFile     //!< @deprecated use cvCreateFileCapture() instead
#define cvCreateAVIWriter cvCreateVideoWriter  //!< @deprecated use cvCreateVideoWriter() instead
#define cvWriteToAVI cvWriteFrame              //!< @deprecated use cvWriteFrame() instead
//!  @} Obsolete...

//! @} videoio_c

#ifdef __cplusplus
}
#endif

#endif //OPENCV_VIDEOIO_H
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

- **CvVideoWriter**: A class/struct defined in this file
- **CvCapture**: A class/struct defined in this file

### Functions and Methods

- **is()**: A function/method defined in this file
- **may()**: A function/method defined in this file
- **OPENCV_VIDEOIO_H()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **struct()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/core_c.h`
- `opencv2/videoio/legacy/constants_c.h`

**Python Imports:**
- `video`
- `camera`
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

