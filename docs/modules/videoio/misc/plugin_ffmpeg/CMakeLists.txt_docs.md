# Documentation for `modules/videoio/misc/plugin_ffmpeg/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_ffmpeg/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 906 bytes
- **File Type**: .txt
- **Link to Source**: [modules/videoio/misc/plugin_ffmpeg/CMakeLists.txt](../../../../modules/videoio/misc/plugin_ffmpeg/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_ffmpeg` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.5)

get_filename_component(OpenCV_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../.." ABSOLUTE)
include("${OpenCV_SOURCE_DIR}/cmake/OpenCVPluginStandalone.cmake")

# scan dependencies
set(WITH_FFMPEG ON)
set(OPENCV_FFMPEG_SKIP_BUILD_CHECK ON)
include("${OpenCV_SOURCE_DIR}/modules/videoio/cmake/init.cmake")

set(OPENCV_PLUGIN_DEPS core imgproc imgcodecs)
ocv_create_plugin(videoio "opencv_videoio_ffmpeg" "ocv.3rdparty.ffmpeg" "FFmpeg" "src/cap_ffmpeg.cpp")

message(STATUS "FFMPEG_libavcodec_VERSION=${FFMPEG_libavcodec_VERSION}")
message(STATUS "FFMPEG_libavformat_VERSION=${FFMPEG_libavformat_VERSION}")
message(STATUS "FFMPEG_libavutil_VERSION=${FFMPEG_libavutil_VERSION}")
message(STATUS "FFMPEG_libswscale_VERSION=${FFMPEG_libswscale_VERSION}")
if(OPENCV_FFMPEG_ENABLE_LIBAVDEVICE)
  message(STATUS "FFMPEG_libavdevice_VERSION=${FFMPEG_libavdevice_VERSION}")
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

