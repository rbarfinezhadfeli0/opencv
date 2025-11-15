# Documentation for `modules/gapi/cmake/init.cmake`

## File Metadata

- **Full Path**: `modules/gapi/cmake/init.cmake`
- **File Name**: `init.cmake`
- **File Size**: 1,095 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/gapi/cmake/init.cmake](../../../modules/gapi/cmake/init.cmake)

## Purpose and Role

This file is located in the `modules/gapi/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
OCV_OPTION(WITH_ADE "Enable ADE framework (required for Graph API module)" ON)

OCV_OPTION(WITH_FREETYPE "Enable FreeType framework"     OFF)
OCV_OPTION(WITH_PLAIDML  "Include PlaidML2 support"      OFF)
OCV_OPTION(WITH_OAK      "Include OpenCV AI Kit support" OFF)

if(NOT WITH_ADE)
  return()
endif()

if(ade_DIR)
  # if ade_DIR is set, use ADE-supplied CMake script
  # to set up variables to the prebuilt ADE
  find_package(ade 0.1.0)
endif()

if(NOT TARGET ade)
  # if ade_DIR is not set, try to use automatically
  # downloaded one (if there any)
  include("${CMAKE_CURRENT_LIST_DIR}/DownloadADE.cmake")
endif()

if(WITH_FREETYPE)
  ocv_check_modules(FREETYPE freetype2)
  if (FREETYPE_FOUND)
      set(HAVE_FREETYPE TRUE)
  endif()
endif()

if(WITH_PLAIDML)
  find_package(PlaidML2 CONFIG QUIET)
  if (PLAIDML_FOUND)
      set(HAVE_PLAIDML TRUE)
  endif()
endif()

if(WITH_GAPI_ONEVPL)
    find_package(VPL)
    if(VPL_FOUND)
        set(HAVE_GAPI_ONEVPL TRUE)
    endif()
endif()

if(WITH_OAK)
  find_package(depthai QUIET)
  if(depthai_FOUND)
      set(HAVE_OAK TRUE)
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

