# Documentation for `docs/samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 2,161 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt_docs.md](../../../../../docs/samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/tutorial-2-mixedprocessing/jni` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,045 bytes
- **File Type**: .txt
- **Link to Source**: [samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt](../../../../samples/android/tutorial-2-mixedprocessing/jni/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/android/tutorial-2-mixedprocessing/jni` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.6)

set(target mixed_sample)
project(${target} CXX)

if (OPENCV_FROM_SDK)
  message(STATUS "Using OpenCV from local SDK")
  set(ANDROID_OPENCV_COMPONENTS "opencv_java" CACHE STRING "")
else()
  message(STATUS "Using OpenCV from AAR (Maven repo)")
  set(ANDROID_OPENCV_COMPONENTS "OpenCV::opencv_java${OPENCV_VERSION_MAJOR}" CACHE STRING "")
endif()

message(STATUS "ANDROID_ABI=${ANDROID_ABI}")
find_package(OpenCV REQUIRED COMPONENTS ${ANDROID_OPENCV_COMPONENTS})

# For 16k pages support with NDK prior 27
# Details: https://developer.android.com/guide/practices/page-sizes?hl=en
if(ANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES)
  if(ANDROID_ABI STREQUAL arm64-v8a OR ANDROID_ABI STREQUAL x86_64)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,max-page-size=16384")
  endif()
endif()

file(GLOB srcs *.cpp *.c)
file(GLOB hdrs *.hpp *.h)

include_directories("${CMAKE_CURRENT_LIST_DIR}")
add_library(${target} SHARED ${srcs} ${hdrs})
target_link_libraries(${target} ${ANDROID_OPENCV_COMPONENTS})

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

