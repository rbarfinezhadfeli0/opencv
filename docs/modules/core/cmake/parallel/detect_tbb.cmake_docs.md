# Documentation for `modules/core/cmake/parallel/detect_tbb.cmake`

## File Metadata

- **Full Path**: `modules/core/cmake/parallel/detect_tbb.cmake`
- **File Name**: `detect_tbb.cmake`
- **File Size**: 135 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/core/cmake/parallel/detect_tbb.cmake](../../../../modules/core/cmake/parallel/detect_tbb.cmake)

## Purpose and Role

This file is located in the `modules/core/cmake/parallel` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
include("${OpenCV_SOURCE_DIR}/cmake/OpenCVDetectTBB.cmake")

if(HAVE_TBB)
  ocv_add_external_target(tbb "" "tbb" "HAVE_TBB=1")
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

