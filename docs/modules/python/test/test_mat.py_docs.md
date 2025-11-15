# Documentation for `modules/python/test/test_mat.py`

## File Metadata

- **Full Path**: `modules/python/test/test_mat.py`
- **File Name**: `test_mat.py`
- **File Size**: 5,365 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_mat.py](../../../modules/python/test/test_mat.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
from __future__ import print_function

import numpy as np
import cv2 as cv

import os
import sys
import unittest

from tests_common import NewOpenCVTests

try:
    if sys.version_info[:2] < (3, 0):
        raise unittest.SkipTest('Python 2.x is not supported')


    class MatTest(NewOpenCVTests):

        def test_mat_construct(self):
            data = np.random.random([10, 10, 3])

            #print(np.ndarray.__dictoffset__)  # 0
            #print(cv.Mat.__dictoffset__)  # 88 (> 0)
            #print(cv.Mat)  # <class cv2.Mat>
            #print(cv.Mat.__base__)  # <class 'numpy.ndarray'>

            mat_data0 = cv.Mat(data)
            assert isinstance(mat_data0, cv.Mat)
            assert isinstance(mat_data0, np.ndarray)
            self.assertEqual(mat_data0.wrap_channels, False)
            res0 = cv.utils.dumpInputArray(mat_data0)
            self.assertEqual(res0, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=300 dims(-1)=3 size(-1)=[10 10 3] type(-1)=CV_64FC1")

            mat_data1 = cv.Mat(data, wrap_channels=True)
            assert isinstance(mat_data1, cv.Mat)
            assert isinstance(mat_data1, np.ndarray)
            self.assertEqual(mat_data1.wrap_channels, True)
            res1 = cv.utils.dumpInputArray(mat_data1)
            self.assertEqual(res1, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=100 dims(-1)=2 size(-1)=10x10 type(-1)=CV_64FC3")

            mat_data2 = cv.Mat(mat_data1)
            assert isinstance(mat_data2, cv.Mat)
            assert isinstance(mat_data2, np.ndarray)
            self.assertEqual(mat_data2.wrap_channels, True)  # fail if __array_finalize__ doesn't work
            res2 = cv.utils.dumpInputArray(mat_data2)
            self.assertEqual(res2, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=100 dims(-1)=2 size(-1)=10x10 type(-1)=CV_64FC3")


        def test_mat_construct_4d(self):
            data = np.random.random([5, 10, 10, 3])

            mat_data0 = cv.Mat(data)
            assert isinstance(mat_data0, cv.Mat)
            assert isinstance(mat_data0, np.ndarray)
            self.assertEqual(mat_data0.wrap_channels, False)
            res0 = cv.utils.dumpInputArray(mat_data0)
            self.assertEqual(res0, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=1500 dims(-1)=4 size(-1)=[5 10 10 3] type(-1)=CV_64FC1")

            mat_data1 = cv.Mat(data, wrap_channels=True)
            assert isinstance(mat_data1, cv.Mat)
            assert isinstance(mat_data1, np.ndarray)
            self.assertEqual(mat_data1.wrap_channels, True)
            res1 = cv.utils.dumpInputArray(mat_data1)
            self.assertEqual(res1, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=500 dims(-1)=3 size(-1)=[5 10 10] type(-1)=CV_64FC3")

            mat_data2 = cv.Mat(mat_data1)
            assert isinstance(mat_data2, cv.Mat)
            assert isinstance(mat_data2, np.ndarray)
            self.assertEqual(mat_data2.wrap_channels, True)  # __array_finalize__ doesn't work
            res2 = cv.utils.dumpInputArray(mat_data2)
            self.assertEqual(res2, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=500 dims(-1)=3 size(-1)=[5 10 10] type(-1)=CV_64FC3")


        def test_mat_wrap_channels_fail(self):
            data = np.random.random([2, 3, 4, 520])

            mat_data0 = cv.Mat(data)
            assert isinstance(mat_data0, cv.Mat)
            assert isinstance(mat_data0, np.ndarray)
            self.assertEqual(mat_data0.wrap_channels, False)
            res0 = cv.utils.dumpInputArray(mat_data0)
            self.assertEqual(res0, "InputArray: empty()=false kind=0x00010000 flags=0x01010000 total(-1)=12480 dims(-1)=4 size(-1)=[2 3 4 520] type(-1)=CV_64FC1")

            with self.assertRaises(cv.error):
                mat_data1 = cv.Mat(data, wrap_channels=True)  # argument unable to wrap channels, too high (520 > CV_CN_MAX=512)
                res1 = cv.utils.dumpInputArray(mat_data1)
                print(mat_data1.__dict__)
                print(res1)


        def test_ufuncs(self):
            data = np.arange(10)
            mat_data = cv.Mat(data)
            mat_data2 = 2 * mat_data
            self.assertEqual(type(mat_data2), cv.Mat)
            np.testing.assert_equal(2 * data, 2 * mat_data)


        def test_comparison(self):
            # Undefined behavior, do NOT use that.
            # Behavior may be changed in the future

            data = np.ones((10, 10, 3))
            mat_wrapped = cv.Mat(data, wrap_channels=True)
            mat_simple = cv.Mat(data)
            np.testing.assert_equal(mat_wrapped, mat_simple)  # ???: wrap_channels is not checked for now
            np.testing.assert_equal(data, mat_simple)
            np.testing.assert_equal(data, mat_wrapped)

            #self.assertEqual(mat_wrapped, mat_simple)  # ???
            #self.assertTrue(mat_wrapped == mat_simple)  # ???
            #self.assertTrue((mat_wrapped == mat_simple).all())


except unittest.SkipTest as e:

    message = str(e)

    class TestSkip(unittest.TestCase):
        def setUp(self):
            self.skipTest('Skip tests: ' + message)

        def test_skip():
            pass

    pass


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

- **MatTest**: A class/struct defined in this file
- **cv2**: A class/struct defined in this file
- **TestSkip**: A class/struct defined in this file

### Functions and Methods

- **test_mat_construct_4d()**: A function/method defined in this file
- **test_skip()**: A function/method defined in this file
- **test_mat_wrap_channels_fail()**: A function/method defined in this file
- **test_ufuncs()**: A function/method defined in this file
- **test_comparison()**: A function/method defined in this file
- **test_mat_construct()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `os`
- `NewOpenCVTests`
- `print_function`
- `numpy`
- `cv2`
- `__future__`
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

