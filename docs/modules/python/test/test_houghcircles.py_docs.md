# Documentation for `modules/python/test/test_houghcircles.py`

## File Metadata

- **Full Path**: `modules/python/test/test_houghcircles.py`
- **File Name**: `test_houghcircles.py`
- **File Size**: 4,634 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_houghcircles.py](../../../modules/python/test/test_houghcircles.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/python

'''
This example illustrates how to use cv.HoughCircles() function.
'''

# Python 2/3 compatibility
from __future__ import print_function

import cv2 as cv
import numpy as np
import sys
from numpy import pi, sin, cos

from tests_common import NewOpenCVTests

def circleApproximation(circle):

    nPoints = 30
    dPhi = 2*pi / nPoints
    contour = []
    for i in range(nPoints):
        contour.append(([circle[0] + circle[2]*cos(i*dPhi),
            circle[1] + circle[2]*sin(i*dPhi)]))

    return np.array(contour).astype(int)

def convContoursIntersectiponRate(c1, c2):

    s1 = cv.contourArea(c1)
    s2 = cv.contourArea(c2)

    s, _ = cv.intersectConvexConvex(c1, c2)

    return 2*s/(s1+s2)

class houghcircles_test(NewOpenCVTests):

    def test_houghcircles(self):

        fn = "samples/data/board.jpg"

        src = self.get_sample(fn, 1)
        img = cv.cvtColor(src, cv.COLOR_BGR2GRAY)
        img = cv.medianBlur(img, 5)

        circles = cv.HoughCircles(img, cv.HOUGH_GRADIENT, 1, 10, np.array([]), 100, 30, 1, 30)[0]

        testCircles = [[38, 181, 17.6],
        [99.7, 166, 13.12],
        [142.7, 160, 13.52],
        [223.6, 110, 8.62],
        [79.1, 206.7, 8.62],
        [47.5, 351.6, 11.64],
        [189.5, 354.4, 11.64],
        [189.8, 298.9, 10.64],
        [189.5, 252.4, 14.62],
        [252.5, 393.4, 15.62],
        [602.9, 467.5, 11.42],
        [222, 210.4, 9.12],
        [263.1, 216.7, 9.12],
        [359.8, 222.6, 9.12],
        [518.9, 120.9, 9.12],
        [413.8, 113.4, 9.12],
        [489, 127.2, 9.12],
        [448.4, 121.3, 9.12],
        [384.6, 128.9, 8.62]]

        matches_counter = 0

        for i in range(len(testCircles)):
            for j in range(len(circles)):

                tstCircle = circleApproximation(testCircles[i])
                circle = circleApproximation(circles[j])
                if convContoursIntersectiponRate(tstCircle, circle) > 0.6:
                    matches_counter += 1

        self.assertGreater(float(matches_counter) / len(testCircles), .5)
        self.assertLess(float(len(circles) - matches_counter) / len(circles), .75)

        circles_acc = cv.HoughCirclesWithAccumulator(
            image=img,
            method=cv.HOUGH_GRADIENT,
            dp=1,
            minDist=10,
            circles=np.array([]),
            param1=150,
            param2=45,
            minRadius=1,
            maxRadius=30)

        self.assertEqual(circles_acc.shape, (1, 2, 4))
        self.assertEqual(circles_acc[0, 0, 3], 66.)
        self.assertEqual(circles_acc[0, 1, 3], 62.)

    def test_houghcircles_alt(self):

        fn = "samples/data/board.jpg"

        src = self.get_sample(fn, 1)
        img = cv.cvtColor(src, cv.COLOR_BGR2GRAY)
        img = cv.medianBlur(img, 5)

        circles = cv.HoughCircles(img, cv.HOUGH_GRADIENT_ALT, 1, 10, np.array([]), 300, 0.9, 1, 30)

        self.assertEqual(circles.shape, (1, 18, 3))

        circles = circles[0]

        testCircles = [[38, 181, 17.6],
        [99.7, 166, 13.12],
        [142.7, 160, 13.52],
        [223.6, 110, 8.62],
        [79.1, 206.7, 8.62],
        [47.5, 351.6, 11.64],
        [189.5, 354.4, 11.64],
        [189.8, 298.9, 10.64],
        [189.5, 252.4, 14.62],
        [252.5, 393.4, 15.62],
        [602.9, 467.5, 11.42],
        [222, 210.4, 9.12],
        [263.1, 216.7, 9.12],
        [359.8, 222.6, 9.12],
        [518.9, 120.9, 9.12],
        [413.8, 113.4, 9.12],
        [489, 127.2, 9.12],
        [448.4, 121.3, 9.12],
        [384.6, 128.9, 8.62]]

        matches_counter = 0

        for i in range(len(testCircles)):
            for j in range(len(circles)):

                tstCircle = circleApproximation(testCircles[i])
                circle = circleApproximation(circles[j])
                if convContoursIntersectiponRate(tstCircle, circle) > 0.6:
                    matches_counter += 1

        self.assertGreater(float(matches_counter) / len(testCircles), .5)
        self.assertLess(float(len(circles) - matches_counter) / len(circles), .75)

        circles_acc = cv.HoughCirclesWithAccumulator(
            image=img,
            method=cv.HOUGH_GRADIENT_ALT,
            dp=1,
            minDist=10,
            circles=np.array([]),
            param1=300,
            param2=0.9,
            minRadius=13,
            maxRadius=15)

        self.assertEqual(circles_acc.shape, (1, 3, 4))
        self.assertEqual(circles_acc[0, 0, 3], 62.)
        self.assertEqual(circles_acc[0, 1, 3], 59.)
        self.assertEqual(circles_acc[0, 2, 3], 47.)

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

- **houghcircles_test**: A class/struct defined in this file

### Functions and Methods

- **circleApproximation()**: A function/method defined in this file
- **test_houghcircles()**: A function/method defined in this file
- **test_houghcircles_alt()**: A function/method defined in this file
- **convContoursIntersectiponRate()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `NewOpenCVTests`
- `pi`
- `print_function`
- `numpy`
- `cv2`
- `__future__`


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

