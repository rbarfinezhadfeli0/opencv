# Documentation for `docs/samples/dnn/optical_flow.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/optical_flow.py_docs.md`
- **File Name**: `optical_flow.py_docs.md`
- **File Size**: 8,080 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/optical_flow.py_docs.md](../../../docs/samples/dnn/optical_flow.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/optical_flow.py`

## File Metadata

- **Full Path**: `samples/dnn/optical_flow.py`
- **File Name**: `optical_flow.py`
- **File Size**: 4,890 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/optical_flow.py](../../samples/dnn/optical_flow.py)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
This sample using FlowNet v2 and RAFT model to calculate optical flow.

FlowNet v2 Original Paper: https://arxiv.org/abs/1612.01925.
FlowNet v2 Repo:  https://github.com/lmb-freiburg/flownet2.

Download the converted .caffemodel model from https://drive.google.com/open?id=16qvE9VNmU39NttpZwZs81Ga8VYQJDaWZ
and .prototxt from https://drive.google.com/file/d/1RyNIUsan1ZOh2hpYIH36A-jofAvJlT6a/view?usp=sharing.
Otherwise download original model from https://lmb.informatik.uni-freiburg.de/resources/binaries/flownet2/flownet2-models.tar.gz,
convert .h5 model to .caffemodel and modify original .prototxt using .prototxt from link above.

RAFT Original Paper: https://arxiv.org/pdf/2003.12039.pdf
RAFT Repo: https://github.com/princeton-vl/RAFT

Download the .onnx model from here https://github.com/opencv/opencv_zoo/raw/281d232cd99cd920853106d853c440edd35eb442/models/optical_flow_estimation_raft/optical_flow_estimation_raft_2023aug.onnx.
'''

import argparse
import os.path
import numpy as np
import cv2 as cv


class OpticalFlow(object):
    def __init__(self, model, height, width, proto=""):
        if proto:
            self.net = cv.dnn.readNetFromCaffe(proto, model)
        else:
            self.net = cv.dnn.readNet(model)
        self.net.setPreferableBackend(cv.dnn.DNN_BACKEND_OPENCV)
        self.height = height
        self.width = width

    def compute_flow(self, first_img, second_img):
        inp0 = cv.dnn.blobFromImage(first_img, size=(self.width, self.height))
        inp1 = cv.dnn.blobFromImage(second_img, size=(self.width, self.height))
        self.net.setInputsNames(["img0", "img1"])
        self.net.setInput(inp0, "img0")
        self.net.setInput(inp1, "img1")

        flow = self.net.forward()
        output = self.motion_to_color(flow)
        return output

    def motion_to_color(self, flow):
        arr = np.arange(0, 255, dtype=np.uint8)
        colormap = cv.applyColorMap(arr, cv.COLORMAP_HSV)
        colormap = colormap.squeeze(1)

        flow = flow.squeeze(0)
        fx, fy = flow[0, ...], flow[1, ...]
        rad = np.sqrt(fx**2 + fy**2)
        maxrad = rad.max() if rad.max() != 0 else 1

        ncols = arr.size
        rad = rad[..., np.newaxis] / maxrad
        a = np.arctan2(-fy / maxrad, -fx / maxrad) / np.pi
        fk = (a + 1) / 2.0 * (ncols - 1)
        k0 = fk.astype(np.int32)
        k1 = (k0 + 1) % ncols
        f = fk[..., np.newaxis] - k0[..., np.newaxis]

        col0 = colormap[k0] / 255.0
        col1 = colormap[k1] / 255.0
        col = (1 - f) * col0 + f * col1
        col = np.where(rad <= 1, 1 - rad * (1 - col), col * 0.75)
        output = (255.0 * col).astype(np.uint8)
        return output


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Use this script to calculate optical flow',
                                     formatter_class=argparse.ArgumentDefaultsHelpFormatter)
    parser.add_argument('-input', '-i', required=True, help='Path to input video file. Skip this argument to capture frames from a camera.')
    parser.add_argument('--height', default=320, type=int, help='Input height')
    parser.add_argument('--width', default=448, type=int, help='Input width')
    parser.add_argument('--proto', '-p', default='', help='Path to prototxt.')
    parser.add_argument('--model', '-m', required=True, help='Path to model.')
    args, _ = parser.parse_known_args()

    if not os.path.isfile(args.model):
        raise OSError("Model does not exist")
    if args.proto and not os.path.isfile(args.proto):
        raise OSError("Prototxt does not exist")

    winName = 'Calculation optical flow in OpenCV'
    cv.namedWindow(winName, cv.WINDOW_NORMAL)
    cap = cv.VideoCapture(args.input if args.input else 0)
    hasFrame, first_frame = cap.read()

    if args.proto:
        divisor = 64.
        var = {}
        var['ADAPTED_WIDTH'] = int(np.ceil(args.width/divisor) * divisor)
        var['ADAPTED_HEIGHT'] = int(np.ceil(args.height/divisor) * divisor)
        var['SCALE_WIDTH'] = args.width / float(var['ADAPTED_WIDTH'])
        var['SCALE_HEIGHT'] = args.height / float(var['ADAPTED_HEIGHT'])

        config = ''
        proto = open(args.proto).readlines()
        for line in proto:
            for key, value in var.items():
                tag = "$%s$" % key
                line = line.replace(tag, str(value))
            config += line

        caffemodel = open(args.model, 'rb').read()

        opt_flow = OpticalFlow(caffemodel, var['ADAPTED_HEIGHT'], var['ADAPTED_WIDTH'], bytearray(config.encode()))
    else:
        opt_flow = OpticalFlow(args.model, 360, 480)

    while cv.waitKey(1) < 0:
        hasFrame, second_frame = cap.read()
        if not hasFrame:
            break
        flow = opt_flow.compute_flow(first_frame, second_frame)
        first_frame = second_frame
        cv.imshow(winName, flow)
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

- **OpticalFlow**: A class/struct defined in this file

### Functions and Methods

- **motion_to_color()**: A function/method defined in this file
- **compute_flow()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `here`
- `https`
- `os.path`
- `numpy`
- `argparse`
- `cv2`
- `a`
- `link`


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

