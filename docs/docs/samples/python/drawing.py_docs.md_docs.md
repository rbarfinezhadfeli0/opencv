# Documentation for `docs/samples/python/drawing.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/drawing.py_docs.md`
- **File Name**: `drawing.py_docs.md`
- **File Size**: 10,345 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/drawing.py_docs.md](../../../docs/samples/python/drawing.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/drawing.py`

## File Metadata

- **Full Path**: `samples/python/drawing.py`
- **File Name**: `drawing.py`
- **File Size**: 6,955 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/drawing.py](../../samples/python/drawing.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
    This program demonstrates OpenCV drawing and text output functions by drawing different shapes and text strings
    Usage :
        python3 drawing.py
    Press any button to exit
    '''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

# Drawing Lines
def lines():
    for i in range(NUMBER*2):
        pt1, pt2 = [], []
        pt1.append(np.random.randint(x1, x2))
        pt1.append(np.random.randint(y1, y2))
        pt2.append(np.random.randint(x1, x2))
        pt2.append(np.random.randint(y1, y2))
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        arrowed =  np.random.randint(0, 6)
        if (arrowed<3):
            cv.line(image, tuple(pt1), tuple(pt2), color, np.random.randint(1, 10), lineType)
        else:
            cv.arrowedLine(image, tuple(pt1), tuple(pt2), color, np.random.randint(1, 10), lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY)>=0:
            return

# Drawing Rectangle
def rectangle():
    for i in range(NUMBER*2):
        pt1, pt2 = [], []
        pt1.append(np.random.randint(x1, x2))
        pt1.append(np.random.randint(y1, y2))
        pt2.append(np.random.randint(x1, x2))
        pt2.append(np.random.randint(y1, y2))
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        thickness = np.random.randint(-3, 10)
        marker = np.random.randint(0, 10)
        marker_size = np.random.randint(30, 80)

        if (marker > 5):
            cv.rectangle(image, tuple(pt1), tuple(pt2), color, max(thickness, -1), lineType)
        else:
            cv.drawMarker(image, tuple(pt1), color, marker, marker_size)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY)>=0:
            return

# Drawing ellipse
def ellipse():
    for i in range(NUMBER*2):
        center = []
        center.append(np.random.randint(x1, x2))
        center.append(np.random.randint(y1, y2))
        axes = []
        axes.append(np.random.randint(0, 200))
        axes.append(np.random.randint(0, 200))
        angle = np.random.randint(0, 180)
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        thickness = np.random.randint(-1, 9)
        cv.ellipse(image, tuple(center), tuple(axes), angle, angle-100, angle + 200, color, thickness, lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY)>=0:
            return

# Drawing Polygonal Curves
def polygonal():
    for i in range(NUMBER):
        pt = [(0, 0)]*6
        pt = np.resize(pt, (2, 3, 2))
        pt[0][0][0] = np.random.randint(x1, x2)
        pt[0][0][1] = np.random.randint(y1, y2)
        pt[0][1][0] = np.random.randint(x1, x2)
        pt[0][1][1] = np.random.randint(y1, y2)
        pt[0][2][0] = np.random.randint(x1, x2)
        pt[0][2][1] = np.random.randint(y1, y2)
        pt[1][0][0] = np.random.randint(x1, x2)
        pt[1][0][1] = np.random.randint(y1, y2)
        pt[1][1][0] = np.random.randint(x1, x2)
        pt[1][1][1] = np.random.randint(y1, y2)
        pt[1][2][0] = np.random.randint(x1, x2)
        pt[1][2][1] = np.random.randint(y1, y2)
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        alist = []
        for k in pt[0]:
            alist.append(k)
        for k in pt[1]:
            alist.append(k)
        ppt = np.array(alist)
        cv.polylines(image, [ppt], True, color, thickness = np.random.randint(1, 10), lineType = lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY) >= 0:
            return

# fills an area bounded by several polygonal contours
def fill():
    for i in range(NUMBER):
        pt = [(0, 0)]*6
        pt = np.resize(pt, (2, 3, 2))
        pt[0][0][0] = np.random.randint(x1, x2)
        pt[0][0][1] = np.random.randint(y1, y2)
        pt[0][1][0] = np.random.randint(x1, x2)
        pt[0][1][1] = np.random.randint(y1, y2)
        pt[0][2][0] = np.random.randint(x1, x2)
        pt[0][2][1] = np.random.randint(y1, y2)
        pt[1][0][0] = np.random.randint(x1, x2)
        pt[1][0][1] = np.random.randint(y1, y2)
        pt[1][1][0] = np.random.randint(x1, x2)
        pt[1][1][1] = np.random.randint(y1, y2)
        pt[1][2][0] = np.random.randint(x1, x2)
        pt[1][2][1] = np.random.randint(y1, y2)
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        alist = []
        for k in pt[0]:
            alist.append(k)
        for k in pt[1]:
            alist.append(k)
        ppt = np.array(alist)
        cv.fillPoly(image, [ppt], color, lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY) >= 0:
            return

# Drawing Circles
def circles():
    for i in range(NUMBER):
        center = []
        center.append(np.random.randint(x1, x2))
        center.append(np.random.randint(y1, y2))
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        cv.circle(image, tuple(center), np.random.randint(0, 300), color, np.random.randint(-1, 9), lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY) >= 0:
            return

# Draws a text string
def string():
    for i in range(NUMBER):
        org = []
        org.append(np.random.randint(x1, x2))
        org.append(np.random.randint(y1, y2))
        color = "%06x" % np.random.randint(0, 0xFFFFFF)
        color = tuple(int(color[i:i+2], 16) for i in (0, 2 ,4))
        cv.putText(image, "Testing text rendering", tuple(org), np.random.randint(0, 8), np.random.randint(0, 100)*0.05+0.1, color, np.random.randint(1, 10), lineType)
        cv.imshow(wndname, image)
        if cv.waitKey(DELAY) >= 0:
            return


def string1():
    textsize = cv.getTextSize("OpenCV forever!", cv.FONT_HERSHEY_COMPLEX, 3, 5)
    org = (int((width - textsize[0][0])/2), int((height - textsize[0][1])/2))
    for i in range(0, 255, 2):
        image2 = np.array(image) - i
        cv.putText(image2, "OpenCV forever!", org, cv.FONT_HERSHEY_COMPLEX, 3, (i, i, 255), 5, lineType)
        cv.imshow(wndname, image2)
        if cv.waitKey(DELAY) >= 0:
            return

if __name__ == '__main__':
    print(__doc__)
    wndname = "Drawing Demo"
    NUMBER = 100
    DELAY = 5
    width, height = 1000, 700
    lineType = cv.LINE_AA  # change it to LINE_8 to see non-antialiased graphics
    x1, x2, y1, y2 = -width/2, width*3/2, -height/2, height*3/2
    image = np.zeros((height, width, 3), dtype = np.uint8)
    cv.imshow(wndname, image)
    cv.waitKey(DELAY)
    lines()
    rectangle()
    ellipse()
    polygonal()
    fill()
    circles()
    string()
    string1()
    cv.waitKey(0)
    cv.destroyAllWindows()```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **lines()**: A function/method defined in this file
- **fill()**: A function/method defined in this file
- **circles()**: A function/method defined in this file
- **string1()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **ellipse()**: A function/method defined in this file
- **rectangle()**: A function/method defined in this file
- **string()**: A function/method defined in this file
- **polygonal()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `print_function`
- `__future__`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

