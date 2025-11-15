# Documentation for `docs/samples/dnn/face_detector/solver.prototxt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/dnn/face_detector/solver.prototxt_docs.md`
- **File Name**: `solver.prototxt_docs.md`
- **File Size**: 1,073 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/dnn/face_detector/solver.prototxt_docs.md](../../../../docs/samples/dnn/face_detector/solver.prototxt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/dnn/face_detector` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/dnn/face_detector/solver.prototxt`

## File Metadata

- **Full Path**: `samples/dnn/face_detector/solver.prototxt`
- **File Name**: `solver.prototxt`
- **File Size**: 465 bytes
- **File Type**: .prototxt
- **Link to Source**: [samples/dnn/face_detector/solver.prototxt](../../../samples/dnn/face_detector/solver.prototxt)

## Purpose and Role

This file is located in the `samples/dnn/face_detector` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
train_net: "train.prototxt"
test_net: "test.prototxt"

test_iter: 2312
test_interval: 5000
test_initialization: true

base_lr: 0.01
display: 10
lr_policy: "multistep"
max_iter: 140000
stepvalue: 80000
stepvalue: 120000
gamma: 0.1
momentum: 0.9
weight_decay: 0.0005
average_loss: 500
iter_size: 1
type: "SGD"

solver_mode: GPU
random_seed: 0
debug_info: false
snapshot: 1000
snapshot_prefix: "snapshot/res10_300x300_ssd"

eval_type: "detection"
ap_version: "11point"
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

