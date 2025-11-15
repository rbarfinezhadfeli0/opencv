# Documentation for `modules/videoio/misc/objc/ios/CvAbstractCamera2.mm`

## File Metadata

- **Full Path**: `modules/videoio/misc/objc/ios/CvAbstractCamera2.mm`
- **File Name**: `CvAbstractCamera2.mm`
- **File Size**: 14,233 bytes
- **File Type**: .mm
- **Link to Source**: [modules/videoio/misc/objc/ios/CvAbstractCamera2.mm](../../../../../modules/videoio/misc/objc/ios/CvAbstractCamera2.mm)

## Purpose and Role

This file is located in the `modules/videoio/misc/objc/ios` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  CvAbstractCamera2.mm
//
//  Created by Giles Payne on 2020/04/01.
//

#import "CvCamera2.h"

#pragma mark - Private Interface

@interface CvAbstractCamera2 ()

@property (nonatomic, strong) AVCaptureVideoPreviewLayer* captureVideoPreviewLayer;

- (void)deviceOrientationDidChange:(NSNotification*)notification;
- (void)startCaptureSession;

- (void)setDesiredCameraPosition:(AVCaptureDevicePosition)desiredPosition;

- (void)updateSize;

@end


#pragma mark - Implementation


@implementation CvAbstractCamera2

#pragma mark - Constructors

- (id)init;
{
    self = [super init];
    if (self) {
        // react to device orientation notifications
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(deviceOrientationDidChange:)
                                                     name:UIDeviceOrientationDidChangeNotification
                                                   object:nil];
        [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
        self.currentDeviceOrientation = [[UIDevice currentDevice] orientation];


        // check if camera available
        self.cameraAvailable = [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera];
        NSLog(@"camera available: %@", (self.cameraAvailable ? @"YES" : @"NO") );

        _running = NO;

        // set camera default configuration
        self.defaultAVCaptureDevicePosition = AVCaptureDevicePositionFront;
        self.defaultAVCaptureVideoOrientation = AVCaptureVideoOrientationLandscapeLeft;
        self.defaultFPS = 15;
        self.defaultAVCaptureSessionPreset = AVCaptureSessionPreset352x288;

        self.parentView = nil;
        self.useAVCaptureVideoPreviewLayer = NO;
    }
    return self;
}



- (id)initWithParentView:(UIView*)parent;
{
    self = [super init];
    if (self) {
        // react to device orientation notifications
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(deviceOrientationDidChange:)
                                                     name:UIDeviceOrientationDidChangeNotification
                                                   object:nil];
        [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
        self.currentDeviceOrientation = [[UIDevice currentDevice] orientation];


        // check if camera available
        self.cameraAvailable = [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera];
        NSLog(@"camera available: %@", (self.cameraAvailable ? @"YES" : @"NO") );

        _running = NO;

        // set camera default configuration
        self.defaultAVCaptureDevicePosition = AVCaptureDevicePositionFront;
        self.defaultAVCaptureVideoOrientation = AVCaptureVideoOrientationLandscapeLeft;
        self.defaultFPS = 15;
        self.defaultAVCaptureSessionPreset = AVCaptureSessionPreset640x480;

        self.parentView = parent;
        self.useAVCaptureVideoPreviewLayer = YES;
    }
    return self;
}

- (void)dealloc;
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
    [[UIDevice currentDevice] endGeneratingDeviceOrientationNotifications];
}

#pragma mark - Public interface


- (void)start;
{
    if (![NSThread isMainThread]) {
        NSLog(@"[Camera] Warning: Call start only from main thread");
        [self performSelectorOnMainThread:@selector(start) withObject:nil waitUntilDone:NO];
        return;
    }

    if (self.running == YES) {
        return;
    }
    _running = YES;

    // TODO: update image size data before actually starting (needed for recording)
    [self updateSize];

    if (self.cameraAvailable) {
        [self startCaptureSession];
    }
}


- (void)pause;
{
    _running = NO;
    [self.captureSession stopRunning];
}



