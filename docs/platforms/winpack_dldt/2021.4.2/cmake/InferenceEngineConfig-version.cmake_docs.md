# Documentation for `platforms/winpack_dldt/2021.4.2/cmake/InferenceEngineConfig-version.cmake`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2021.4.2/cmake/InferenceEngineConfig-version.cmake`
- **File Name**: `InferenceEngineConfig-version.cmake`
- **File Size**: 980 bytes
- **File Type**: .cmake
- **Link to Source**: [platforms/winpack_dldt/2021.4.2/cmake/InferenceEngineConfig-version.cmake](../../../../platforms/winpack_dldt/2021.4.2/cmake/InferenceEngineConfig-version.cmake)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2021.4.2/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# Copyright (C) 2018-2021 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
#

set(PACKAGE_VERSION_MAJOR 2021)
set(PACKAGE_VERSION_MINOR 4)
set(PACKAGE_VERSION_PATCH 2)
set(PACKAGE_VERSION "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}.${PACKAGE_VERSION_PATCH}")

set(PACKAGE_VERSION_EXACT False)
set(PACKAGE_VERSION_COMPATIBLE False)

# Compatibility with old versioning for 2.x
if(PACKAGE_FIND_VERSION_MAJOR VERSION_EQUAL 2)
    set(PACKAGE_VERSION_COMPATIBLE True)
    if(${CMAKE_FIND_PACKAGE_NAME}_FIND_REQUIRED)
        message(WARNING "Inference Engine versioning has changed. Use ${PACKAGE_VERSION} instead of ${PACKAGE_FIND_VERSION}")
    endif()
endif()

if(PACKAGE_FIND_VERSION VERSION_EQUAL PACKAGE_VERSION)
    set(PACKAGE_VERSION_EXACT True)
    set(PACKAGE_VERSION_COMPATIBLE True)
endif()

if(PACKAGE_FIND_VERSION_MAJOR EQUAL PACKAGE_VERSION_MAJOR AND
   PACKAGE_FIND_VERSION VERSION_LESS PACKAGE_VERSION)
    set(PACKAGE_VERSION_COMPATIBLE True)
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

