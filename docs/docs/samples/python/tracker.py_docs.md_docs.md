# Documentation for `docs/samples/python/tracker.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/tracker.py_docs.md`
- **File Name**: `tracker.py_docs.md`
- **File Size**: 11,898 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/tracker.py_docs.md](../../../docs/samples/python/tracker.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/tracker.py`

## File Metadata

- **Full Path**: `samples/python/tracker.py`
- **File Name**: `tracker.py`
- **File Size**: 8,591 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/tracker.py](../../samples/python/tracker.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
Tracker demo

For usage download models by following links
For GOTURN:
    goturn.prototxt and goturn.caffemodel: https://github.com/opencv/opencv_extra/tree/c4219d5eb3105ed8e634278fad312a1a8d2c182d/testdata/tracking
For DaSiamRPN:
    network:     https://www.dropbox.com/s/rr1lk9355vzolqv/dasiamrpn_model.onnx?dl=0
    kernel_r1:   https://www.dropbox.com/s/999cqx5zrfi7w4p/dasiamrpn_kernel_r1.onnx?dl=0
    kernel_cls1: https://www.dropbox.com/s/qvmtszx5h339a0w/dasiamrpn_kernel_cls1.onnx?dl=0
For NanoTrack:
    nanotrack_backbone: https://github.com/HonglinChu/SiamTrackers/blob/master/NanoTrack/models/nanotrackv2/nanotrack_backbone_sim.onnx
    nanotrack_headneck: https://github.com/HonglinChu/SiamTrackers/blob/master/NanoTrack/models/nanotrackv2/nanotrack_head_sim.onnx

USAGE:
    tracker.py [-h] [--input INPUT_VIDEO]
                    [--tracker_algo TRACKER_ALGO (mil, goturn, dasiamrpn, nanotrack, vittrack)]
                    [--goturn GOTURN_PROTOTXT]
                    [--goturn_model GOTURN_MODEL]
                    [--dasiamrpn_net DASIAMRPN_NET]
                    [--dasiamrpn_kernel_r1 DASIAMRPN_KERNEL_R1]
                    [--dasiamrpn_kernel_cls1 DASIAMRPN_KERNEL_CLS1]
                    [--nanotrack_backbone NANOTRACK_BACKBONE]
                    [--nanotrack_headneck NANOTRACK_TARGET]
                    [--vittrack_net VITTRACK_MODEL]
                    [--vittrack_net VITTRACK_MODEL]
                    [--tracking_score_threshold TRACKING SCORE THRESHOLD FOR ONLY VITTRACK]
                    [--backend CHOOSE ONE OF COMPUTATION BACKEND]
                    [--target CHOOSE ONE OF COMPUTATION TARGET]
'''

# Python 2/3 compatibility
from __future__ import print_function

import sys

import numpy as np
import cv2 as cv
import argparse

from video import create_capture, presets

backends = (cv.dnn.DNN_BACKEND_DEFAULT, cv.dnn.DNN_BACKEND_HALIDE, cv.dnn.DNN_BACKEND_INFERENCE_ENGINE, cv.dnn.DNN_BACKEND_OPENCV,
            cv.dnn.DNN_BACKEND_VKCOM, cv.dnn.DNN_BACKEND_CUDA)
targets = (cv.dnn.DNN_TARGET_CPU, cv.dnn.DNN_TARGET_OPENCL, cv.dnn.DNN_TARGET_OPENCL_FP16, cv.dnn.DNN_TARGET_MYRIAD,
           cv.dnn.DNN_TARGET_VULKAN, cv.dnn.DNN_TARGET_CUDA, cv.dnn.DNN_TARGET_CUDA_FP16)

class App(object):

    def __init__(self, args):
        self.args = args
        self.trackerAlgorithm = args.tracker_algo
        self.tracker = self.createTracker()

    def createTracker(self):
        if self.trackerAlgorithm == 'mil':
            tracker = cv.TrackerMIL_create()
        elif self.trackerAlgorithm == 'goturn':
            params = cv.TrackerGOTURN_Params()
            params.modelTxt = self.args.goturn
            params.modelBin = self.args.goturn_model
            tracker = cv.TrackerGOTURN_create(params)
        elif self.trackerAlgorithm == 'dasiamrpn':
            params = cv.TrackerDaSiamRPN_Params()
            params.model = self.args.dasiamrpn_net
            params.kernel_cls1 = self.args.dasiamrpn_kernel_cls1
            params.kernel_r1 = self.args.dasiamrpn_kernel_r1
            params.backend = args.backend
            params.target = args.target
            tracker = cv.TrackerDaSiamRPN_create(params)
        elif self.trackerAlgorithm == 'nanotrack':
            params = cv.TrackerNano_Params()
            params.backbone = args.nanotrack_backbone
            params.neckhead = args.nanotrack_headneck
            params.backend = args.backend
            params.target = args.target
            tracker = cv.TrackerNano_create(params)
        elif self.trackerAlgorithm == 'vittrack':
            params = cv.TrackerVit_Params()
            params.net = args.vittrack_net
            params.tracking_score_threshold = args.tracking_score_threshold
            params.backend = args.backend
            params.target = args.target
            tracker = cv.TrackerVit_create(params)
        else:
            sys.exit("Tracker {} is not recognized. Please use one of three available: mil, goturn, dasiamrpn, nanotrack.".format(self.trackerAlgorithm))
        return tracker

    def initializeTracker(self, image):
        while True:
            print('==> Select object ROI for tracker ...')
            bbox = cv.selectROI('tracking', image)
            print('ROI: {}'.format(bbox))
            if bbox[2] <= 0 or bbox[3] <= 0:
                sys.exit("ROI selection cancelled. Exiting...")

            try:
                self.tracker.init(image, bbox)
            except Exception as e:
                print('Unable to initialize tracker with requested bounding box. Is there any object?')
                print(e)
                print('Try again ...')
                continue

            return

    def run(self):
        videoPath = self.args.input
        print('Using video: {}'.format(videoPath))
        camera = create_capture(cv.samples.findFileOrKeep(videoPath), presets['cube'])
        if not camera.isOpened():
            sys.exit("Can't open video stream: {}".format(videoPath))

        ok, image = camera.read()
        if not ok:
            sys.exit("Can't read first frame")
        assert image is not None

        cv.namedWindow('tracking')
        self.initializeTracker(image)

        print("==> Tracking is started. Press 'SPACE' to re-initialize tracker or 'ESC' for exit...")

        while camera.isOpened():
            ok, image = camera.read()
            if not ok:
                print("Can't read frame")
                break

            ok, newbox = self.tracker.update(image)
            #print(ok, newbox)

            if ok:
                cv.rectangle(image, newbox, (200,0,0))

            cv.imshow("tracking", image)
            k = cv.waitKey(1)
            if k == 32:  # SPACE
                self.initializeTracker(image)
            if k == 27:  # ESC
                break

        print('Done')


if __name__ == '__main__':
    print(__doc__)
    parser = argparse.ArgumentParser(description="Run tracker")
    parser.add_argument("--input", type=str, default="vtest.avi", help="Path to video source")
    parser.add_argument("--tracker_algo", type=str, default="nanotrack", help="One of available tracking algorithms: mil, goturn, dasiamrpn, nanotrack, vittrack")
    parser.add_argument("--goturn", type=str, default="goturn.prototxt", help="Path to GOTURN architecture")
    parser.add_argument("--goturn_model", type=str, default="goturn.caffemodel", help="Path to GOTERN model")
    parser.add_argument("--dasiamrpn_net", type=str, default="dasiamrpn_model.onnx", help="Path to onnx model of DaSiamRPN net")
    parser.add_argument("--dasiamrpn_kernel_r1", type=str, default="dasiamrpn_kernel_r1.onnx", help="Path to onnx model of DaSiamRPN kernel_r1")
    parser.add_argument("--dasiamrpn_kernel_cls1", type=str, default="dasiamrpn_kernel_cls1.onnx", help="Path to onnx model of DaSiamRPN kernel_cls1")
    parser.add_argument("--nanotrack_backbone", type=str, default="nanotrack_backbone_sim.onnx", help="Path to onnx model of NanoTrack backBone")
    parser.add_argument("--nanotrack_headneck", type=str, default="nanotrack_head_sim.onnx", help="Path to onnx model of NanoTrack headNeck")
    parser.add_argument("--vittrack_net", type=str, default="vitTracker.onnx", help="Path to onnx model of  vittrack")
    parser.add_argument('--tracking_score_threshold', type=float,  help="Tracking score threshold. If a bbox of score >= 0.3, it is considered as found ")
    parser.add_argument('--backend', choices=backends, default=cv.dnn.DNN_BACKEND_DEFAULT, type=int,
                help="Choose one of computation backends: "
                        "%d: automatically (by default), "
                        "%d: Halide language (http://halide-lang.org/), "
                        "%d: Intel's Deep Learning Inference Engine (https://software.intel.com/openvino-toolkit), "
                        "%d: OpenCV implementation, "
                        "%d: VKCOM, "
                        "%d: CUDA"% backends)
    parser.add_argument("--target", choices=targets, default=cv.dnn.DNN_TARGET_CPU, type=int,
                help="Choose one of target computation devices: "
                        '%d: CPU target (by default), '
                        '%d: OpenCL, '
                        '%d: OpenCL fp16 (half-float precision), '
                        '%d: VPU, '
                        '%d: VULKAN, '
                        '%d: CUDA, '
                        '%d: CUDA fp16 (half-float preprocess)'% targets)

    args = parser.parse_args()
    App(args).run()
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

- **App**: A class/struct defined in this file

### Functions and Methods

- **initializeTracker()**: A function/method defined in this file
- **run()**: A function/method defined in this file
- **createTracker()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
- `print_function`
- `numpy`
- `cv2`
- `argparse`
- `video`
- `__future__`
- `create_capture`


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

