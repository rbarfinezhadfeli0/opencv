# Documentation for `modules/videoio/cmake/detect_msmf.cmake`

## File Metadata

- **Full Path**: `modules/videoio/cmake/detect_msmf.cmake`
- **File Name**: `detect_msmf.cmake`
- **File Size**: 531 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/videoio/cmake/detect_msmf.cmake](../../../modules/videoio/cmake/detect_msmf.cmake)

## Purpose and Role

This file is located in the `modules/videoio/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# --- VideoInput/Microsoft Media Foundation ---
if(NOT HAVE_MSMF)
  check_include_file(mfapi.h HAVE_MFAPI)
  if(HAVE_MFAPI)
    set(HAVE_MSMF TRUE)
  endif()
endif()

if(HAVE_MSMF)
  if(WITH_MSMF_DXVA)
    check_include_file(d3d11.h HAVE_D3D11)
    check_include_file(d3d11_4.h HAVE_D3D11_4)
    if(HAVE_D3D11 AND HAVE_D3D11_4)
      set(HAVE_MSMF_DXVA TRUE)
    endif()
  endif()
  set(defs "HAVE_MSMF")
  if(HAVE_MSMF_DXVA)
    list(APPEND defs "HAVE_MSMF_DXVA")
  endif()
  ocv_add_external_target(msmf "" "" "${defs}")
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

