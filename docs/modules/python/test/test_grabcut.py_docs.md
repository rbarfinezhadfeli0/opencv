# Documentation for `modules/python/test/test_grabcut.py`

## File Metadata

- **Full Path**: `modules/python/test/test_grabcut.py`
- **File Name**: `test_grabcut.py`
- **File Size**: 2,500 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_grabcut.py](../../../modules/python/test/test_grabcut.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
===============================================================================
Interactive Image Segmentation using GrabCut algorithm.
===============================================================================
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv
import sys

from tests_common import NewOpenCVTests

class grabcut_test(NewOpenCVTests):

    def verify(self, mask, exp):

        maxDiffRatio = 0.02
        expArea = np.count_nonzero(exp)
        nonIntersectArea = np.count_nonzero(mask != exp)
        curRatio = float(nonIntersectArea) / expArea
        return curRatio < maxDiffRatio

    def scaleMask(self, mask):

        return np.where((mask==cv.GC_FGD) + (mask==cv.GC_PR_FGD),255,0).astype('uint8')

    def test_grabcut(self):

        img = self.get_sample('cv/shared/airplane.png')
        mask_prob = self.get_sample("cv/grabcut/mask_probpy.png", 0)
        exp_mask1 = self.get_sample("cv/grabcut/exp_mask1py.png", 0)
        exp_mask2 = self.get_sample("cv/grabcut/exp_mask2py.png", 0)

        if img is None:
            self.assertTrue(False, 'Missing test data')

        rect = (24, 126, 459, 168)
        mask = np.zeros(img.shape[:2], dtype = np.uint8)
        bgdModel = np.zeros((1,65),np.float64)
        fgdModel = np.zeros((1,65),np.float64)
        cv.grabCut(img, mask, rect, bgdModel, fgdModel, 0, cv.GC_INIT_WITH_RECT)
        cv.grabCut(img, mask, rect, bgdModel, fgdModel, 2, cv.GC_EVAL)

        if mask_prob is None:
            mask_prob = mask.copy()
            cv.imwrite(self.extraTestDataPath + '/cv/grabcut/mask_probpy.png', mask_prob)
        if exp_mask1 is None:
            exp_mask1 = self.scaleMask(mask)
            cv.imwrite(self.extraTestDataPath + '/cv/grabcut/exp_mask1py.png', exp_mask1)

        self.assertEqual(self.verify(self.scaleMask(mask), exp_mask1), True)

        mask = mask_prob
        bgdModel = np.zeros((1,65),np.float64)
        fgdModel = np.zeros((1,65),np.float64)
        cv.grabCut(img, mask, rect, bgdModel, fgdModel, 0, cv.GC_INIT_WITH_MASK)
        cv.grabCut(img, mask, rect, bgdModel, fgdModel, 1, cv.GC_EVAL)

        if exp_mask2 is None:
            exp_mask2 = self.scaleMask(mask)
            cv.imwrite(self.extraTestDataPath + '/cv/grabcut/exp_mask2py.png', exp_mask2)

        self.assertEqual(self.verify(self.scaleMask(mask), exp_mask2), True)


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

- **grabcut_test**: A class/struct defined in this file

### Functions and Methods

- **test_grabcut()**: A function/method defined in this file
- **scaleMask()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **verify()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `NewOpenCVTests`
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

