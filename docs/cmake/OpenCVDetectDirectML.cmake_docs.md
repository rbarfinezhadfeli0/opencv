# Documentation for `cmake/OpenCVDetectDirectML.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVDetectDirectML.cmake`
- **File Name**: `OpenCVDetectDirectML.cmake`
- **File Size**: 411 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVDetectDirectML.cmake](../cmake/OpenCVDetectDirectML.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(WIN32)
  try_compile(__VALID_DIRECTML
    "${OpenCV_BINARY_DIR}"
    "${OpenCV_SOURCE_DIR}/cmake/checks/directml.cpp"
    LINK_LIBRARIES d3d12 dxcore directml
    OUTPUT_VARIABLE TRY_OUT
  )
  if(NOT __VALID_DIRECTML)
    message(STATUS "No support for DirectML. d3d12, dxcore, directml libs are required, first bundled with Windows SDK 10.0.19041.0.")
    return()
  endif()
  set(HAVE_DIRECTML ON)
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

