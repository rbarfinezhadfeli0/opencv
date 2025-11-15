# Documentation for `modules/js/common.cmake`

## File Metadata

- **Full Path**: `modules/js/common.cmake`
- **File Name**: `common.cmake`
- **File Size**: 366 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/js/common.cmake](../../modules/js/common.cmake)

## Purpose and Role

This file is located in the `modules/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# get list of modules to wrap
if(HAVE_opencv_js)
  message(STATUS "Wrapped in JavaScript(js):")
endif()
set(OPENCV_JS_MODULES "")
foreach(m ${OPENCV_MODULES_BUILD})
  if(";${OPENCV_MODULE_${m}_WRAPPERS};" MATCHES ";js;" AND HAVE_${m})
    list(APPEND OPENCV_JS_MODULES ${m})
    if(HAVE_opencv_js)
      message(STATUS "    ${m}")
    endif()
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

