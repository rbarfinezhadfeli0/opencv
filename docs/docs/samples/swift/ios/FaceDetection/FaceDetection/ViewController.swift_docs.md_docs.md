# Documentation for `docs/samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift_docs.md`

## File Metadata

- **Full Path**: `docs/samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift_docs.md`
- **File Name**: `ViewController.swift_docs.md`
- **File Size**: 6,332 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift_docs.md](../../../../../../docs/samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift_docs.md)

## Purpose and Role

This file is located in the `docs/samples/swift/ios/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift`

## File Metadata

- **Full Path**: `samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift`
- **File Name**: `ViewController.swift`
- **File Size**: 3,098 bytes
- **File Type**: .swift
- **Link to Source**: [samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift](../../../../../samples/swift/ios/FaceDetection/FaceDetection/ViewController.swift)

## Purpose and Role

This file is located in the `samples/swift/ios/FaceDetection/FaceDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  ViewController.swift
//
//  Created by Giles Payne on 2020/03/02.
//

import UIKit
import OpenCV

extension Rect {
    func rotateClockwise(parentHeight:Int32) {
        let tmpX = self.x
        self.x = parentHeight - (self.y + self.height)
        self.y = tmpX
        swapDims()
    }

    func rotateCounterclockwise(parentWidth:Int32) {
        let tmpY = self.y
        self.y = parentWidth - (self.x + self.width)
        self.x = tmpY
        swapDims()
    }

    func swapDims() {
        let tmpWidth = self.width
        self.width = self.height
        self.height = tmpWidth
    }
}

class ViewController: UIViewController, CvVideoCameraDelegate2 {

    let swiftDetector = CascadeClassifier(filename: Bundle(for: ViewController.self).path(forResource:"lbpcascade_frontalface", ofType:"xml")!)
    let nativeDetector = DetectionBasedTracker(cascadeName: Bundle(for: ViewController.self).path(forResource:"lbpcascade_frontalface", ofType:"xml")!, minFaceSize: 0)
    var rgba: Mat? = nil
    var gray: Mat = Mat()
    var relativeFaceSize: Float = 0.2
    var absoluteFaceSize: Int32 = 0
    let FACE_RECT_COLOR = Scalar(0.0, 255.0, 0.0, 255.0)
    let FACE_RECT_THICKNESS: Int32 = 4

    func processImage(_ image: Mat!) {
        let orientation = UIDevice.current.orientation
        switch orientation {
        case .landscapeLeft:
            rgba = Mat()
            Core.rotate(src: image, dst: rgba!, rotateCode: .ROTATE_90_COUNTERCLOCKWISE)
        case .landscapeRight:
            rgba = Mat()
            Core.rotate(src: image, dst: rgba!, rotateCode: .ROTATE_90_CLOCKWISE)
        default:
            rgba = image
        }

        Imgproc.cvtColor(src: rgba!, dst: gray, code: .COLOR_RGB2GRAY)

        if (absoluteFaceSize == 0) {
            let height = gray.rows()
            if (round(Float(height) * relativeFaceSize) > 0) {
                absoluteFaceSize = Int32(round(Float(height) * relativeFaceSize))
            }
        }

        var faces = [Rect]()

        swiftDetector.detectMultiScale(image: gray, objects: &faces, scaleFactor: 1.1, minNeighbors: Int32(2), flags: Int32(2), minSize: Size(width: absoluteFaceSize, height: absoluteFaceSize), maxSize: Size())
        //let facesArray = NSMutableArray()
        //nativeDetector!.detect(gray, faces: facesArray)
        //faces.append(contentsOf: facesArray)

        for face in faces {
            if orientation == .landscapeLeft {
                face.rotateClockwise(parentHeight: gray.rows())
            } else if orientation == .landscapeRight {
                face.rotateCounterclockwise(parentWidth: gray.cols())
            }
            Imgproc.rectangle(img: image, pt1: face.tl(), pt2: face.br(), color: FACE_RECT_COLOR, thickness: FACE_RECT_THICKNESS)
        }
    }

    var camera: CvVideoCamera2? = nil

    @IBOutlet weak var cameraHolder: UIView!
    override func viewDidLoad() {
        super.viewDidLoad()
        camera = CvVideoCamera2(parentView: cameraHolder)
        camera?.rotateVideo = true
        camera?.delegate = self
        camera?.start()
    }
}
```

## High-Level Overview

This is a .swift source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **ViewController**: A class/struct defined in this file

### Functions and Methods

- **rotateCounterclockwise()**: A function/method defined in this file
- **rotateClockwise()**: A function/method defined in this file
- **swapDims()**: A function/method defined in this file
- **processImage()**: A function/method defined in this file
- **viewDidLoad()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `UIKit`
- `OpenCV`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

