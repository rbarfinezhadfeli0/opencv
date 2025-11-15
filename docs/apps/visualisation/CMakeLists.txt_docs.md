# Documentation for `apps/visualisation/CMakeLists.txt`

## File Metadata

- **Full Path**: `apps/visualisation/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 162 bytes
- **File Type**: .txt
- **Link to Source**: [apps/visualisation/CMakeLists.txt](../../apps/visualisation/CMakeLists.txt)

## Purpose and Role

This file is located in the `apps/visualisation` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
ocv_add_application(opencv_visualisation
    MODULES opencv_core opencv_highgui opencv_imgproc opencv_videoio opencv_imgcodecs
    SRCS opencv_visualisation.cpp)

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

