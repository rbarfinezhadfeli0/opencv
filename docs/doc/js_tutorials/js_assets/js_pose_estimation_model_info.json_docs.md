# Documentation for `doc/js_tutorials/js_assets/js_pose_estimation_model_info.json`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_pose_estimation_model_info.json`
- **File Name**: `js_pose_estimation_model_info.json`
- **File Size**: 1,463 bytes
- **File Type**: .json
- **Link to Source**: [doc/js_tutorials/js_assets/js_pose_estimation_model_info.json](../../../doc/js_tutorials/js_assets/js_pose_estimation_model_info.json)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "caffe": [
        {
            "model": "body_25",
            "inputSize": "368, 368",
            "mean": "0, 0, 0",
            "std": "0.00392",
            "swapRB": "false",
            "dataset": "BODY_25",
            "modelUrl": "http://posefs1.perception.cs.cmu.edu/OpenPose/models/pose/body_25/pose_iter_584000.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/CMU-Perceptual-Computing-Lab/openpose/master/models/pose/body_25/pose_deploy.prototxt"
        },
        {
            "model": "coco",
            "inputSize": "368, 368",
            "mean": "0, 0, 0",
            "std": "0.00392",
            "swapRB": "false",
            "dataset": "COCO",
            "modelUrl": "http://posefs1.perception.cs.cmu.edu/OpenPose/models/pose/coco/pose_iter_440000.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/CMU-Perceptual-Computing-Lab/openpose/master/models/pose/coco/pose_deploy_linevec.prototxt"
        },
        {
            "model": "mpi",
            "inputSize": "368, 368",
            "mean": "0, 0, 0",
            "std": "0.00392",
            "swapRB": "false",
            "dataset": "MPI",
            "modelUrl": "http://posefs1.perception.cs.cmu.edu/OpenPose/models/pose/mpi/pose_iter_160000.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/CMU-Perceptual-Computing-Lab/openpose/master/models/pose/mpi/pose_deploy_linevec.prototxt"
        }
    ]
}
```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.

