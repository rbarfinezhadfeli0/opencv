# Documentation for `modules/python/test/tests_common.py`

## File Metadata

- **Full Path**: `modules/python/test/tests_common.py`
- **File Name**: `tests_common.py`
- **File Size**: 4,025 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/tests_common.py](../../../modules/python/test/tests_common.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

from __future__ import print_function

import os
import sys
import unittest
import hashlib
import random
import argparse

import numpy as np
#sys.OpenCV_LOADER_DEBUG = True
import cv2 as cv

# Python 3 moved urlopen to urllib.requests
try:
    from urllib.request import urlopen
except ImportError:
    from urllib import urlopen

class NewOpenCVTests(unittest.TestCase):

    # path to local repository folder containing 'samples' folder
    repoPath = None
    extraTestDataPath = None
    extraDnnTestDataPath = None
    # github repository url
    repoUrl = 'https://raw.github.com/opencv/opencv/4.x'

    def find_file(self, filename, searchPaths=[], required=True):
        searchPaths = searchPaths if searchPaths else [self.repoPath, self.extraTestDataPath, self.extraDnnTestDataPath]
        for path in searchPaths:
            if path is not None:
                candidate = path + '/' + filename
                if os.path.isfile(candidate):
                    return candidate
        if required:
            self.fail('File ' + filename + ' not found')
        else:
            self.skipTest('File ' + filename + ' not found')
        return None


    def get_sample(self, filename, iscolor = None):
        if iscolor is None:
            iscolor = cv.IMREAD_COLOR
        if not filename in self.image_cache:
            filepath = self.find_file(filename)
            with open(filepath, 'rb') as f:
                filedata = f.read()
            self.image_cache[filename] = cv.imdecode(np.frombuffer(filedata, dtype=np.uint8), iscolor)
        return self.image_cache[filename]

    def setUp(self):
        cv.setRNGSeed(10)
        self.image_cache = {}

    def hashimg(self, im):
        """ Compute a hash for an image, useful for image comparisons """
        return hashlib.md5(im.tobytes()).hexdigest()

    if sys.version_info[:2] == (2, 6):
        def assertLess(self, a, b, msg=None):
            if not a < b:
                self.fail('%s not less than %s' % (repr(a), repr(b)))

        def assertLessEqual(self, a, b, msg=None):
            if not a <= b:
                self.fail('%s not less than or equal to %s' % (repr(a), repr(b)))

        def assertGreater(self, a, b, msg=None):
            if not a > b:
                self.fail('%s not greater than %s' % (repr(a), repr(b)))

    @staticmethod
    def bootstrap():
        parser = argparse.ArgumentParser(description='run OpenCV python tests')
        parser.add_argument('--repo', help='use sample image files from local git repository (path to folder), '
                                           'if not set, samples will be downloaded from github.com')
        parser.add_argument('--data', help='<not used> use data files from local folder (path to folder), '
                                            'if not set, data files will be downloaded from docs.opencv.org')
        args, other = parser.parse_known_args()
        print("Testing OpenCV", cv.__version__)
        print("Local repo path:", args.repo)
        NewOpenCVTests.repoPath = args.repo

        try:
            NewOpenCVTests.extraTestDataPath = os.environ['OPENCV_TEST_DATA_PATH']
        except KeyError:
            print('Missing opencv extra repository. Some of tests may fail.')

        try:
            NewOpenCVTests.extraDnnTestDataPath = os.environ['OPENCV_DNN_TEST_DATA_PATH']
        except KeyError:
            pass

        random.seed(0)
        unit_argv = [sys.argv[0]] + other
        unittest.main(argv=unit_argv)


def intersectionRate(s1, s2):

    x1, y1, x2, y2 = s1
    s1 = np.array([[x1, y1], [x2,y1], [x2, y2], [x1, y2]])

    x1, y1, x2, y2 = s2
    s2 = np.array([[x1, y1], [x2,y1], [x2, y2], [x1, y2]])

    area, _intersection = cv.intersectConvexConvex(s1, s2)
    return 2 * area / (cv.contourArea(s1) + cv.contourArea(s2))

def isPointInRect(p, rect):
    if rect[0] <= p[0] and rect[1] <=p[1] and p[0] <= rect[2] and p[1] <= rect[3]:
        return True
    else:
        return False
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

- **NewOpenCVTests**: A class/struct defined in this file

### Functions and Methods

- **find_file()**: A function/method defined in this file
- **assertLess()**: A function/method defined in this file
- **assertGreater()**: A function/method defined in this file
- **get_sample()**: A function/method defined in this file
- **bootstrap()**: A function/method defined in this file
- **intersectionRate()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **assertLessEqual()**: A function/method defined in this file
- **isPointInRect()**: A function/method defined in this file
- **hashimg()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `github.com`
- `sys`
- `local`
- `os`
- `random`
- `hashlib`
- `urllib.request`
- `docs.opencv.org`
- `urllib`
- `print_function`
- `numpy`
- `cv2`
- `argparse`
- `__future__`
- `urlopen`
- `unittest`


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

