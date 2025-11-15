# Documentation for `samples/python/mouse_and_match.py`

## File Metadata

- **Full Path**: `samples/python/mouse_and_match.py`
- **File Name**: `mouse_and_match.py`
- **File Size**: 3,161 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/mouse_and_match.py](../../samples/python/mouse_and_match.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
mouse_and_match.py [-i path | --input path: default ../data/]

Demonstrate using a mouse to interact with an image:
 Read in the images in a directory one by one
 Allow the user to select parts of an image with a mouse
 When they let go of the mouse, it correlates (using matchTemplate) that patch with the image.

 SPACE for next image
 ESC to exit
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

# built-in modules
import os
import sys
import glob
import argparse
from math import *


class App():
    drag_start = None
    sel = (0,0,0,0)

    def onmouse(self, event, x, y, flags, param):
        if event == cv.EVENT_LBUTTONDOWN:
            self.drag_start = x, y
            self.sel = (0,0,0,0)
        elif event == cv.EVENT_LBUTTONUP:
            if self.sel[2] > self.sel[0] and self.sel[3] > self.sel[1]:
                patch = self.gray[self.sel[1]:self.sel[3], self.sel[0]:self.sel[2]]
                result = cv.matchTemplate(self.gray, patch, cv.TM_CCOEFF_NORMED)
                result = np.abs(result)**3
                _val, result = cv.threshold(result, 0.01, 0, cv.THRESH_TOZERO)
                result8 = cv.normalize(result, None, 0, 255, cv.NORM_MINMAX, cv.CV_8U)
                cv.imshow("result", result8)
            self.drag_start = None
        elif self.drag_start:
            #print flags
            if flags & cv.EVENT_FLAG_LBUTTON:
                minpos = min(self.drag_start[0], x), min(self.drag_start[1], y)
                maxpos = max(self.drag_start[0], x), max(self.drag_start[1], y)
                self.sel = (minpos[0], minpos[1], maxpos[0], maxpos[1])
                img = cv.cvtColor(self.gray, cv.COLOR_GRAY2BGR)
                cv.rectangle(img, (self.sel[0], self.sel[1]), (self.sel[2], self.sel[3]), (0,255,255), 1)
                cv.imshow("gray", img)
            else:
                print("selection is complete")
                self.drag_start = None

    def run(self):
        parser = argparse.ArgumentParser(description='Demonstrate mouse interaction with images')
        parser.add_argument("-i","--input", default='../data/', help="Input directory.")
        args = parser.parse_args()
        path = args.input

        cv.namedWindow("gray",1)
        cv.setMouseCallback("gray", self.onmouse)
        '''Loop through all the images in the directory'''
        for infile in glob.glob( os.path.join(path, '*.*') ):
            ext = os.path.splitext(infile)[1][1:] #get the filename extension
            if ext == "png" or ext == "jpg" or ext == "bmp" or ext == "tiff" or ext == "pbm":
                print(infile)

                img = cv.imread(infile, cv.IMREAD_COLOR)
                if img is None:
                    continue
                self.sel = (0,0,0,0)
                self.drag_start = None
                self.gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
                cv.imshow("gray", self.gray)
                if cv.waitKey() == 27:
                    break

        print('Done')


if __name__ == '__main__':
    print(__doc__)
    App().run()
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

- **App**: A class/struct defined in this file

### Functions and Methods

- **onmouse()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **run()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `glob`
- `sys`
- `os`
- `print_function`
- `numpy`
- `cv2`
- `argparse`
- `math`
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

