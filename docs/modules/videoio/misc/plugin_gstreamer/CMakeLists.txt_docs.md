# Documentation for `modules/videoio/misc/plugin_gstreamer/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_gstreamer/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 524 bytes
- **File Type**: .txt
- **Link to Source**: [modules/videoio/misc/plugin_gstreamer/CMakeLists.txt](../../../../modules/videoio/misc/plugin_gstreamer/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_gstreamer` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.5)

get_filename_component(OpenCV_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../.." ABSOLUTE)
include("${OpenCV_SOURCE_DIR}/cmake/OpenCVPluginStandalone.cmake")

# scan dependencies
set(WITH_GSTREAMER ON)
include("${OpenCV_SOURCE_DIR}/modules/videoio/cmake/init.cmake")

set(OPENCV_PLUGIN_DEPS core imgproc imgcodecs)
ocv_create_plugin(videoio "opencv_videoio_gstreamer" "ocv.3rdparty.gstreamer" "GStreamer" "src/cap_gstreamer.cpp")

message(STATUS "Using GStreamer: ${GSTREAMER_VERSION}")

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

