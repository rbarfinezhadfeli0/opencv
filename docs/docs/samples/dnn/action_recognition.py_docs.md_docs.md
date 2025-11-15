# Documentation for `docs/samples/dnn/action_recognition.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/action_recognition.py_docs.md`
- **File Name**: `action_recognition.py_docs.md`
- **File Size**: 6,513 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/action_recognition.py_docs.md](../../../docs/samples/dnn/action_recognition.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/action_recognition.py`

## File Metadata

- **Full Path**: `samples/dnn/action_recognition.py`
- **File Name**: `action_recognition.py`
- **File Size**: 3,370 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/action_recognition.py](../../samples/dnn/action_recognition.py)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import os
import numpy as np
import cv2 as cv
import argparse
from common import findFile

parser = argparse.ArgumentParser(description='Use this script to run action recognition using 3D ResNet34',
                                 formatter_class=argparse.ArgumentDefaultsHelpFormatter)
parser.add_argument('--input', '-i', help='Path to input video file. Skip this argument to capture frames from a camera.')
parser.add_argument('--model', required=True, help='Path to model.')
parser.add_argument('--classes', default=findFile('action_recongnition_kinetics.txt'), help='Path to classes list.')

# To get net download original repository https://github.com/kenshohara/video-classification-3d-cnn-pytorch
# For correct ONNX export modify file: video-classification-3d-cnn-pytorch/models/resnet.py
# change
# - def downsample_basic_block(x, planes, stride):
# -     out = F.avg_pool3d(x, kernel_size=1, stride=stride)
# -     zero_pads = torch.Tensor(out.size(0), planes - out.size(1),
# -                              out.size(2), out.size(3),
# -                              out.size(4)).zero_()
# -     if isinstance(out.data, torch.cuda.FloatTensor):
# -         zero_pads = zero_pads.cuda()
# -
# -     out = Variable(torch.cat([out.data, zero_pads], dim=1))
# -     return out

# To
# + def downsample_basic_block(x, planes, stride):
# +     out = F.avg_pool3d(x, kernel_size=1, stride=stride)
# +     out = F.pad(out, (0, 0, 0, 0, 0, 0, 0, int(planes - out.size(1)), 0, 0), "constant", 0)
# +     return out

# To ONNX export use torch.onnx.export(model, inputs, model_name)

def get_class_names(path):
    class_names = []
    with open(path) as f:
        for row in f:
            class_names.append(row[:-1])
    return class_names

def classify_video(video_path, net_path):
    SAMPLE_DURATION = 16
    SAMPLE_SIZE = 112
    mean = (114.7748, 107.7354, 99.4750)
    class_names = get_class_names(args.classes)

    net = cv.dnn.readNet(net_path)
    net.setPreferableBackend(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE)
    net.setPreferableTarget(cv.dnn.DNN_TARGET_CPU)

    winName = 'Deep learning image classification in OpenCV'
    cv.namedWindow(winName, cv.WINDOW_AUTOSIZE)
    cap = cv.VideoCapture(video_path)
    while cv.waitKey(1) < 0:
        frames = []
        for _ in range(SAMPLE_DURATION):
            hasFrame, frame = cap.read()
            if not hasFrame:
                exit(0)
            frames.append(frame)

        inputs = cv.dnn.blobFromImages(frames, 1, (SAMPLE_SIZE, SAMPLE_SIZE), mean, True, crop=True)
        inputs = np.transpose(inputs, (1, 0, 2, 3))
        inputs = np.expand_dims(inputs, axis=0)
        net.setInput(inputs)
        outputs = net.forward()
        class_pred = np.argmax(outputs)
        label = class_names[class_pred]

        for frame in frames:
            labelSize, baseLine = cv.getTextSize(label, cv.FONT_HERSHEY_SIMPLEX, 0.5, 1)
            cv.rectangle(frame, (0, 10 - labelSize[1]),
                                (labelSize[0], 10 + baseLine), (255, 255, 255), cv.FILLED)
            cv.putText(frame, label, (0, 10), cv.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0))
            cv.imshow(winName, frame)
        if cv.waitKey(1) & 0xFF == ord('q'):
            break

if __name__ == "__main__":
    args, _ = parser.parse_known_args()
    classify_video(args.input if args.input else 0, args.model)
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

### Functions and Methods

- **classify_video()**: A function/method defined in this file
- **downsample_basic_block()**: A function/method defined in this file
- **get_class_names()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `os`
- `common`
- `numpy`
- `cv2`
- `argparse`
- `findFile`
- `a`


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

