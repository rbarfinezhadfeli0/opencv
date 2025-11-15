# Documentation for `modules/python/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/python/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,018 bytes
- **File Type**: .txt
- **Link to Source**: [modules/python/CMakeLists.txt](../../modules/python/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/python` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# ----------------------------------------------------------------------------
#  CMake file for python support
# ----------------------------------------------------------------------------
if(DEFINED OPENCV_INITIAL_PASS)  # OpenCV build

if(ANDROID OR APPLE_FRAMEWORK OR WINRT)
  ocv_module_disable_(python2)
  ocv_module_disable_(python3)
  return()
elseif(BUILD_opencv_world OR (WIN32 AND CMAKE_BUILD_TYPE STREQUAL "Debug"))
  if(NOT DEFINED BUILD_opencv_python2)
    set(__disable_python2 ON)
  endif()
  if(NOT DEFINED BUILD_opencv_python3)
    set(__disable_python3 ON)
  endif()
endif()

add_subdirectory(bindings)

add_subdirectory(test)

if(__disable_python2)
  ocv_module_disable_(python2)
endif()
if(__disable_python3)
  ocv_module_disable_(python3)
endif()
if(__disable_python2 AND __disable_python3)
  return()
endif()

add_subdirectory(python2)
add_subdirectory(python3)

else()  # standalone build

cmake_minimum_required(VERSION 3.5)
project(OpenCVPython CXX C)
include("./standalone.cmake")

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

