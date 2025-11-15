# Documentation for `samples/python/tutorial_code/imgProc/changing_contrast_brightness_image/changing_contrast_brightness_image.py`

## File Metadata

- **Full Path**: `samples/python/tutorial_code/imgProc/changing_contrast_brightness_image/changing_contrast_brightness_image.py`
- **File Name**: `changing_contrast_brightness_image.py`
- **File Size**: 2,591 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/tutorial_code/imgProc/changing_contrast_brightness_image/changing_contrast_brightness_image.py](../../../../../samples/python/tutorial_code/imgProc/changing_contrast_brightness_image/changing_contrast_brightness_image.py)

## Purpose and Role

This file is located in the `samples/python/tutorial_code/imgProc/changing_contrast_brightness_image` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
from __future__ import print_function
from __future__ import division
import cv2 as cv
import numpy as np
import argparse

alpha = 1.0
alpha_max = 500
beta = 0
beta_max = 200
gamma = 1.0
gamma_max = 200

def basicLinearTransform():
    res = cv.convertScaleAbs(img_original, alpha=alpha, beta=beta)
    img_corrected = cv.hconcat([img_original, res])
    cv.imshow("Brightness and contrast adjustments", img_corrected)

def gammaCorrection():
    ## [changing-contrast-brightness-gamma-correction]
    lookUpTable = np.empty((1,256), np.uint8)
    for i in range(256):
        lookUpTable[0,i] = np.clip(pow(i / 255.0, gamma) * 255.0, 0, 255)

    res = cv.LUT(img_original, lookUpTable)
    ## [changing-contrast-brightness-gamma-correction]

    img_gamma_corrected = cv.hconcat([img_original, res])
    cv.imshow("Gamma correction", img_gamma_corrected)

def on_linear_transform_alpha_trackbar(val):
    global alpha
    alpha = val / 100
    basicLinearTransform()

def on_linear_transform_beta_trackbar(val):
    global beta
    beta = val - 100
    basicLinearTransform()

def on_gamma_correction_trackbar(val):
    global gamma
    gamma = val / 100
    gammaCorrection()

parser = argparse.ArgumentParser(description='Code for Changing the contrast and brightness of an image! tutorial.')
parser.add_argument('--input', help='Path to input image.', default='lena.jpg')
args = parser.parse_args()

img_original = cv.imread(cv.samples.findFile(args.input))
if img_original is None:
    print('Could not open or find the image: ', args.input)
    exit(0)

img_corrected = np.empty((img_original.shape[0], img_original.shape[1]*2, img_original.shape[2]), img_original.dtype)
img_gamma_corrected = np.empty((img_original.shape[0], img_original.shape[1]*2, img_original.shape[2]), img_original.dtype)

img_corrected = cv.hconcat([img_original, img_original])
img_gamma_corrected = cv.hconcat([img_original, img_original])

cv.namedWindow('Brightness and contrast adjustments')
cv.namedWindow('Gamma correction')

alpha_init = int(alpha *100)
cv.createTrackbar('Alpha gain (contrast)', 'Brightness and contrast adjustments', alpha_init, alpha_max, on_linear_transform_alpha_trackbar)
beta_init = beta + 100
cv.createTrackbar('Beta bias (brightness)', 'Brightness and contrast adjustments', beta_init, beta_max, on_linear_transform_beta_trackbar)
gamma_init = int(gamma * 100)
cv.createTrackbar('Gamma correction', 'Gamma correction', gamma_init, gamma_max, on_gamma_correction_trackbar)

on_linear_transform_alpha_trackbar(alpha_init)
on_gamma_correction_trackbar(gamma_init)

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

- **on_gamma_correction_trackbar()**: A function/method defined in this file
- **gammaCorrection()**: A function/method defined in this file
- **on_linear_transform_beta_trackbar()**: A function/method defined in this file
- **from()**: A function/method defined in this file
- **basicLinearTransform()**: A function/method defined in this file
- **on_linear_transform_alpha_trackbar()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `print_function`
- `numpy`
- `division`
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

