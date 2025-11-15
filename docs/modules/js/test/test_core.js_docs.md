# Documentation for `modules/js/test/test_core.js`

## File Metadata

- **Full Path**: `modules/js/test/test_core.js`
- **File Name**: `test_core.js`
- **File Size**: 13,080 bytes
- **File Type**: .js
- **Link to Source**: [modules/js/test/test_core.js](../../../modules/js/test/test_core.js)

## Purpose and Role

This file is located in the `modules/js/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

QUnit.module('Core', {});

QUnit.test('test_operations_on_arrays', function(assert) {
    // Transpose
    {
        let mat1 = cv.Mat.eye(9, 7, cv.CV_8UC3);
        let mat2 = new cv.Mat();

        cv.transpose(mat1, mat2);

        // Verify result.
        let size = mat2.size();
        assert.equal(mat2.channels(), 3);
        assert.equal(size.height, 7);
        assert.equal(size.width, 9);
    }

    // Concat
    {
        let mat = cv.Mat.ones({height: 10, width: 5}, cv.CV_8UC3);
        let mat2 = cv.Mat.eye({height: 10, width: 5}, cv.CV_8UC3);
        let mat3 = cv.Mat.eye({height: 10, width: 5}, cv.CV_8UC3);

        let out = new cv.Mat();
        let input = new cv.MatVector();
        input.push_back(mat);
        input.push_back(mat2);
        input.push_back(mat3);

        cv.vconcat(input, out);

        // Verify result.
        let size = out.size();
        assert.equal(out.channels(), 3);
        assert.equal(size.height, 30);
        assert.equal(size.width, 5);
        assert.equal(out.elemSize1(), 1);

        cv.hconcat(input, out);

        // Verify result.
        size = out.size();
        assert.equal(out.channels(), 3);
        assert.equal(size.height, 10);
        assert.equal(size.width, 15);
        assert.equal(out.elemSize1(), 1);

        input.delete();
        out.delete();
    }

    // Min, Max
    {
        let data1 = new Uint8Array([1, 2, 3, 4, 5, 6, 7, 8, 9]);
        let data2 = new Uint8Array([0, 4, 0, 8, 0, 12, 0, 16, 0]);

        let expectedMin = new Uint8Array([0, 2, 0, 4, 0, 6, 0, 8, 0]);
        let expectedMax = new Uint8Array([1, 4, 3, 8, 5, 12, 7, 16, 9]);

        let dataPtr = cv._malloc(3*3*1);
        let dataPtr2 = cv._malloc(3*3*1);

        let dataHeap = new Uint8Array(cv.HEAPU8.buffer, dataPtr, 3*3*1);
        dataHeap.set(new Uint8Array(data1.buffer));

        let dataHeap2 = new Uint8Array(cv.HEAPU8.buffer, dataPtr2, 3*3*1);
        dataHeap2.set(new Uint8Array(data2.buffer));

        let mat1 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr, 0);
        let mat2 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr2, 0);
        let mat3 = new cv.Mat();

        cv.min(mat1, mat2, mat3);

        // Verify result.
        let size = mat2.size();
        assert.equal(mat2.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedMin);

        cv.max(mat1, mat2, mat3);

        // Verify result.
        size = mat2.size();
        assert.equal(mat2.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedMax);

        cv._free(dataPtr);
        cv._free(dataPtr2);
    }

    // Bitwise operations
    {
        let data1 = new Uint8Array([0, 1, 2, 4, 8, 16, 32, 64, 128]);
        let data2 = new Uint8Array([255, 255, 255, 255, 255, 255, 255, 255, 255]);

        let expectedAnd = new Uint8Array([0, 1, 2, 4, 8, 16, 32, 64, 128]);
        let expectedOr = new Uint8Array([255, 255, 255, 255, 255, 255, 255, 255, 255]);
        let expectedXor = new Uint8Array([255, 254, 253, 251, 247, 239, 223, 191, 127]);

        let expectedNot = new Uint8Array([255, 254, 253, 251, 247, 239, 223, 191, 127]);

        let dataPtr = cv._malloc(3*3*1);
        let dataPtr2 = cv._malloc(3*3*1);

        let dataHeap = new Uint8Array(cv.HEAPU8.buffer, dataPtr, 3*3*1);
        dataHeap.set(new Uint8Array(data1.buffer));

        let dataHeap2 = new Uint8Array(cv.HEAPU8.buffer, dataPtr2, 3*3*1);
        dataHeap2.set(new Uint8Array(data2.buffer));

        let mat1 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr, 0);
        let mat2 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr2, 0);
        let mat3 = new cv.Mat();
        let none = new cv.Mat();

        cv.bitwise_not(mat1, mat3, none);

        // Verify result.
        let size = mat3.size();
        assert.equal(mat3.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedNot);

        cv.bitwise_and(mat1, mat2, mat3, none);

        // Verify result.
        size = mat3.size();
        assert.equal(mat3.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedAnd);

        cv.bitwise_or(mat1, mat2, mat3, none);

        // Verify result.
        size = mat3.size();
        assert.equal(mat3.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedOr);

        cv.bitwise_xor(mat1, mat2, mat3, none);

        // Verify result.
        size = mat3.size();
        assert.equal(mat3.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(mat3.data, expectedXor);

        cv._free(dataPtr);
        cv._free(dataPtr2);
    }

    // Arithmetic operations
    {
        let data1 = new Uint8Array([0, 1, 2, 3, 4, 5, 6, 7, 8]);
        let data2 = new Uint8Array([0, 2, 4, 6, 8, 10, 12, 14, 16]);
        let data3 = new Uint8Array([0, 1, 0, 1, 0, 1, 0, 1, 0]);

        // |data1 - data2|
        let expectedAbsDiff = new Uint8Array([0, 1, 2, 3, 4, 5, 6, 7, 8]);
        let expectedAdd = new Uint8Array([0, 3, 6, 9, 12, 15, 18, 21, 24]);

        const alpha = 4;
        const beta = -1;
        const gamma = 3;
        // 4*data1 - data2 + 3
        let expectedWeightedAdd = new Uint8Array([3, 5, 7, 9, 11, 13, 15, 17, 19]);

        let dataPtr = cv._malloc(3*3*1);
        let dataPtr2 = cv._malloc(3*3*1);
        let dataPtr3 = cv._malloc(3*3*1);

        let dataHeap = new Uint8Array(cv.HEAPU8.buffer, dataPtr, 3*3*1);
        dataHeap.set(new Uint8Array(data1.buffer));
        let dataHeap2 = new Uint8Array(cv.HEAPU8.buffer, dataPtr2, 3*3*1);
        dataHeap2.set(new Uint8Array(data2.buffer));
        let dataHeap3 = new Uint8Array(cv.HEAPU8.buffer, dataPtr3, 3*3*1);
        dataHeap3.set(new Uint8Array(data3.buffer));

        let mat1 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr, 0);
        let mat2 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr2, 0);
        let mat3 = new cv.Mat(3, 3, cv.CV_8UC1, dataPtr3, 0);

        let dst = new cv.Mat();
        let none = new cv.Mat();

        cv.absdiff(mat1, mat2, dst);

        // Verify result.
        let size = dst.size();
        assert.equal(dst.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(dst.data, expectedAbsDiff);

        cv.add(mat1, mat2, dst, none, -1);

        // Verify result.
        size = dst.size();
        assert.equal(dst.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);
        assert.deepEqual(dst.data, expectedAdd);

        cv.addWeighted(mat1, alpha, mat2, beta, gamma, dst, -1);

        // Verify result.
        size = dst.size();
        assert.equal(dst.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);
        assert.deepEqual(dst.data, expectedWeightedAdd);

        // default parameter
        cv.addWeighted(mat1, alpha, mat2, beta, gamma, dst);

        // Verify result.
        size = dst.size();
        assert.equal(dst.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);

        assert.deepEqual(dst.data, expectedWeightedAdd);

        mat1.delete();
        mat2.delete();
        mat3.delete();
        dst.delete();
        none.delete();
    }

    // Invert
    {
        let inv1 = new cv.Mat();
        let inv2 = new cv.Mat();
        let inv3 = new cv.Mat();
        let inv4 = new cv.Mat();

        let data1 = new Float32Array([1, 0, 0,
                                      0, 1, 0,
                                      0, 0, 1]);
        let data2 = new Float32Array([0, 0, 0,
                                      0, 5, 0,
                                      0, 0, 0]);
        let data3 = new Float32Array([1, 1, 1, 0,
                                      0, 3, 1, 2,
                                      2, 3, 1, 0,
                                      1, 0, 2, 1]);
        let data4 = new Float32Array([1, 4, 5,
                                      4, 2, 2,
                                      5, 2, 2]);

        let expected1 = new Float32Array([1, 0, 0,
                                          0, 1, 0,
                                          0, 0, 1]);
        // Inverse does not exist!
        let expected3 = new Float32Array([-3, -1/2, 3/2, 1,
                                          1, 1/4, -1/4, -1/2,
                                          3, 1/4, -5/4, -1/2,
                                          -3, 0, 1, 1]);
        let expected4 = new Float32Array([0, -1, 1,
                                          -1, 23/2, -9,
                                          1, -9, 7]);

        let dataPtr1 = cv._malloc(3*3*4);
        let dataPtr2 = cv._malloc(3*3*4);
        let dataPtr3 = cv._malloc(4*4*4);
        let dataPtr4 = cv._malloc(3*3*4);

        let dataHeap = new Float32Array(cv.HEAP32.buffer, dataPtr1, 3*3);
        dataHeap.set(new Float32Array(data1.buffer));
        let dataHeap2 = new Float32Array(cv.HEAP32.buffer, dataPtr2, 3*3);
        dataHeap2.set(new Float32Array(data2.buffer));
        let dataHeap3 = new Float32Array(cv.HEAP32.buffer, dataPtr3, 4*4);
        dataHeap3.set(new Float32Array(data3.buffer));
        let dataHeap4 = new Float32Array(cv.HEAP32.buffer, dataPtr4, 3*3);
        dataHeap4.set(new Float32Array(data4.buffer));

        let mat1 = new cv.Mat(3, 3, cv.CV_32FC1, dataPtr1, 0);
        let mat2 = new cv.Mat(3, 3, cv.CV_32FC1, dataPtr2, 0);
        let mat3 = new cv.Mat(4, 4, cv.CV_32FC1, dataPtr3, 0);
        let mat4 = new cv.Mat(3, 3, cv.CV_32FC1, dataPtr4, 0);

        QUnit.assert.deepEqualWithTolerance = function( value, expected, tolerance ) {
            for (let i = 0; i < value.length; i= i+1) {
                this.pushResult( {
                    result: Math.abs(value[i]-expected[i]) < tolerance,
                    actual: value[i],
                    expected: expected[i],
                } );
            }
        };

        cv.invert(mat1, inv1, 0);

        // Verify result.
        let size = inv1.size();
        assert.equal(inv1.channels(), 1);
        assert.equal(size.height, 3);
        assert.equal(size.width, 3);
        assert.deepEqualWithTolerance(inv1.data32F, expected1, 0.0001);

        cv.invert(mat2, inv2, 0);

        // Verify result.
        assert.deepEqualWithTolerance(inv3.data32F, expected3, 0.0001);

        cv.invert(mat3, inv3, 0);

        // Verify result.
        size = inv3.size();
        assert.equal(inv3.channels(), 1);
        assert.equal(size.height, 4);
        assert.equal(size.width, 4);
        assert.deepEqualWithTolerance(inv3.data32F, expected3, 0.0001);

        cv.invert(mat3, inv3, 1);

        // Verify result.
        assert.deepEqualWithTolerance(inv3.data32F, expected3, 0.0001);

        cv.invert(mat4, inv4, 2);

        // Verify result.
        assert.deepEqualWithTolerance(inv4.data32F, expected4, 0.0001);

        cv.invert(mat4, inv4, 3);

        // Verify result.
        assert.deepEqualWithTolerance(inv4.data32F, expected4, 0.0001);

        mat1.delete();
        mat2.delete();
        mat3.delete();
        mat4.delete();
        inv1.delete();
        inv2.delete();
        inv3.delete();
        inv4.delete();
    }

    //Rotate
    {
        let dst = new cv.Mat();
        let src = cv.matFromArray(3, 2, cv.CV_8U, [1,2,3,4,5,6]);

        cv.rotate(src, dst, cv.ROTATE_90_CLOCKWISE);

        let size = dst.size();
        assert.equal(size.height, 2, "ROTATE_HEIGHT");
        assert.equal(size.width, 3, "ROTATE_WIGTH");

        let expected = new Uint8Array([5,3,1,6,4,2]);

        assert.deepEqual(dst.data, expected);

        dst.delete();
        src.delete();
    }
});

QUnit.test('test_LUT', function(assert) {
    {
        let src = cv.matFromArray(3, 3, cv.CV_8UC1, [255, 128, 0, 0, 128, 255, 1, 2, 254]);
        let lutTable = [];
        for (let i = 0; i < 256; i++)
        {
           lutTable[i] = 255 - i;
        }
        let lut = cv.matFromArray(1, 256, cv.CV_8UC1, lutTable);
        let dst = new cv.Mat();

        cv.LUT(src, lut, dst);

        // Verify result.
        assert.equal(dst.ucharAt(0), 0);
        assert.equal(dst.ucharAt(1), 127);
        assert.equal(dst.ucharAt(2), 255);
        assert.equal(dst.ucharAt(3), 255);
        assert.equal(dst.ucharAt(4), 127);
        assert.equal(dst.ucharAt(5), 0);
        assert.equal(dst.ucharAt(6), 254);
        assert.equal(dst.ucharAt(7), 253);
        assert.equal(dst.ucharAt(8), 1);

        src.delete();
        lut.delete();
        dst.delete();
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