- (void)stop;
{
    _running = NO;

    // Release any retained subviews of the main view.
    // e.g. self.myOutlet = nil;
    if (self.captureSession) {
        for (AVCaptureInput *input in self.captureSession.inputs) {
            [self.captureSession removeInput:input];
        }

        for (AVCaptureOutput *output in self.captureSession.outputs) {
            [self.captureSession removeOutput:output];
        }

        [self.captureSession stopRunning];
    }

    _captureSessionLoaded = NO;
}



// use front/back camera
- (void)switchCameras;
{
    BOOL was_running = self.running;
    if (was_running) {
        [self stop];
    }
    if (self.defaultAVCaptureDevicePosition == AVCaptureDevicePositionFront) {
        self.defaultAVCaptureDevicePosition = AVCaptureDevicePositionBack;
    } else {
        self.defaultAVCaptureDevicePosition  = AVCaptureDevicePositionFront;
    }
    if (was_running) {
        [self start];
    }
}



#pragma mark - Device Orientation Changes


- (void)deviceOrientationDidChange:(NSNotification*)notification
{
    (void)notification;
    UIDeviceOrientation orientation = [UIDevice currentDevice].orientation;

    switch (orientation)
    {
        case UIDeviceOrientationPortrait:
        case UIDeviceOrientationPortraitUpsideDown:
        case UIDeviceOrientationLandscapeLeft:
        case UIDeviceOrientationLandscapeRight:
            self.currentDeviceOrientation = orientation;
            break;

        case UIDeviceOrientationFaceUp:
        case UIDeviceOrientationFaceDown:
        default:
            break;
    }
    NSLog(@"deviceOrientationDidChange: %d", (int)orientation);

    [self updateOrientation];
}



#pragma mark - Private Interface

- (void)createCaptureSession;
{
    // set a av capture session preset
    self.captureSession = [[AVCaptureSession alloc] init];
    if ([self.captureSession canSetSessionPreset:self.defaultAVCaptureSessionPreset]) {
        [self.captureSession setSessionPreset:self.defaultAVCaptureSessionPreset];
    } else if ([self.captureSession canSetSessionPreset:AVCaptureSessionPresetLow]) {
        [self.captureSession setSessionPreset:AVCaptureSessionPresetLow];
    } else {
        NSLog(@"[Camera] Error: could not set session preset");
    }
}

- (void)createCaptureDevice;
{
    // setup the device
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    [self setDesiredCameraPosition:self.defaultAVCaptureDevicePosition];
    NSLog(@"[Camera] device connected? %@", device.connected ? @"YES" : @"NO");
    NSLog(@"[Camera] device position %@", (device.position == AVCaptureDevicePositionBack) ? @"back" : @"front");
}


- (void)createVideoPreviewLayer;
{
    self.captureVideoPreviewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:self.captureSession];

    if ([self.captureVideoPreviewLayer.connection isVideoOrientationSupported])
    {
        [self.captureVideoPreviewLayer.connection setVideoOrientation:self.defaultAVCaptureVideoOrientation];
    }

    if (self.parentView != nil) {
        self.captureVideoPreviewLayer.frame = self.parentView.bounds;
        self.captureVideoPreviewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
        [self.parentView.layer addSublayer:self.captureVideoPreviewLayer];
    }
    NSLog(@"[Camera] created AVCaptureVideoPreviewLayer");
}

- (void)setDesiredCameraPosition:(AVCaptureDevicePosition)desiredPosition;
{
    for (AVCaptureDevice *device in [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo]) {
        if ([device position] == desiredPosition) {
            [self.captureSession beginConfiguration];

            NSError* error = nil;
            AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:device error:&error];
            if (!input) {
                NSLog(@"error creating input %@", [error description]);
            }

            // support for autofocus
            if ([device isFocusModeSupported:AVCaptureFocusModeContinuousAutoFocus]) {
                error = nil;
                if ([device lockForConfiguration:&error]) {
                    device.focusMode = AVCaptureFocusModeContinuousAutoFocus;
                    [device unlockForConfiguration];
                } else {
                    NSLog(@"unable to lock device for autofocus configuration %@", [error description]);
                }
            }
            [self.captureSession addInput:input];

            for (AVCaptureInput *oldInput in self.captureSession.inputs) {
                [self.captureSession removeInput:oldInput];
            }
            [self.captureSession addInput:input];
            [self.captureSession commitConfiguration];

            break;
        }
    }
}



