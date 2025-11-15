# Documentation for `samples/swift/ios/ColorBlobDetection/ColorBlobDetection/ColorBlobDetector.swift`

## File Metadata

- **Full Path**: `samples/swift/ios/ColorBlobDetection/ColorBlobDetection/ColorBlobDetector.swift`
- **File Name**: `ColorBlobDetector.swift`
- **File Size**: 3,040 bytes
- **File Type**: .swift
- **Link to Source**: [samples/swift/ios/ColorBlobDetection/ColorBlobDetection/ColorBlobDetector.swift](../../../../../samples/swift/ios/ColorBlobDetection/ColorBlobDetection/ColorBlobDetector.swift)

## Purpose and Role

This file is located in the `samples/swift/ios/ColorBlobDetection/ColorBlobDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  ColorBlobDetector.swift
//
//  Created by Giles Payne on 2020/04/04.
//

import OpenCV

public class ColorBlobDetector {
    // Lower and Upper bounds for range checking in HSV color space
    var lowerBound = Scalar(0.0)
    var upperBound = Scalar(0.0)
    // Minimum contour area in percent for contours filtering
    static let minContourArea = 0.1
    // Color radius for range checking in HSV color space
    var colorRadius = Scalar(25.0, 50.0, 50.0, 0.0)
    let spectrum = Mat()
    var contours = [[Point]]()

    // Cache
    let pyrDownMat = Mat()
    let hsvMat = Mat()
    let mask = Mat()
    let dilatedMask = Mat()
    let hierarchy = Mat()

    func setHsvColor(hsvColor:Scalar) {
        let minH = (hsvColor.val[0].doubleValue >= colorRadius.val[0].doubleValue) ? hsvColor.val[0].doubleValue - colorRadius.val[0].doubleValue : 0
        let maxH = (hsvColor.val[0].doubleValue + colorRadius.val[0].doubleValue <= 255) ? hsvColor.val[0].doubleValue + colorRadius.val[0].doubleValue : 255

        lowerBound = Scalar(minH, hsvColor.val[1].doubleValue - colorRadius.val[1].doubleValue, hsvColor.val[2].doubleValue - colorRadius.val[2].doubleValue, 0)
        upperBound = Scalar(maxH, hsvColor.val[1].doubleValue + colorRadius.val[1].doubleValue, hsvColor.val[2].doubleValue + colorRadius.val[2].doubleValue, 255)

        let spectrumHsv = Mat(rows: 1, cols: (Int32)(maxH-minH), type:CvType.CV_8UC3);

        for j:Int32 in 0..<Int32(maxH - minH) {
            let tmp:[Double] = [Double(Int32(minH) + j), 255, 255]
            try! spectrumHsv.put(row: 0, col: j, data: tmp)
        }

        Imgproc.cvtColor(src: spectrumHsv, dst: spectrum, code: .COLOR_HSV2RGB_FULL, dstCn: 4)
    }

    func process(rgbaImage:Mat) {
        Imgproc.pyrDown(src: rgbaImage, dst: pyrDownMat)
        Imgproc.pyrDown(src: pyrDownMat, dst: pyrDownMat)

        Imgproc.cvtColor(src: pyrDownMat, dst: hsvMat, code: .COLOR_RGB2HSV_FULL)

        Core.inRange(src: hsvMat, lowerb: lowerBound, upperb: upperBound, dst: mask)
        Imgproc.dilate(src: mask, dst: dilatedMask, kernel: Mat())

        var contoursTmp = [[Point]]()

        Imgproc.findContours(image: dilatedMask, contours: &contoursTmp, hierarchy: hierarchy, mode: .RETR_EXTERNAL, method: .CHAIN_APPROX_SIMPLE)

        // Find max contour area
        var maxArea = 0.0
        for contour in contoursTmp {
            let contourMat = MatOfPoint(array: contour)
            let area = Imgproc.contourArea(contour: contourMat)
            maxArea = max(area, maxArea)
        }

        // Filter contours by area and resize to fit the original image size
        contours.removeAll()
        for contour in contoursTmp {
            let contourMat = MatOfPoint(array: contour)
            if (Imgproc.contourArea(contour: contourMat) > ColorBlobDetector.minContourArea * maxArea) {
                Core.multiply(src1: contourMat, srcScalar: Scalar(4.0,4.0), dst: contourMat)
                contours.append(contourMat.toArray())
            }
        }
    }
}
```

## High-Level Overview

This is a .swift source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **ColorBlobDetector**: A class/struct defined in this file

### Functions and Methods

- **setHsvColor()**: A function/method defined in this file
- **process()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
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

