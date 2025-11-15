# Documentation for `modules/videoio/misc/objc/ios/CvCamera2.h`

## File Metadata

- **Full Path**: `modules/videoio/misc/objc/ios/CvCamera2.h`
- **File Name**: `CvCamera2.h`
- **File Size**: 2,981 bytes
- **File Type**: .h
- **Link to Source**: [modules/videoio/misc/objc/ios/CvCamera2.h](../../../../../modules/videoio/misc/objc/ios/CvCamera2.h)

## Purpose and Role

This file is located in the `modules/videoio/misc/objc/ios` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  CvCamera2.h
//
//  Created by Giles Payne on 2020/03/11.
//

#import <UIKit/UIKit.h>
#import <Accelerate/Accelerate.h>
#import <AVFoundation/AVFoundation.h>
#import <ImageIO/ImageIO.h>
#import "CVObjcUtil.h"

@class Mat;

@class CvAbstractCamera2;

CV_EXPORTS @interface CvAbstractCamera2 : NSObject

@property UIDeviceOrientation currentDeviceOrientation;
@property BOOL cameraAvailable;
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

- (void)start;
- (void)stop;
- (void)switchCameras;
- (id)initWithParentView:(UIView*)parent;
- (void)createCaptureOutput;
- (void)createVideoPreviewLayer;
- (void)updateOrientation;
- (void)lockFocus;
- (void)unlockFocus;
- (void)lockExposure;
- (void)unlockExposure;
- (void)lockBalance;
- (void)unlockBalance;
@end

///////////////////////////////// CvVideoCamera ///////////////////////////////////////////
@class CvVideoCamera2;

@protocol CvVideoCameraDelegate2 <NSObject>
- (void)processImage:(Mat*)image;
@end

CV_EXPORTS @interface CvVideoCamera2 : CvAbstractCamera2<AVCaptureVideoDataOutputSampleBufferDelegate>
@property (nonatomic, weak) id<CvVideoCameraDelegate2> delegate;
@property (nonatomic, assign) BOOL grayscaleMode;
@property (nonatomic, assign) BOOL recordVideo;
@property (nonatomic, assign) BOOL rotateVideo;
@property (nonatomic, strong) AVAssetWriterInput* recordAssetWriterInput;
@property (nonatomic, strong) AVAssetWriterInputPixelBufferAdaptor* recordPixelBufferAdaptor;
@property (nonatomic, strong) AVAssetWriter* recordAssetWriter;
- (void)adjustLayoutToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation;
- (void)layoutPreviewLayer;
- (void)saveVideo;
- (NSURL *)videoFileURL;
- (NSString *)videoFileString;
@end

///////////////////////////////// CvPhotoCamera ///////////////////////////////////////////
@class CvPhotoCamera2;

@protocol CvPhotoCameraDelegate2 <NSObject>
- (void)photoCamera:(CvPhotoCamera2*)photoCamera capturedImage:(UIImage*)image;
- (void)photoCameraCancel:(CvPhotoCamera2*)photoCamera;
@end

CV_EXPORTS @interface CvPhotoCamera2 : CvAbstractCamera2<AVCapturePhotoCaptureDelegate>
@property (nonatomic, weak) id<CvPhotoCameraDelegate2> delegate;
- (void)takePicture;
@end
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

- **Mat**: A class/struct defined in this file
- **CvAbstractCamera2**: A class/struct defined in this file
- **CvVideoCamera2**: A class/struct defined in this file
- **CvPhotoCamera2**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

