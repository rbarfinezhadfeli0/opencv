# Documentation for `modules/core/misc/objc/test/MatTest.swift`

## File Metadata

- **Full Path**: `modules/core/misc/objc/test/MatTest.swift`
- **File Name**: `MatTest.swift`
- **File Size**: 51,836 bytes
- **File Type**: .swift
- **Link to Source**: [modules/core/misc/objc/test/MatTest.swift](../../../../../modules/core/misc/objc/test/MatTest.swift)

## Purpose and Role

This file is located in the `modules/core/misc/objc/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  StitchAppTests.swift
//
//  Created by Giles Payne on 2020/01/19.
//

import XCTest
import OpenCV

class MatTests: OpenCVTestCase {

    override func setUp() {
        super.setUp()
    }

    override func tearDown() {
        super.tearDown()
    }

    func testInitWithSize() {
        let size = Size(width: 7, height: 9)
        let mat = Mat(size: size, type: CvType.CV_8U)
        assertSizeEquals(size, mat.size())
    }

    func testAdjustROI() throws {
        let roi = gray0.submat(rowStart: 3, rowEnd: 5, colStart: 7, colEnd: 10)
        let originalroi = roi.clone()
        let adjusted = roi.adjustRoi(top: 2, bottom: 2, left: 2, right: 2)
        try assertMatEqual(adjusted, roi)
        assertSizeEquals(Size(width: 5, height: 6), adjusted.size())
        XCTAssertEqual(originalroi.type(), adjusted.type())
        XCTAssertTrue(adjusted.isSubmatrix())
        XCTAssertFalse(adjusted.isContinuous())

        let offset = Point()
        let size = Size()
        adjusted.locateROI(wholeSize: size, offset: offset)
        assertPointEquals(Point(x: 5, y: 1), offset);
        assertSizeEquals(gray0.size(), size);
    }

    func testAssignToMat() throws {
        gray0.assign(to: dst)
        try assertMatEqual(gray0, dst)
        gray255.assign(to: dst)
        try assertMatEqual(gray255, dst)
    }

    func testAssignToMatInt() throws {
        gray255.assign(to: dst, type: CvType.CV_32F)
        try assertMatEqual(gray255_32f, dst, OpenCVTestCase.EPS)
    }

    func testChannels() {
        XCTAssertEqual(1, gray0.channels())
        XCTAssertEqual(3, rgbLena.channels())
        XCTAssertEqual(4, rgba0.channels())
    }

    func testCheckVectorInt() {
        // ! returns N if the matrix is 1-channel (N x ptdim) or ptdim-channel
        // (1 x N) or (N x 1); negative number otherwise
        XCTAssertEqual(2, Mat(rows: 2, cols: 10, type: CvType.CV_8U).checkVector(elemChannels: 10))
        XCTAssertEqual(2, Mat(rows: 1, cols: 2, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10))
        XCTAssertEqual(2, Mat(rows: 2, cols: 1, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10))
        XCTAssertEqual(10, Mat(rows: 1, cols: 10, type: CvType.CV_8UC2).checkVector(elemChannels: 2))

        XCTAssert(0 > Mat().checkVector(elemChannels: 0))
        XCTAssert(0 > Mat(rows: 10, cols: 1, type: CvType.CV_8U).checkVector(elemChannels: 10))
        XCTAssert(0 > Mat(rows: 10, cols: 20, type: CvType.CV_8U).checkVector(elemChannels: 10))
    }

    func testCheckVectorIntInt() {
        XCTAssertEqual(2, Mat(rows: 2, cols: 10, type: CvType.CV_8U).checkVector(elemChannels: 10, depth: CvType.CV_8U))
        XCTAssertEqual(2, Mat(rows: 1, cols: 2, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10, depth: CvType.CV_8U))
        XCTAssertEqual(2, Mat(rows: 2, cols: 1, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10, depth: CvType.CV_8U))
        XCTAssertEqual(10, Mat(rows: 1, cols: 10, type: CvType.CV_8UC2).checkVector(elemChannels: 2, depth: CvType.CV_8U))

        XCTAssert(0 > Mat(rows: 2, cols: 10, type: CvType.CV_8U).checkVector(elemChannels: 10, depth: CvType.CV_8S));
        XCTAssert(0 > Mat(rows: 1, cols: 2, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10, depth: CvType.CV_8S));
        XCTAssert(0 > Mat(rows: 2, cols: 1, type: CvType.CV_8UC(10)).checkVector(elemChannels: 10, depth: CvType.CV_8S));
        XCTAssert(0 > Mat(rows: 1, cols: 10, type: CvType.CV_8UC2).checkVector(elemChannels: 10, depth: CvType.CV_8S));
    }

    func testCheckVectorIntIntBoolean() {
        let mm = Mat(rows: 5, cols: 1, type: CvType.CV_8UC(10))
        let roi = Mat(rows: 5, cols: 3, type: CvType.CV_8UC(10)).submat(rowStart: 1, rowEnd: 3, colStart: 2, colEnd: 3);

        XCTAssertEqual(5, mm.checkVector(elemChannels: 10, depth: CvType.CV_8U, requireContinuous: true));
        XCTAssertEqual(5, mm.checkVector(elemChannels: 10, depth: CvType.CV_8U, requireContinuous: false));
        XCTAssertEqual(2, roi.checkVector(elemChannels: 10, depth: CvType.CV_8U, requireContinuous: false));
        XCTAssert(0 > roi.checkVector(elemChannels: 10, depth: CvType.CV_8U, requireContinuous: true));
    }

    func testClone() throws {
        dst = gray0.clone()
        try assertMatEqual(gray0, dst)
        XCTAssertFalse(dst.isSameMat(gray0))
    }

    func testCol() {
        let col = gray0.col(0)
        XCTAssertEqual(1, col.cols())
        XCTAssertEqual(gray0.rows(), col.rows())
    }

    func testColRangeIntInt() {
        let cols = gray0.colRange(start: 0, end: gray0.cols() / 2)
        XCTAssertEqual(gray0.cols() / 2, cols.cols())
        XCTAssertEqual(gray0.rows(), cols.rows())
    }

    func testColRangeRange() throws {
        let range = Range(start: 0, end: 5)
        dst = gray0.colRange(range)

        truth = Mat(rows: 10, cols: 5, type: CvType.CV_8UC1, scalar: Scalar(0.0))
        try assertMatEqual(truth!, dst)
    }

    func testCols() {
        XCTAssertEqual(OpenCVTestCase.matSize, gray0.cols())
    }

    func testConvertToMatInt() throws {
        gray255.convert(to: dst, rtype: CvType.CV_32F)

        truth = Mat(rows: OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_32F, scalar: Scalar(255));
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testConvertToMatIntDouble() throws {
        gray2.convert(to: dst, rtype: CvType.CV_16U, alpha: 2.0)

        truth = Mat(rows: OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_16U, scalar: Scalar(4))
        try assertMatEqual(truth!, dst)
    }

    func testConvertToMatIntDoubleDouble() throws {
        gray0_32f.convert(to: dst, rtype: CvType.CV_8U, alpha: 2.0, beta: 4.0)

        truth = Mat(rows: OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_8U, scalar: Scalar(4))
        try assertMatEqual(truth!, dst)
    }

    func testCopyToMat() throws {
        rgbLena.copy(to:dst)
        try assertMatEqual(rgbLena, dst)
    }

    func testCopyToMatMat() throws {
        let src = Mat(rows: 4, cols: 4, type: CvType.CV_8U, scalar: Scalar(5))
        let mask = makeMask(src.clone())

        src.copy(to: dst, mask: mask)

        truth = Mat(rows: 4, cols: 4, type: CvType.CV_8U)
        try XCTAssertEqual(truth!.put(row: 0, col: 0, data: [0, 0, 5, 5] as [Int8]), 4)
        try XCTAssertEqual(truth!.put(row: 1, col: 0, data: [0, 0, 5, 5] as [Int8]), 4)
        try XCTAssertEqual(truth!.put(row: 2, col: 0, data: [0, 0, 5, 5] as [Int8]), 4)
        try XCTAssertEqual(truth!.put(row: 3, col: 0, data: [0, 0, 5, 5] as [Int8]), 4)
        try assertMatEqual(truth!, dst)
    }

    func testCreateIntIntInt() {
        gray255.create(rows: 4, cols: 5, type: CvType.CV_32F)

        XCTAssertEqual(4, gray255.rows())
        XCTAssertEqual(5, gray255.cols())
        XCTAssertEqual(CvType.CV_32F, gray255.type())
    }

    func testCreateSizeInt() {
        let size = Size(width: 5, height: 5)
        dst.create(size: size, type: CvType.CV_16U)

        XCTAssertEqual(5, dst.rows())
        XCTAssertEqual(5, dst.cols())
        XCTAssertEqual(CvType.CV_16U, dst.type())
    }

    func testCreateIntArrayInt() {
        dst.create(sizes:[5, 6, 7], type:CvType.CV_16U)

        XCTAssertEqual(5, dst.size(0))
        XCTAssertEqual(6, dst.size(1))
        XCTAssertEqual(7, dst.size(2))
        XCTAssertEqual(CvType.CV_16U, dst.type())
    }

    func testCross() throws {
        let answer = Mat(rows: 1, cols: 3, type: CvType.CV_32F)
        try XCTAssertEqual(answer.put(row: 0, col: 0, data: [7.0, 1.0, -5.0] as [Float]), 12)

        let cross = v1.cross(v2)
        try assertMatEqual(answer, cross, OpenCVTestCase.EPS)
    }

    func testDepth() {
        XCTAssertEqual(CvType.CV_8U, gray0.depth())
        XCTAssertEqual(CvType.CV_32F, gray0_32f.depth())
    }

    func testDiag() throws {
        dst = gray0.diag()
        truth = Mat(rows: 10, cols: 1, type: CvType.CV_8UC1, scalar: Scalar(0))
        try assertMatEqual(truth!, dst)
    }

    func testDiagInt() throws {
        dst = gray255.diag(2)
        truth = Mat(rows: 8, cols: 1, type: CvType.CV_8UC1, scalar: Scalar(255))
        try assertMatEqual(truth!, dst)
    }

    func testDiagMat() throws {
        let diagVector = Mat(rows: OpenCVTestCase.matSize, cols: 1, type: CvType.CV_32F, scalar: Scalar(1))
        dst = Mat.diag(diagVector)
        try assertMatEqual(grayE_32f, dst, OpenCVTestCase.EPS);
    }

    func testDot() {
        let s = v1.dot(v2)
        XCTAssertEqual(11.0, s)
    }

    func testDump() {
        XCTAssertEqual("[1, 3, 2]", v1.dump())
    }

    func testElemSize() {
        XCTAssertEqual(MemoryLayout<UInt8>.size * Int(gray0.channels()), gray0.elemSize())
        XCTAssertEqual(MemoryLayout<Float>.size * Int(gray0_32f.channels()), gray0_32f.elemSize())
        XCTAssertEqual(MemoryLayout<UInt8>.size * Int(rgbLena.channels()), rgbLena.elemSize())
    }

    func testElemSize1() {
        XCTAssertEqual(MemoryLayout<UInt8>.size, gray255.elemSize1())
        XCTAssertEqual(MemoryLayout<Double>.size, gray0_64f.elemSize1())
        XCTAssertEqual(MemoryLayout<UInt8>.size, rgbLena.elemSize1())
    }

    func testEmpty() {
        XCTAssert(dst.empty())
        XCTAssertFalse(gray0.empty())
    }

    func testEyeIntIntInt() throws {
        let eye = Mat.eye(rows: 3, cols: 3, type: CvType.CV_32FC1)
        try assertMatEqual(eye, eye.inv(), OpenCVTestCase.EPS)
    }

    func testEyeSizeInt() {
        let size = Size(width: 5, height: 5)
        let eye = Mat.eye(size: size, type: CvType.CV_32S)
        XCTAssertEqual(5, Core.countNonZero(src: eye))
    }

    func getTestMat(size:Int32, type:Int32) throws -> Mat {
        let ret = Mat(rows: size, cols: size, type: type)
        let ch = CvType.channels(type)
        var buff:[Double] = []
        for i: Int32 in (0..<size) {
            for j: Int32 in (0..<size) {
                for k: Int32 in (0..<ch) {
                    buff.append(Double(100 * i + 10 * j + k))
                }
            }
        }
        try _ = ret.put(row:0, col:0, data:buff)
        return ret
    }

    func testGetIntInt_8U() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_8UC2)

        // whole Mat
        XCTAssert([0.0, 1.0] == m.get(row: 0, col: 0))
        XCTAssert([240, 241] == m.get(row: 2, col: 4))
        XCTAssert([255, 255] == m.get(row: 4, col: 4))

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5);
        XCTAssert([230, 231] == sm.get(row: 0, col: 0))
        XCTAssert([255, 255] == sm.get(row: 1, col: 1))
    }

    func testGetIntInt_32S() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_32SC3)

        // whole Mat
        XCTAssert([0, 1, 2] == m.get(row: 0, col: 0))
        XCTAssert([240, 241, 242] == m.get(row: 2, col: 4))
        XCTAssert([440, 441, 442] == m.get(row: 4, col: 4))

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5);
        XCTAssert([230, 231, 232] == sm.get(row: 0, col: 0));
        XCTAssert([340, 341, 342] == sm.get(row: 1, col: 1));
    }

    func testGetIntInt_64F() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_64FC1)

        // whole Mat
        XCTAssert([0] ==  m.get(row: 0, col: 0))
        XCTAssert([240] == m.get(row: 2, col: 4))
        XCTAssert([440] == m.get(row: 4, col: 4))

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        XCTAssert([230] == sm.get(row: 0, col: 0))
        XCTAssert([340] == sm.get(row: 1, col: 1))
    }

    func testGetIntIntInt8Array() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_8SC3)
        var goodData = [Int8](repeating: 0, count: 9)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &goodData)

        XCTAssertEqual(9, bytesNum)
        XCTAssert([110, 111, 112, 120, 121, 122, 127, 127, 127] == goodData)

        var badData = [Int8](repeating: 0, count: 7)
        XCTAssertThrowsError(bytesNum = try m.get(row: 0, col: 0, data: &badData))

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [Int8](repeating: 0, count: 3)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(3, bytesNum)
        XCTAssert(buff00 == [127, 127, 127])
        var buff11 = [Int8](repeating: 0, count: 3)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(3, bytesNum)
        XCTAssert(buff11 == [127, 127, 127])
    }

    func testGetIntIntUInt8Array() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_8UC3)
        var goodData = [UInt8](repeating: 0, count: 9)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &goodData)

        XCTAssertEqual(9, bytesNum)
        XCTAssert([110, 111, 112, 120, 121, 122, 130, 131, 132] == goodData)

        var badData = [UInt8](repeating: 0, count: 7)
        XCTAssertThrowsError(bytesNum = try m.get(row: 0, col: 0, data: &badData))

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [UInt8](repeating: 0, count: 3)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(3, bytesNum)
        XCTAssert(buff00 == [230, 231, 232])
        var buff11 = [UInt8](repeating: 0, count: 3)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(3, bytesNum)
        XCTAssert(buff11 == [255, 255, 255])
    }

    func testGetIntIntDoubleArray() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_64F)
        var buff = [Double](repeating: 0, count: 4)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &buff)

        XCTAssertEqual(32, bytesNum)
        XCTAssert(buff == [110, 120, 130, 140])

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [Double](repeating: 0, count: 2)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(16, bytesNum)
        XCTAssert(buff00 == [230, 240])
        var buff11 = [Double](repeating: 0, count: 2)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(8, bytesNum)
        XCTAssert(buff11 == [340, 0])
    }

    func testGetIntIntFloatArray() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_32F)
        var buff = [Float](repeating: 0, count: 4)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &buff)

        XCTAssertEqual(16, bytesNum)
        XCTAssert(buff == [110, 120, 130, 140])

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [Float](repeating: 0, count: 2)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(8, bytesNum);
        XCTAssert(buff00 == [230, 240])
        var buff11 = [Float](repeating: 0, count: 2)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(4, bytesNum);
        XCTAssert(buff11 == [340, 0])
    }

    func testGetIntIntIntArray() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_32SC2)
        var buff = [Int32](repeating: 0, count: 6)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &buff)

        XCTAssertEqual(24, bytesNum)
        XCTAssert(buff == [110, 111, 120, 121, 130, 131])

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [Int32](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(16, bytesNum)
        XCTAssert(buff00 == [230, 231, 240, 241])
        var buff11 = [Int32](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(8, bytesNum)
        XCTAssert(buff11 == [340, 341, 0, 0])
    }

    func testGetIntIntInt16Array() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_16SC2)
        var buff = [Int16](repeating: 0, count: 6)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &buff)

        XCTAssertEqual(12, bytesNum)
        XCTAssert(buff == [110, 111, 120, 121, 130, 131])

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [Int16](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(8, bytesNum)
        XCTAssert(buff00 == [230, 231, 240, 241])
        var buff11 = [Int16](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(4, bytesNum)
        XCTAssert(buff11 == [340, 341, 0, 0])

        let m2 = Mat(sizes: [5, 6, 8], type: CvType.CV_16S)
        let data:[Int16] = (0..<m2.total()).map { Int16($0) }
        try m2.put(indices: [0, 0, 0], data: data)
        let matNonContinuous = m2.submat(ranges:[Range(start:1, end:4), Range(start:2, end:5), Range(start:3, end:6)])
        let matContinuous = matNonContinuous.clone()
        var outNonContinuous = [Int16](repeating:0, count: matNonContinuous.total())
        try matNonContinuous.get(indices: [0, 0, 0], data: &outNonContinuous)
        var outContinuous = [Int16](repeating: 0, count:matNonContinuous.total())
        try matContinuous.get(indices: [0, 0, 0], data: &outContinuous)
        XCTAssertEqual(outNonContinuous, outContinuous)
        let subMat2 = m2.submat(ranges:[Range(start:1, end:4), Range(start:1, end:5), Range(start:0, end:8)])
        let subMatClone2 = subMat2.clone()
        var outNonContinuous2 = [Int16](repeating:0, count: subMat2.total())
        try subMat2.get(indices: [0, 1, 1], data: &outNonContinuous2)
        var outContinuous2 = [Int16](repeating:0, count:subMat2.total())
        try subMatClone2.get(indices: [0, 1, 1], data: &outContinuous2)
        XCTAssertEqual(outNonContinuous2, outContinuous2)
    }

    func testGetIntIntUInt16Array() throws {
        let m = try getTestMat(size: 5, type: CvType.CV_16UC2)
        var buff = [UInt16](repeating: 0, count: 6)

        // whole Mat
        var bytesNum = try m.get(row: 1, col: 1, data: &buff)

        XCTAssertEqual(12, bytesNum);
        XCTAssert(buff == [110, 111, 120, 121, 130, 131])

        // sub-Mat
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff00 = [UInt16](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff00)
        XCTAssertEqual(8, bytesNum)
        XCTAssert(buff00 == [230, 231, 240, 241])
        var buff11 = [UInt16](repeating: 0, count: 4)
        bytesNum = try sm.get(row: 1, col: 1, data: &buff11)
        XCTAssertEqual(4, bytesNum);
        XCTAssert(buff11 == [340, 341, 0, 0])
    }

    func testHeight() {
        XCTAssertEqual(gray0.rows(), gray0.height())
        XCTAssertEqual(rgbLena.rows(), rgbLena.height())
        XCTAssertEqual(rgba128.rows(), rgba128.height())
    }

    func testInv() throws {
        dst = grayE_32f.inv()
        try assertMatEqual(grayE_32f, dst, OpenCVTestCase.EPS)
    }

    func testInvInt() throws {
        let src = Mat(rows: 2, cols: 2, type: CvType.CV_32F)
        try XCTAssertEqual(src.put(row: 0, col: 0, data: [1.0] as [Float]), 4)
        try XCTAssertEqual(src.put(row: 0, col: 1, data: [2.0] as [Float]), 4)
        try XCTAssertEqual(src.put(row: 1, col: 0, data: [1.5] as [Float]), 4)
        try XCTAssertEqual(src.put(row: 1, col: 1, data: [4.0] as [Float]), 4)

        dst = src.inv(DecompTypes.DECOMP_CHOLESKY.rawValue)

        truth = Mat(rows: 2, cols: 2, type: CvType.CV_32F)
        try XCTAssertEqual(truth!.put(row: 0, col: 0, data: [4.0]), 4)
        try XCTAssertEqual(truth!.put(row: 0, col: 1, data: [-2.0]), 4)
        try XCTAssertEqual(truth!.put(row: 1, col: 0, data: [-1.5]), 4)
        try XCTAssertEqual(truth!.put(row: 1, col: 1, data: [1.0]), 4)

        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testIsContinuous() {
        XCTAssert(gray0.isContinuous())

        let subMat = gray0.submat(rowStart: 0, rowEnd: gray0.rows() / 2, colStart: 0, colEnd: gray0.cols() / 2)
        XCTAssertFalse(subMat.isContinuous())
    }

    func testIsSubmatrix() {
        XCTAssertFalse(gray0.isSubmatrix())
        let subMat = gray0.submat(rowStart: 0, rowEnd: gray0.rows() / 2, colStart: 0, colEnd: gray0.cols() / 2)
        XCTAssert(subMat.isSubmatrix())
    }

    func testLocateROI() {
        let roi = gray0.submat(rowStart: 3, rowEnd: 5, colStart: 7, colEnd: 10)
        let offset = Point()
        let size = Size()

        roi.locateROI(wholeSize: size, offset: offset)

        assertPointEquals(Point(x: 7, y: 3), offset)
        assertSizeEquals(Size(width: 10, height: 10), size)
    }

    func testMat() {
        let m = Mat()
        XCTAssertNotNil(m)
        XCTAssert(m.empty())
    }

    func testMatIntIntCvType() {
        let gray = Mat(rows: 1, cols: 1, type: CvType.CV_8UC1)
        XCTAssertFalse(gray.empty())

        let rgb = Mat(rows: 1, cols: 1, type: CvType.CV_8UC3)
        XCTAssertFalse(rgb.empty())
    }

    func testMatIntIntCvTypeScalar() throws {
        dst = Mat(rows: gray127.rows(), cols: gray127.cols(), type: CvType.CV_8U, scalar: Scalar(127))
        XCTAssertFalse(dst.empty())
        try assertMatEqual(dst, gray127)

        dst = Mat(rows: rgba128.rows(), cols: rgba128.cols(), type: CvType.CV_8UC4, scalar: Scalar.all(128))
        XCTAssertFalse(dst.empty())
        try assertMatEqual(dst, rgba128)
    }

    func testMatIntIntInt() {
        let gray = Mat(rows: 1, cols: 1, type: CvType.CV_8U)
        XCTAssertFalse(gray.empty())

        let rgb = Mat(rows: 1, cols: 1, type: CvType.CV_8U)
        XCTAssertFalse(rgb.empty())
    }

    func testMatIntIntIntScalar() throws {
        let m1 = Mat(rows: gray127.rows(), cols: gray127.cols(), type: CvType.CV_8U, scalar: Scalar(127))
        XCTAssertFalse(m1.empty())
        try assertMatEqual(m1, gray127)

        let m2 = Mat(rows: gray0_32f.rows(), cols: gray0_32f.cols(), type: CvType.CV_32F, scalar: Scalar(0))
        XCTAssertFalse(m2.empty())
        try assertMatEqual(m2, gray0_32f, OpenCVTestCase.EPS)
    }

    func testMatMatRange() throws {
        dst = Mat(mat: gray0, rowRange: Range(start: 0, end: 5))

        truth = Mat(rows: 5, cols: 10, type: CvType.CV_8UC1, scalar: Scalar(0))
        XCTAssertFalse(dst.empty())
        try assertMatEqual(truth!, dst)
    }

    func testMatMatRangeRange() throws {
        dst = Mat(mat: gray255_32f, rowRange: Range(start: 0, end: 5), colRange: Range(start: 0, end: 5))

        truth = Mat(rows: 5, cols: 5, type: CvType.CV_32FC1, scalar: Scalar(255))

        XCTAssertFalse(dst.empty())
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testMatMatRangeArray() throws {
        dst = Mat(mat: gray255_32f_3d, ranges: [Range(start: 0, end: 5), Range(start: 0, end: 5), Range(start: 0, end: 5)])

        truth = Mat(sizes:[5, 5, 5], type:CvType.CV_32FC1, scalar:Scalar(255))

        XCTAssertFalse(dst.empty())
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testMatMatRect() throws {
        let m = Mat(rows: 7, cols: 6, type: CvType.CV_32SC1)
        try XCTAssertEqual(m.put(row: 0, col: 0,
              data: [ 0,  1,  2,  3,  4,  5,
                     10, 11, 12, 13, 14, 15,
                     20, 21, 22, 23, 24, 25,
                     30, 31, 32, 33, 34, 35,
                     40, 41, 42, 43, 44, 45,
                     50, 51, 52, 53, 54, 55,
                     60, 61, 62, 63, 64, 65] as [Int32]), 168)

        dst = Mat(mat: m, rect: Rect(x: 1, y: 2, width: 3, height: 4))

        truth = Mat(rows: 4, cols: 3, type: CvType.CV_32SC1)
        try XCTAssertEqual(truth!.put(row: 0, col: 0,
                   data: [21, 22, 23,
                          31, 32, 33,
                          41, 42, 43,
                          51, 52, 53] as [Int32]), 48)

        XCTAssertFalse(dst.empty())
        try assertMatEqual(truth!, dst)
    }

    func testMatSizeInt() {
        dst = Mat(size: Size(width: 10, height: 10), type: CvType.CV_8U)

        XCTAssertFalse(dst.empty())
    }

    func testMatSizeIntScalar() throws {
        dst = Mat(size: Size(width: 10, height: 10), type: CvType.CV_32F, scalar: Scalar(255))

        XCTAssertFalse(dst.empty())
        try assertMatEqual(gray255_32f, dst, OpenCVTestCase.EPS)
    }

    func testMatIntArrayIntScalar() throws {
        dst = Mat(sizes:[10, 10, 10], type:CvType.CV_32F, scalar:Scalar(255))

        XCTAssertFalse(dst.empty());
        try assertMatEqual(gray255_32f_3d, dst, OpenCVTestCase.EPS)
    }

    func testMulMat() throws {
        try assertMatEqual(gray0, gray0.mul(gray255))

        let m1 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(2))
        let m2 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(3))

        dst = m1.mul(m2)

        truth = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(6))
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testMulMat3d() throws {
        let m1 = Mat(sizes: [2, 2, 2], type: CvType.CV_32F, scalar: Scalar(2))
        let m2 = Mat(sizes: [2, 2, 2], type: CvType.CV_32F, scalar: Scalar(3))

        dst = m1.mul(m2)

        truth = Mat(sizes: [2, 2, 2], type: CvType.CV_32F, scalar: Scalar(6))
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testMulMatDouble() throws{
        let m1 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(2))
        let m2 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(3))

        dst = m1.mul(m2, scale: 3.0)

        truth = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(18))
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testMatMulMat() throws {
        let m1 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(2))
        let m2 = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(3))

        dst = m1.matMul(m2)

        truth = Mat(rows: 2, cols: 2, type: CvType.CV_32F, scalar: Scalar(12))
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testOnesIntIntInt() throws {
        dst = Mat.ones(rows: OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_32F)

        truth = Mat(rows: OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_32F, scalar: Scalar(1))
        try assertMatEqual(truth!, dst, OpenCVTestCase.EPS)
    }

    func testOnesSizeInt() throws {
        dst = Mat.ones(size: Size(width: 2, height: 2), type: CvType.CV_16S)
        truth = Mat(rows: 2, cols: 2, type: CvType.CV_16S, scalar: Scalar(1))
        try assertMatEqual(truth!, dst)
    }

    func testOnesIntArrayInt() throws {
        dst = Mat.ones(sizes: [2, 2, 2], type: CvType.CV_16S)
        truth = Mat(sizes: [2, 2, 2], type: CvType.CV_16S, scalar: Scalar(1))
        try assertMatEqual(truth!, dst)
    }

    func testPush_back() throws {
        let m1 = Mat(rows: 2, cols: 4, type: CvType.CV_32F, scalar: Scalar(2))
        let m2 = Mat(rows: 3, cols: 4, type: CvType.CV_32F, scalar: Scalar(3))

        m1.push_back(m2)

        truth = Mat(rows: 5, cols: 4, type: CvType.CV_32FC1)
        try XCTAssertEqual(truth!.put(row: 0, col: 0, data: [2, 2, 2, 2] as [Float]), 16)
        try XCTAssertEqual(truth!.put(row: 1, col: 0, data: [2, 2, 2, 2] as [Float]), 16)
        try XCTAssertEqual(truth!.put(row: 2, col: 0, data: [3, 3, 3, 3] as [Float]), 16)
        try XCTAssertEqual(truth!.put(row: 3, col: 0, data: [3, 3, 3, 3] as [Float]), 16)
        try XCTAssertEqual(truth!.put(row: 4, col: 0, data: [3, 3, 3, 3] as [Float]), 16)

        try assertMatEqual(truth!, m1, OpenCVTestCase.EPS)
    }

    func testPutIntIntInt8Array() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_8SC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff = [Int8](repeating: 0, count: 6)
        let buff0:[Int8] = [10, 20, 30, 40, 50, 60]
        let buff1:[Int8] = [-1, -2, -3, -4, -5, -6]

        var bytesNum = try m.put(row:1, col:2, data:buff0)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(row: 1, col: 2, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)

        bytesNum = try sm.put(row:0, col:0, data:buff1)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)
        bytesNum = try m.get(row: 2, col: 3, data: &buff)
        XCTAssertEqual(6, bytesNum);
        XCTAssert(buff == buff1)

        let m1 = m.row(1)
        bytesNum = try m1.get(row: 0, col: 2, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)
    }

    func testPutIntIntUInt8Array() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_8UC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff = [UInt8](repeating: 0, count: 6)
        let buff0:[UInt8] = [10, 20, 30, 40, 50, 60]
        let buff1:[UInt8] = [255, 254, 253, 252, 251, 250]

        var bytesNum = try m.put(row:1, col:2, data:buff0)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(row: 1, col: 2, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)

        bytesNum = try sm.put(row:0, col:0, data:buff1)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)
        bytesNum = try m.get(row: 2, col: 3, data: &buff)
        XCTAssertEqual(6, bytesNum);
        XCTAssert(buff == buff1)

        let m1 = m.row(1)
        bytesNum = try m1.get(row: 0, col: 2, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)
    }

    func testPutIntArrayInt8Array() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_8SC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(ranges: [Range(start: 0, end: 2), Range(start: 1, end: 3), Range(start: 2, end: 4)])
        var buff = [Int8](repeating: 0, count: 6)
        let buff0:[Int8] = [10, 20, 30, 40, 50, 60]
        let buff1:[Int8] = [-1, -2, -3, -4, -5, -6]

        var bytesNum = try m.put(indices:[1, 2, 0], data:buff0)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(indices: [1, 2, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)

        bytesNum = try sm.put(indices: [0, 0, 0], data: buff1)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try sm.get(indices: [0, 0, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)

        bytesNum = try m.get(indices: [0, 1, 2], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)

        let m1 = m.submat(ranges: [Range(start: 1,end: 2), Range.all(), Range.all()])
        bytesNum = try m1.get(indices: [0, 2, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)
    }

    func testPutIntArrayUInt8Array() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_8UC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(ranges: [Range(start: 0, end: 2), Range(start: 1, end: 3), Range(start: 2, end: 4)])
        var buff = [UInt8](repeating: 0, count: 6)
        let buff0:[UInt8] = [10, 20, 30, 40, 50, 60]
        let buff1:[UInt8] = [255, 254, 253, 252, 251, 250]

        var bytesNum = try m.put(indices:[1, 2, 0], data:buff0)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(indices: [1, 2, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)

        bytesNum = try sm.put(indices: [0, 0, 0], data: buff1)

        XCTAssertEqual(6, bytesNum)
        bytesNum = try sm.get(indices: [0, 0, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)

        bytesNum = try m.get(indices: [0, 1, 2], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff1)

        let m1 = m.submat(ranges: [Range(start: 1,end: 2), Range.all(), Range.all()])
        bytesNum = try m1.get(indices: [0, 2, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == buff0)
    }

    func testPutIntIntDoubleArray() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_8UC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(rowStart: 2, rowEnd: 4, colStart: 3, colEnd: 5)
        var buff = [UInt8](repeating: 0, count: 6)

        var bytesNum = try m.put(row: 1, col: 2, data: [10, 20, 30, 40, 50, 60] as [Double])

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(row: 1, col: 2, data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == [10, 20, 30, 40, 50, 60])

        bytesNum = try sm.put(row: 0, col: 0, data:[255, 254, 253, 252, 251, 250] as [Double])

        XCTAssertEqual(6, bytesNum)
        bytesNum = try sm.get(row: 0, col: 0, data: &buff)
        XCTAssertEqual(6, bytesNum);
        XCTAssert(buff == [255, 254, 253, 252, 251, 250])
        bytesNum = try m.get(row: 2, col: 3, data: &buff)
        XCTAssertEqual(6, bytesNum);
        XCTAssert(buff == [255, 254, 253, 252, 251, 250])
    }

    func testPutIntArrayDoubleArray() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_8UC3, scalar: Scalar(1, 2, 3))
        let sm = m.submat(ranges: [Range(start: 0, end: 2), Range(start: 1, end: 3), Range(start: 2, end: 4)])
        var buff = [UInt8](repeating: 0, count: 6)

        var bytesNum = try m.put(indices: [1, 2, 0], data: [10, 20, 30, 40, 50, 60] as [Double])

        XCTAssertEqual(6, bytesNum)
        bytesNum = try m.get(indices: [1, 2, 0], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == [10, 20, 30, 40, 50, 60])

        bytesNum = try sm.put(indices: [0, 0, 0], data: [255, 254, 253, 252, 251, 250] as [Double])

        XCTAssertEqual(6, bytesNum);
        bytesNum = try sm.get(indices: [0, 0, 0], data: &buff)
        XCTAssertEqual(6, bytesNum);
        XCTAssert(buff == [255, 254, 253, 252, 251, 250])
        bytesNum = try m.get(indices: [0, 1, 2], data: &buff)
        XCTAssertEqual(6, bytesNum)
        XCTAssert(buff == [255, 254, 253, 252, 251, 250])
    }

    func testPutIntIntFloatArray() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_32FC3, scalar: Scalar(1, 2, 3))
        let elements:[Float] = [10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(row: 4, col: 3, data: elements)

        XCTAssertEqual(Int32(elements.count * 4), bytesNum);
        let m1 = m.row(4)
        var buff = [Float](repeating: 0, count: 3)
        bytesNum = try m1.get(row: 0, col: 4, data: &buff)
        XCTAssertEqual(Int32(buff.count * 4), bytesNum)
        XCTAssert(buff == [40, 50, 60])
        XCTAssert([10, 20, 30] == m.get(row: 4, col: 3));
    }

    func testPutIntArrayFloatArray() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_32FC3, scalar: Scalar(1, 2, 3))
        let elements:[Float] = [10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(indices: [0, 4, 3], data: elements)

        XCTAssertEqual(Int32(elements.count * 4), bytesNum)
        let m1 = m.submat(ranges: [Range.all(), Range(start: 4, end: 5), Range.all()])
        var buff = [Float](repeating: 0, count: 3)
        bytesNum = try m1.get(indices: [0, 0, 4], data: &buff)
        XCTAssertEqual(Int32(buff.count * 4), bytesNum)
        XCTAssert(buff == [40, 50, 60])
        XCTAssert([10, 20, 30] == m.get(indices: [0, 4, 3]))
    }

    func testPutIntIntIntArray() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_32SC3, scalar: Scalar(-1, -2, -3))
        let elements: [Int32] = [10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(row: 0, col: 4, data: elements)

        XCTAssertEqual(Int32(elements.count * 4), bytesNum)
        let m1 = m.col(4)
        var buff = [Int32](repeating: 0, count: 3)
        bytesNum = try m1.get(row: 0, col: 0, data: &buff)
        XCTAssertEqual(Int32(buff.count * 4), bytesNum)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(row: 1, col: 0))
    }

    func testPutIntArrayIntArray() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_32SC3, scalar: Scalar(-1, -2, -3))
        let elements: [Int32] = [10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(indices: [0, 0, 4], data: elements)

        XCTAssertEqual(Int32(elements.count * 4), bytesNum);
        let m1 = m.submat(ranges: [Range.all(), Range.all(), Range(start: 4, end: 5)])
        var buff = [Int32](repeating: 0, count: 3)
        bytesNum = try m1.get(indices: [0, 0, 0], data: &buff)
        XCTAssertEqual(Int32(buff.count * 4), bytesNum)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(indices: [0, 1, 0]))
    }

    func testPutIntIntInt16Array() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_16SC3, scalar: Scalar(-1, -2, -3))
        let elements: [Int16] = [ 10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(row: 2, col: 3, data: elements)

        XCTAssertEqual(Int32(elements.count * 2), bytesNum)
        let m1 = m.col(3)
        var buff = [Int16](repeating: 0, count: 3)
        bytesNum = try m1.get(row: 2, col: 0, data: &buff)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(row: 2, col: 4))
    }

    func testPutIntIntUInt16Array() throws {
        let m = Mat(rows: 5, cols: 5, type: CvType.CV_16UC3, scalar: Scalar(-1, -2, -3))
        let elements: [UInt16] = [ 10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(row: 2, col: 3, data: elements)

        XCTAssertEqual(Int32(elements.count * 2), bytesNum)
        let m1 = m.col(3)
        var buff = [UInt16](repeating: 0, count: 3)
        bytesNum = try m1.get(row: 2, col: 0, data: &buff)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(row: 2, col: 4))
    }

    func testPutIntArrayInt16Array() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_16SC3, scalar: Scalar(-1, -2, -3))
        let elements: [Int16] = [ 10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(indices: [0, 2, 3], data: elements)

        XCTAssertEqual(Int32(elements.count * 2), bytesNum)
        let m1 = m.submat(ranges: [Range.all(), Range.all(), Range(start: 3, end: 4)])
        var buff = [Int16](repeating: 0, count: 3)
        bytesNum = try m1.get(indices: [0, 2, 0], data: &buff)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(indices: [0, 2, 4]))
    }

    func testPutIntArrayUInt16Array() throws {
        let m = Mat(sizes: [5, 5, 5], type: CvType.CV_16UC3, scalar: Scalar(-1, -2, -3))
        let elements: [UInt16] = [ 10, 20, 30, 40, 50, 60]

        var bytesNum = try m.put(indices: [0, 2, 3], data: elements)

        XCTAssertEqual(Int32(elements.count * 2), bytesNum)
        let m1 = m.submat(ranges: [Range.all(), Range.all(), Range(start: 3, end: 4)])
        var buff = [UInt16](repeating: 0, count: 3)
        bytesNum = try m1.get(indices: [0, 2, 0], data: &buff)
        XCTAssert(buff == [10, 20, 30])
        XCTAssert([40, 50, 60] == m.get(indices: [0, 2, 4]))
    }

    func testReshapeInt() throws {
        let src = Mat(rows: 4, cols: 4, type: CvType.CV_8U, scalar: Scalar(0))
        dst = src.reshape(channels: 4)

        truth = Mat(rows: 4, cols: 1, type: CvType.CV_8UC4, scalar: Scalar(0))
        try assertMatEqual(truth!, dst)
    }

    func testReshapeIntInt() throws {
        let src = Mat(rows: 5, cols: 7, type: CvType.CV_8U, scalar: Scalar(0))
        dst = src.reshape(channels: 7, rows: 5)

        truth = Mat(rows: 5, cols: 1, type: CvType.CV_8UC(7), scalar: Scalar(0))
        try assertMatEqual(truth!, dst)
    }

    func testReshapeIntIntArray() {
        // 2D -> 4D
        let src = Mat(rows: 6, cols: 5, type: CvType.CV_8UC3, scalar: Scalar(0))
        XCTAssertEqual(2, src.dims())
        XCTAssertEqual(src.rows(), src.size(0))
        XCTAssertEqual(src.cols(), src.size(1))

        let newShape = [1, src.channels() * src.cols(), 1, src.rows()]
        dst = src.reshape(channels: 1, newshape: newShape as [NSNumber])
        XCTAssertEqual(newShape.count, Int(dst.dims()))
        for i in 0..<newShape.count {
            XCTAssertEqual(newShape[i], dst.size(Int32(i)))
        }

        // 3D -> 2D
        let src2 = Mat(sizes: [4, 6, 7], type: CvType.CV_8UC3, scalar: Scalar(0))
        XCTAssertEqual(3, src2.dims())
        XCTAssertEqual(4, src2.size(0))
        XCTAssertEqual(6, src2.size(1))
        XCTAssertEqual(7, src2.size(2))

        let newShape2 = [src2.channels() * src2.size(2), src2.size(0) * src2.size(1)]
        dst = src2.reshape(channels: 1, newshape: newShape2 as [NSNumber])
        XCTAssertEqual(newShape2.count, Int(dst.dims()))
        for i in 0..<newShape2.count {
            XCTAssertEqual(newShape2[i], dst.size(Int32(i)))
        }
    }

    func testCopySize() {
        let src = Mat(sizes: [1, 1, 10, 10], type: CvType.CV_8UC1, scalar: Scalar(1))
        XCTAssertEqual(4, src.dims())
        XCTAssertEqual(1, src.size(0))
        XCTAssertEqual(1, src.size(1))
        XCTAssertEqual(10, src.size(2))
        XCTAssertEqual(10, src.size(3))
        let other = Mat(sizes: [10, 10], type: src.type())

        src.copySize(other)
        XCTAssertEqual(other.dims(), src.dims())
        for i in 0..<other.dims() {
            XCTAssertEqual(other.size(i), src.size(i))
        }
    }

    func testRow() {
        let row = gray0.row(0)
        XCTAssertEqual(1, row.rows())
        XCTAssertEqual(gray0.cols(), row.cols())
    }

    func testRowRangeIntInt() {
        let rows = gray0.rowRange(start:0, end: gray0.rows() / 2)
        XCTAssertEqual(gray0.rows() / 2, rows.rows())
        XCTAssertEqual(gray0.cols(), rows.cols())
    }

    func testRowRangeRange() {
        let rows = gray255.rowRange(Range(start: 0, end: 5))
        XCTAssertEqual(gray255.rows() / 2, rows.rows())
        XCTAssertEqual(gray255.cols(), rows.cols())
    }

    func testRows() {
        XCTAssertEqual(OpenCVTestCase.matSize, gray0.rows())
    }

    func testSetToMat() throws {
        let vals = Mat(rows: 7, cols: 1, type: CvType.CV_8U)
        try XCTAssertEqual(vals.put(row: 0, col: 0, data: [1, 2, 3, 4, 5, 6, 7] as [Int8]), 7)
        let dst = Mat(rows: 1, cols: 1, type: CvType.CV_8UC(7))

        dst.setTo(value: vals)

        let truth = Mat(rows: 1, cols: 1, type: CvType.CV_8UC(7))
        try XCTAssertEqual(truth.put(row: 0, col: 0, data: [1, 2, 3, 4, 5, 6, 7] as [Int8]), 7)
        try assertMatEqual(truth, dst)
    }

    func testSetToMatMat() throws {
        let vals = Mat(rows: 7, cols: 1, type: CvType.CV_8U)
        try XCTAssertEqual(vals.put(row: 0, col: 0, data: [1, 2, 3, 4, 5, 6, 7] as [Int8]), 7)
        let dst = Mat.zeros(2, cols: 1, type: CvType.CV_8UC(7))
        let mask = Mat(rows: 2, cols: 1, type: CvType.CV_8U)
        try XCTAssertEqual(mask.put(row: 0, col: 0, data: [0, 1] as [Int8]), 2)

        dst.setTo(value: vals, mask: mask)

        let truth = Mat(rows: 2, cols: 1, type: CvType.CV_8UC(7))
        try XCTAssertEqual(truth.put(row: 0, col: 0, data: [0, 0, 0, 0, 0, 0, 0] as [Int8]), 7)
        try XCTAssertEqual(truth.put(row: 1, col: 0, data: [1, 2, 3, 4, 5, 6, 7] as [Int8]), 7)
        try assertMatEqual(truth, dst)
    }

    func testSetToScalar() throws {
        gray0.setTo(scalar: Scalar(127))
        try assertMatEqual(gray127, gray0)
    }

    func testSetToScalarMask() throws {
        let mask = gray0.clone()
        try XCTAssertEqual(mask.put(row: 1, col: 1, data: [1, 2, 3] as [Int8]), 3)
        gray0.setTo(scalar: Scalar(1), mask: mask)
        XCTAssertEqual(3, Core.countNonZero(src: gray0))
        Core.subtract(src1: gray0, src2: mask, dst: gray0)
        XCTAssertEqual(0, Core.countNonZero(src: gray0))
    }

    func testSize() {
        XCTAssertEqual(Size(width: OpenCVTestCase.matSize, height: OpenCVTestCase.matSize), gray0.size())

        XCTAssertEqual(Size(width: 3, height: 1), v1.size())
    }

    func testStep1() {
        XCTAssertEqual(OpenCVTestCase.matSize * CvType.channels(CvType.CV_8U), Int32(gray0.step1()))

        XCTAssertEqual(3, v2.step1())
    }

    func testStep1Int() {
        let roi = rgba0.submat(rowStart: 3, rowEnd: 5, colStart: 7, colEnd: 10)
        let m = roi.clone()

        XCTAssert(rgba0.channels() * rgba0.cols() <= roi.step1(0))
        XCTAssertEqual(rgba0.channels(), Int32(roi.step1(1)))
        XCTAssert(m.channels() * (10 - 7) <= m.step1(0))
        XCTAssertEqual(m.channels(), Int32(m.step1(1)))
    }

    func testSubmatIntIntIntInt() {
        let submat = gray0.submat(rowStart: 0, rowEnd: gray0.rows() / 2, colStart: 0, colEnd: gray0.cols() / 2)

        XCTAssert(submat.isSubmatrix())
        XCTAssertFalse(submat.isContinuous())
        XCTAssertEqual(gray0.rows() / 2, submat.rows())
        XCTAssertEqual(gray0.cols() / 2, submat.cols())
    }

    func testSubmatRangeRange() {
        let submat = gray255.submat(rowRange: Range(start: 2, end: 4), colRange: Range(start: 2, end: 4))
        XCTAssert(submat.isSubmatrix())
        XCTAssertFalse(submat.isContinuous())

        XCTAssertEqual(2, submat.rows())
        XCTAssertEqual(2, submat.cols())
    }

    func testSubmatRangeArray() {
        let submat = gray255_32f_3d.submat(ranges: [Range(start: 2, end: 4), Range(start: 2, end: 4), Range(start: 3, end: 6)])
        XCTAssert(submat.isSubmatrix())
        XCTAssertFalse(submat.isContinuous())

        XCTAssertEqual(2, submat.size(0))
        XCTAssertEqual(2, submat.size(1))
        XCTAssertEqual(3, submat.size(2))
    }

    func testSubmatRect() {
        let submat = gray255.submat(roi: Rect(x: 5, y: 5, width: gray255.cols() / 2, height: gray255.rows() / 2))
        XCTAssert(submat.isSubmatrix())
        XCTAssertFalse(submat.isContinuous())

        XCTAssertEqual(gray255.rows() / 2, submat.rows())
        XCTAssertEqual(gray255.cols() / 2, submat.cols())
    }

    func testT() throws {
        try assertMatEqual(gray255, gray255.t())

        let src = Mat(rows: 3, cols: 3, type: CvType.CV_16U)
        try XCTAssertEqual(src.put(row: 0, col: 0, data: [1, 2, 4] as [Int16]), 6)
        try XCTAssertEqual(src.put(row: 1, col: 0, data: [7, 5, 0] as [Int16]), 6)
        try XCTAssertEqual(src.put(row: 2, col: 0, data: [3, 4, 6] as [Int16]), 6)

        dst = src.t()

        truth = Mat(rows: 3, cols: 3, type: CvType.CV_16U)

        try XCTAssertEqual(truth!.put(row: 0, col: 0, data: [1, 7, 3] as [Int16]), 6)
        try XCTAssertEqual(truth!.put(row: 1, col: 0, data: [2, 5, 4] as [Int16]), 6)
        try XCTAssertEqual(truth!.put(row: 2, col: 0, data: [4, 0, 6] as [Int16]), 6)
        try assertMatEqual(truth!, dst)
    }

    func testToString() {
        let gray0String = "\(gray0)"
        XCTAssert(gray0String.starts(with: "Mat [ 10*10*CV_8UC1, isCont=YES, isSubmat=NO, nativeObj="))
    }

    func testTotal() {
        let nElements = gray0.rows() * gray0.cols()
        XCTAssertEqual(nElements, Int32(gray0.total()))
    }

    func testType() {
        XCTAssertEqual(CvType.CV_8UC1, gray0.type())
        XCTAssertEqual(CvType.CV_32FC1, gray0_32f.type())
        XCTAssertEqual(CvType.CV_8UC3, rgbLena.type())
    }

    func testWidth() {
        XCTAssertEqual(gray0.cols(), gray0.width())
        XCTAssertEqual(rgbLena.cols(), rgbLena.width())
        XCTAssertEqual(rgba128.cols(), rgba128.width())
    }

    func testZerosIntIntInt() throws {
        dst = Mat.zeros(OpenCVTestCase.matSize, cols: OpenCVTestCase.matSize, type: CvType.CV_32F)

        try assertMatEqual(gray0_32f, dst, OpenCVTestCase.EPS)
    }

    func testZerosSizeInt() throws {
        dst = Mat.zeros(Size(width: 2, height: 2), type: CvType.CV_16S)

        truth = Mat(rows: 2, cols: 2, type: CvType.CV_16S, scalar: Scalar(0))
        try assertMatEqual(truth!, dst)
    }

    func testZerosIntArray() throws {
        dst = Mat.zeros(sizes: [2, 3, 4], type: CvType.CV_16S)

        truth = Mat(sizes: [2, 3, 4], type: CvType.CV_16S, scalar: Scalar(0))
        try assertMatEqual(truth!, dst)
    }

    func testMatFromByteBuffer() {
        var bufferIn = [Int8](repeating:0, count: 64*64)
        bufferIn[0] = 1;
        bufferIn[1] = 1;
        bufferIn[2] = 1;
        bufferIn[3] = 1;
        var m: Mat? = Mat(rows:64, cols:64, type:CvType.CV_8UC1, data:bufferIn)
        XCTAssertEqual(4, Core.countNonZero(src: m!))
        Core.add(src1: m!, srcScalar: Scalar(1), dst: m!)
        XCTAssertEqual(4096, Core.countNonZero(src: m!))
        m = nil
        let data = bufferIn.withUnsafeBufferPointer { Data(buffer: $0) }
        m = Mat(rows:64, cols:64, type:CvType.CV_8UC1, data:data)
        Core.add(src1: m!, srcScalar: Scalar(1), dst: m!)
        m = nil
        let bufferOut = [UInt8](data as Data)
        XCTAssertEqual(2, bufferOut[0])
        XCTAssertEqual(1, bufferOut[4095])
    }

    func testMatFromByteBufferWithStep() {
        var bufferIn = [Int8](repeating:0, count: 80*64)
        bufferIn[0] = 1
        bufferIn[1] = 1
        bufferIn[2] = 1
        bufferIn[3] = 1
        bufferIn[64] = 2
        bufferIn[65] = 2
        bufferIn[66] = 2
        bufferIn[67] = 2
        bufferIn[80] = 3
        bufferIn[81] = 3
        bufferIn[82] = 3
        bufferIn[83] = 3
        var m:Mat? = Mat(rows:64, cols:64, type:CvType.CV_8UC1, data:bufferIn, step:80)
        XCTAssertEqual(8, Core.countNonZero(src: m!))
        Core.add(src1: m!, srcScalar: Scalar(5), dst: m!);
        XCTAssertEqual(4096, Core.countNonZero(src: m!))
        m = nil
        let data = bufferIn.withUnsafeBufferPointer { Data(buffer: $0) }
        m = Mat(rows:64, cols:64, type:CvType.CV_8UC1, data:data, step:80)
        Core.add(src1: m!, srcScalar: Scalar(5), dst: m!)
        m = nil
        let bufferOut = [UInt8](data as Data)
        XCTAssertEqual(6, bufferOut[0])
        XCTAssertEqual(5, bufferOut[63])
        XCTAssertEqual(2, bufferOut[64])
        XCTAssertEqual(0, bufferOut[79])
        XCTAssertEqual(8, bufferOut[80])
        XCTAssertEqual(5, bufferOut[63*80 + 63])
    }

    func testMatAt() {
        let uc1 = Mat(rows: 2, cols: 3, type: CvType.CV_8U)
        try! uc1.put(row: 0, col: 0, data: [1, 2, 3, 4, 5, 6] as [Int8])
        XCTAssertEqual(UInt8(1), uc1.at(row: 0, col: 0).v)
        XCTAssertEqual(UInt8(2), uc1.at(row: 0, col: 1).v)
        XCTAssertEqual(UInt8(3), uc1.at(row: 0, col: 2).v)
        XCTAssertEqual(UInt8(4), uc1.at(row: 1, col: 0).v)
        XCTAssertEqual(UInt8(5), uc1.at(row: 1, col: 1).v)
        XCTAssertEqual(UInt8(6), uc1.at(row: 1, col: 2).v)
        uc1.at(row: 0, col: 0).v = UInt8(7)
        uc1.at(row: 0, col: 1).v = UInt8(8)
        uc1.at(row: 0, col: 2).v = UInt8(9)
        uc1.at(row: 1, col: 0).v = UInt8(10)
        uc1.at(row: 1, col: 1).v = UInt8(11)
        uc1.at(row: 1, col: 2).v = UInt8(12)
        var data = [Int8](repeating: 0, count: 6)
        try! uc1.get(row: 0, col: 0, data: &data)
        XCTAssertEqual(data, [7, 8, 9, 10, 11, 12] as [Int8])
        let (b, g, r): T3<UInt8>  = rgbLena.at(row: 0, col: 0).v3c
        XCTAssertEqual(b, UInt8(128))
        XCTAssertEqual(g, UInt8(138))
        XCTAssertEqual(r, UInt8(225))
    }

}
```

