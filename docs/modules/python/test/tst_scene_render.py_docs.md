# Documentation for `modules/python/test/tst_scene_render.py`

## File Metadata

- **Full Path**: `modules/python/test/tst_scene_render.py`
- **File Name**: `tst_scene_render.py`
- **File Size**: 3,952 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/tst_scene_render.py](../../../modules/python/test/tst_scene_render.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python


# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
from numpy import pi, sin, cos

import cv2 as cv

defaultSize = 512

class TestSceneRender():

    def __init__(self, bgImg = None, fgImg = None, deformation = False, noise = 0.0, speed = 0.25, **params):
        self.time = 0.0
        self.timeStep = 1.0 / 30.0
        self.foreground = fgImg
        self.deformation = deformation
        self.noise = noise
        self.speed = speed

        if bgImg is not None:
            self.sceneBg = bgImg.copy()
        else:
            self.sceneBg = np.zeros(defaultSize, defaultSize, np.uint8)

        self.w = self.sceneBg.shape[0]
        self.h = self.sceneBg.shape[1]

        if fgImg is not None:
            self.foreground = fgImg.copy()
            self.center = self.currentCenter = (int(self.w/2 - fgImg.shape[0]/2), int(self.h/2 - fgImg.shape[1]/2))

            self.xAmpl = self.sceneBg.shape[0] - (self.center[0] + fgImg.shape[0])
            self.yAmpl = self.sceneBg.shape[1] - (self.center[1] + fgImg.shape[1])

        self.initialRect = np.array([ (self.h/2, self.w/2), (self.h/2, self.w/2 + self.w/10),
         (self.h/2 + self.h/10, self.w/2 + self.w/10), (self.h/2 + self.h/10, self.w/2)]).astype(int)
        self.currentRect = self.initialRect
        np.random.seed(10)

    def getXOffset(self, time):
        return int(self.xAmpl*cos(time*self.speed))


    def getYOffset(self, time):
        return int(self.yAmpl*sin(time*self.speed))

    def setInitialRect(self, rect):
        self.initialRect = rect

    def getRectInTime(self, time):

        if self.foreground is not None:
            tmp = np.array(self.center) + np.array((self.getXOffset(time), self.getYOffset(time)))
            x0, y0 = tmp
            x1, y1 = tmp + self.foreground.shape[0:2]
            return np.array([y0, x0, y1, x1])
        else:
            x0, y0 = self.initialRect[0] + np.array((self.getXOffset(time), self.getYOffset(time)))
            x1, y1 = self.initialRect[2] + np.array((self.getXOffset(time), self.getYOffset(time)))
            return np.array([y0, x0, y1, x1])

    def getCurrentRect(self):

        if self.foreground is not None:

            x0 = self.currentCenter[0]
            y0 = self.currentCenter[1]
            x1 = self.currentCenter[0] + self.foreground.shape[0]
            y1 = self.currentCenter[1] + self.foreground.shape[1]
            return np.array([y0, x0, y1, x1])
        else:
            x0, y0 = self.currentRect[0]
            x1, y1 = self.currentRect[2]
            return np.array([x0, y0, x1, y1])

    def getNextFrame(self):
        img = self.sceneBg.copy()

        if self.foreground is not None:
            self.currentCenter = (self.center[0] + self.getXOffset(self.time), self.center[1] + self.getYOffset(self.time))
            img[self.currentCenter[0]:self.currentCenter[0]+self.foreground.shape[0],
             self.currentCenter[1]:self.currentCenter[1]+self.foreground.shape[1]] = self.foreground
        else:
            self.currentRect = self.initialRect + int( 30*cos(self.time) + 50*sin(self.time/3))
            if self.deformation:
                self.currentRect[1:3] += int(self.h/20*cos(self.time))
            cv.fillConvexPoly(img, self.currentRect, (0, 0, 255))

        self.time += self.timeStep

        if self.noise:
            noise = np.zeros(self.sceneBg.shape, np.int8)
            cv.randn(noise, np.zeros(3), np.ones(3)*255*self.noise)
            img = cv.add(img, noise, dtype=cv.CV_8UC3)
        return img

    def resetTime(self):
        self.time = 0.0


if __name__ == '__main__':

    backGr = cv.imread('../../../samples/data/lena.jpg')

    render = TestSceneRender(backGr, noise = 0.5)

    while True:

        img = render.getNextFrame()
        cv.imshow('img', img)

        ch = cv.waitKey(3)
        if ch == 27:
            break
    cv.destroyAllWindows()
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

- **TestSceneRender**: A class/struct defined in this file

### Functions and Methods

- **getNextFrame()**: A function/method defined in this file
- **getRectInTime()**: A function/method defined in this file
- **getXOffset()**: A function/method defined in this file
- **getCurrentRect()**: A function/method defined in this file
- **setInitialRect()**: A function/method defined in this file
- **resetTime()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **getYOffset()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
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

