# Documentation for `modules/videoio/include/opencv2/videoio/cap_ios.h`

## File Metadata

- **Full Path**: `modules/videoio/include/opencv2/videoio/cap_ios.h`
- **File Name**: `cap_ios.h`
- **File Size**: 4,914 bytes
- **File Type**: .h
- **Link to Source**: [modules/videoio/include/opencv2/videoio/cap_ios.h](../../../../../modules/videoio/include/opencv2/videoio/cap_ios.h)

## Purpose and Role

This file is located in the `modules/videoio/include/opencv2/videoio` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*  For iOS video I/O
 *  by Eduard Feicho on 29/07/12
 *  Copyright 2012. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 *    this list of conditions and the following disclaimer in the documentation
 *    and/or other materials provided with the distribution.
 * 3. The name of the author may not be used to endorse or promote products
 *    derived from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE AUTHOR "AS IS" AND ANY EXPRESS OR IMPLIED
 * WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
 * MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
 * EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
 * SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
 * OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
 * WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
 * OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 */

#import <UIKit/UIKit.h>
#import <Accelerate/Accelerate.h>
#import <AVFoundation/AVFoundation.h>
#import <ImageIO/ImageIO.h>
#include "opencv2/core.hpp"

//! @addtogroup videoio_ios
//! @{

/////////////////////////////////////// CvAbstractCamera /////////////////////////////////////

@class CvAbstractCamera;

CV_EXPORTS @interface CvAbstractCamera : NSObject
{
    UIDeviceOrientation currentDeviceOrientation;

    BOOL cameraAvailable;
}

@property (nonatomic, strong) AVCaptureSession* captureSession;
@property (nonatomic, strong) AVCaptureConnection* videoCaptureConnection;

@property (nonatomic, readonly) BOOL running;
@property (nonatomic, readonly) BOOL captureSessionLoaded;

@property (nonatomic, assign) int defaultFPS;
@property (nonatomic, readonly) AVCaptureVideoPreviewLayer *captureVideoPreviewLayer;
@property (nonatomic, assign) AVCaptureDevicePosition defaultAVCaptureDevicePosition;
@property (nonatomic, assign) AVCaptureVideoOrientation defaultAVCaptureVideoOrientation;
@property (nonatomic, assign) BOOL useAVCaptureVideoPreviewLayer;
@property (nonatomic, strong) NSString *const defaultAVCaptureSessionPreset;

@property (nonatomic, assign) int imageWidth;
@property (nonatomic, assign) int imageHeight;

@property (nonatomic, strong) UIView* parentView;

- CV_UNUSED(start);
- CV_UNUSED(stop);
- CV_UNUSED(switchCameras);

- (id)initWithParentView:(UIView*)parent;

- CV_UNUSED(createCaptureOutput);
- CV_UNUSED(createVideoPreviewLayer);
- CV_UNUSED(updateOrientation);

- CV_UNUSED(lockFocus);
- CV_UNUSED(unlockFocus);
- CV_UNUSED(lockExposure);
- CV_UNUSED(unlockExposure);
- CV_UNUSED(lockBalance);
- CV_UNUSED(unlockBalance);

@end

///////////////////////////////// CvVideoCamera ///////////////////////////////////////////

@class CvVideoCamera;

CV_EXPORTS @protocol CvVideoCameraDelegate <NSObject>

#ifdef __cplusplus
// delegate method for processing image frames
- (void)processImage:(cv::Mat&)image;
#endif

@end

CV_EXPORTS @interface CvVideoCamera : CvAbstractCamera<AVCaptureVideoDataOutputSampleBufferDelegate>
{
    AVCaptureVideoDataOutput *videoDataOutput;

    dispatch_queue_t videoDataOutputQueue;
    CALayer *customPreviewLayer;

    CMTime lastSampleTime;

}

@property (nonatomic, weak) id<CvVideoCameraDelegate> delegate;
@property (nonatomic, assign) BOOL grayscaleMode;

@property (nonatomic, assign) BOOL recordVideo;
@property (nonatomic, assign) BOOL rotateVideo;
@property (nonatomic, strong) AVAssetWriterInput* recordAssetWriterInput;
@property (nonatomic, strong) AVAssetWriterInputPixelBufferAdaptor* recordPixelBufferAdaptor;
@property (nonatomic, strong) AVAssetWriter* recordAssetWriter;

- (void)adjustLayoutToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation;
- CV_UNUSED(layoutPreviewLayer);
- CV_UNUSED(saveVideo);
- (NSURL *)videoFileURL;
- (NSString *)videoFileString;


@end

///////////////////////////////// CvPhotoCamera ///////////////////////////////////////////

@class CvPhotoCamera;

CV_EXPORTS @protocol CvPhotoCameraDelegate <NSObject>

- (void)photoCamera:(CvPhotoCamera*)photoCamera capturedImage:(UIImage *)image;
- (void)photoCameraCancel:(CvPhotoCamera*)photoCamera;

@end

CV_EXPORTS @interface CvPhotoCamera : CvAbstractCamera
{
    AVCaptureStillImageOutput *stillImageOutput;
}

@property (nonatomic, weak) id<CvPhotoCameraDelegate> delegate;

- CV_UNUSED(takePicture);

@end

//! @} videoio_ios
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

- **CvPhotoCamera**: A class/struct defined in this file
- **CvAbstractCamera**: A class/struct defined in this file
- **CvVideoCamera**: A class/struct defined in this file

### Functions and Methods

- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`

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

