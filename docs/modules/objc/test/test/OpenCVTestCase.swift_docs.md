# Documentation for `modules/objc/test/test/OpenCVTestCase.swift`

## File Metadata

- **Full Path**: `modules/objc/test/test/OpenCVTestCase.swift`
- **File Name**: `OpenCVTestCase.swift`
- **File Size**: 11,263 bytes
- **File Type**: .swift
- **Link to Source**: [modules/objc/test/test/OpenCVTestCase.swift](../../../../modules/objc/test/test/OpenCVTestCase.swift)

## Purpose and Role

This file is located in the `modules/objc/test/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  OpenCVTestCase.swift
//
//  Created by Giles Payne on 2020/01/19.
//

import XCTest
import OpenCV

enum OpenCVTestError: Error {
    case unsupportedOperationError(String)
}

open class OpenCVTestCase: XCTestCase {

    //change to 'true' to unblock fail on fail("Not yet implemented")
    static let passNYI = true

    static let isTestCaseEnabled = true

    static let XFEATURES2D = "xfeatures2d"
    static let DEFAULT_FACTORY = "create"

    static let matSize: Int32 = 10
    static let EPS = 0.001
    static let FEPS:Float = 0.001
    static let weakEPS = 0.5
    static let weakFEPS:Float = 0.5

    var dst: Mat = Mat()
    var truth: Mat? = nil

    let colorBlack = Scalar(0)
    let colorWhite = Scalar(255, 255, 255)

    // Naming notation: <channels info>_[depth]_[dimensions]_value
    // examples: gray0 - single channel 8U 2d Mat filled with 0
    // grayRnd - single channel 8U 2d Mat filled with random numbers
    // gray0_32f_1d

    let gray0 = Mat(rows:matSize, cols:matSize, type:CvType.CV_8U, scalar:Scalar(0))
    let gray1 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(1))
    let gray2 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(2))
    let gray3 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(3))
    let gray9 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(9))
    let gray127 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(127))
    let gray128 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(128))
    let gray255 = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U, scalar: Scalar(255))
    let grayRnd = Mat(rows:matSize, cols:matSize, type: CvType.CV_8U)

    let gray_16u_256 = Mat(rows: matSize, cols: matSize, type: CvType.CV_16U, scalar: Scalar(256))
    let gray_16s_1024 = Mat(rows: matSize, cols: matSize, type: CvType.CV_16S, scalar: Scalar(1024))

    let gray0_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F, scalar: Scalar(0.0))
    let gray1_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F, scalar: Scalar(1.0))
    let gray3_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F, scalar: Scalar(3.0))
    let gray9_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F, scalar: Scalar(9.0))
    let gray255_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F, scalar: Scalar(255.0))
    let grayE_32f = Mat.eye(rows: matSize, cols: matSize, type: CvType.CV_32FC1)
    let grayRnd_32f = Mat(rows: matSize, cols: matSize, type: CvType.CV_32F)

    let gray0_64f = Mat(rows: matSize, cols: matSize, type: CvType.CV_64F, scalar: Scalar(0.0))
    let gray0_32f_1d = Mat(rows: 1, cols: matSize, type: CvType.CV_32F, scalar: Scalar(0.0))
    let gray0_64f_1d = Mat(rows: 1, cols: matSize, type: CvType.CV_64F, scalar: Scalar(0.0))

    let rgba0 = Mat(rows: matSize, cols: matSize, type: CvType.CV_8UC4, scalar: Scalar.all(0))
    let rgba128 = Mat(rows: matSize, cols: matSize, type: CvType.CV_8UC4, scalar: Scalar.all(128))

    let rgbLena: Mat = {
        return Imgcodecs.imread(filename: Bundle(for: OpenCVTestCase.self).path(forResource:"lena", ofType:"png", inDirectory:"resources")!)
    }()

    let grayChess: Mat = {
        return Imgcodecs.imread(filename: Bundle(for: OpenCVTestCase.self).path(forResource:"chessboard", ofType:"jpg", inDirectory:"resources")!, flags: ImreadModes.IMREAD_GRAYSCALE.rawValue)
    }()

    let gray255_32f_3d = Mat(sizes: [matSize, matSize, matSize] as [NSNumber], type: CvType.CV_32F, scalar: Scalar(255.0))

    let v1 = Mat(rows: 1, cols: 3, type: CvType.CV_32F)
    let v2 = Mat(rows: 1, cols: 3, type: CvType.CV_32F)

    override open func setUp() {
        //Core.setErrorVerbosity(false)
        Core.randu(dst: grayRnd, low: 0, high: 255)
        Core.randu(dst: grayRnd_32f, low:0, high: 255)
        do {
            try v1.put(row: 0,col: 0, data: [1.0, 3.0, 2.0])
            try v2.put(row: 0,col: 0, data: [2.0, 1.0, 3.0])
        } catch {
            print("Fatal error in start-up")
        }
    }

    override open func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
    }

    func assertMatEqual(_ expected:Mat, _ actual:Mat, file: StaticString = #file, line: UInt = #line) throws {
        try compareMats(expected, actual, true, file:file, line:line)
    }

    func assertMatNotEqual(_ expected:Mat, _ actual:Mat, file: StaticString = #file, line: UInt = #line) throws {
        try compareMats(expected, actual, false, file:file, line:line)
    }

    func assertMatEqual(_ expected:Mat, _ actual:Mat, _ eps:Double, file: StaticString = #file, line: UInt = #line) throws {
        try compareMats(expected, actual, eps, true, file:file, line:line);
    }

    func assertMatNotEqual(_ expected:Mat, _ actual:Mat, _ eps:Double, file: StaticString = #file, line: UInt = #line) throws {
        try compareMats(expected, actual, eps, false, file:file, line:line);
    }

    func assertSizeEquals(_ expected:Size,_ actual: Size, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.width, actual.width,  msg, file:file, line:line)
        XCTAssertEqual(expected.height, actual.height,  msg, file:file, line:line)
    }

    func assertSize2fEquals(_ expected:Size2f,_ actual: Size2f,_ eps: Float, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.width, actual.width, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.height, actual.height, accuracy:eps, msg, file:file, line:line)
    }

    func assertSize2dEquals(_ expected:Size2d,_ actual: Size2d,_ eps: Double, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.width, actual.width, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.height, actual.height, accuracy:eps, msg, file:file, line:line)
    }

    func assertPointEquals(_ expected:Point, _ actual: Point, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.x, actual.x, msg, file:file, line:line)
        XCTAssertEqual(expected.y, actual.y, msg, file:file, line:line)
    }

    func assertPoint2dEquals(_ expected:Point2d, _ actual: Point2d, _ eps: Double, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.x, actual.x, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.y, actual.y, accuracy:eps, msg, file:file, line:line)
    }

    func assertPoint2fEquals(_ expected:Point2f, _ actual: Point2f, _ eps: Float, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.x, actual.x, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.y, actual.y, accuracy:eps, msg, file:file, line:line)
    }

    func assertScalarEqual(_ expected:Scalar, _ actual:Scalar, _ eps: Double, file: StaticString = #file, line: UInt = #line) {
        let msg = "expected:<\(expected)> but was:<\(actual)>"
        XCTAssertEqual(expected.val[0] as! Double, actual.val[0] as! Double, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.val[1] as! Double, actual.val[1] as! Double, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.val[2] as! Double, actual.val[2] as! Double, accuracy:eps, msg, file:file, line:line)
        XCTAssertEqual(expected.val[3] as! Double, actual.val[3] as! Double, accuracy:eps, msg, file:file, line:line)
    }

    func assertArrayEquals(_ expected:[Double], _ actual:[Double], _ eps: Double, file: StaticString = #file, line: UInt = #line) {
        XCTAssertEqual(expected.count, actual.count, "Arrays have different sizes.", file:file, line:line)

        for i in 0..<expected.count {
            XCTAssertEqual(expected[i], actual[i], accuracy:eps, file:file, line:line)
        }
    }

    func compareMats(_ expected:Mat, _ actual:Mat, _ isEqualityMeasured:Bool, file: StaticString = #file, line: UInt = #line) throws {
        if expected.type() != actual.type() || !dimensionsEqual(expected, actual) {
            throw OpenCVTestError.unsupportedOperationError("Incompatible matrices")
        }

        if (expected.depth() == CvType.CV_32F || expected.depth() == CvType.CV_64F) {
            if isEqualityMeasured {
                throw OpenCVTestError.unsupportedOperationError("Floating-point Mats must not be checked for exact match. Use assertMatEqual(expected:Mat, actual:Mat, eps:Double) instead.")
            } else {
                throw OpenCVTestError.unsupportedOperationError("Floating-point Mats must not be checked for exact match. Use assertMatNotEqual(expected:Mat, actual:Mat, eps:Double) instead.")
            }
        }

        let diff = Mat()
        Core.absdiff(src1: expected, src2: actual, dst: diff);
        let reshaped = diff.reshape(channels: 1)
        let mistakes = Core.countNonZero(src: reshaped)

        if isEqualityMeasured {
            XCTAssertTrue(mistakes == 0, "Mats are different in \(mistakes) points", file:file, line:line)
        } else {
            XCTAssertFalse(mistakes == 0, "Mats are equal", file:file, line:line)
        }
    }

    func compareMats(_ expected: Mat, _ actual:Mat, _ eps:Double, _ isEqualityMeasured:Bool, file: StaticString = #file, line: UInt = #line) throws {
        if expected.type() != actual.type() || !dimensionsEqual(expected, actual) {
            throw OpenCVTestError.unsupportedOperationError("Incompatible matrices")
        }

        let diff = Mat()
        Core.absdiff(src1: expected, src2: actual, dst: diff)
        let maxDiff = Core.norm(src1: diff, normType: .NORM_INF)

        if isEqualityMeasured {
            XCTAssertTrue(maxDiff <= eps, "Max difference between expected and actual Mats is \(maxDiff), that bigger than \(eps)", file:file, line:line)
        } else {
            XCTAssertFalse(maxDiff <= eps, "Max difference between expected and actual Mats is \(maxDiff), that less than \(eps)", file:file, line:line)
        }
    }

    func dimensionsEqual(_ expected: Mat, _ actual: Mat) -> Bool {
        if expected.dims() != actual.dims() {
            return false
        }
        if expected.dims() > 2 {
            return (0..<expected.dims()).allSatisfy { expected.size($0) == actual.size($0) }
        } else {
            return expected.cols() == actual.cols() && expected.rows() == actual.rows();
        }
    }

    func getMat(_ type:Int32, vals:[NSNumber]) -> Mat {
        return Mat(rows:OpenCVTestCase.matSize, cols:OpenCVTestCase.matSize, type:type, scalar:Scalar(vals: vals))
    }

    func makeMask(_ mat:Mat, vals:[Double] = []) -> Mat {
        mat.submat(rowStart: 0, rowEnd: mat.rows(), colStart: 0, colEnd: mat.cols() / 2).setTo(scalar: Scalar(vals: vals as [NSNumber]))
        return mat
    }
}
```

## High-Level Overview

This is a .swift source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **OpenCVTestCase**: A class/struct defined in this file

### Functions and Methods

- **tearDown()**: A function/method defined in this file
- **assertMatNotEqual()**: A function/method defined in this file
- **assertPoint2fEquals()**: A function/method defined in this file
- **assertSize2dEquals()**: A function/method defined in this file
- **assertScalarEqual()**: A function/method defined in this file
- **assertMatEqual()**: A function/method defined in this file
- **compareMats()**: A function/method defined in this file
- **getMat()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **assertPointEquals()**: A function/method defined in this file
- **assertSizeEquals()**: A function/method defined in this file
- **assertPoint2dEquals()**: A function/method defined in this file
- **dimensionsEqual()**: A function/method defined in this file
- **assertSize2fEquals()**: A function/method defined in this file
- **makeMask()**: A function/method defined in this file
- **assertArrayEquals()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `OpenCV`
- `XCTest`


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

