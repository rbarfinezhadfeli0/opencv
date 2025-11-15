# Documentation for `modules/ml/misc/python/test/test_digits.py`

## File Metadata

- **Full Path**: `modules/ml/misc/python/test/test_digits.py`
- **File Name**: `test_digits.py`
- **File Size**: 6,582 bytes
- **File Type**: .py
- **Link to Source**: [modules/ml/misc/python/test/test_digits.py](../../../../../modules/ml/misc/python/test/test_digits.py)

## Purpose and Role

This file is located in the `modules/ml/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
SVM and KNearest digit recognition.

Sample loads a dataset of handwritten digits from '../data/digits.png'.
Then it trains a SVM and KNearest classifiers on it and evaluates
their accuracy.

Following preprocessing is applied to the dataset:
 - Moment-based image deskew (see deskew())
 - Digit images are split into 4 10x10 cells and 16-bin
   histogram of oriented gradients is computed for each
   cell
 - Transform histograms to space with Hellinger metric (see [1] (RootSIFT))


[1] R. Arandjelovic, A. Zisserman
    "Three things everyone should know to improve object retrieval"
    http://www.robots.ox.ac.uk/~vgg/publications/2012/Arandjelovic12/arandjelovic12.pdf

'''


# Python 2/3 compatibility
from __future__ import print_function

# built-in modules
from multiprocessing.pool import ThreadPool

import cv2 as cv

import numpy as np
from numpy.linalg import norm


SZ = 20 # size of each digit is SZ x SZ
CLASS_N = 10
DIGITS_FN = 'samples/data/digits.png'

def split2d(img, cell_size, flatten=True):
    h, w = img.shape[:2]
    sx, sy = cell_size
    cells = [np.hsplit(row, w//sx) for row in np.vsplit(img, h//sy)]
    cells = np.array(cells)
    if flatten:
        cells = cells.reshape(-1, sy, sx)
    return cells

def deskew(img):
    m = cv.moments(img)
    if abs(m['mu02']) < 1e-2:
        return img.copy()
    skew = m['mu11']/m['mu02']
    M = np.float32([[1, skew, -0.5*SZ*skew], [0, 1, 0]])
    img = cv.warpAffine(img, M, (SZ, SZ), flags=cv.WARP_INVERSE_MAP | cv.INTER_LINEAR)
    return img

class StatModel(object):
    def load(self, fn):
        self.model.load(fn)  # Known bug: https://github.com/opencv/opencv/issues/4969
    def save(self, fn):
        self.model.save(fn)

class KNearest(StatModel):
    def __init__(self, k = 3):
        self.k = k
        self.model = cv.ml.KNearest_create()

    def train(self, samples, responses):
        self.model.train(samples, cv.ml.ROW_SAMPLE, responses)

    def predict(self, samples):
        _retval, results, _neigh_resp, _dists = self.model.findNearest(samples, self.k)
        return results.ravel()

class SVM(StatModel):
    def __init__(self, C = 1, gamma = 0.5):
        self.model = cv.ml.SVM_create()
        self.model.setGamma(gamma)
        self.model.setC(C)
        self.model.setKernel(cv.ml.SVM_RBF)
        self.model.setType(cv.ml.SVM_C_SVC)

    def train(self, samples, responses):
        self.model.train(samples, cv.ml.ROW_SAMPLE, responses)

    def predict(self, samples):
        return self.model.predict(samples)[1].ravel()


def evaluate_model(model, digits, samples, labels):
    resp = model.predict(samples)
    err = (labels != resp).mean()

    confusion = np.zeros((10, 10), np.int32)
    for i, j in zip(labels, resp):
        confusion[int(i), int(j)] += 1

    return err, confusion

def preprocess_simple(digits):
    return np.float32(digits).reshape(-1, SZ*SZ) / 255.0

def preprocess_hog(digits):
    samples = []
    for img in digits:
        gx = cv.Sobel(img, cv.CV_32F, 1, 0)
        gy = cv.Sobel(img, cv.CV_32F, 0, 1)
        mag, ang = cv.cartToPolar(gx, gy)
        bin_n = 16
        bin = np.int32(bin_n*ang/(2*np.pi))
        bin_cells = bin[:10,:10], bin[10:,:10], bin[:10,10:], bin[10:,10:]
        mag_cells = mag[:10,:10], mag[10:,:10], mag[:10,10:], mag[10:,10:]
        hists = [np.bincount(b.ravel(), m.ravel(), bin_n) for b, m in zip(bin_cells, mag_cells)]
        hist = np.hstack(hists)

        # transform to Hellinger kernel
        eps = 1e-7
        hist /= hist.sum() + eps
        hist = np.sqrt(hist)
        hist /= norm(hist) + eps

        samples.append(hist)
    return np.float32(samples)

from tests_common import NewOpenCVTests

class digits_test(NewOpenCVTests):

    def load_digits(self, fn):
        digits_img = self.get_sample(fn, 0)
        digits = split2d(digits_img, (SZ, SZ))
        labels = np.repeat(np.arange(CLASS_N), len(digits)/CLASS_N)
        return digits, labels

    def test_digits(self):

        digits, labels = self.load_digits(DIGITS_FN)

        # shuffle digits
        rand = np.random.RandomState(321)
        shuffle = rand.permutation(len(digits))
        digits, labels = digits[shuffle], labels[shuffle]

        digits2 = list(map(deskew, digits))
        samples = preprocess_hog(digits2)

        train_n = int(0.9*len(samples))
        _digits_train, digits_test = np.split(digits2, [train_n])
        samples_train, samples_test = np.split(samples, [train_n])
        labels_train, labels_test = np.split(labels, [train_n])
        errors = list()
        confusionMatrixes = list()

        model = KNearest(k=4)
        model.train(samples_train, labels_train)
        error, confusion = evaluate_model(model, digits_test, samples_test, labels_test)
        errors.append(error)
        confusionMatrixes.append(confusion)

        model = SVM(C=2.67, gamma=5.383)
        model.train(samples_train, labels_train)
        error, confusion = evaluate_model(model, digits_test, samples_test, labels_test)
        errors.append(error)
        confusionMatrixes.append(confusion)

        eps = 0.001
        normEps = len(samples_test) * 0.02

        confusionKNN = [[45,  0,  0,  0,  0,  0,  0,  0,  0,  0],
         [ 0, 57,  0,  0,  0,  0,  0,  0,  0,  0],
         [ 0,  0, 59,  1,  0,  0,  0,  0,  1,  0],
         [ 0,  0,  0, 43,  0,  0,  0,  1,  0,  0],
         [ 0,  0,  0,  0, 38,  0,  2,  0,  0,  0],
         [ 0,  0,  0,  2,  0, 48,  0,  0,  1,  0],
         [ 0,  1,  0,  0,  0,  0, 51,  0,  0,  0],
         [ 0,  0,  1,  0,  0,  0,  0, 54,  0,  0],
         [ 0,  0,  0,  0,  0,  1,  0,  0, 46,  0],
         [ 1,  1,  0,  1,  1,  0,  0,  0,  2, 42]]

        confusionSVM = [[45,  0,  0,  0,  0,  0,  0,  0,  0,  0],
          [ 0, 57,  0,  0,  0,  0,  0,  0,  0,  0],
          [ 0,  0, 59,  2,  0,  0,  0,  0,  0,  0],
          [ 0,  0,  0, 43,  0,  0,  0,  1,  0,  0],
          [ 0,  0,  0,  0, 40,  0,  0,  0,  0,  0],
          [ 0,  0,  0,  1,  0, 50,  0,  0,  0,  0],
          [ 0,  0,  0,  0,  1,  0,  51, 0,  0,  0],
          [ 0,  0,  1,  0,  0,  0,  0,  54, 0,  0],
          [ 0,  0,  0,  0,  0,  0,  0,  0, 47,  0],
          [ 0,  1,  0,  1,  0,  0,  0,  0,  1, 45]]

        self.assertLess(cv.norm(confusionMatrixes[0] - confusionKNN, cv.NORM_L1), normEps)
        self.assertLess(cv.norm(confusionMatrixes[1] - confusionSVM, cv.NORM_L1), normEps)

        self.assertLess(errors[0] - 0.034, eps)
        self.assertLess(errors[1] - 0.018, eps)


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

- **KNearest**: A class/struct defined in this file
- **StatModel**: A class/struct defined in this file
- **digits_test**: A class/struct defined in this file
- **SVM**: A class/struct defined in this file

### Functions and Methods

- **predict()**: A function/method defined in this file
- **train()**: A function/method defined in this file
- **deskew()**: A function/method defined in this file
- **test_digits()**: A function/method defined in this file
- **load_digits()**: A function/method defined in this file
- **load()**: A function/method defined in this file
- **evaluate_model()**: A function/method defined in this file
- **preprocess_hog()**: A function/method defined in this file
- **save()**: A function/method defined in this file
- **preprocess_simple()**: A function/method defined in this file
- **split2d()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `norm`
- `NewOpenCVTests`
- `ThreadPool`
- `print_function`
- `numpy.linalg`
- `multiprocessing.pool`
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

