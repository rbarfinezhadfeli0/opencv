# Documentation for `modules/gapi/misc/python/test/test_gapi_stateful_kernel.py`

## File Metadata

- **Full Path**: `modules/gapi/misc/python/test/test_gapi_stateful_kernel.py`
- **File Name**: `test_gapi_stateful_kernel.py`
- **File Size**: 6,511 bytes
- **File Type**: .py
- **Link to Source**: [modules/gapi/misc/python/test/test_gapi_stateful_kernel.py](../../../../../modules/gapi/misc/python/test/test_gapi_stateful_kernel.py)

## Purpose and Role

This file is located in the `modules/gapi/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

import numpy as np
import cv2 as cv
import os
import sys
import unittest

from tests_common import NewOpenCVTests


try:

    if sys.version_info[:2] < (3, 0):
        raise unittest.SkipTest('Python 2.x is not supported')


    class CounterState:
        def __init__(self):
            self.counter = 0


    @cv.gapi.op('stateful_counter',
                in_types=[cv.GOpaque.Int],
                out_types=[cv.GOpaque.Int])
    class GStatefulCounter:
        """Accumulates state counter on every call"""

        @staticmethod
        def outMeta(desc):
            return cv.empty_gopaque_desc()


    @cv.gapi.kernel(GStatefulCounter)
    class GStatefulCounterImpl:
        """Implementation for GStatefulCounter operation."""

        @staticmethod
        def setup(desc):
            return CounterState()

        @staticmethod
        def run(value, state):
            state.counter += value
            return state.counter


    class SumState:
        def __init__(self):
            self.sum = 0


    @cv.gapi.op('stateful_sum',
                in_types=[cv.GOpaque.Int, cv.GOpaque.Int],
                out_types=[cv.GOpaque.Int])
    class GStatefulSum:
        """Accumulates sum on every call"""

        @staticmethod
        def outMeta(lhs_desc, rhs_desc):
            return cv.empty_gopaque_desc()


    class gapi_sample_pipelines(NewOpenCVTests):
        def test_stateful_kernel_single_instance(self):
            g_in  = cv.GOpaque.Int()
            g_out = GStatefulCounter.on(g_in)
            comp  = cv.GComputation(cv.GIn(g_in), cv.GOut(g_out))
            pkg   = cv.gapi.kernels(GStatefulCounterImpl)

            nums = [i for i in range(10)]
            acc = 0
            for v in nums:
                acc = comp.apply(cv.gin(v), args=cv.gapi.compile_args(pkg))

            self.assertEqual(sum(nums), acc)


        def test_stateful_kernel_multiple_instances(self):
            # NB: Every counter has his own independent state.
            g_in   = cv.GOpaque.Int()
            g_out0 = GStatefulCounter.on(g_in)
            g_out1 = GStatefulCounter.on(g_in)
            comp   = cv.GComputation(cv.GIn(g_in), cv.GOut(g_out0, g_out1))
            pkg    = cv.gapi.kernels(GStatefulCounterImpl)

            nums = [i for i in range(10)]
            acc0 = acc1 = 0
            for v in nums:
                acc0, acc1 = comp.apply(cv.gin(v), args=cv.gapi.compile_args(pkg))

            ref = sum(nums)
            self.assertEqual(ref, acc0)
            self.assertEqual(ref, acc1)


        def test_stateful_throw_setup(self):
            @cv.gapi.kernel(GStatefulCounter)
            class GThrowStatefulCounterImpl:
                """Implementation for GStatefulCounter operation
                   that throw exception in setup method"""

                @staticmethod
                def setup(desc):
                    raise Exception('Throw from setup method')

                @staticmethod
                def run(value, state):
                    raise Exception('Unreachable')

            g_in  = cv.GOpaque.Int()
            g_out = GStatefulCounter.on(g_in)
            comp  = cv.GComputation(cv.GIn(g_in), cv.GOut(g_out))
            pkg   = cv.gapi.kernels(GThrowStatefulCounterImpl)

            with self.assertRaises(Exception): comp.apply(cv.gin(42),
                                                          args=cv.gapi.compile_args(pkg))


        def test_stateful_reset(self):
            g_in  = cv.GOpaque.Int()
            g_out = GStatefulCounter.on(g_in)
            comp  = cv.GComputation(cv.GIn(g_in), cv.GOut(g_out))
            pkg   = cv.gapi.kernels(GStatefulCounterImpl)

            cc = comp.compileStreaming(args=cv.gapi.compile_args(pkg))

            cc.setSource(cv.gin(1))
            cc.start()
            for i in range(1, 10):
                _, actual = cc.pull()
                self.assertEqual(i, actual)
            cc.stop()

            cc.setSource(cv.gin(2))
            cc.start()
            for i in range(2, 10, 2):
                _, actual = cc.pull()
                self.assertEqual(i, actual)
            cc.stop()


        def test_stateful_multiple_inputs(self):
            @cv.gapi.kernel(GStatefulSum)
            class GStatefulSumImpl:
                """Implementation for GStatefulCounter operation."""

                @staticmethod
                def setup(lhs_desc, rhs_desc):
                    return SumState()

                @staticmethod
                def run(lhs, rhs, state):
                    state.sum+= lhs + rhs
                    return state.sum


            g_in1 = cv.GOpaque.Int()
            g_in2 = cv.GOpaque.Int()
            g_out = GStatefulSum.on(g_in1, g_in2)
            comp = cv.GComputation(cv.GIn(g_in1, g_in2), cv.GOut(g_out))
            pkg  = cv.gapi.kernels(GStatefulSumImpl)

            lhs_list = [1, 10, 15]
            rhs_list = [2, 14, 32]

            ref_out = 0
            for lhs, rhs in zip(lhs_list, rhs_list):
                ref_out += lhs + rhs
                gapi_out = comp.apply(cv.gin(lhs, rhs), cv.gapi.compile_args(pkg))
                self.assertEqual(ref_out, gapi_out)


        def test_stateful_multiple_inputs_throw(self):
            @cv.gapi.kernel(GStatefulSum)
            class GStatefulSumImplIncorrect:
                """Incorrect implementation for GStatefulCounter operation."""

                # NB: setup methods is intentionally
                # incorrect - accepts one meta arg instead of two
                @staticmethod
                def setup(desc):
                    return SumState()

                @staticmethod
                def run(lhs, rhs, state):
                    state.sum+= lhs + rhs
                    return state.sum


            g_in1 = cv.GOpaque.Int()
            g_in2 = cv.GOpaque.Int()
            g_out = GStatefulSum.on(g_in1, g_in2)
            comp = cv.GComputation(cv.GIn(g_in1, g_in2), cv.GOut(g_out))
            pkg  = cv.gapi.kernels(GStatefulSumImplIncorrect)

            with self.assertRaises(Exception): comp.apply(cv.gin(42, 42),
                                                          args=cv.gapi.compile_args(pkg))


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

- **GStatefulCounter**: A class/struct defined in this file
- **GThrowStatefulCounterImpl**: A class/struct defined in this file
- **SumState**: A class/struct defined in this file
- **GStatefulSumImpl**: A class/struct defined in this file
- **GStatefulCounterImpl**: A class/struct defined in this file
- **GStatefulSumImplIncorrect**: A class/struct defined in this file
- **GStatefulSum**: A class/struct defined in this file
- **TestSkip**: A class/struct defined in this file
- **CounterState**: A class/struct defined in this file
- **gapi_sample_pipelines**: A class/struct defined in this file

### Functions and Methods

- **test_skip()**: A function/method defined in this file
- **test_stateful_kernel_multiple_instances()**: A function/method defined in this file
- **test_stateful_throw_setup()**: A function/method defined in this file
- **test_stateful_multiple_inputs_throw()**: A function/method defined in this file
- **test_stateful_reset()**: A function/method defined in this file
- **outMeta()**: A function/method defined in this file
- **run()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **setup()**: A function/method defined in this file
- **test_stateful_multiple_inputs()**: A function/method defined in this file
- **test_stateful_kernel_single_instance()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `os`
- `NewOpenCVTests`
- `numpy`
- `cv2`
- `setup`
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

