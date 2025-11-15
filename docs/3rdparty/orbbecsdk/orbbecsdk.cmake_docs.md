# Documentation for `3rdparty/orbbecsdk/orbbecsdk.cmake`

## File Metadata

- **Full Path**: `3rdparty/orbbecsdk/orbbecsdk.cmake`
- **File Name**: `orbbecsdk.cmake`
- **File Size**: 1,536 bytes
- **File Type**: .cmake
- **Link to Source**: [3rdparty/orbbecsdk/orbbecsdk.cmake](../../3rdparty/orbbecsdk/orbbecsdk.cmake)

## Purpose and Role

This file is located in the `3rdparty/orbbecsdk` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
ocv_update(ORBBEC_SDK_VERSION "2")
ocv_update(ORBBEC_SDK_DOWNLOAD_DIR "${OpenCV_BINARY_DIR}/3rdparty/orbbecsdk")

function(download_orbbec_sdk root_var)
    if(ORBBEC_SDK_VERSION STREQUAL "1")
        set(ORBBEC_SDK_FILE_HASH_CMAKE "e7566fa915a1b0c02640df41891916fe")
        set(ORBBEC_SDK_GIT_TAG "1.9.4")
        add_definitions(-DORBBEC_SDK_VERSION_MAJOR=1)
    elseif(ORBBEC_SDK_VERSION STREQUAL "2")
        set(ORBBEC_SDK_FILE_HASH_CMAKE "d828ac15618a56b9ae325bada8676e28")
        set(ORBBEC_SDK_GIT_TAG "2.5.5")
        add_definitions(-DORBBEC_SDK_VERSION_MAJOR=2)
    else()
        message(STATUS "Unsupported OrbbecSDK version: ${ORBBEC_SDK_VERSION}, use default version 2")
        set(ORBBEC_SDK_FILE_HASH_CMAKE "d828ac15618a56b9ae325bada8676e28")
        set(ORBBEC_SDK_GIT_TAG "2.5.5")
        add_definitions(-DORBBEC_SDK_VERSION_MAJOR=2)
    endif()

    ocv_download(FILENAME "v${ORBBEC_SDK_GIT_TAG}.tar.gz"
                HASH ${ORBBEC_SDK_FILE_HASH_CMAKE}
                URL "https://github.com/orbbec/OrbbecSDK/archive/refs/tags/v${ORBBEC_SDK_GIT_TAG}/"
                DESTINATION_DIR ${ORBBEC_SDK_DOWNLOAD_DIR}
                ID OrbbecSDK
                STATUS res
                UNPACK RELATIVE_URL
                )
    if(${res})
        message(STATUS "OrbbecSDK downloaded to: ${ORBBEC_SDK_DOWNLOAD_DIR}")
        set(${root_var} "${ORBBEC_SDK_DOWNLOAD_DIR}/OrbbecSDK-${ORBBEC_SDK_GIT_TAG}" PARENT_SCOPE)
    else()
        message(FATAL_ERROR "Failed to download OrbbecSDK")
    endif()
endfunction()
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

