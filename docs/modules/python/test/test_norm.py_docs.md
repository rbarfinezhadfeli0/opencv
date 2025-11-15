# Documentation for `modules/python/test/test_norm.py`

## File Metadata

- **Full Path**: `modules/python/test/test_norm.py`
- **File Name**: `test_norm.py`
- **File Size**: 5,778 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_norm.py](../../../modules/python/test/test_norm.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

from itertools import product
from functools import reduce

import numpy as np
import cv2 as cv

from tests_common import NewOpenCVTests


def norm_inf(x, y=None):
    def norm(vec):
        return np.linalg.norm(vec.flatten(), np.inf)

    x = x.astype(np.float64)
    return norm(x) if y is None else norm(x - y.astype(np.float64))


def norm_l1(x, y=None):
    def norm(vec):
        return np.linalg.norm(vec.flatten(), 1)

    x = x.astype(np.float64)
    return norm(x) if y is None else norm(x - y.astype(np.float64))


def norm_l2(x, y=None):
    def norm(vec):
        return np.linalg.norm(vec.flatten())

    x = x.astype(np.float64)
    return norm(x) if y is None else norm(x - y.astype(np.float64))


def norm_l2sqr(x, y=None):
    def norm(vec):
        return np.square(vec).sum()

    x = x.astype(np.float64)
    return norm(x) if y is None else norm(x - y.astype(np.float64))


def norm_hamming(x, y=None):
    def norm(vec):
        return sum(bin(i).count('1') for i in vec.flatten())

    return norm(x) if y is None else norm(np.bitwise_xor(x, y))


def norm_hamming2(x, y=None):
    def norm(vec):
        def element_norm(element):
            binary_str = bin(element).split('b')[-1]
            if len(binary_str) % 2 == 1:
                binary_str = '0' + binary_str
            gen = filter(lambda p: p != '00',
                         (binary_str[i:i+2]
                          for i in range(0, len(binary_str), 2)))
            return sum(1 for _ in gen)

        return sum(element_norm(element) for element in vec.flatten())

    return norm(x) if y is None else norm(np.bitwise_xor(x, y))


norm_type_under_test = {
    cv.NORM_INF: norm_inf,
    cv.NORM_L1: norm_l1,
    cv.NORM_L2: norm_l2,
    cv.NORM_L2SQR: norm_l2sqr,
    cv.NORM_HAMMING: norm_hamming,
    cv.NORM_HAMMING2: norm_hamming2
}

norm_name = {
    cv.NORM_INF: 'inf',
    cv.NORM_L1: 'L1',
    cv.NORM_L2: 'L2',
    cv.NORM_L2SQR: 'L2SQR',
    cv.NORM_HAMMING: 'Hamming',
    cv.NORM_HAMMING2: 'Hamming2'
}


def get_element_types(norm_type):
    if norm_type in (cv.NORM_HAMMING, cv.NORM_HAMMING2):
        return (np.uint8,)
    else:
        return (np.uint8, np.int8, np.uint16, np.int16, np.int32, np.float32,
                np.float64, np.float16)


def generate_vector(shape, dtype):
    if np.issubdtype(dtype, np.integer):
        return np.random.randint(0, 100, shape).astype(dtype)
    else:
        return np.random.normal(10., 12.5, shape).astype(dtype)


shapes = (1, 2, 3, 5, 7, 16, (1, 1), (2, 2), (3, 5), (1, 7))


class norm_test(NewOpenCVTests):

    def test_norm_for_one_array(self):
        np.random.seed(123)
        for norm_type, norm in norm_type_under_test.items():
            element_types = get_element_types(norm_type)
            for shape, element_type in product(shapes, element_types):
                array = generate_vector(shape, element_type)
                expected = norm(array)
                actual = cv.norm(array, norm_type)
                self.assertAlmostEqual(
                    expected, actual, places=2,
                    msg='Array {0} of {1} and norm {2}'.format(
                        array, element_type.__name__, norm_name[norm_type]
                    )
                )

    def test_norm_for_two_arrays(self):
        np.random.seed(456)
        for norm_type, norm in norm_type_under_test.items():
            element_types = get_element_types(norm_type)
            for shape, element_type in product(shapes, element_types):
                first = generate_vector(shape, element_type)
                second = generate_vector(shape, element_type)
                expected = norm(first, second)
                actual = cv.norm(first, second, norm_type)
                self.assertAlmostEqual(
                    expected, actual, places=2,
                    msg='Arrays {0} {1} of type {2} and norm {3}'.format(
                        first, second, element_type.__name__,
                        norm_name[norm_type]
                    )
                )

    def test_norm_fails_for_wrong_type(self):
        for norm_type in (cv.NORM_HAMMING, cv.NORM_HAMMING2):
            with self.assertRaises(Exception,
                                   msg='Type is not checked {0}'.format(
                                       norm_name[norm_type]
                                   )):
                cv.norm(np.array([1, 2], dtype=np.int32), norm_type)

    def test_norm_fails_for_array_and_scalar(self):
        for norm_type in norm_type_under_test:
            with self.assertRaises(Exception,
                                   msg='Exception is not thrown for {0}'.format(
                                       norm_name[norm_type]
                                   )):
                cv.norm(np.array([1, 2], dtype=np.uint8), 123, norm_type)

    def test_norm_fails_for_scalar_and_array(self):
        for norm_type in norm_type_under_test:
            with self.assertRaises(Exception,
                                   msg='Exception is not thrown for {0}'.format(
                                       norm_name[norm_type]
                                   )):
                cv.norm(4, np.array([1, 2], dtype=np.uint8), norm_type)

    def test_norm_fails_for_array_and_norm_type_as_scalar(self):
        for norm_type in norm_type_under_test:
            with self.assertRaises(Exception,
                                   msg='Exception is not thrown for {0}'.format(
                                       norm_name[norm_type]
                                   )):
                cv.norm(np.array([3, 4, 5], dtype=np.uint8),
                        norm_type, normType=norm_type)


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

- **norm_test**: A class/struct defined in this file

### Functions and Methods

- **test_norm_for_two_arrays()**: A function/method defined in this file
- **norm_hamming2()**: A function/method defined in this file
- **test_norm_fails_for_scalar_and_array()**: A function/method defined in this file
- **norm()**: A function/method defined in this file
- **norm_l2sqr()**: A function/method defined in this file
- **element_norm()**: A function/method defined in this file
- **get_element_types()**: A function/method defined in this file
- **norm_inf()**: A function/method defined in this file
- **test_norm_for_one_array()**: A function/method defined in this file
- **norm_l1()**: A function/method defined in this file
- **test_norm_fails_for_wrong_type()**: A function/method defined in this file
- **norm_hamming()**: A function/method defined in this file
- **norm_l2()**: A function/method defined in this file
- **test_norm_fails_for_array_and_scalar()**: A function/method defined in this file
- **generate_vector()**: A function/method defined in this file
- **test_norm_fails_for_array_and_norm_type_as_scalar()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `reduce`
- `tests_common`
- `NewOpenCVTests`
- `functools`
- `numpy`
- `cv2`
- `product`
- `itertools`


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

