# Documentation for `modules/gapi/misc/python/test/test_gapi_infer.py`

## File Metadata

- **Full Path**: `modules/gapi/misc/python/test/test_gapi_infer.py`
- **File Name**: `test_gapi_infer.py`
- **File Size**: 15,046 bytes
- **File Type**: .py
- **Link to Source**: [modules/gapi/misc/python/test/test_gapi_infer.py](../../../../../modules/gapi/misc/python/test/test_gapi_infer.py)

## Purpose and Role

This file is located in the `modules/gapi/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

import numpy as np
import cv2 as cv
import os
import sys
import unittest

from tests_common import NewOpenCVTests


try:

    if sys.version_info[:2] < (3, 0):
        raise unittest.SkipTest('Python 2.x is not supported')


    class test_gapi_infer(NewOpenCVTests):

        def infer_reference_network(self, model_path, weights_path, img):
            net = cv.dnn.readNetFromModelOptimizer(model_path, weights_path)
            net.setPreferableBackend(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE)
            net.setPreferableTarget(cv.dnn.DNN_TARGET_CPU)

            blob = cv.dnn.blobFromImage(img)

            net.setInput(blob)
            return net.forward(net.getUnconnectedOutLayersNames())


        def make_roi(self, img, roi):
            return img[roi[1]:roi[1] + roi[3], roi[0]:roi[0] + roi[2], ...]


        def test_age_gender_infer(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/age-gender-recognition-retail-0013/FP32/age-gender-recognition-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            device_id    = 'CPU'

            img_path  = self.find_file('cv/face/david2.jpg', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            img       = cv.resize(cv.imread(img_path), (62,62))

            # OpenCV DNN
            dnn_age, dnn_gender = self.infer_reference_network(model_path, weights_path, img)

            # OpenCV G-API
            g_in   = cv.GMat()
            inputs = cv.GInferInputs()
            inputs.setInput('data', g_in)

            outputs  = cv.gapi.infer("net", inputs)
            age_g    = outputs.at("age_conv3")
            gender_g = outputs.at("prob")

            comp = cv.GComputation(cv.GIn(g_in), cv.GOut(age_g, gender_g))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_age, gapi_gender = comp.apply(cv.gin(img), args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Check
            self.assertEqual(0.0, cv.norm(dnn_gender, gapi_gender, cv.NORM_INF))
            self.assertEqual(0.0, cv.norm(dnn_age, gapi_age, cv.NORM_INF))


        def test_age_gender_infer_roi(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/age-gender-recognition-retail-0013/FP32/age-gender-recognition-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            device_id    = 'CPU'

            img_path = self.find_file('cv/face/david2.jpg', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            img = cv.imread(img_path)
            roi      = (10, 10, 62, 62)

            # OpenCV DNN
            dnn_age, dnn_gender = self.infer_reference_network(model_path,
                                                               weights_path,
                                                               self.make_roi(img, roi))

            # OpenCV G-API
            g_in   = cv.GMat()
            g_roi  = cv.GOpaqueT(cv.gapi.CV_RECT)
            inputs = cv.GInferInputs()
            inputs.setInput('data', g_in)

            outputs  = cv.gapi.infer("net", g_roi, inputs)
            age_g    = outputs.at("age_conv3")
            gender_g = outputs.at("prob")

            comp = cv.GComputation(cv.GIn(g_in, g_roi), cv.GOut(age_g, gender_g))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_age, gapi_gender = comp.apply(cv.gin(img, roi), args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Check
            self.assertEqual(0.0, cv.norm(dnn_gender, gapi_gender, cv.NORM_INF))
            self.assertEqual(0.0, cv.norm(dnn_age, gapi_age, cv.NORM_INF))


        def test_age_gender_infer_roi_list(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/age-gender-recognition-retail-0013/FP32/age-gender-recognition-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            device_id    = 'CPU'

            rois = [(10, 15, 62, 62), (23, 50, 62, 62), (14, 100, 62, 62), (80, 50, 62, 62)]
            img_path = self.find_file('cv/face/david2.jpg', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            img = cv.imread(img_path)

            # OpenCV DNN
            dnn_age_list    = []
            dnn_gender_list = []
            for roi in rois:
                age, gender = self.infer_reference_network(model_path,
                                                           weights_path,
                                                           self.make_roi(img, roi))
                dnn_age_list.append(age)
                dnn_gender_list.append(gender)

            # OpenCV G-API
            g_in   = cv.GMat()
            g_rois = cv.GArrayT(cv.gapi.CV_RECT)
            inputs = cv.GInferInputs()
            inputs.setInput('data', g_in)

            outputs  = cv.gapi.infer("net", g_rois, inputs)
            age_g    = outputs.at("age_conv3")
            gender_g = outputs.at("prob")

            comp = cv.GComputation(cv.GIn(g_in, g_rois), cv.GOut(age_g, gender_g))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_age_list, gapi_gender_list = comp.apply(cv.gin(img, rois),
                                                         args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Check
            for gapi_age, gapi_gender, dnn_age, dnn_gender in zip(gapi_age_list,
                                                                  gapi_gender_list,
                                                                  dnn_age_list,
                                                                  dnn_gender_list):
                self.assertEqual(0.0, cv.norm(dnn_gender, gapi_gender, cv.NORM_INF))
                self.assertEqual(0.0, cv.norm(dnn_age, gapi_age, cv.NORM_INF))


        def test_age_gender_infer2_roi(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/age-gender-recognition-retail-0013/FP32/age-gender-recognition-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            device_id    = 'CPU'

            rois = [(10, 15, 62, 62), (23, 50, 62, 62), (14, 100, 62, 62), (80, 50, 62, 62)]
            img_path = self.find_file('cv/face/david2.jpg', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            img = cv.imread(img_path)

            # OpenCV DNN
            dnn_age_list    = []
            dnn_gender_list = []
            for roi in rois:
                age, gender = self.infer_reference_network(model_path,
                                                           weights_path,
                                                           self.make_roi(img, roi))
                dnn_age_list.append(age)
                dnn_gender_list.append(gender)

            # OpenCV G-API
            g_in   = cv.GMat()
            g_rois = cv.GArrayT(cv.gapi.CV_RECT)
            inputs = cv.GInferListInputs()
            inputs.setInput('data', g_rois)

            outputs  = cv.gapi.infer2("net", g_in, inputs)
            age_g    = outputs.at("age_conv3")
            gender_g = outputs.at("prob")

            comp = cv.GComputation(cv.GIn(g_in, g_rois), cv.GOut(age_g, gender_g))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_age_list, gapi_gender_list = comp.apply(cv.gin(img, rois),
                                                         args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Check
            for gapi_age, gapi_gender, dnn_age, dnn_gender in zip(gapi_age_list,
                                                                  gapi_gender_list,
                                                                  dnn_age_list,
                                                                  dnn_gender_list):
                self.assertEqual(0.0, cv.norm(dnn_gender, gapi_gender, cv.NORM_INF))
                self.assertEqual(0.0, cv.norm(dnn_age, gapi_age, cv.NORM_INF))



        def test_person_detection_retail_0013(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/person-detection-retail-0013/FP32/person-detection-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            img_path     = self.find_file('gpu/lbpcascade/er.png', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            device_id    = 'CPU'
            img          = cv.resize(cv.imread(img_path), (544, 320))

            # OpenCV DNN
            net = cv.dnn.readNetFromModelOptimizer(model_path, weights_path)
            net.setPreferableBackend(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE)
            net.setPreferableTarget(cv.dnn.DNN_TARGET_CPU)

            blob = cv.dnn.blobFromImage(img)

            def parseSSD(detections, size):
                h, w = size
                bboxes = []
                detections = detections.reshape(-1, 7)
                for sample_id, class_id, confidence, xmin, ymin, xmax, ymax in detections:
                    if confidence >= 0.5:
                        x      = int(xmin * w)
                        y      = int(ymin * h)
                        width  = int(xmax * w - x)
                        height = int(ymax * h - y)
                        bboxes.append((x, y, width, height))

                return bboxes

            net.setInput(blob)
            dnn_detections = net.forward()
            dnn_boxes = parseSSD(np.array(dnn_detections), img.shape[:2])

            # OpenCV G-API
            g_in   = cv.GMat()
            inputs = cv.GInferInputs()
            inputs.setInput('data', g_in)

            g_sz       = cv.gapi.streaming.size(g_in)
            outputs    = cv.gapi.infer("net", inputs)
            detections = outputs.at("detection_out")
            bboxes     = cv.gapi.parseSSD(detections, g_sz, 0.5, False, False)

            comp = cv.GComputation(cv.GIn(g_in), cv.GOut(bboxes))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_boxes = comp.apply(cv.gin(img.astype(np.float32)),
                                    args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Comparison
            self.assertEqual(0.0, cv.norm(np.array(dnn_boxes).flatten(),
                                          np.array(gapi_boxes).flatten(),
                                          cv.NORM_INF))


        def test_person_detection_retail_0013(self):
            # NB: Check IE
            if not cv.dnn.DNN_TARGET_CPU in cv.dnn.getAvailableTargets(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE):
                return

            root_path    = '/omz_intel_models/intel/person-detection-retail-0013/FP32/person-detection-retail-0013'
            model_path   = self.find_file(root_path + '.xml',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            weights_path = self.find_file(root_path + '.bin',   [os.environ.get('OPENCV_DNN_TEST_DATA_PATH')], required=False)
            img_path     = self.find_file('gpu/lbpcascade/er.png', [os.environ.get('OPENCV_TEST_DATA_PATH')])
            device_id    = 'CPU'
            img          = cv.resize(cv.imread(img_path), (544, 320))

            # OpenCV DNN
            net = cv.dnn.readNetFromModelOptimizer(model_path, weights_path)
            net.setPreferableBackend(cv.dnn.DNN_BACKEND_INFERENCE_ENGINE)
            net.setPreferableTarget(cv.dnn.DNN_TARGET_CPU)

            blob = cv.dnn.blobFromImage(img)

            def parseSSD(detections, size):
                h, w = size
                bboxes = []
                detections = detections.reshape(-1, 7)
                for sample_id, class_id, confidence, xmin, ymin, xmax, ymax in detections:
                    if confidence >= 0.5:
                        x      = int(xmin * w)
                        y      = int(ymin * h)
                        width  = int(xmax * w - x)
                        height = int(ymax * h - y)
                        bboxes.append((x, y, width, height))

                return bboxes

            net.setInput(blob)
            dnn_detections = net.forward()
            dnn_boxes = parseSSD(np.array(dnn_detections), img.shape[:2])

            # OpenCV G-API
            g_in   = cv.GMat()
            inputs = cv.GInferInputs()
            inputs.setInput('data', g_in)

            g_sz       = cv.gapi.streaming.size(g_in)
            outputs    = cv.gapi.infer("net", inputs)
            detections = outputs.at("detection_out")
            bboxes     = cv.gapi.parseSSD(detections, g_sz, 0.5, False, False)

            comp = cv.GComputation(cv.GIn(g_in), cv.GOut(bboxes))
            pp = cv.gapi.ie.params("net", model_path, weights_path, device_id)

            gapi_boxes = comp.apply(cv.gin(img.astype(np.float32)),
                                    args=cv.gapi.compile_args(cv.gapi.networks(pp)))

            # Comparison
            self.assertEqual(0.0, cv.norm(np.array(dnn_boxes).flatten(),
                                          np.array(gapi_boxes).flatten(),
                                          cv.NORM_INF))


except unittest.SkipTest as e:

    message = str(e)

    class TestSkip(unittest.TestCase):
        def setUp(self):
            self.skipTest('Skip tests: ' + message)

        def test_skip():
            pass

    pass


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

- **test_gapi_infer**: A class/struct defined in this file
- **TestSkip**: A class/struct defined in this file

### Functions and Methods

- **infer_reference_network()**: A function/method defined in this file
- **test_age_gender_infer_roi_list()**: A function/method defined in this file
- **test_skip()**: A function/method defined in this file
- **make_roi()**: A function/method defined in this file
- **test_age_gender_infer2_roi()**: A function/method defined in this file
- **test_age_gender_infer_roi()**: A function/method defined in this file
- **test_age_gender_infer()**: A function/method defined in this file
- **parseSSD()**: A function/method defined in this file
- **setUp()**: A function/method defined in this file
- **test_person_detection_retail_0013()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `os`
- `NewOpenCVTests`
- `numpy`
- `cv2`
- `unittest`


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

