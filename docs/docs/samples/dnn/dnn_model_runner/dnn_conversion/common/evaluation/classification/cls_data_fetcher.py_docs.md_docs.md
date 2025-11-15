# Documentation for `docs/samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py_docs.md`
- **File Name**: `cls_data_fetcher.py_docs.md`
- **File Size**: 6,772 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py_docs.md](../../../../../../../../docs/samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py`

## File Metadata

- **Full Path**: `samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py`
- **File Name**: `cls_data_fetcher.py`
- **File Size**: 2,774 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py](../../../../../../../samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification/cls_data_fetcher.py)

## Purpose and Role

This file is located in the `samples/dnn/dnn_model_runner/dnn_conversion/common/evaluation/classification` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import os
from abc import ABCMeta, abstractmethod

import cv2
import numpy as np

from ...img_utils import read_rgb_img, get_pytorch_preprocess
from ...test.configs.default_preprocess_config import PYTORCH_RSZ_HEIGHT, PYTORCH_RSZ_WIDTH


class DataFetch(object):
    imgs_dir = ''
    frame_size = 0
    bgr_to_rgb = False

    __metaclass__ = ABCMeta

    @abstractmethod
    def preprocess(self, img):
        pass

    @staticmethod
    def reshape_img(img):
        img = img[:, :, 0:3].transpose(2, 0, 1)
        return np.expand_dims(img, 0)

    def center_crop(self, img):
        cols = img.shape[1]
        rows = img.shape[0]

        y1 = round((rows - self.frame_size) / 2)
        y2 = round(y1 + self.frame_size)
        x1 = round((cols - self.frame_size) / 2)
        x2 = round(x1 + self.frame_size)
        return img[y1:y2, x1:x2]

    def initial_preprocess(self, img):
        min_dim = min(img.shape[-3], img.shape[-2])
        resize_ratio = self.frame_size / float(min_dim)

        img = cv2.resize(img, (0, 0), fx=resize_ratio, fy=resize_ratio)
        img = self.center_crop(img)
        return img

    def get_preprocessed_img(self, img_path):
        image_data = read_rgb_img(img_path, self.bgr_to_rgb)
        image_data = self.preprocess(image_data)
        return self.reshape_img(image_data)

    def get_batch(self, img_names):
        assert type(img_names) is list
        batch = np.zeros((len(img_names), 3, self.frame_size, self.frame_size)).astype(np.float32)

        for i in range(len(img_names)):
            img_name = img_names[i]
            img_file = os.path.join(self.imgs_dir, img_name)
            assert os.path.exists(img_file)

            batch[i] = self.get_preprocessed_img(img_file)
        return batch


class PyTorchPreprocessedFetch(DataFetch):
    def __init__(self, pytorch_cls_config, preprocess_input=None):
        self.imgs_dir = pytorch_cls_config.img_root_dir
        self.frame_size = pytorch_cls_config.frame_size
        self.bgr_to_rgb = pytorch_cls_config.bgr_to_rgb
        self.preprocess_input = preprocess_input

    def preprocess(self, img):
        img = cv2.resize(img, (PYTORCH_RSZ_WIDTH, PYTORCH_RSZ_HEIGHT))
        img = self.center_crop(img)
        if self.preprocess_input:
            return self.presprocess_input(img)
        return get_pytorch_preprocess(img)


class TFPreprocessedFetch(DataFetch):
    def __init__(self, tf_cls_config, preprocess_input):
        self.imgs_dir = tf_cls_config.img_root_dir
        self.frame_size = tf_cls_config.frame_size
        self.bgr_to_rgb = tf_cls_config.bgr_to_rgb
        self.preprocess_input = preprocess_input

    def preprocess(self, img):
        img = self.initial_preprocess(img)
        return self.preprocess_input(img)
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

- **PyTorchPreprocessedFetch**: A class/struct defined in this file
- **TFPreprocessedFetch**: A class/struct defined in this file
- **DataFetch**: A class/struct defined in this file

### Functions and Methods

- **preprocess()**: A function/method defined in this file
- **initial_preprocess()**: A function/method defined in this file
- **get_preprocessed_img()**: A function/method defined in this file
- **center_crop()**: A function/method defined in this file
- **reshape_img()**: A function/method defined in this file
- **get_batch()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `abc`
- `read_rgb_img`
- `os`
- `...img_utils`
- `PYTORCH_RSZ_HEIGHT`
- `ABCMeta`
- `...test.configs.default_preprocess_config`
- `numpy`
- `cv2`


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