## High-Level Overview

This is a .swift source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **MatTests**: A class/struct defined in this file

### Functions and Methods

- **testSetToScalarMask()**: A function/method defined in this file
- **testDiagInt()**: A function/method defined in this file
- **testSize()**: A function/method defined in this file
- **testCopySize()**: A function/method defined in this file
- **testPush_back()**: A function/method defined in this file
- **testMatMatRangeArray()**: A function/method defined in this file
- **testMat()**: A function/method defined in this file
- **testHeight()**: A function/method defined in this file
- **testAssignToMatInt()**: A function/method defined in this file
- **testMulMat3d()**: A function/method defined in this file
- **testCross()**: A function/method defined in this file
- **testIsSubmatrix()**: A function/method defined in this file
- **testRowRangeIntInt()**: A function/method defined in this file
- **testGetIntIntIntArray()**: A function/method defined in this file
- **testCreateIntArrayInt()**: A function/method defined in this file
- **testReshapeIntIntArray()**: A function/method defined in this file
- **testDiag()**: A function/method defined in this file
- **testMatSizeIntScalar()**: A function/method defined in this file
- **testMatIntIntIntScalar()**: A function/method defined in this file
- **testColRangeRange()**: A function/method defined in this file
- **testColRangeIntInt()**: A function/method defined in this file
- **testPutIntIntUInt8Array()**: A function/method defined in this file
- **testGetIntIntFloatArray()**: A function/method defined in this file
- **testDump()**: A function/method defined in this file
- **testCheckVectorIntInt()**: A function/method defined in this file
- **testReshapeInt()**: A function/method defined in this file
- **testSetToMatMat()**: A function/method defined in this file
- **testPutIntIntFloatArray()**: A function/method defined in this file
- **testEyeIntIntInt()**: A function/method defined in this file
- **testGetIntIntDoubleArray()**: A function/method defined in this file
- **testPutIntArrayIntArray()**: A function/method defined in this file
- **testAdjustROI()**: A function/method defined in this file
- **testMatSizeInt()**: A function/method defined in this file
- **testMatMatRange()**: A function/method defined in this file
- **testElemSize()**: A function/method defined in this file
- **testEmpty()**: A function/method defined in this file
- **testOnesIntIntInt()**: A function/method defined in this file
- **testRow()**: A function/method defined in this file
- **testPutIntIntDoubleArray()**: A function/method defined in this file
- **testGetIntInt_64F()**: A function/method defined in this file
- **testPutIntIntInt16Array()**: A function/method defined in this file
- **testGetIntInt_8U()**: A function/method defined in this file
- **testCopyToMatMat()**: A function/method defined in this file
- **testDiagMat()**: A function/method defined in this file
- **testGetIntIntInt8Array()**: A function/method defined in this file
- **testChannels()**: A function/method defined in this file
- **testMatMatRect()**: A function/method defined in this file
- **testPutIntArrayInt8Array()**: A function/method defined in this file
- **testRowRangeRange()**: A function/method defined in this file
- **testPutIntIntIntArray()**: A function/method defined in this file
- **tearDown()**: A function/method defined in this file
- **testOnesIntArrayInt()**: A function/method defined in this file
- **testPutIntArrayInt16Array()**: A function/method defined in this file
- **testCreateIntIntInt()**: A function/method defined in this file
- **testInitWithSize()**: A function/method defined in this file
- **testDepth()**: A function/method defined in this file
- **testCheckVectorInt()**: A function/method defined in this file
- **testStep1()**: A function/method defined in this file
- **testCols()**: A function/method defined in this file
- **testCreateSizeInt()**: A function/method defined in this file
- **testAssignToMat()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **testPutIntIntUInt16Array()**: A function/method defined in this file
- **testGetIntIntUInt16Array()**: A function/method defined in this file
- **testPutIntArrayUInt8Array()**: A function/method defined in this file
- **testMatMulMat()**: A function/method defined in this file
- **testConvertToMatIntDouble()**: A function/method defined in this file
- **testMatIntIntInt()**: A function/method defined in this file
- **testIsContinuous()**: A function/method defined in this file
- **testCol()**: A function/method defined in this file
- **testRows()**: A function/method defined in this file
- **testMatIntArrayIntScalar()**: A function/method defined in this file
- **testEyeSizeInt()**: A function/method defined in this file
- **testConvertToMatInt()**: A function/method defined in this file
- **testMatMatRangeRange()**: A function/method defined in this file
- **testCheckVectorIntIntBoolean()**: A function/method defined in this file
- **testConvertToMatIntDoubleDouble()**: A function/method defined in this file
- **testGetIntIntInt16Array()**: A function/method defined in this file
- **testGetIntIntUInt8Array()**: A function/method defined in this file
- **testClone()**: A function/method defined in this file
- **testGetIntInt_32S()**: A function/method defined in this file
- **testOnesSizeInt()**: A function/method defined in this file
- **testMatIntIntCvTypeScalar()**: A function/method defined in this file
- **testPutIntIntInt8Array()**: A function/method defined in this file
- **testInv()**: A function/method defined in this file
- **testElemSize1()**: A function/method defined in this file
- **testMulMat()**: A function/method defined in this file
- **testCopyToMat()**: A function/method defined in this file
- **testPutIntArrayDoubleArray()**: A function/method defined in this file
- **testInvInt()**: A function/method defined in this file
- **testPutIntArrayFloatArray()**: A function/method defined in this file
- **testSetToScalar()**: A function/method defined in this file
- **testReshapeIntInt()**: A function/method defined in this file
- **testSetToMat()**: A function/method defined in this file
- **testMulMatDouble()**: A function/method defined in this file
- **getTestMat()**: A function/method defined in this file
- **testLocateROI()**: A function/method defined in this file
- **testDot()**: A function/method defined in this file
- **testMatIntIntCvType()**: A function/method defined in this file
- **testPutIntArrayUInt16Array()**: A function/method defined in this file


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