- (void)startCaptureSession
{
    if (!self.cameraAvailable) {
        return;
    }

    if (self.captureSessionLoaded == NO) {
        [self createCaptureSession];
        [self createCaptureDevice];
        [self createCaptureOutput];

        // setup preview layer
        if (self.useAVCaptureVideoPreviewLayer) {
            [self createVideoPreviewLayer];
        } else {
            [self createCustomVideoPreview];
        }

        _captureSessionLoaded = YES;
    }

    [self.captureSession startRunning];
}


- (void)createCaptureOutput;
{
    [NSException raise:NSInternalInconsistencyException
                format:@"You must override %s in a subclass", __FUNCTION__];
}

- (void)createCustomVideoPreview;
{
    [NSException raise:NSInternalInconsistencyException
                format:@"You must override %s in a subclass", __FUNCTION__];
}

- (void)updateOrientation;
{
    // nothing to do here
}


- (void)updateSize;
{
    if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPresetPhoto]) {
        //TODO: find the correct resolution
        self.imageWidth = 640;
        self.imageHeight = 480;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPresetHigh]) {
        //TODO: find the correct resolution
        self.imageWidth = 640;
        self.imageHeight = 480;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPresetMedium]) {
        //TODO: find the correct resolution
        self.imageWidth = 640;
        self.imageHeight = 480;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPresetLow]) {
        //TODO: find the correct resolution
        self.imageWidth = 640;
        self.imageHeight = 480;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPreset352x288]) {
        self.imageWidth = 352;
        self.imageHeight = 288;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPreset640x480]) {
        self.imageWidth = 640;
        self.imageHeight = 480;
    } else if ([self.defaultAVCaptureSessionPreset isEqualToString:AVCaptureSessionPreset1280x720]) {
        self.imageWidth = 1280;
        self.imageHeight = 720;
    } else {
        self.imageWidth = 640;
        self.imageHeight = 480;
    }
}

- (void)lockFocus;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isFocusModeSupported:AVCaptureFocusModeLocked]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.focusMode = AVCaptureFocusModeLocked;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for locked focus configuration %@", [error description]);
        }
    }
}

- (void) unlockFocus;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isFocusModeSupported:AVCaptureFocusModeContinuousAutoFocus]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.focusMode = AVCaptureFocusModeContinuousAutoFocus;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for autofocus configuration %@", [error description]);
        }
    }
}

- (void)lockExposure;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isExposureModeSupported:AVCaptureExposureModeLocked]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.exposureMode = AVCaptureExposureModeLocked;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for locked exposure configuration %@", [error description]);
        }
    }
}

- (void) unlockExposure;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isExposureModeSupported:AVCaptureExposureModeContinuousAutoExposure]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.exposureMode = AVCaptureExposureModeContinuousAutoExposure;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for autoexposure configuration %@", [error description]);
        }
    }
}

- (void)lockBalance;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isWhiteBalanceModeSupported:AVCaptureWhiteBalanceModeLocked]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.whiteBalanceMode = AVCaptureWhiteBalanceModeLocked;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for locked white balance configuration %@", [error description]);
        }
    }
}

- (void) unlockBalance;
{
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    if ([device isWhiteBalanceModeSupported:AVCaptureWhiteBalanceModeContinuousAutoWhiteBalance]) {
        NSError *error = nil;
        if ([device lockForConfiguration:&error]) {
            device.whiteBalanceMode = AVCaptureWhiteBalanceModeContinuousAutoWhiteBalance;
            [device unlockForConfiguration];
        } else {
            NSLog(@"unable to lock device for auto white balance configuration %@", [error description]);
        }
    }
}

@end
```

## High-Level Overview

This is a .mm source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **CvAbstractCamera2**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `main`


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

