# Documentation for `docs/samples/android/face-detection/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/face-detection/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 1,786 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/face-detection/CMakeLists.txt_docs.md](../../../../docs/samples/android/face-detection/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/face-detection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/face-detection/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/android/face-detection/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 755 bytes
- **File Type**: .txt
- **Link to Source**: [samples/android/face-detection/CMakeLists.txt](../../../samples/android/face-detection/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/android/face-detection` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(sample example-face-detection)

ocv_download(FILENAME "face_detection_yunet_2023mar.onnx"
             HASH "4ae92eeb150c82ce15ac80738b3b8167"
             URL
               "${OPENCV_FACE_DETECT_YN_URL}"
               "$ENV{OPENCV_FACE_DETECT_YN_URL}"
               "https://media.githubusercontent.com/media/opencv/opencv_zoo/main/models/face_detection_yunet/face_detection_yunet_2023mar.onnx"
             DESTINATION_DIR "${CMAKE_CURRENT_LIST_DIR}/res/raw"
             ID OPENCV_FACE_DETECT_YN
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

