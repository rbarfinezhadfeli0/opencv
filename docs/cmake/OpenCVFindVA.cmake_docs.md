# Documentation for `cmake/OpenCVFindVA.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVFindVA.cmake`
- **File Name**: `OpenCVFindVA.cmake`
- **File Size**: 511 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVFindVA.cmake](../cmake/OpenCVFindVA.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# Output:
#   HAVE_VA - libva is available
#   HAVE_VA_INTEL - OpenCL/libva Intel interoperability extension is available

find_path(
    VA_INCLUDE_DIR
    NAMES va/va.h
    PATHS ${VA_ROOT_DIR}
    PATH_SUFFIXES include
    DOC "Path to libva headers"
)

if(VA_INCLUDE_DIR)
    set(HAVE_VA TRUE)
    if(NOT DEFINED VA_LIBRARIES AND NOT OPENCV_LIBVA_LINK)
      set(VA_LIBRARIES "va" "va-drm")
    endif()
else()
    set(HAVE_VA FALSE)
    message(STATUS "libva: missing va.h header (VA_INCLUDE_DIR)")
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

