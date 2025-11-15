# Documentation for `docs/doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown_docs.md`
- **File Name**: `android_dnn_intro.markdown_docs.md`
- **File Size**: 4,818 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown_docs.md](../../../../../docs/doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/introduction/android_binary_package` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown`
- **File Name**: `android_dnn_intro.markdown`
- **File Size**: 4,030 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown](../../../../doc/tutorials/introduction/android_binary_package/android_dnn_intro.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/introduction/android_binary_package` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# How to run deep networks on Android device {#tutorial_android_dnn_intro}

@tableofcontents

@prev_tutorial{tutorial_dev_with_OCV_on_Android}
@next_tutorial{tutorial_android_ocl_intro}

@see @ref tutorial_table_of_content_dnn

|    |    |
| -: | :- |
| Original author | Dmitry Kurtaev |
| Compatibility | OpenCV >= 4.9 |

@tableofcontents

## Introduction
In this tutorial you'll know how to run deep learning networks on Android device
using OpenCV deep learning module.
Tutorial was written for Android Studio 2022.2.1.

## Requirements

- Download and install Android Studio from https://developer.android.com/studio.

- Get the latest pre-built OpenCV for Android release from https://github.com/opencv/opencv/releases
and unpack it (for example, `opencv-4.X.Y-android-sdk.zip`, minimum version 4.9 is required).

- Download MobileNet object detection model from https://github.com/chuanqi305/MobileNet-SSD.
Configuration file `MobileNetSSD_deploy.prototxt` and model weights `MobileNetSSD_deploy.caffemodel`
are required.

## Create an empty Android Studio project and add OpenCV dependency

Use @ref tutorial_dev_with_OCV_on_Android tutorial to initialize your project and add OpenCV.

## Make an app

Our sample will takes pictures from a camera, forwards it into a deep network and
receives a set of rectangles, class identifiers and confidence values in range [0, 1].

- First of all, we need to add a necessary widget which displays processed
frames. Modify `app/src/main/res/layout/activity_main.xml`:
@include android/mobilenet-objdetect/res/layout/activity_main.xml

- Modify `/app/src/main/AndroidManifest.xml` to enable full-screen mode, set up
a correct screen orientation and allow to use a camera.
@code{.xml}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:label="@string/app_name">
@endcode
@snippet android/mobilenet-objdetect/gradle/AndroidManifest.xml mobilenet_tutorial

- Replace content of `app/src/main/java/com/example/myapplication/MainActivity.java` and set a custom package name if necessary:

@snippet android/mobilenet-objdetect/src/org/opencv/samples/opencv_mobilenet/MainActivity.java mobilenet_tutorial_package
@snippet android/mobilenet-objdetect/src/org/opencv/samples/opencv_mobilenet/MainActivity.java mobilenet_tutorial

- Put downloaded `deploy.prototxt` and `mobilenet_iter_73000.caffemodel`
into `app/src/main/res/raw` folder. OpenCV DNN model is mainly designed to load ML and DNN models
from file. Modern Android does not allow it without extra permissions, but provides Java API to load
bytes from resources. The sample uses alternative DNN API that initializes a model from in-memory
buffer rather than a file. The following function reads model file from resources and converts it to
`MatOfBytes` (analog of `std::vector<char>` in C++ world) object suitable for OpenCV Java API:

@snippet android/mobilenet-objdetect/src/org/opencv/samples/opencv_mobilenet/MainActivity.java mobilenet_tutorial_resource

And then the network initialization is done with the following lines:

@snippet android/mobilenet-objdetect/src/org/opencv/samples/opencv_mobilenet/MainActivity.java init_model_from_memory

See also [Android documentation on resources](https://developer.android.com/guide/topics/resources/providing-resources.html)

- Take a look how DNN model input is prepared and inference result is interpreted:

@snippet android/mobilenet-objdetect/src/org/opencv/samples/opencv_mobilenet/MainActivity.java mobilenet_handle_frame

`Dnn.blobFromImage` converts camera frame to neural network input tensor. Resize and statistical
normalization are applied. Each line of network output tensor contains information on one detected
object in the following order: confidence in range [0, 1], class id, left, top, right, bottom box
coordinates. All coordinates are in range [0, 1] and should be scaled to image size before rendering.

- Launch an application and make a fun!
![](images/11_demo.jpg)

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

