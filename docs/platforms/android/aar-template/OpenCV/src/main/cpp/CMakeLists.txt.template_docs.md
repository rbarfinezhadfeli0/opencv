# Documentation for `platforms/android/aar-template/OpenCV/src/main/cpp/CMakeLists.txt.template`

## File Metadata

- **Full Path**: `platforms/android/aar-template/OpenCV/src/main/cpp/CMakeLists.txt.template`
- **File Name**: `CMakeLists.txt.template`
- **File Size**: 108 bytes
- **File Type**: .template
- **Link to Source**: [platforms/android/aar-template/OpenCV/src/main/cpp/CMakeLists.txt.template](../../../../../../../platforms/android/aar-template/OpenCV/src/main/cpp/CMakeLists.txt.template)

## Purpose and Role

This file is located in the `platforms/android/aar-template/OpenCV/src/main/cpp` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
cmake_minimum_required(VERSION 3.6)

project("opencv")

add_library(${LIB_NAME} ${LIB_TYPE} native-lib.cpp)

```

## General Information

This file is part of the OpenCV repository infrastructure.

