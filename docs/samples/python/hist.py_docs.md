# Documentation for `samples/python/hist.py`

## File Metadata

- **Full Path**: `samples/python/hist.py`
- **File Name**: `hist.py`
- **File Size**: 3,705 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/hist.py](../../samples/python/hist.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

''' This is a sample for histogram plotting for RGB images and grayscale images for better understanding of colour distribution

Benefit : Learn how to draw histogram of images
          Get familier with cv.calcHist, cv.equalizeHist,cv.normalize and some drawing functions

Level : Beginner or Intermediate

Functions : 1) hist_curve : returns histogram of an image drawn as curves
            2) hist_lines : return histogram of an image drawn as bins ( only for grayscale images )

Usage : python hist.py <image_file>

Abid Rahman 3/14/12 debug Gary Bradski
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

bins = np.arange(256).reshape(256,1)

def hist_curve(im):
    h = np.zeros((300,256,3))
    if len(im.shape) == 2:
        color = [(255,255,255)]
    elif im.shape[2] == 3:
        color = [ (255,0,0),(0,255,0),(0,0,255) ]
    for ch, col in enumerate(color):
        hist_item = cv.calcHist([im],[ch],None,[256],[0,256])
        cv.normalize(hist_item,hist_item,0,255,cv.NORM_MINMAX)
        hist=np.int32(np.around(hist_item))
        pts = np.int32(np.column_stack((bins,hist)))
        cv.polylines(h,[pts],False,col)
    y=np.flipud(h)
    return y

def hist_lines(im):
    h = np.zeros((300,256,3))
    if len(im.shape)!=2:
        print("hist_lines applicable only for grayscale images")
        #print("so converting image to grayscale for representation"
        im = cv.cvtColor(im,cv.COLOR_BGR2GRAY)
    hist_item = cv.calcHist([im],[0],None,[256],[0,256])
    cv.normalize(hist_item,hist_item,0,255,cv.NORM_MINMAX)
    hist = np.int32(np.around(hist_item))
    for x,y in enumerate(hist):
        cv.line(h,(x,0),(x,y[0]),(255,255,255))
    y = np.flipud(h)
    return y


def main():
    import sys

    if len(sys.argv)>1:
        fname = sys.argv[1]
    else :
        fname = 'lena.jpg'
        print("usage : python hist.py <image_file>")

    im = cv.imread(cv.samples.findFile(fname))

    if im is None:
        print('Failed to load image file:', fname)
        sys.exit(1)

    gray = cv.cvtColor(im,cv.COLOR_BGR2GRAY)


    print(''' Histogram plotting \n
    Keymap :\n
    a - show histogram for color image in curve mode \n
    b - show histogram in bin mode \n
    c - show equalized histogram (always in bin mode) \n
    d - show histogram for gray image in curve mode \n
    e - show histogram for a normalized image in curve mode \n
    Esc - exit \n
    ''')

    cv.imshow('image',im)
    while True:
        k = cv.waitKey(0)
        if k == ord('a'):
            curve = hist_curve(im)
            cv.imshow('histogram',curve)
            cv.imshow('image',im)
            print('a')
        elif k == ord('b'):
            print('b')
            lines = hist_lines(im)
            cv.imshow('histogram',lines)
            cv.imshow('image',gray)
        elif k == ord('c'):
            print('c')
            equ = cv.equalizeHist(gray)
            lines = hist_lines(equ)
            cv.imshow('histogram',lines)
            cv.imshow('image',equ)
        elif k == ord('d'):
            print('d')
            curve = hist_curve(gray)
            cv.imshow('histogram',curve)
            cv.imshow('image',gray)
        elif k == ord('e'):
            print('e')
            norm = cv.normalize(gray, gray, alpha = 0,beta = 255,norm_type = cv.NORM_MINMAX)
            lines = hist_lines(norm)
            cv.imshow('histogram',lines)
            cv.imshow('image',norm)
        elif k == 27:
            print('ESC')
            cv.destroyAllWindows()
            break

    print('Done')


if __name__ == '__main__':
    print(__doc__)
    main()
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

### Functions and Methods

- **hist_curve()**: A function/method defined in this file
- **hist_lines()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **main()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
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

