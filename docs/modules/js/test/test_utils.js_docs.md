# Documentation for `modules/js/test/test_utils.js`

## File Metadata

- **Full Path**: `modules/js/test/test_utils.js`
- **File Name**: `test_utils.js`
- **File Size**: 8,926 bytes
- **File Type**: .js
- **Link to Source**: [modules/js/test/test_utils.js](../../../modules/js/test/test_utils.js)

## Purpose and Role

This file is located in the `modules/js/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//  //////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//

// //////////////////////////////////////////////////////////////////////////////////////
// Author: Sajjad Taheri, University of California, Irvine. sajjadt[at]uci[dot]edu
//
//                             LICENSE AGREEMENT
// Copyright (c) 2015 The Regents of the University of California (Regents)
//
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions are met:
// 1. Redistributions of source code must retain the above copyright
//    notice, this list of conditions and the following disclaimer.
// 2. Redistributions in binary form must reproduce the above copyright
//    notice, this list of conditions and the following disclaimer in the
//    documentation and/or other materials provided with the distribution.
// 3. Neither the name of the University nor the
//    names of its contributors may be used to endorse or promote products
//    derived from this software without specific prior written permission.
//
// THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS ''AS IS'' AND ANY
// EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
// WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
// DISCLAIMED. IN NO EVENT SHALL CONTRIBUTORS BE LIABLE FOR ANY
// DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
// (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
// LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
// ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
// (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
// SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
//

QUnit.module('Utils', {});
QUnit.test('Test vectors', function(assert) {
    {
        let pointVector = new cv.PointVector();
        for (let i=0; i<100; ++i) {
            pointVector.push_back({x: i, y: 2*i});
        }

        assert.equal(pointVector.size(), 100);

        let index = 10;
        let item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        index = 0;
        item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        index = 99;
        item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        pointVector.delete();
    }

    {
        let pointVector = new cv.PointVector();
        for (let i=0; i<100; ++i) {
            pointVector.push_back(new cv.Point(i, 2*i));
        }

        pointVector.push_back(new cv.Point());

        assert.equal(pointVector.size(), 101);

        let index = 10;
        let item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        index = 0;
        item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        index = 99;
        item = pointVector.get(index);
        assert.equal(item.x, index);
        assert.equal(item.y, 2*index);

        index = 100;
        item = pointVector.get(index);
        assert.equal(item.x, 0);
        assert.equal(item.y, 0);

        pointVector.delete();
    }
});
QUnit.test('Test Rect', function(assert) {
    let rectVector = new cv.RectVector();
    let rect = {x: 1, y: 2, width: 3, height: 4};
    rectVector.push_back(rect);
    rectVector.push_back(new cv.Rect());
    rectVector.push_back(new cv.Rect(rect));
    rectVector.push_back(new cv.Rect({x: 5, y: 6}, {width: 7, height: 8}));
    rectVector.push_back(new cv.Rect(9, 10, 11, 12));

    assert.equal(rectVector.size(), 5);

    let item = rectVector.get(0);
    assert.equal(item.x, 1);
    assert.equal(item.y, 2);
    assert.equal(item.width, 3);
    assert.equal(item.height, 4);

    item = rectVector.get(1);
    assert.equal(item.x, 0);
    assert.equal(item.y, 0);
    assert.equal(item.width, 0);
    assert.equal(item.height, 0);

    item = rectVector.get(2);
    assert.equal(item.x, 1);
    assert.equal(item.y, 2);
    assert.equal(item.width, 3);
    assert.equal(item.height, 4);

    item = rectVector.get(3);
    assert.equal(item.x, 5);
    assert.equal(item.y, 6);
    assert.equal(item.width, 7);
    assert.equal(item.height, 8);

    item = rectVector.get(4);
    assert.equal(item.x, 9);
    assert.equal(item.y, 10);
    assert.equal(item.width, 11);
    assert.equal(item.height, 12);

    rectVector.delete();
});
QUnit.test('Test Size', function(assert) {
    {
        let mat = new cv.Mat();
        mat.create({width: 5, height: 10}, cv.CV_8UC4);
        let size = mat.size();

        assert.ok(mat.type() === cv.CV_8UC4);
        assert.ok(size.height === 10);
        assert.ok(size.width === 5);
        assert.ok(mat.channels() === 4);

        mat.delete();
    }

    {
        let mat = new cv.Mat();
        mat.create(new cv.Size(5, 10), cv.CV_8UC4);
        let size = mat.size();

        assert.ok(mat.type() === cv.CV_8UC4);
        assert.ok(size.height === 10);
        assert.ok(size.width === 5);
        assert.ok(mat.channels() === 4);

        mat.delete();
    }
});


QUnit.test('test_rotated_rect', function(assert) {
    {
        let rect = {center: {x: 100, y: 100}, size: {height: 100, width: 50}, angle: 30};

        assert.equal(rect.center.x, 100);
        assert.equal(rect.center.y, 100);
        assert.equal(rect.angle, 30);
        assert.equal(rect.size.height, 100);
        assert.equal(rect.size.width, 50);
    }

    {
        let rect = new cv.RotatedRect();

        assert.equal(rect.center.x, 0);
        assert.equal(rect.center.y, 0);
        assert.equal(rect.angle, 0);
        assert.equal(rect.size.height, 0);
        assert.equal(rect.size.width, 0);

        let points = cv.RotatedRect.points(rect);

        assert.equal(points[0].x, 0);
        assert.equal(points[0].y, 0);
        assert.equal(points[1].x, 0);
        assert.equal(points[1].y, 0);
        assert.equal(points[2].x, 0);
        assert.equal(points[2].y, 0);
        assert.equal(points[3].x, 0);
        assert.equal(points[3].y, 0);
    }

    {
        let rect = new cv.RotatedRect({x: 100, y: 100}, {height: 100, width: 50}, 30);

        assert.equal(rect.center.x, 100);
        assert.equal(rect.center.y, 100);
        assert.equal(rect.angle, 30);
        assert.equal(rect.size.height, 100);
        assert.equal(rect.size.width, 50);

        let points = cv.RotatedRect.points(rect);

        assert.equal(points[0].x, cv.RotatedRect.boundingRect2f(rect).x);
        assert.equal(points[1].y, cv.RotatedRect.boundingRect2f(rect).y);

        let points1 = cv.boxPoints(rect);
        assert.deepEqual(points, points1);
    }
});
```

## High-Level Overview

This is a .js source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

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

