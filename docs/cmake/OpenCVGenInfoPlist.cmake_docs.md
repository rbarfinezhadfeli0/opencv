# Documentation for `cmake/OpenCVGenInfoPlist.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVGenInfoPlist.cmake`
- **File Name**: `OpenCVGenInfoPlist.cmake`
- **File Size**: 1,136 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVGenInfoPlist.cmake](../cmake/OpenCVGenInfoPlist.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(OPENCV_APPLE_BUNDLE_NAME "OpenCV")
set(OPENCV_APPLE_BUNDLE_ID "org.opencv")

if(IOS)
  if(MAC_CATALYST)
    # Copy the iOS plist over to the OSX directory if building iOS library for Catalyst
    configure_file("${OpenCV_SOURCE_DIR}/platforms/ios/Info.plist.in"
                  "${CMAKE_BINARY_DIR}/osx/Info.plist")
  elseif(APPLE_FRAMEWORK AND DYNAMIC_PLIST)
    configure_file("${OpenCV_SOURCE_DIR}/platforms/ios/Info.Dynamic.plist.in"
                   "${CMAKE_BINARY_DIR}/ios/Info.plist")
  else()
    configure_file("${OpenCV_SOURCE_DIR}/platforms/ios/Info.plist.in"
                   "${CMAKE_BINARY_DIR}/ios/Info.plist")
  endif()
elseif(XROS)
  if(APPLE_FRAMEWORK AND DYNAMIC_PLIST)
    configure_file("${OpenCV_SOURCE_DIR}/platforms/ios/Info.Dynamic.plist.in"
                   "${CMAKE_BINARY_DIR}/visionos/Info.plist")
  else()
    configure_file("${OpenCV_SOURCE_DIR}/platforms/ios/Info.plist.in"
                   "${CMAKE_BINARY_DIR}/visionos/Info.plist")
  endif()
elseif(APPLE)
  configure_file("${OpenCV_SOURCE_DIR}/platforms/osx/Info.plist.in"
                 "${CMAKE_BINARY_DIR}/osx/Info.plist")
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

