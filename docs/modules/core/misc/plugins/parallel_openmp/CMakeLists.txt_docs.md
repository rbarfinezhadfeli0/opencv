# Documentation for `modules/core/misc/plugins/parallel_openmp/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/core/misc/plugins/parallel_openmp/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 520 bytes
- **File Type**: .txt
- **Link to Source**: [modules/core/misc/plugins/parallel_openmp/CMakeLists.txt](../../../../../modules/core/misc/plugins/parallel_openmp/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/core/misc/plugins/parallel_openmp` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.5)
project(opencv_core_parallel_openmp CXX)

get_filename_component(OpenCV_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../../.." ABSOLUTE)
include("${OpenCV_SOURCE_DIR}/cmake/OpenCVPluginStandalone.cmake")

# scan dependencies
set(WITH_OPENMP ON)
include("${OpenCV_SOURCE_DIR}/modules/core/cmake/parallel/init.cmake")

message(STATUS "OpenMP: ${OpenMP_CXX_VERSION}")
ocv_create_plugin(core "opencv_core_parallel_openmp" "ocv.3rdparty.openmp" "OPENMP" "src/parallel/parallel_openmp.cpp")

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

