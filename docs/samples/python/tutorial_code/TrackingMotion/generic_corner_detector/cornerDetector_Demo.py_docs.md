# Documentation for `samples/python/tutorial_code/TrackingMotion/generic_corner_detector/cornerDetector_Demo.py`

## File Metadata

- **Full Path**: `samples/python/tutorial_code/TrackingMotion/generic_corner_detector/cornerDetector_Demo.py`
- **File Name**: `cornerDetector_Demo.py`
- **File Size**: 2,939 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/tutorial_code/TrackingMotion/generic_corner_detector/cornerDetector_Demo.py](../../../../../samples/python/tutorial_code/TrackingMotion/generic_corner_detector/cornerDetector_Demo.py)

## Purpose and Role

This file is located in the `samples/python/tutorial_code/TrackingMotion/generic_corner_detector` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
from __future__ import print_function
import cv2 as cv
import numpy as np
import argparse
import random as rng

myHarris_window = 'My Harris corner detector'
myShiTomasi_window = 'My Shi Tomasi corner detector'
myHarris_qualityLevel = 50
myShiTomasi_qualityLevel = 50
max_qualityLevel = 100
rng.seed(12345)

def myHarris_function(val):
    myHarris_copy = np.copy(src)
    myHarris_qualityLevel = max(val, 1)

    for i in range(src_gray.shape[0]):
        for j in range(src_gray.shape[1]):
            if Mc[i,j] > myHarris_minVal + ( myHarris_maxVal - myHarris_minVal )*myHarris_qualityLevel/max_qualityLevel:
                cv.circle(myHarris_copy, (j,i), 4, (rng.randint(0,256), rng.randint(0,256), rng.randint(0,256)), cv.FILLED)

    cv.imshow(myHarris_window, myHarris_copy)

def myShiTomasi_function(val):
    myShiTomasi_copy = np.copy(src)
    myShiTomasi_qualityLevel = max(val, 1)

    for i in range(src_gray.shape[0]):
        for j in range(src_gray.shape[1]):
            if myShiTomasi_dst[i,j] > myShiTomasi_minVal + ( myShiTomasi_maxVal - myShiTomasi_minVal )*myShiTomasi_qualityLevel/max_qualityLevel:
                cv.circle(myShiTomasi_copy, (j,i), 4, (rng.randint(0,256), rng.randint(0,256), rng.randint(0,256)), cv.FILLED)

    cv.imshow(myShiTomasi_window, myShiTomasi_copy)

# Load source image and convert it to gray
parser = argparse.ArgumentParser(description='Code for Creating your own corner detector tutorial.')
parser.add_argument('--input', help='Path to input image.', default='building.jpg')
args = parser.parse_args()

src = cv.imread(cv.samples.findFile(args.input))
if src is None:
    print('Could not open or find the image:', args.input)
    exit(0)

src_gray = cv.cvtColor(src, cv.COLOR_BGR2GRAY)

# Set some parameters
blockSize = 3
apertureSize = 3

# My Harris matrix -- Using cornerEigenValsAndVecs
myHarris_dst = cv.cornerEigenValsAndVecs(src_gray, blockSize, apertureSize)

# calculate Mc
Mc = np.empty(src_gray.shape, dtype=np.float32)
for i in range(src_gray.shape[0]):
    for j in range(src_gray.shape[1]):
        lambda_1 = myHarris_dst[i,j,0]
        lambda_2 = myHarris_dst[i,j,1]
        Mc[i,j] = lambda_1*lambda_2 - 0.04*pow( ( lambda_1 + lambda_2 ), 2 )

myHarris_minVal, myHarris_maxVal, _, _ = cv.minMaxLoc(Mc)

# Create Window and Trackbar
cv.namedWindow(myHarris_window)
cv.createTrackbar('Quality Level:', myHarris_window, myHarris_qualityLevel, max_qualityLevel, myHarris_function)
myHarris_function(myHarris_qualityLevel)

# My Shi-Tomasi -- Using cornerMinEigenVal
myShiTomasi_dst = cv.cornerMinEigenVal(src_gray, blockSize, apertureSize)

myShiTomasi_minVal, myShiTomasi_maxVal, _, _ = cv.minMaxLoc(myShiTomasi_dst)

# Create Window and Trackbar
cv.namedWindow(myShiTomasi_window)
cv.createTrackbar('Quality Level:', myShiTomasi_window, myShiTomasi_qualityLevel, max_qualityLevel, myShiTomasi_function)
myShiTomasi_function(myShiTomasi_qualityLevel)

cv.waitKey()
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

- **myHarris_function()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **myShiTomasi_function()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `random`
- `print_function`
- `numpy`
- `cv2`
- `argparse`
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

