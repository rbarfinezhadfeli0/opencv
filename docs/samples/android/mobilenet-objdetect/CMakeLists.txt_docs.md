# Documentation for `samples/android/mobilenet-objdetect/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/android/mobilenet-objdetect/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,269 bytes
- **File Type**: .txt
- **Link to Source**: [samples/android/mobilenet-objdetect/CMakeLists.txt](../../../samples/android/mobilenet-objdetect/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/android/mobilenet-objdetect` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(sample example-mobilenet-objdetect)

ocv_download(FILENAME "mobilenet_iter_73000.caffemodel"
             HASH "bbcb3b6a0afe1ec89e1288096b5b8c66"
             URL
               "${OPENCV_MOBILENET_SSD_WEIGHTS_URL}"
               "$ENV{OPENCV_MOBILENET_SSD_WEIGHTS_URL}"
               "https://raw.githubusercontent.com/chuanqi305/MobileNet-SSD/97406996b1eee2d40eb0a00ae567cf41e23369f9/mobilenet_iter_73000.caffemodel"
             DESTINATION_DIR "${CMAKE_CURRENT_LIST_DIR}/res/raw"
             ID OPENCV_MOBILENET_SSD_WEIGHTS
             STATUS res)

ocv_download(FILENAME "deploy.prototxt"
             HASH "f1978dc4fe20c680e850ce99830c5945"
             URL
               "${OPENCV_MOBILENET_SSD_CONFIG_URL}"
               "$ENV{OPENCV_MOBILENET_SSD_CONFIG_URL}"
               "https://raw.githubusercontent.com/chuanqi305/MobileNet-SSD/97406996b1eee2d40eb0a00ae567cf41e23369f9/deploy.prototxt"
             DESTINATION_DIR "${CMAKE_CURRENT_LIST_DIR}/res/raw"
             ID OPENCV_MOBILENET_SSD_CONFIG
             STATUS res)

add_android_project(${sample} "${CMAKE_CURRENT_SOURCE_DIR}" LIBRARY_DEPS "${OPENCV_ANDROID_LIB_DIR}" SDK_TARGET 11 "${ANDROID_SDK_TARGET}")
if(TARGET ${sample})
  add_dependencies(opencv_android_examples ${sample})
endif()

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

