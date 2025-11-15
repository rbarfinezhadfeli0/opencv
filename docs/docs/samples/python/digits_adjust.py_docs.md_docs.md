# Documentation for `docs/samples/python/digits_adjust.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/digits_adjust.py_docs.md`
- **File Name**: `digits_adjust.py_docs.md`
- **File Size**: 7,965 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/digits_adjust.py_docs.md](../../../docs/samples/python/digits_adjust.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/digits_adjust.py`

## File Metadata

- **Full Path**: `samples/python/digits_adjust.py`
- **File Name**: `digits_adjust.py`
- **File Size**: 4,376 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/digits_adjust.py](../../samples/python/digits_adjust.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Digit recognition adjustment.
Grid search is used to find the best parameters for SVM and KNearest classifiers.
SVM adjustment follows the guidelines given in
http://www.csie.ntu.edu.tw/~cjlin/papers/guide/guide.pdf

Usage:
  digits_adjust.py [--model {svm|knearest}]

  --model {svm|knearest}   - select the classifier (SVM is the default)

'''

# Python 2/3 compatibility
from __future__ import print_function
import sys
PY3 = sys.version_info[0] == 3

if PY3:
    xrange = range

import numpy as np
import cv2 as cv

from multiprocessing.pool import ThreadPool

from digits import *

def cross_validate(model_class, params, samples, labels, kfold = 3, pool = None):
    n = len(samples)
    folds = np.array_split(np.arange(n), kfold)
    def f(i):
        model = model_class(**params)
        test_idx = folds[i]
        train_idx = list(folds)
        train_idx.pop(i)
        train_idx = np.hstack(train_idx)
        train_samples, train_labels = samples[train_idx], labels[train_idx]
        test_samples, test_labels = samples[test_idx], labels[test_idx]
        model.train(train_samples, train_labels)
        resp = model.predict(test_samples)
        score = (resp != test_labels).mean()
        print(".", end='')
        return score
    if pool is None:
        scores = list(map(f, xrange(kfold)))
    else:
        scores = pool.map(f, xrange(kfold))
    return np.mean(scores)


class App(object):
    def __init__(self):
        self._samples, self._labels = self.preprocess()

    def preprocess(self):
        digits, labels = load_digits(DIGITS_FN)
        shuffle = np.random.permutation(len(digits))
        digits, labels = digits[shuffle], labels[shuffle]
        digits2 = list(map(deskew, digits))
        samples = preprocess_hog(digits2)
        return samples, labels

    def get_dataset(self):
        return self._samples, self._labels

    def run_jobs(self, f, jobs):
        pool = ThreadPool(processes=cv.getNumberOfCPUs())
        ires = pool.imap_unordered(f, jobs)
        return ires

    def adjust_SVM(self):
        Cs = np.logspace(0, 10, 15, base=2)
        gammas = np.logspace(-7, 4, 15, base=2)
        scores = np.zeros((len(Cs), len(gammas)))
        scores[:] = np.nan

        print('adjusting SVM (may take a long time) ...')
        def f(job):
            i, j = job
            samples, labels = self.get_dataset()
            params = dict(C = Cs[i], gamma=gammas[j])
            score = cross_validate(SVM, params, samples, labels)
            return i, j, score

        ires = self.run_jobs(f, np.ndindex(*scores.shape))
        for count, (i, j, score) in enumerate(ires):
            scores[i, j] = score
            print('%d / %d (best error: %.2f %%, last: %.2f %%)' %
                  (count+1, scores.size, np.nanmin(scores)*100, score*100))
        print(scores)

        print('writing score table to "svm_scores.npz"')
        np.savez('svm_scores.npz', scores=scores, Cs=Cs, gammas=gammas)

        i, j = np.unravel_index(scores.argmin(), scores.shape)
        best_params = dict(C = Cs[i], gamma=gammas[j])
        print('best params:', best_params)
        print('best error: %.2f %%' % (scores.min()*100))
        return best_params

    def adjust_KNearest(self):
        print('adjusting KNearest ...')
        def f(k):
            samples, labels = self.get_dataset()
            err = cross_validate(KNearest, dict(k=k), samples, labels)
            return k, err
        best_err, best_k = np.inf, -1
        for k, err in self.run_jobs(f, xrange(1, 9)):
            if err < best_err:
                best_err, best_k = err, k
            print('k = %d, error: %.2f %%' % (k, err*100))
        best_params = dict(k=best_k)
        print('best params:', best_params, 'err: %.2f' % (best_err*100))
        return best_params


if __name__ == '__main__':
    import getopt
    import sys

    print(__doc__)

    args, _ = getopt.getopt(sys.argv[1:], '', ['model='])
    args = dict(args)
    args.setdefault('--model', 'svm')
    args.setdefault('--env', '')
    if args['--model'] not in ['svm', 'knearest']:
        print('unknown model "%s"' % args['--model'])
        sys.exit(1)

    t = clock()
    app = App()
    if args['--model'] == 'knearest':
        app.adjust_KNearest()
    else:
        app.adjust_SVM()
    print('work time: %f s' % (clock() - t))
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

- **App**: A class/struct defined in this file

### Functions and Methods

- **adjust_KNearest()**: A function/method defined in this file
- **preprocess()**: A function/method defined in this file
- **run_jobs()**: A function/method defined in this file
- **adjust_SVM()**: A function/method defined in this file
- **get_dataset()**: A function/method defined in this file
- **f()**: A function/method defined in this file
- **cross_validate()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `digits`
- `sys`
- `ThreadPool`
- `getopt`
- `print_function`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

