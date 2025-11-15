# Documentation for `docs/samples/dnn/person_reid.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/person_reid.py_docs.md`
- **File Name**: `person_reid.py_docs.md`
- **File Size**: 13,238 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/person_reid.py_docs.md](../../../docs/samples/dnn/person_reid.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/person_reid.py`

## File Metadata

- **Full Path**: `samples/dnn/person_reid.py`
- **File Name**: `person_reid.py`
- **File Size**: 9,738 bytes
- **File Type**: .py
- **Link to Source**: [samples/dnn/person_reid.py](../../samples/dnn/person_reid.py)

## Purpose and Role

This file is located in the `samples/dnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
'''
You can download a baseline ReID model and sample input from:
https://github.com/ReID-Team/ReID_extra_testdata

Authors of samples and Youtu ReID baseline:
        Xing Sun <winfredsun@tencent.com>
        Feng Zheng <zhengf@sustech.edu.cn>
        Xinyang Jiang <sevjiang@tencent.com>
        Fufu Yu <fufuyu@tencent.com>
        Enwei Zhang <miyozhang@tencent.com>

Copyright (C) 2020-2021, Tencent.
Copyright (C) 2020-2021, SUSTech.
'''
import argparse
import os.path
import numpy as np
import cv2 as cv

backends = (cv.dnn.DNN_BACKEND_DEFAULT,
    cv.dnn.DNN_BACKEND_INFERENCE_ENGINE,
    cv.dnn.DNN_BACKEND_OPENCV,
    cv.dnn.DNN_BACKEND_VKCOM,
    cv.dnn.DNN_BACKEND_CUDA)

targets = (cv.dnn.DNN_TARGET_CPU,
    cv.dnn.DNN_TARGET_OPENCL,
    cv.dnn.DNN_TARGET_OPENCL_FP16,
    cv.dnn.DNN_TARGET_MYRIAD,
    cv.dnn.DNN_TARGET_HDDL,
    cv.dnn.DNN_TARGET_VULKAN,
    cv.dnn.DNN_TARGET_CUDA,
    cv.dnn.DNN_TARGET_CUDA_FP16)

MEAN = (0.485, 0.456, 0.406)
STD = (0.229, 0.224, 0.225)

def preprocess(images, height, width):
    """
    Create 4-dimensional blob from image
    :param image: input image
    :param height: the height of the resized input image
    :param width: the width of the resized input image
    """
    img_list = []
    for image in images:
        image = cv.resize(image, (width, height))
        img_list.append(image[:, :, ::-1])

    images = np.array(img_list)
    images = (images / 255.0 - MEAN) / STD

    input = cv.dnn.blobFromImages(images.astype(np.float32), ddepth = cv.CV_32F)
    return input

def extract_feature(img_dir, model_path, batch_size = 32, resize_h = 384, resize_w = 128, backend=cv.dnn.DNN_BACKEND_OPENCV, target=cv.dnn.DNN_TARGET_CPU):
    """
    Extract features from images in a target directory
    :param img_dir: the input image directory
    :param model_path: path to ReID model
    :param batch_size: the batch size for each network inference iteration
    :param resize_h: the height of the input image
    :param resize_w: the width of the input image
    :param backend: name of computation backend
    :param target: name of computation target
    """
    feat_list = []
    path_list = os.listdir(img_dir)
    path_list = [os.path.join(img_dir, img_name) for img_name in path_list]
    count = 0

    for i in range(0, len(path_list), batch_size):
        print('Feature Extraction for images in', img_dir, 'Batch:', count, '/', len(path_list))
        batch = path_list[i : min(i + batch_size, len(path_list))]
        imgs = read_data(batch)
        inputs = preprocess(imgs, resize_h, resize_w)

        feat = run_net(inputs, model_path, backend, target)

        feat_list.append(feat)
        count += batch_size

    feats = np.concatenate(feat_list, axis = 0)
    return feats, path_list

def run_net(inputs, model_path, backend=cv.dnn.DNN_BACKEND_OPENCV, target=cv.dnn.DNN_TARGET_CPU):
    """
    Forward propagation for a batch of images.
    :param inputs: input batch of images
    :param model_path: path to ReID model
    :param backend: name of computation backend
    :param target: name of computation target
    """
    net = cv.dnn.readNet(model_path)
    net.setPreferableBackend(backend)
    net.setPreferableTarget(target)
    net.setInput(inputs)
    out = net.forward()
    out = np.reshape(out, (out.shape[0], out.shape[1]))
    return out

def read_data(path_list):
    """
    Read all images from a directory into a list
    :param path_list: the list of image path
    """
    img_list = []
    for img_path in path_list:
        img = cv.imread(img_path)
        if img is None:
            continue
        img_list.append(img)
    return img_list

def normalize(nparray, order=2, axis=0):
    """
    Normalize a N-D numpy array along the specified axis.
    :param nparry: the array of vectors to be normalized
    :param order: order of the norm
    :param axis: the axis of x along which to compute the vector norms
    """
    norm = np.linalg.norm(nparray, ord=order, axis=axis, keepdims=True)
    return nparray / (norm + np.finfo(np.float32).eps)

def similarity(array1, array2):
    """
    Compute the euclidean or cosine distance of all pairs.
    :param  array1: numpy array with shape [m1, n]
    :param  array2: numpy array with shape [m2, n]
    Returns:
      numpy array with shape [m1, m2]
    """
    array1 = normalize(array1, axis=1)
    array2 = normalize(array2, axis=1)
    dist = np.matmul(array1, array2.T)
    return dist

def topk(query_feat, gallery_feat, topk = 5):
    """
    Return the index of top K gallery images most similar to the query images
    :param query_feat: array of feature vectors of query images
    :param gallery_feat: array of feature vectors of gallery images
    :param topk: number of gallery images to return
    """
    sim = similarity(query_feat, gallery_feat)
    index = np.argsort(-sim, axis = 1)
    return [i[0:int(topk)] for i in index]

def drawRankList(query_name, gallery_list, output_size = (128, 384)):
    """
    Draw the rank list
    :param query_name: path of the query image
    :param gallery_name: path of the gallery image
    "param output_size: the output size of each image in the rank list
    """
    def addBorder(im, color):
        bordersize = 5
        border = cv.copyMakeBorder(
            im,
            top = bordersize,
            bottom = bordersize,
            left = bordersize,
            right = bordersize,
            borderType = cv.BORDER_CONSTANT,
            value = color
        )
        return border
    query_img = cv.imread(query_name)
    query_img = cv.resize(query_img, output_size)
    query_img = addBorder(query_img, [0, 0, 0])
    cv.putText(query_img, 'Query', (10, 30), cv.FONT_HERSHEY_COMPLEX, 1., (0,255,0), 2)

    gallery_img_list = []
    for i, gallery_name in enumerate(gallery_list):
        gallery_img = cv.imread(gallery_name)
        gallery_img = cv.resize(gallery_img, output_size)
        gallery_img = addBorder(gallery_img, [255, 255, 255])
        cv.putText(gallery_img, 'G%02d'%i, (10, 30), cv.FONT_HERSHEY_COMPLEX, 1., (0,255,0), 2)
        gallery_img_list.append(gallery_img)
    ret = np.concatenate([query_img] + gallery_img_list, axis = 1)
    return ret


def visualization(topk_idx, query_names, gallery_names, output_dir = 'vis'):
    """
    Visualize the retrieval results with the person ReID model
    :param topk_idx: the index of ranked gallery images for each query image
    :param query_names: the list of paths of query images
    :param gallery_names: the list of paths of gallery images
    :param output_dir: the path to save the visualize results
    """
    if not os.path.exists(output_dir):
        os.mkdir(output_dir)
    for i, idx in enumerate(topk_idx):
        query_name = query_names[i]
        topk_names = [gallery_names[j] for j in idx]
        vis_img = drawRankList(query_name, topk_names)
        output_path = os.path.join(output_dir, '%03d_%s'%(i, os.path.basename(query_name)))
        cv.imwrite(output_path, vis_img)

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Use this script to run human parsing using JPPNet',
                                     formatter_class=argparse.ArgumentDefaultsHelpFormatter)
    parser.add_argument('--query_dir', '-q', required=True, help='Path to query image.')
    parser.add_argument('--gallery_dir', '-g', required=True, help='Path to gallery directory.')
    parser.add_argument('--resize_h', default = 256, help='The height of the input for model inference.')
    parser.add_argument('--resize_w', default = 128, help='The width of the input for model inference')
    parser.add_argument('--model', '-m', default='reid.onnx', help='Path to pb model.')
    parser.add_argument('--visualization_dir', default='vis', help='Path for the visualization results')
    parser.add_argument('--topk', default=10, help='Number of images visualized in the rank list')
    parser.add_argument('--batchsize', default=32, help='The batch size of each inference')
    parser.add_argument('--backend', choices=backends, default=cv.dnn.DNN_BACKEND_DEFAULT, type=int,
                        help="Choose one of computation backends: "
                             "%d: automatically (by default), "
                             "%d: Intel's Deep Learning Inference Engine (https://software.intel.com/openvino-toolkit), "
                             "%d: OpenCV implementation, "
                             "%d: VKCOM, "
                             "%d: CUDA backend"% backends)
    parser.add_argument('--target', choices=targets, default=cv.dnn.DNN_TARGET_CPU, type=int,
                        help='Choose one of target computation devices: '
                             '%d: CPU target (by default), '
                             '%d: OpenCL, '
                             '%d: OpenCL fp16 (half-float precision), '
                             '%d: NCS2 VPU, '
                             '%d: HDDL VPU, '
                             '%d: Vulkan, '
                             '%d: CUDA, '
                             '%d: CUDA FP16'
                             % targets)
    args, _ = parser.parse_known_args()

    if not os.path.isfile(args.model):
        raise OSError("Model not exist")

    query_feat, query_names = extract_feature(args.query_dir, args.model, args.batchsize, args.resize_h, args.resize_w, args.backend, args.target)
    gallery_feat, gallery_names = extract_feature(args.gallery_dir, args.model, args.batchsize, args.resize_h, args.resize_w, args.backend, args.target)

    topk_idx = topk(query_feat, gallery_feat, args.topk)
    visualization(topk_idx, query_names, gallery_names, output_dir = args.visualization_dir)
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

- **similarity()**: A function/method defined in this file
- **preprocess()**: A function/method defined in this file
- **drawRankList()**: A function/method defined in this file
- **addBorder()**: A function/method defined in this file
- **normalize()**: A function/method defined in this file
- **extract_feature()**: A function/method defined in this file
- **read_data()**: A function/method defined in this file
- **run_net()**: A function/method defined in this file
- **topk()**: A function/method defined in this file
- **visualization()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `image`
- `numpy`
- `os.path`
- `cv2`
- `argparse`
- `images`
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

