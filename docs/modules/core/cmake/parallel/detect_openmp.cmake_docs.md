# Documentation for `modules/core/cmake/parallel/detect_openmp.cmake`

## File Metadata

- **Full Path**: `modules/core/cmake/parallel/detect_openmp.cmake`
- **File Name**: `detect_openmp.cmake`
- **File Size**: 393 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/core/cmake/parallel/detect_openmp.cmake](../../../../modules/core/cmake/parallel/detect_openmp.cmake)

## Purpose and Role

This file is located in the `modules/core/cmake/parallel` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(CMAKE_VERSION VERSION_LESS "3.9")
  message(STATUS "OpenMP detection requires CMake 3.9+")  # OpenMP::OpenMP_CXX target
endif()

find_package(OpenMP)
if(OpenMP_FOUND)
  if(TARGET OpenMP::OpenMP_CXX)
    set(HAVE_OPENMP 1)
    ocv_add_external_target(openmp "" "OpenMP::OpenMP_CXX" "HAVE_OPENMP=1")
  else()
    message(WARNING "OpenMP: missing OpenMP::OpenMP_CXX target")
  endif()
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

