# Documentation for `modules/videoio/cmake/detect_ios.cmake`

## File Metadata

- **Full Path**: `modules/videoio/cmake/detect_ios.cmake`
- **File Name**: `detect_ios.cmake`
- **File Size**: 346 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/videoio/cmake/detect_ios.cmake](../../../modules/videoio/cmake/detect_ios.cmake)

## Purpose and Role

This file is located in the `modules/videoio/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(APPLE AND IOS)
  set(HAVE_CAP_IOS TRUE)
  set(libs
    "-framework Accelerate"
    "-framework AVFoundation"
    "-framework CoreGraphics"
    "-framework CoreImage"
    "-framework CoreMedia"
    "-framework CoreVideo"
    "-framework QuartzCore"
    "-framework UIKit")
  ocv_add_external_target(cap_ios "" "${libs}" "HAVE_CAP_IOS")
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

