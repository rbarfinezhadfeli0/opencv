# Documentation for `modules/objc/common.cmake`

## File Metadata

- **Full Path**: `modules/objc/common.cmake`
- **File Name**: `common.cmake`
- **File Size**: 367 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/objc/common.cmake](../../modules/objc/common.cmake)

## Purpose and Role

This file is located in the `modules/objc` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
ocv_warnings_disable(CMAKE_CXX_FLAGS -Wdeprecated-declarations)

# get list of modules to wrap
# message(STATUS "Wrapped in Objective-C:")
set(OPENCV_OBJC_MODULES)
foreach(m ${OPENCV_MODULES_BUILD})
  if (";${OPENCV_MODULE_${m}_WRAPPERS};" MATCHES ";objc;" AND HAVE_${m})
    list(APPEND OPENCV_OBJC_MODULES ${m})
    #message(STATUS "\t${m}")
  endif()
endforeach()

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

