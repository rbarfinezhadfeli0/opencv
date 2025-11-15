# Documentation for `modules/python/test/test_cuda.py`

## File Metadata

- **Full Path**: `modules/python/test/test_cuda.py`
- **File Name**: `test_cuda.py`
- **File Size**: 6,805 bytes
- **File Type**: .py
- **Link to Source**: [modules/python/test/test_cuda.py](../../../modules/python/test/test_cuda.py)

## Purpose and Role

This file is located in the `modules/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
CUDA-accelerated Computer Vision functions
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv
import os

from tests_common import NewOpenCVTests, unittest

class cuda_test(NewOpenCVTests):
    def setUp(self):
        super(cuda_test, self).setUp()
        if not cv.cuda.getCudaEnabledDeviceCount():
            self.skipTest("No CUDA-capable device is detected")

    def test_cuda_upload_download(self):
        npMat = (np.random.random((128, 128, 3)) * 255).astype(np.uint8)
        cuMat = cv.cuda_GpuMat()
        cuMat.upload(npMat)

        self.assertTrue(np.allclose(cuMat.download(), npMat))

    def test_cuda_upload_download_stream(self):
        stream = cv.cuda_Stream()
        npMat = (np.random.random((128, 128, 3)) * 255).astype(np.uint8)
        cuMat = cv.cuda_GpuMat(128,128, cv.CV_8UC3)
        cuMat.upload(npMat, stream)
        npMat2 = cuMat.download(stream=stream)
        stream.waitForCompletion()
        self.assertTrue(np.allclose(npMat2, npMat))

    def test_cuda_interop(self):
        npMat = (np.random.random((128, 128, 3)) * 255).astype(np.uint8)
        cuMat = cv.cuda_GpuMat()
        cuMat.upload(npMat)
        self.assertTrue(cuMat.cudaPtr() != 0)
        cuMatFromPtrSz = cv.cuda.createGpuMatFromCudaMemory(cuMat.size(),cuMat.type(),cuMat.cudaPtr(), cuMat.step)
        self.assertTrue(cuMat.cudaPtr() == cuMatFromPtrSz.cudaPtr())
        cuMatFromPtrRc = cv.cuda.createGpuMatFromCudaMemory(cuMat.size()[1],cuMat.size()[0],cuMat.type(),cuMat.cudaPtr(), cuMat.step)
        self.assertTrue(cuMat.cudaPtr() == cuMatFromPtrRc.cudaPtr())
        stream = cv.cuda_Stream()
        self.assertTrue(stream.cudaPtr() != 0)
        streamFromPtr = cv.cuda.wrapStream(stream.cudaPtr())
        self.assertTrue(stream.cudaPtr() == streamFromPtr.cudaPtr())
        asyncstream = cv.cuda_Stream(1)  # cudaStreamNonBlocking
        self.assertTrue(asyncstream.cudaPtr() != 0)

    def test_cuda_buffer_pool(self):
        cv.cuda.setBufferPoolUsage(True)
        cv.cuda.setBufferPoolConfig(cv.cuda.getDevice(), 1024 * 1024 * 64, 2)
        stream_a = cv.cuda.Stream()
        pool_a = cv.cuda.BufferPool(stream_a)
        cuMat = pool_a.getBuffer(1024, 1024, cv.CV_8UC3)
        cv.cuda.setBufferPoolUsage(False)
        self.assertEqual(cuMat.size(), (1024, 1024))
        self.assertEqual(cuMat.type(), cv.CV_8UC3)

    def test_cuda_release(self):
        npMat = (np.random.random((128, 128, 3)) * 255).astype(np.uint8)
        cuMat = cv.cuda_GpuMat()
        cuMat.upload(npMat)
        cuMat.release()
        self.assertTrue(cuMat.cudaPtr() == 0)
        self.assertTrue(cuMat.step == 0)
        self.assertTrue(cuMat.size() == (0, 0))

    @unittest.skip("failed test")
    def test_cuda_convertTo(self):
        # setup
        npMat_8UC4 = (np.random.random((128, 128, 4)) * 255).astype(np.uint8)
        npMat_32FC4 = npMat_8UC4.astype(np.single)
        new_type = cv.CV_32FC4

        # sync
        # in/out
        cuMat_8UC4 = cv.cuda_GpuMat(npMat_8UC4)
        cuMat_32FC4 = cv.cuda_GpuMat(cuMat_8UC4.size(), new_type)
        cuMat_32FC4_out = cuMat_8UC4.convertTo(new_type, cuMat_32FC4)
        self.assertTrue(cuMat_32FC4.cudaPtr() == cuMat_32FC4_out.cudaPtr())
        npMat_32FC4_out = cuMat_32FC4.download()
        self.assertTrue(np.array_equal(npMat_32FC4, npMat_32FC4_out))
        # out
        cuMat_32FC4_out = cuMat_8UC4.convertTo(new_type)
        npMat_32FC4_out = cuMat_32FC4.download()
        self.assertTrue(np.array_equal(npMat_32FC4, npMat_32FC4_out))

        # async
        stream = cv.cuda.Stream()
        cuMat_32FC4 = cv.cuda_GpuMat(cuMat_8UC4.size(), new_type)
        cuMat_32FC4_out = cuMat_8UC4.convertTo(new_type, cuMat_32FC4)
        # in/out
        cuMat_32FC4_out = cuMat_8UC4.convertTo(new_type, 1, 0, stream, cuMat_32FC4)
        self.assertTrue(cuMat_32FC4.cudaPtr() == cuMat_32FC4_out.cudaPtr())
        npMat_32FC4_out = cuMat_32FC4.download(stream)
        stream.waitForCompletion()
        self.assertTrue(np.array_equal(npMat_32FC4, npMat_32FC4_out))
        # out
        cuMat_32FC4_out = cuMat_8UC4.convertTo(new_type, 1, 0, stream)
        npMat_32FC4_out = cuMat_32FC4.download(stream)
        stream.waitForCompletion()
        self.assertTrue(np.array_equal(npMat_32FC4, npMat_32FC4_out))

    @unittest.skip("failed test")
    def test_cuda_copyTo(self):
        # setup
        npMat_8UC4 = (np.random.random((128, 128, 4)) * 255).astype(np.uint8)

        # sync
        # in/out
        cuMat_8UC4 = cv.cuda_GpuMat(npMat_8UC4)
        cuMat_8UC4_dst = cv.cuda_GpuMat(cuMat_8UC4.size(), cuMat_8UC4.type())
        cuMat_8UC4_out = cuMat_8UC4.copyTo(cuMat_8UC4_dst)
        self.assertTrue(cuMat_8UC4_out.cudaPtr() == cuMat_8UC4_dst.cudaPtr())
        npMat_8UC4_out = cuMat_8UC4_out.download()
        self.assertTrue(np.array_equal(npMat_8UC4, npMat_8UC4_out))
        # out
        cuMat_8UC4_out =  cuMat_8UC4.copyTo()
        npMat_8UC4_out = cuMat_8UC4_out.download()
        self.assertTrue(np.array_equal(npMat_8UC4, npMat_8UC4_out))

        # async
        stream = cv.cuda.Stream()
        # in/out
        cuMat_8UC4 = cv.cuda_GpuMat(npMat_8UC4)
        cuMat_8UC4_dst = cv.cuda_GpuMat(cuMat_8UC4.size(), cuMat_8UC4.type())
        cuMat_8UC4_out = cuMat_8UC4.copyTo(cuMat_8UC4_dst, stream)
        self.assertTrue(cuMat_8UC4_out.cudaPtr() == cuMat_8UC4_out.cudaPtr())
        npMat_8UC4_out = cuMat_8UC4_dst.download(stream)
        stream.waitForCompletion()
        self.assertTrue(np.array_equal(npMat_8UC4, npMat_8UC4_out))
        # out
        cuMat_8UC4_out = cuMat_8UC4.copyTo(stream)
        npMat_8UC4_out = cuMat_8UC4_out.download(stream)
        stream.waitForCompletion()
        self.assertTrue(np.array_equal(npMat_8UC4, npMat_8UC4_out))

    def test_cuda_denoising(self):
        self.assertEqual(True, hasattr(cv.cuda, 'fastNlMeansDenoising'))
        self.assertEqual(True, hasattr(cv.cuda, 'fastNlMeansDenoisingColored'))
        self.assertEqual(True, hasattr(cv.cuda, 'nonLocalMeans'))

    def test_dlpack_GpuMat(self):
        for dtype in [np.int8, np.uint8, np.int16, np.uint16, np.float16, np.int32, np.float32, np.float64]:
            for channels in [2, 3, 5]:
                ref = (np.random.random((64, 128, channels)) * 255).astype(dtype)
                src = cv.cuda_GpuMat()
                src.upload(ref)
                dst = cv.cuda_GpuMat.from_dlpack(src)
                test = dst.download()
                equal = np.array_equal(ref, test)
                if not equal:
                    print(f"Failed test with dtype {dtype} and {channels} channels")
                self.assertTrue(equal)

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

- **cuda_test**: A class/struct defined in this file

### Functions and Methods

- **test_cuda_interop()**: A function/method defined in this file
- **test_cuda_buffer_pool()**: A function/method defined in this file
- **test_cuda_copyTo()**: A function/method defined in this file
- **test_cuda_denoising()**: A function/method defined in this file
- **test_dlpack_GpuMat()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **test_cuda_upload_download_stream()**: A function/method defined in this file
- **test_cuda_convertTo()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **test_cuda_upload_download()**: A function/method defined in this file
- **test_cuda_release()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `os`
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

