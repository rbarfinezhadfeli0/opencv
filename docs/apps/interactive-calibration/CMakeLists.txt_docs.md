# Documentation for `apps/interactive-calibration/CMakeLists.txt`

## File Metadata

- **Full Path**: `apps/interactive-calibration/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 220 bytes
- **File Type**: .txt
- **Link to Source**: [apps/interactive-calibration/CMakeLists.txt](../../apps/interactive-calibration/CMakeLists.txt)

## Purpose and Role

This file is located in the `apps/interactive-calibration` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(DEPS opencv_core opencv_imgproc opencv_features2d opencv_highgui opencv_calib3d opencv_videoio opencv_objdetect)
file(GLOB SRCS *.cpp)
ocv_add_application(opencv_interactive-calibration MODULES ${DEPS} SRCS ${SRCS})

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

