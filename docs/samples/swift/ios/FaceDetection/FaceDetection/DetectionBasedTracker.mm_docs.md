# Documentation for `samples/swift/ios/FaceDetection/FaceDetection/DetectionBasedTracker.mm`

## File Metadata

- **Full Path**: `samples/swift/ios/FaceDetection/FaceDetection/DetectionBasedTracker.mm`
- **File Name**: `DetectionBasedTracker.mm`
- **File Size**: 2,523 bytes
- **File Type**: .mm
- **Link to Source**: [samples/swift/ios/FaceDetection/FaceDetection/DetectionBasedTracker.mm](../../../../../samples/swift/ios/FaceDetection/FaceDetection/DetectionBasedTracker.mm)

## Purpose and Role

This file is located in the `samples/swift/ios/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  DetectionBasedTracker.mm
//
//  Created by Giles Payne on 2020/04/05.
//

#import "DetectionBasedTracker.h"
#import "Mat.h"
#import "Rect2i.h"
#import "CVObjcUtil.h"

class CascadeDetectorAdapter: public cv::DetectionBasedTracker::IDetector
{
public:
    CascadeDetectorAdapter(cv::Ptr<cv::CascadeClassifier> detector):IDetector(), Detector(detector) {}

    void detect(const cv::Mat &Image, std::vector<cv::Rect> &objects)
    {
        Detector->detectMultiScale(Image, objects, scaleFactor, minNeighbours, 0, minObjSize, maxObjSize);
    }

    virtual ~CascadeDetectorAdapter() {}

private:
    CascadeDetectorAdapter();
    cv::Ptr<cv::CascadeClassifier> Detector;
};


struct DetectorAgregator
{
    cv::Ptr<CascadeDetectorAdapter> mainDetector;
    cv::Ptr<CascadeDetectorAdapter> trackingDetector;
    cv::Ptr<cv::DetectionBasedTracker> tracker;
    DetectorAgregator(cv::Ptr<CascadeDetectorAdapter>& _mainDetector, cv::Ptr<CascadeDetectorAdapter>& _trackingDetector):mainDetector(_mainDetector), trackingDetector(_trackingDetector) {
        CV_Assert(_mainDetector);
        CV_Assert(_trackingDetector);
        cv::DetectionBasedTracker::Parameters DetectorParams;
        tracker = cv::makePtr<cv::DetectionBasedTracker>(mainDetector, trackingDetector, DetectorParams);
    }
};

@implementation DetectionBasedTracker {
    DetectorAgregator* agregator;
}

- (instancetype)initWithCascadeName:(NSString*)cascadeName minFaceSize:(int)faceSize {
    self = [super init];
    if (self) {
        auto mainDetector = cv::makePtr<CascadeDetectorAdapter>(cv::makePtr<cv::CascadeClassifier>(cascadeName.UTF8String));
        auto trackingDetector = cv::makePtr<CascadeDetectorAdapter>(
            cv::makePtr<cv::CascadeClassifier>(cascadeName.UTF8String));
        agregator = new DetectorAgregator(mainDetector, trackingDetector);
        if (faceSize > 0) {
            agregator->mainDetector->setMinObjectSize(cv::Size(faceSize, faceSize));
        }
    }
    return self;
}

- (void)dealloc
{
    delete agregator;
}

- (void)start {
    agregator->tracker->run();
}

- (void)stop {
    agregator->tracker->stop();
}

- (void)setFaceSize:(int)size {
    agregator->mainDetector->setMinObjectSize(cv::Size(size, size));
}

- (void)detect:(Mat*)imageGray faces:(NSMutableArray<Rect2i*>*)faces {
    std::vector<cv::Rect> rectFaces;
    agregator->tracker->process(*((cv::Mat*)imageGray.nativePtr));
    agregator->tracker->getObjects(rectFaces);
    CV2OBJC(cv::Rect, Rect2i, rectFaces, faces);
}

@end
```

## High-Level Overview

This is a .mm source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **DetectorAgregator**: A class/struct defined in this file
- **CascadeDetectorAdapter**: A class/struct defined in this file


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

