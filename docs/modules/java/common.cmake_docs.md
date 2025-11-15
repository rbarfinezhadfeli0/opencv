# Documentation for `modules/java/common.cmake`

## File Metadata

- **Full Path**: `modules/java/common.cmake`
- **File Name**: `common.cmake`
- **File Size**: 687 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/java/common.cmake](../../modules/java/common.cmake)

## Purpose and Role

This file is located in the `modules/java` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(ANDROID)
  ocv_update(OPENCV_JAVA_LIB_NAME_SUFFIX "${OPENCV_VERSION_MAJOR}")
  ocv_update(JAVA_INSTALL_ROOT "sdk/java")
else()
  ocv_update(OPENCV_JAVA_LIB_NAME_SUFFIX "${OPENCV_VERSION_MAJOR}${OPENCV_VERSION_MINOR}${OPENCV_VERSION_PATCH}")
endif()

if(MSVC)
  ocv_warnings_disable(CMAKE_CXX_FLAGS /wd4996)
else()
  ocv_warnings_disable(CMAKE_CXX_FLAGS -Wdeprecated-declarations)
endif()

# get list of modules to wrap
# message(STATUS "Wrapped in java:")
set(OPENCV_JAVA_MODULES)
foreach(m ${OPENCV_MODULES_BUILD})
  if (";${OPENCV_MODULE_${m}_WRAPPERS};" MATCHES ";java;" AND HAVE_${m})
    list(APPEND OPENCV_JAVA_MODULES ${m})
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

