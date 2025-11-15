# Documentation for `modules/core/misc/objc/test/RectTest.swift`

## File Metadata

- **Full Path**: `modules/core/misc/objc/test/RectTest.swift`
- **File Name**: `RectTest.swift`
- **File Size**: 4,498 bytes
- **File Type**: .swift
- **Link to Source**: [modules/core/misc/objc/test/RectTest.swift](../../../../../modules/core/misc/objc/test/RectTest.swift)

## Purpose and Role

This file is located in the `modules/core/misc/objc/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//
//  RectTest.swift
//
//  Created by Giles Payne on 2020/01/31.
//

import XCTest
import OpenCV

class RectTest: OpenCVTestCase {

    let r = Rect()
    let rect = Rect(x: 0, y: 0, width: 10, height: 10)

    func testArea() {
        let area = rect.area()
        XCTAssertEqual(100.0, area)
    }

    func testBr() {
        let p_br = rect.br()
        let truth = Point(x: 10, y: 10)
        XCTAssertEqual(truth, p_br)
    }

    func testClone() {
        let r = rect.clone()
        XCTAssertEqual(rect, r)
    }

    func testContains() {
        let rect = Rect(x: 0, y: 0, width: 10, height: 10)

        let p_inner = Point(x: 5, y: 5)
        let p_outer = Point(x: 5, y: 55)
        let p_bl = Point(x: 0, y: 0)
        let p_br = Point(x: 10, y: 0)
        let p_tl = Point(x: 0, y: 10)
        let p_tr = Point(x: 10, y: 10)

        XCTAssert(rect.contains(p_inner))
        XCTAssert(rect.contains(p_bl))

        XCTAssertFalse(rect.contains(p_outer))
        XCTAssertFalse(rect.contains(p_br))
        XCTAssertFalse(rect.contains(p_tl))
        XCTAssertFalse(rect.contains(p_tr))
    }

    func testEqualsObject() {
        var flag = rect == r
        XCTAssertFalse(flag)

        let r = rect.clone()
        flag = rect == r
        XCTAssert(flag)
    }

    func testHashCode() {
        XCTAssertEqual(rect.hash(), rect.hash())
    }

    func testRect() {
        let r = Rect()

        XCTAssertEqual(0, r.x)
        XCTAssertEqual(0, r.y)
        XCTAssertEqual(0, r.width)
        XCTAssertEqual(0, r.height)
    }

    func testRectDoubleArray() {
        let vals:[Double] = [1, 3, 5, 2]
        let r = Rect(vals: vals as [NSNumber])

        XCTAssertEqual(1, r.x)
        XCTAssertEqual(3, r.y)
        XCTAssertEqual(5, r.width)
        XCTAssertEqual(2, r.height)
    }

    func testRectIntIntIntInt() {
        let rect = Rect(x: 1, y: 3, width: 5, height: 2)

        XCTAssertNotNil(rect)
        XCTAssertEqual(1, rect.x)
        XCTAssertEqual(3, rect.y)
        XCTAssertEqual(5, rect.width)
        XCTAssertEqual(2, rect.height)
    }

    func testRectPointPoint() {
        let p1 = Point(x:4, y:4)
        let p2 = Point(x: 2, y: 3)

        let r = Rect(point: p1, point: p2)
        XCTAssertNotNil(r);
        XCTAssertEqual(2, r.x);
        XCTAssertEqual(3, r.y);
        XCTAssertEqual(2, r.width);
        XCTAssertEqual(1, r.height);
    }

    func testRect2fPointPoint() {
        let p1 = Point2f(x:4.3, y:4.1)
        let p2 = Point2f(x:2.7, y:3.9)

        let r = Rect2f(point: p1, point: p2)
        XCTAssertNotNil(r);
        XCTAssertEqual(2.7, r.x);
        XCTAssertEqual(3.9, r.y);
        XCTAssertEqual(1.6, r.width, accuracy: OpenCVTestCase.FEPS);
        XCTAssertEqual(0.2, r.height, accuracy: OpenCVTestCase.FEPS);
    }

    func testRect2dPointPoint() {
        let p1 = Point2d(x:4.7879839, y:4.9922311)
        let p2 = Point2d(x:2.1213123, y:3.1122129)

        let r = Rect2d(point: p1, point: p2)
        XCTAssertNotNil(r);
        XCTAssertEqual(2.1213123, r.x);
        XCTAssertEqual(3.1122129, r.y);
        XCTAssertEqual(2.6666716, r.width, accuracy: OpenCVTestCase.EPS);
        XCTAssertEqual(1.8800182, r.height, accuracy: OpenCVTestCase.EPS);
    }

    func testRectPointSize() {
        let p1 = Point(x: 4, y: 4)
        let sz = Size(width: 3, height: 1)
        let r = Rect(point: p1, size: sz)

        XCTAssertEqual(4, r.x)
        XCTAssertEqual(4, r.y)
        XCTAssertEqual(3, r.width)
        XCTAssertEqual(1, r.height)
    }

    func testSet() {
        let vals1:[Double] = []
        let r1 = Rect(vals:vals1 as [NSNumber])

        XCTAssertEqual(0, r1.x)
        XCTAssertEqual(0, r1.y)
        XCTAssertEqual(0, r1.width)
        XCTAssertEqual(0, r1.height)

        let vals2:[Double] = [2, 2, 10, 5]
        let r = Rect(vals: vals2 as [NSNumber])

        XCTAssertEqual(2, r.x)
        XCTAssertEqual(2, r.y)
        XCTAssertEqual(10, r.width)
        XCTAssertEqual(5, r.height)
    }

    func testSize() {
        let s1 = Size(width: 0, height: 0)
        XCTAssertEqual(s1, r.size())

        let s2 = Size(width: 10, height: 10)
        XCTAssertEqual(s2, rect.size())
    }

    func testTl() {
        let p_tl = rect.tl()
        let truth = Point(x: 0, y: 0)
        XCTAssertEqual(truth, p_tl)
    }

    func testToString() {
        let actual = "\(rect)"
        let expected = "Rect2i {0,0,10,10}"
        XCTAssertEqual(expected, actual);
    }

}
```

## High-Level Overview

This is a .swift source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **RectTest**: A class/struct defined in this file

### Functions and Methods

- **testClone()**: A function/method defined in this file
- **testSet()**: A function/method defined in this file
- **testRect2fPointPoint()**: A function/method defined in this file
- **testBr()**: A function/method defined in this file
- **testRect2dPointPoint()**: A function/method defined in this file
- **testSize()**: A function/method defined in this file
- **testEqualsObject()**: A function/method defined in this file
- **testArea()**: A function/method defined in this file
- **testRectDoubleArray()**: A function/method defined in this file
- **testRectIntIntIntInt()**: A function/method defined in this file
- **testTl()**: A function/method defined in this file
- **testToString()**: A function/method defined in this file
- **testRectPointPoint()**: A function/method defined in this file
- **testRect()**: A function/method defined in this file
- **testContains()**: A function/method defined in this file
- **testRectPointSize()**: A function/method defined in this file
- **testHashCode()**: A function/method defined in this file


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

