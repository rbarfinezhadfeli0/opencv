# Documentation for `modules/python/test/test_squares.py`

## File Metadata

- **Full Path**: `modules/python/test/test_squares.py`
- **File Name**: `test_squares.py`
- **File Size**: 2,792 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_squares.py](../../../modules/python/test/test_squares.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Simple "Square Detector" program.

Loads several images sequentially and tries to find squares in each image.
'''

# Python 2/3 compatibility
import sys
PY3 = sys.version_info[0] == 3

if PY3:
    xrange = range

import numpy as np
import cv2 as cv


def angle_cos(p0, p1, p2):
    d1, d2 = (p0-p1).astype('float'), (p2-p1).astype('float')
    return abs( np.dot(d1, d2) / np.sqrt( np.dot(d1, d1)*np.dot(d2, d2) ) )

def find_squares(img):
    img = cv.GaussianBlur(img, (5, 5), 0)
    squares = []
    for gray in cv.split(img):
        for thrs in xrange(0, 255, 26):
            if thrs == 0:
                bin = cv.Canny(gray, 0, 50, apertureSize=5)
                bin = cv.dilate(bin, None)
            else:
                _retval, bin = cv.threshold(gray, thrs, 255, cv.THRESH_BINARY)
            contours, _hierarchy = cv.findContours(bin, cv.RETR_LIST, cv.CHAIN_APPROX_SIMPLE)
            for cnt in contours:
                cnt_len = cv.arcLength(cnt, True)
                cnt = cv.approxPolyDP(cnt, 0.02*cnt_len, True)
                if len(cnt) == 4 and cv.contourArea(cnt) > 1000 and cv.isContourConvex(cnt):
                    cnt = cnt.reshape(-1, 2)
                    max_cos = np.max([angle_cos( cnt[i], cnt[(i+1) % 4], cnt[(i+2) % 4] ) for i in xrange(4)])
                    if max_cos < 0.1 and filterSquares(squares, cnt):
                        squares.append(cnt)

    return squares

def intersectionRate(s1, s2):
    area, _intersection = cv.intersectConvexConvex(np.array(s1), np.array(s2))
    return 2 * area / (cv.contourArea(np.array(s1)) + cv.contourArea(np.array(s2)))

def filterSquares(squares, square):

    for i in range(len(squares)):
        if intersectionRate(squares[i], square) > 0.95:
            return False

    return True

from tests_common import NewOpenCVTests

class squares_test(NewOpenCVTests):

    def test_squares(self):

        img = self.get_sample('samples/data/pic1.png')
        squares = find_squares(img)

        testSquares = [
        [[43, 25],
        [43, 129],
        [232, 129],
        [232, 25]],

        [[252, 87],
        [324, 40],
        [387, 137],
        [315, 184]],

        [[154, 178],
        [196, 180],
        [198, 278],
        [154, 278]],

        [[0, 0],
        [400, 0],
        [400, 300],
        [0, 300]]
        ]

        matches_counter = 0
        for i in range(len(squares)):
            for j in range(len(testSquares)):
                if intersectionRate(squares[i], testSquares[j]) > 0.9:
                    matches_counter += 1

        self.assertGreater(matches_counter / len(testSquares), 0.9)
        self.assertLess( (len(squares) - matches_counter) / len(squares), 0.2)

if __name__ == '__main__':
    NewOpenCVTests.bootstrap()
```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **squares_test**: A class/struct defined in this file

### Functions and Methods

- **find_squares()**: A function/method defined in this file
- **intersectionRate()**: A function/method defined in this file
- **angle_cos()**: A function/method defined in this file
- **filterSquares()**: A function/method defined in this file
- **test_squares()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `NewOpenCVTests`
- `numpy`
- `cv2`


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

