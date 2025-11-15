# Documentation for `modules/videoio/cmake/detect_ueye.cmake`

## File Metadata

- **Full Path**: `modules/videoio/cmake/detect_ueye.cmake`
- **File Name**: `detect_ueye.cmake`
- **File Size**: 657 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/videoio/cmake/detect_ueye.cmake](../../../modules/videoio/cmake/detect_ueye.cmake)

## Purpose and Role

This file is located in the `modules/videoio/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(NOT HAVE_UEYE)
  if(WIN32)
    if(X86_64)
      set(_WIN_LIB_SUFFIX "_64")
    endif()
  endif()
  find_path(UEYE_INCLUDE "ueye.h"
    PATHS "${UEYE_ROOT}" ENV UEYE_ROOT "/usr" "C:/Program Files/IDS/uEye/Develop"
    HINTS "${regpath}"
    PATH_SUFFIXES "include")
  find_library(UEYE_LIBRARY ueye_api${_WIN_LIB_SUFFIX}
    PATHS "${UEYE_ROOT}" ENV UEYE_ROOT "/usr" "C:/Program Files/IDS/uEye/Develop"
    HINTS "${regpath}"
    PATH_SUFFIXES "lib")
  if(UEYE_INCLUDE AND UEYE_LIBRARY)
    set(HAVE_UEYE TRUE)
  endif()
endif()
unset(_WIN_LIB_SUFFIX)

if(HAVE_UEYE)
  ocv_add_external_target(ueye "${UEYE_INCLUDE}" "${UEYE_LIBRARY}" "HAVE_UEYE")
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

