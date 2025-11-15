# Documentation for `doc/js_tutorials/js_assets/js_image_classification_model_info.json`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_image_classification_model_info.json`
- **File Name**: `js_image_classification_model_info.json`
- **File Size**: 3,258 bytes
- **File Type**: .json
- **Link to Source**: [doc/js_tutorials/js_assets/js_image_classification_model_info.json](../../../doc/js_tutorials/js_assets/js_image_classification_model_info.json)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "caffe": [
        {
            "model": "alexnet",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "needSoftmax": "false",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/classification_classes_ILSVRC2012.txt",
            "modelUrl": "http://dl.caffe.berkeleyvision.org/bvlc_alexnet.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/BVLC/caffe/master/models/bvlc_alexnet/deploy.prototxt"
        },
        {
            "model": "densenet",
            "mean": "127.5, 127.5, 127.5",
            "std": "0.007843",
            "swapRB": "false",
            "needSoftmax": "true",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/classification_classes_ILSVRC2012.txt",
            "modelUrl": "https://drive.google.com/open?id=0B7ubpZO7HnlCcHlfNmJkU2VPelE",
            "configUrl": "https://raw.githubusercontent.com/shicai/DenseNet-Caffe/master/DenseNet_121.prototxt"
        },
        {
            "model": "googlenet",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "needSoftmax": "false",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/classification_classes_ILSVRC2012.txt",
            "modelUrl": "http://dl.caffe.berkeleyvision.org/bvlc_googlenet.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/BVLC/caffe/master/models/bvlc_googlenet/deploy.prototxt"
        },
        {
            "model": "squeezenet",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "needSoftmax": "false",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/classification_classes_ILSVRC2012.txt",
            "modelUrl": "https://raw.githubusercontent.com/forresti/SqueezeNet/master/SqueezeNet_v1.0/squeezenet_v1.0.caffemodel",
            "configUrl": "https://raw.githubusercontent.com/forresti/SqueezeNet/master/SqueezeNet_v1.0/deploy.prototxt"
        },
        {
            "model": "VGG",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "needSoftmax": "false",
            "labelsUrl": "https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/dnn/classification_classes_ILSVRC2012.txt",
            "modelUrl": "http://www.robots.ox.ac.uk/~vgg/software/very_deep/caffe/VGG_ILSVRC_19_layers.caffemodel",
            "configUrl": "https://gist.githubusercontent.com/ksimonyan/3785162f95cd2d5fee77/raw/f02f8769e64494bcd3d7e97d5d747ac275825721/VGG_ILSVRC_19_layers_deploy.prototxt"
        }
    ],
    "tensorflow": [
        {
            "model": "inception",
            "mean": "123, 117, 104",
            "std": "1",
            "swapRB": "true",
            "needSoftmax": "false",
            "labelsUrl": "https://raw.githubusercontent.com/petewarden/tf_ios_makefile_example/master/data/imagenet_comp_graph_label_strings.txt",
            "modelUrl": "https://raw.githubusercontent.com/petewarden/tf_ios_makefile_example/master/data/tensorflow_inception_graph.pb"
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

