# Documentation for `modules/videoio/src/cap_ios_photo_camera.mm`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_ios_photo_camera.mm`
- **File Name**: `cap_ios_photo_camera.mm`
- **File Size**: 5,376 bytes
- **File Type**: .mm
- **Link to Source**: [modules/videoio/src/cap_ios_photo_camera.mm](../../../modules/videoio/src/cap_ios_photo_camera.mm)

## Purpose and Role

This file is located in the `modules/videoio/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 *  cap_ios_photo_camera.mm
 *  For iOS video I/O
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


#import "opencv2/videoio/cap_ios.h"
#include "precomp.hpp"

#pragma mark - Private Interface


@interface CvPhotoCamera ()
{
    id<CvPhotoCameraDelegate> _delegate;
}

@property (nonatomic, strong) AVCaptureStillImageOutput* stillImageOutput;

@end



#pragma mark - Implementation


@implementation CvPhotoCamera



#pragma mark Public

@synthesize stillImageOutput;

- (void)setDelegate:(id<CvPhotoCameraDelegate>)newDelegate {
    _delegate = newDelegate;
}

- (id<CvPhotoCameraDelegate>)delegate {
    return _delegate;
}

#pragma mark - Public interface


- (void)takePicture
{
    if (cameraAvailable == NO) {
        return;
    }
    cameraAvailable = NO;


    [self.stillImageOutput captureStillImageAsynchronouslyFromConnection:self.videoCaptureConnection
                                                       completionHandler:
     ^(CMSampleBufferRef imageSampleBuffer, NSError *error)
     {
         if (error == nil && imageSampleBuffer != NULL)
         {
             // TODO check
             //			 NSNumber* imageOrientation = [UIImage cgImageOrientationForUIDeviceOrientation:currentDeviceOrientation];
             //			 CMSetAttachment(imageSampleBuffer, kCGImagePropertyOrientation, imageOrientation, 1);

             NSData *jpegData = [AVCaptureStillImageOutput jpegStillImageNSDataRepresentation:imageSampleBuffer];

             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.captureSession stopRunning];

                 // Make sure we create objects on the main thread in the main context
                 UIImage* newImage = [UIImage imageWithData:jpegData];

                 //UIImageOrientation orientation = [newImage imageOrientation];

                 // TODO: only apply rotation, don't scale, since we can set this directly in the camera
                 /*
                  switch (orientation) {
                  case UIImageOrientationUp:
                  case UIImageOrientationDown:
                  newImage = [newImage imageWithAppliedRotationAndMaxSize:CGSizeMake(640.0, 480.0)];
                  break;
                  case UIImageOrientationLeft:
                  case UIImageOrientationRight:
                  newImage = [newImage imageWithMaxSize:CGSizeMake(640.0, 480.0)];
                  default:
                  break;
                  }
                  */

                 // We have captured the image, we can allow the user to take another picture
                 cameraAvailable = YES;

                 NSLog(@"CvPhotoCamera captured image");
                 [self.delegate photoCamera:self capturedImage:newImage];

                 [self.captureSession startRunning];
             });
         }
     }];


}

- (void)stop;
{
    [super stop];
    self.stillImageOutput = nil;
}


#pragma mark - Private Interface


- (void)createStillImageOutput;
{
    // setup still image output with jpeg codec
    self.stillImageOutput = [[AVCaptureStillImageOutput alloc] init];
    NSDictionary *outputSettings = [NSDictionary dictionaryWithObjectsAndKeys:AVVideoCodecTypeJPEG, AVVideoCodecKey, nil];
    [self.stillImageOutput setOutputSettings:outputSettings];
    [self.captureSession addOutput:self.stillImageOutput];

    for (AVCaptureConnection *connection in self.stillImageOutput.connections) {
        for (AVCaptureInputPort *port in [connection inputPorts]) {
            if ([port.mediaType isEqual:AVMediaTypeVideo]) {
                self.videoCaptureConnection = connection;
                break;
            }
        }
        if (self.videoCaptureConnection) {
            break;
        }
    }
    NSLog(@"[Camera] still image output created");
}


- (void)createCaptureOutput;
{
    [self createStillImageOutput];
}

- (void)createCustomVideoPreview;
{
    //do nothing, always use AVCaptureVideoPreviewLayer
}


@end
```

## High-Level Overview

This is a .mm source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **CvPhotoCamera**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`

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

