# Documentation for `doc/js_tutorials/js_assets/js_object_detection_model_info.json`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_object_detection_model_info.json`
- **File Name**: `js_object_detection_model_info.json`
- **File Size**: 1,738 bytes
- **File Type**: .json
- **Link to Source**: [doc/js_tutorials/js_assets/js_object_detection_model_info.json](../../../doc/js_tutorials/js_assets/js_object_detection_model_info.json)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "caffe": [
        {
            "model": "mobilenet_SSD",
            "inputSize": "300, 300",
            "mean": "127.5, 127.5, 127.5",
            "std": "0.007843",
            "swapRB": "false",
            "outType": "SSD",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/object_detection_classes_pascal_voc.txt",
            "modelUrl": "https://raw.githubusercontent.com/chuanqi305/MobileNet-SSD/master/mobilenet_iter_73000.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/chuanqi305/MobileNet-SSD/master/deploy.prototxt"
        },
        {
            "model": "VGG_SSD",
            "inputSize": "300, 300",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "outType": "SSD",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/object_detection_classes_pascal_voc.txt",
            "modelUrl": "https://drive.google.com/uc?id=0BzKzrI_SkD1_WVVTSmQxU0dVRzA&export=download",
            "configUrl": "https://drive.google.com/uc?id=0BzKzrI_SkD1_WVVTSmQxU0dVRzA&export=download"
        }
    ],
    "darknet": [
        {
            "model": "yolov2_tiny",
            "inputSize": "416, 416",
            "mean": "0, 0, 0",
            "std": "0.00392",
            "swapRB": "false",
            "outType": "YOLO",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/object_detection_classes_yolov3.txt",
            "modelUrl": "https://pjreddie.com/media/files/yolov2-tiny.weights",
            "configUrl": "https://raw.githubusercontent.com/pjreddie/darknet/master/cfg/yolov2-tiny.cfg"
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

