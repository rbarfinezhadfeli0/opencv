# Documentation for `samples/python/letter_recog.py`

## File Metadata

- **Full Path**: `samples/python/letter_recog.py`
- **File Name**: `letter_recog.py`
- **File Size**: 6,325 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/letter_recog.py](../../samples/python/letter_recog.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
The sample demonstrates how to train Random Trees classifier
(or Boosting classifier, or MLP, or Knearest, or Support Vector Machines) using the provided dataset.

We use the sample database letter-recognition.data
from UCI Repository, here is the link:

Newman, D.J. & Hettich, S. & Blake, C.L. & Merz, C.J. (1998).
UCI Repository of machine learning databases
[http://www.ics.uci.edu/~mlearn/MLRepository.html].
Irvine, CA: University of California, Department of Information and Computer Science.

The dataset consists of 20000 feature vectors along with the
responses - capital latin letters A..Z.
The first 10000 samples are used for training
and the remaining 10000 - to test the classifier.
======================================================
USAGE:
  letter_recog.py [--model <model>]
                  [--data <data fn>]
                  [--load <model fn>] [--save <model fn>]

  Models: RTrees, KNearest, Boost, SVM, MLP
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

def load_base(fn):
    a = np.loadtxt(fn, np.float32, delimiter=',', converters={ 0 : lambda ch : ord(ch)-ord('A') })
    samples, responses = a[:,1:], a[:,0]
    return samples, responses

class LetterStatModel(object):
    class_n = 26
    train_ratio = 0.5

    def load(self, fn):
        self.model = self.model.load(fn)
    def save(self, fn):
        self.model.save(fn)

    def unroll_samples(self, samples):
        sample_n, var_n = samples.shape
        new_samples = np.zeros((sample_n * self.class_n, var_n+1), np.float32)
        new_samples[:,:-1] = np.repeat(samples, self.class_n, axis=0)
        new_samples[:,-1] = np.tile(np.arange(self.class_n), sample_n)
        return new_samples

    def unroll_responses(self, responses):
        sample_n = len(responses)
        new_responses = np.zeros(sample_n*self.class_n, np.int32)
        resp_idx = np.int32( responses + np.arange(sample_n)*self.class_n )
        new_responses[resp_idx] = 1
        return new_responses

class RTrees(LetterStatModel):
    def __init__(self):
        self.model = cv.ml.RTrees_create()

    def train(self, samples, responses):
        self.model.setMaxDepth(20)
        self.model.train(samples, cv.ml.ROW_SAMPLE, responses.astype(int))

    def predict(self, samples):
        _ret, resp = self.model.predict(samples)
        return resp.ravel()


class KNearest(LetterStatModel):
    def __init__(self):
        self.model = cv.ml.KNearest_create()

    def train(self, samples, responses):
        self.model.train(samples, cv.ml.ROW_SAMPLE, responses)

    def predict(self, samples):
        _retval, results, _neigh_resp, _dists = self.model.findNearest(samples, k = 10)
        return results.ravel()


class Boost(LetterStatModel):
    def __init__(self):
        self.model = cv.ml.Boost_create()

    def train(self, samples, responses):
        _sample_n, var_n = samples.shape
        new_samples = self.unroll_samples(samples)
        new_responses = self.unroll_responses(responses)
        var_types = np.array([cv.ml.VAR_NUMERICAL] * var_n + [cv.ml.VAR_CATEGORICAL, cv.ml.VAR_CATEGORICAL], np.uint8)

        self.model.setWeakCount(15)
        self.model.setMaxDepth(10)
        self.model.train(cv.ml.TrainData_create(new_samples, cv.ml.ROW_SAMPLE, new_responses.astype(int), varType = var_types))

    def predict(self, samples):
        new_samples = self.unroll_samples(samples)
        _ret, resp = self.model.predict(new_samples)

        return resp.ravel().reshape(-1, self.class_n).argmax(1)


class SVM(LetterStatModel):
    def __init__(self):
        self.model = cv.ml.SVM_create()

    def train(self, samples, responses):
        self.model.setType(cv.ml.SVM_C_SVC)
        self.model.setC(1)
        self.model.setKernel(cv.ml.SVM_RBF)
        self.model.setGamma(.1)
        self.model.train(samples, cv.ml.ROW_SAMPLE, responses.astype(int))

    def predict(self, samples):
        _ret, resp = self.model.predict(samples)
        return resp.ravel()


class MLP(LetterStatModel):
    def __init__(self):
        self.model = cv.ml.ANN_MLP_create()

    def train(self, samples, responses):
        _sample_n, var_n = samples.shape
        new_responses = self.unroll_responses(responses).reshape(-1, self.class_n)
        layer_sizes = np.int32([var_n, 100, 100, self.class_n])

        self.model.setLayerSizes(layer_sizes)
        self.model.setTrainMethod(cv.ml.ANN_MLP_BACKPROP)
        self.model.setBackpropMomentumScale(0.0)
        self.model.setBackpropWeightScale(0.001)
        self.model.setTermCriteria((cv.TERM_CRITERIA_COUNT, 20, 0.01))
        self.model.setActivationFunction(cv.ml.ANN_MLP_SIGMOID_SYM, 2, 1)

        self.model.train(samples, cv.ml.ROW_SAMPLE, np.float32(new_responses))

    def predict(self, samples):
        _ret, resp = self.model.predict(samples)
        return resp.argmax(-1)



def main():
    import getopt
    import sys

    models = [RTrees, KNearest, Boost, SVM, MLP] # NBayes
    models = dict( [(cls.__name__.lower(), cls) for cls in models] )


    args, dummy = getopt.getopt(sys.argv[1:], '', ['model=', 'data=', 'load=', 'save='])
    args = dict(args)
    args.setdefault('--model', 'svm')
    args.setdefault('--data', 'letter-recognition.data')

    datafile = cv.samples.findFile(args['--data'])

    print('loading data %s ...' % datafile)
    samples, responses = load_base(datafile)
    Model = models[args['--model']]
    model = Model()

    train_n = int(len(samples)*model.train_ratio)
    if '--load' in args:
        fn = args['--load']
        print('loading model from %s ...' % fn)
        model.load(fn)
    else:
        print('training %s ...' % Model.__name__)
        model.train(samples[:train_n], responses[:train_n])

    print('testing...')
    train_rate = np.mean(model.predict(samples[:train_n]) == responses[:train_n].astype(int))
    test_rate  = np.mean(model.predict(samples[train_n:]) == responses[train_n:].astype(int))

    print('train rate: %f  test rate: %f' % (train_rate*100, test_rate*100))

    if '--save' in args:
        fn = args['--save']
        print('saving model to %s ...' % fn)
        model.save(fn)

    print('Done')


if __name__ == '__main__':
    print(__doc__)
    main()
    cv.destroyAllWindows()
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

- **SVM**: A class/struct defined in this file
- **MLP**: A class/struct defined in this file
- **RTrees**: A class/struct defined in this file
- **LetterStatModel**: A class/struct defined in this file
- **KNearest**: A class/struct defined in this file
- **Boost**: A class/struct defined in this file

### Functions and Methods

- **predict()**: A function/method defined in this file
- **load_base()**: A function/method defined in this file
- **train()**: A function/method defined in this file
- **load()**: A function/method defined in this file
- **unroll_responses()**: A function/method defined in this file
- **unroll_samples()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **save()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
- `getopt`
- `print_function`
- `UCI`
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

