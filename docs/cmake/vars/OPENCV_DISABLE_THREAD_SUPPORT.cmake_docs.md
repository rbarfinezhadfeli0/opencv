# Documentation for `cmake/vars/OPENCV_DISABLE_THREAD_SUPPORT.cmake`

## File Metadata

- **Full Path**: `cmake/vars/OPENCV_DISABLE_THREAD_SUPPORT.cmake`
- **File Name**: `OPENCV_DISABLE_THREAD_SUPPORT.cmake`
- **File Size**: 804 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/vars/OPENCV_DISABLE_THREAD_SUPPORT.cmake](../../cmake/vars/OPENCV_DISABLE_THREAD_SUPPORT.cmake)

## Purpose and Role

This file is located in the `cmake/vars` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# Force removal of code conditionally compiled with `#if
# HAVE_PTHREAD`.
ocv_update(HAVE_PTHREAD 0)

# There components are disabled because they require
# multi-threaded execution.
ocv_update(WITH_PROTOBUF OFF)
ocv_update(WITH_GSTREAMER OFF)
ocv_update(WITH_IPP OFF)
ocv_update(WITH_ITT OFF)
ocv_update(WITH_OPENCL OFF)
ocv_update(WITH_VA OFF)
ocv_update(WITH_VA_INTEL OFF)

# Disable bindings
ocv_update(BUILD_opencv_python2 OFF)
ocv_update(BUILD_opencv_python3 OFF)
ocv_update(BUILD_JAVA OFF)
ocv_update(BUILD_opencv_java OFF)

# These modules require `#include
# <[thread|mutex|condition_variable|future]>` and linkage into
# `libpthread` to work.
ocv_update(BUILD_opencv_objdetect OFF)
ocv_update(BUILD_opencv_gapi OFF)
ocv_update(BUILD_opencv_dnn OFF)

set(OPJ_USE_THREAD "OFF" CACHE INTERNAL "")

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

