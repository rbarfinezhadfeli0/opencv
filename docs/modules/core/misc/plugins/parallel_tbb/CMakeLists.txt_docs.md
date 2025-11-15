# Documentation for `modules/core/misc/plugins/parallel_tbb/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/core/misc/plugins/parallel_tbb/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 558 bytes
- **File Type**: .txt
- **Link to Source**: [modules/core/misc/plugins/parallel_tbb/CMakeLists.txt](../../../../../modules/core/misc/plugins/parallel_tbb/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/core/misc/plugins/parallel_tbb` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.5)
project(opencv_core_parallel_tbb CXX)

get_filename_component(OpenCV_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../../.." ABSOLUTE)
include("${OpenCV_SOURCE_DIR}/cmake/OpenCVPluginStandalone.cmake")

# scan dependencies
set(WITH_TBB ON)
include("${OpenCV_SOURCE_DIR}/modules/core/cmake/parallel/init.cmake")

message(STATUS "TBB: ver ${TBB_VERSION_MAJOR}.${TBB_VERSION_MINOR} interface ${TBB_INTERFACE_VERSION}")
ocv_create_plugin(core "opencv_core_parallel_tbb" "ocv.3rdparty.tbb" "TBB" "src/parallel/parallel_tbb.cpp")

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

