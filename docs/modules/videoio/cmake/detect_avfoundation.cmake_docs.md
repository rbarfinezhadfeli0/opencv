# Documentation for `modules/videoio/cmake/detect_avfoundation.cmake`

## File Metadata

- **Full Path**: `modules/videoio/cmake/detect_avfoundation.cmake`
- **File Name**: `detect_avfoundation.cmake`
- **File Size**: 435 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/videoio/cmake/detect_avfoundation.cmake](../../../modules/videoio/cmake/detect_avfoundation.cmake)

## Purpose and Role

This file is located in the `modules/videoio/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(APPLE)
  set(HAVE_AVFOUNDATION TRUE)
  if(IOS)
    set(libs "-framework AVFoundation" "-framework QuartzCore")
  else()
    set(libs
      "-framework Cocoa"
      "-framework Accelerate"
      "-framework AVFoundation"
      "-framework CoreGraphics"
      "-framework CoreMedia"
      "-framework CoreVideo"
      "-framework QuartzCore")
  endif()
  ocv_add_external_target(avfoundation "" "${libs}" "HAVE_AVFOUNDATION")
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

