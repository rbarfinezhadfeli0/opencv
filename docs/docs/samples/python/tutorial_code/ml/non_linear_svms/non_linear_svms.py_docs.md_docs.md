# Documentation for `docs/samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py_docs.md`
- **File Name**: `non_linear_svms.py_docs.md`
- **File Size**: 7,347 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py_docs.md](../../../../../../docs/samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python/tutorial_code/ml/non_linear_svms` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py`

## File Metadata

- **Full Path**: `samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py`
- **File Name**: `non_linear_svms.py`
- **File Size**: 4,000 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py](../../../../../samples/python/tutorial_code/ml/non_linear_svms/non_linear_svms.py)

## Purpose and Role

This file is located in the `samples/python/tutorial_code/ml/non_linear_svms` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
from __future__ import print_function
import cv2 as cv
import numpy as np
import random as rng

NTRAINING_SAMPLES = 100 # Number of training samples per class
FRAC_LINEAR_SEP = 0.9   # Fraction of samples which compose the linear separable part

# Data for visual representation
WIDTH = 512
HEIGHT = 512
I = np.zeros((HEIGHT, WIDTH, 3), dtype=np.uint8)

# --------------------- 1. Set up training data randomly ---------------------------------------
trainData = np.empty((2*NTRAINING_SAMPLES, 2), dtype=np.float32)
labels = np.empty((2*NTRAINING_SAMPLES, 1), dtype=np.int32)

rng.seed(100) # Random value generation class

# Set up the linearly separable part of the training data
nLinearSamples = int(FRAC_LINEAR_SEP * NTRAINING_SAMPLES)

## [setup1]
# Generate random points for the class 1
trainClass = trainData[0:nLinearSamples,:]
# The x coordinate of the points is in [0, 0.4)
c = trainClass[:,0:1]
c[:] = np.random.uniform(0.0, 0.4 * WIDTH, c.shape)
# The y coordinate of the points is in [0, 1)
c = trainClass[:,1:2]
c[:] = np.random.uniform(0.0, HEIGHT, c.shape)

# Generate random points for the class 2
trainClass = trainData[2*NTRAINING_SAMPLES-nLinearSamples:2*NTRAINING_SAMPLES,:]
# The x coordinate of the points is in [0.6, 1]
c = trainClass[:,0:1]
c[:] = np.random.uniform(0.6*WIDTH, WIDTH, c.shape)
# The y coordinate of the points is in [0, 1)
c = trainClass[:,1:2]
c[:] = np.random.uniform(0.0, HEIGHT, c.shape)
## [setup1]

#------------------ Set up the non-linearly separable part of the training data ---------------
## [setup2]
# Generate random points for the classes 1 and 2
trainClass = trainData[nLinearSamples:2*NTRAINING_SAMPLES-nLinearSamples,:]
# The x coordinate of the points is in [0.4, 0.6)
c = trainClass[:,0:1]
c[:] = np.random.uniform(0.4*WIDTH, 0.6*WIDTH, c.shape)
# The y coordinate of the points is in [0, 1)
c = trainClass[:,1:2]
c[:] = np.random.uniform(0.0, HEIGHT, c.shape)
## [setup2]

#------------------------- Set up the labels for the classes ---------------------------------
labels[0:NTRAINING_SAMPLES,:] = 1                   # Class 1
labels[NTRAINING_SAMPLES:2*NTRAINING_SAMPLES,:] = 2 # Class 2

#------------------------ 2. Set up the support vector machines parameters --------------------
print('Starting training process')
## [init]
svm = cv.ml.SVM_create()
svm.setType(cv.ml.SVM_C_SVC)
svm.setC(0.1)
svm.setKernel(cv.ml.SVM_LINEAR)
svm.setTermCriteria((cv.TERM_CRITERIA_MAX_ITER, int(1e7), 1e-6))
## [init]

#------------------------ 3. Train the svm ----------------------------------------------------
## [train]
svm.train(trainData, cv.ml.ROW_SAMPLE, labels)
## [train]
print('Finished training process')

#------------------------ 4. Show the decision regions ----------------------------------------
## [show]
green = (0,100,0)
blue = (100,0,0)
for i in range(I.shape[0]):
    for j in range(I.shape[1]):
        sampleMat = np.matrix([[j,i]], dtype=np.float32)
        response = svm.predict(sampleMat)[1]

        if response == 1:
            I[i,j] = green
        elif response == 2:
            I[i,j] = blue
## [show]

#----------------------- 5. Show the training data --------------------------------------------
## [show_data]
thick = -1
# Class 1
for i in range(NTRAINING_SAMPLES):
    px = trainData[i,0]
    py = trainData[i,1]
    cv.circle(I, (int(px), int(py)), 3, (0, 255, 0), thick)

# Class 2
for i in range(NTRAINING_SAMPLES, 2*NTRAINING_SAMPLES):
    px = trainData[i,0]
    py = trainData[i,1]
    cv.circle(I, (int(px), int(py)), 3, (255, 0, 0), thick)
## [show_data]

#------------------------- 6. Show support vectors --------------------------------------------
## [show_vectors]
thick = 2
sv = svm.getUncompressedSupportVectors()

for i in range(sv.shape[0]):
    cv.circle(I, (int(sv[i,0]), int(sv[i,1])), 6, (128, 128, 128), thick)
## [show_vectors]

cv.imwrite('result.png', I)                      # save the Image
cv.imshow('SVM for Non-Linear Training Data', I) # show it to the user
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

### Classes and Structures

- **2**: A class/struct defined in this file
- **1**: A class/struct defined in this file
- **FRAC_LINEAR_SEP**: A class/struct defined in this file

### Functions and Methods

- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `random`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

