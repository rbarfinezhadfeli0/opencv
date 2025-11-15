# Documentation for `docs/samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py_docs.md`
- **File Name**: `py_to_py_resnet50.py_docs.md`
- **File Size**: 7,814 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py_docs.md](../../../../../../../docs/samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py`

## File Metadata

- **Full Path**: `samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py`
- **File Name**: `py_to_py_resnet50.py`
- **File Size**: 4,055 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py](../../../../../../samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification/py_to_py_resnet50.py)

## Purpose and Role

This file is located in the `samples/dnn/dnn_model_runner/dnn_conversion/pytorch/classification` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import os

import cv2
import numpy as np
import torch
import torch.onnx
from torch.autograd import Variable
from torchvision import models


def get_pytorch_onnx_model(original_model):
    # define the directory for further converted model save
    onnx_model_path = "models"
    # define the name of further converted model
    onnx_model_name = "resnet50.onnx"

    # create directory for further converted model
    os.makedirs(onnx_model_path, exist_ok=True)

    # get full path to the converted model
    full_model_path = os.path.join(onnx_model_path, onnx_model_name)

    # generate model input
    generated_input = Variable(
        torch.randn(1, 3, 224, 224)
    )

    # model export into ONNX format
    torch.onnx.export(
        original_model,
        generated_input,
        full_model_path,
        verbose=True,
        input_names=["input"],
        output_names=["output"],
        opset_version=11
    )

    return full_model_path


def get_preprocessed_img(img_path):
    # read the image
    input_img = cv2.imread(img_path, cv2.IMREAD_COLOR)
    input_img = input_img.astype(np.float32)

    input_img = cv2.resize(input_img, (256, 256))

    # define preprocess parameters
    mean = np.array([0.485, 0.456, 0.406]) * 255.0
    scale = 1 / 255.0
    std = [0.229, 0.224, 0.225]

    # prepare input blob to fit the model input:
    # 1. subtract mean
    # 2. scale to set pixel values from 0 to 1
    input_blob = cv2.dnn.blobFromImage(
        image=input_img,
        scalefactor=scale,
        size=(224, 224),  # img target size
        mean=mean,
        swapRB=True,  # BGR -> RGB
        crop=True  # center crop
    )
    # 3. divide by std
    input_blob[0] /= np.asarray(std, dtype=np.float32).reshape(3, 1, 1)
    return input_blob


def get_imagenet_labels(labels_path):
    with open(labels_path) as f:
        imagenet_labels = [line.strip() for line in f.readlines()]
    return imagenet_labels


def get_opencv_dnn_prediction(opencv_net, preproc_img, imagenet_labels):
    # set OpenCV DNN input
    opencv_net.setInput(preproc_img)

    # OpenCV DNN inference
    out = opencv_net.forward()
    print("OpenCV DNN prediction: \n")
    print("* shape: ", out.shape)

    # get the predicted class ID
    imagenet_class_id = np.argmax(out)

    # get confidence
    confidence = out[0][imagenet_class_id]
    print("* class ID: {}, label: {}".format(imagenet_class_id, imagenet_labels[imagenet_class_id]))
    print("* confidence: {:.4f}".format(confidence))


def get_pytorch_dnn_prediction(original_net, preproc_img, imagenet_labels):
    original_net.eval()
    preproc_img = torch.FloatTensor(preproc_img)

    # inference
    with torch.no_grad():
        out = original_net(preproc_img)

    print("\nPyTorch model prediction: \n")
    print("* shape: ", out.shape)

    # get the predicted class ID
    imagenet_class_id = torch.argmax(out, axis=1).item()
    print("* class ID: {}, label: {}".format(imagenet_class_id, imagenet_labels[imagenet_class_id]))

    # get confidence
    confidence = out[0][imagenet_class_id]
    print("* confidence: {:.4f}".format(confidence.item()))


def main():
    # initialize PyTorch ResNet-50 model
    original_model = models.resnet50(pretrained=True)

    # get the path to the converted into ONNX PyTorch model
    full_model_path = get_pytorch_onnx_model(original_model)

    # read converted .onnx model with OpenCV API
    opencv_net = cv2.dnn.readNetFromONNX(full_model_path)
    print("OpenCV model was successfully read. Layer IDs: \n", opencv_net.getLayerNames())

    # get preprocessed image
    input_img = get_preprocessed_img("../data/squirrel_cls.jpg")

    # get ImageNet labels
    imagenet_labels = get_imagenet_labels("../data/dnn/classification_classes_ILSVRC2012.txt")

    # obtain OpenCV DNN predictions
    get_opencv_dnn_prediction(opencv_net, input_img, imagenet_labels)

    # obtain original PyTorch ResNet50 predictions
    get_pytorch_dnn_prediction(original_model, input_img, imagenet_labels)


if __name__ == "__main__":
    main()
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

- **ID**: A class/struct defined in this file

### Functions and Methods

- **get_opencv_dnn_prediction()**: A function/method defined in this file
- **get_imagenet_labels()**: A function/method defined in this file
- **get_preprocessed_img()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **get_pytorch_dnn_prediction()**: A function/method defined in this file
- **get_pytorch_onnx_model()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `os`
- `models`
- `torch.autograd`
- `torchvision`
- `numpy`
- `torch`
- `cv2`
- `0`
- `torch.onnx`
- `Variable`


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

