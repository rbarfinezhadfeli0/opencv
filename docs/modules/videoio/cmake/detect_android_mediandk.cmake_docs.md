# Documentation for `modules/videoio/cmake/detect_android_mediandk.cmake`

## File Metadata

- **Full Path**: `modules/videoio/cmake/detect_android_mediandk.cmake`
- **File Name**: `detect_android_mediandk.cmake`
- **File Size**: 328 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/videoio/cmake/detect_android_mediandk.cmake](../../../modules/videoio/cmake/detect_android_mediandk.cmake)

## Purpose and Role

This file is located in the `modules/videoio/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# if(ANDROID AND ANDROID_NATIVE_API_LEVEL GREATER_EQUAL 21)  <-- would be nicer but requires CMake 3.7 or later
if(ANDROID AND ANDROID_NATIVE_API_LEVEL GREATER 20)
  set(HAVE_ANDROID_MEDIANDK TRUE)
  set(libs "-landroid -llog -lmediandk")
  ocv_add_external_target(android_mediandk "" "${libs}" "HAVE_ANDROID_MEDIANDK")
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

