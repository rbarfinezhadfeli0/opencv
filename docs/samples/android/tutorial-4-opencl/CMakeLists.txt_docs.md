# Documentation for `samples/android/tutorial-4-opencl/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/android/tutorial-4-opencl/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 439 bytes
- **File Type**: .txt
- **Link to Source**: [samples/android/tutorial-4-opencl/CMakeLists.txt](../../../samples/android/tutorial-4-opencl/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/android/tutorial-4-opencl` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(sample example-tutorial-4-opencl)

if(BUILD_FAT_JAVA_LIB)
  set(native_deps opencv_java)
else()
  set(native_deps opencv_imgproc)
endif()

add_android_project(${sample} "${CMAKE_CURRENT_SOURCE_DIR}"
    LIBRARY_DEPS "${OPENCV_ANDROID_LIB_DIR}"
    SDK_TARGET 21 "${ANDROID_SDK_TARGET}"
    NATIVE_DEPS ${native_deps} -lGLESv2 -lEGL
    COPY_LIBS YES
)
if(TARGET ${sample})
  add_dependencies(opencv_android_examples ${sample})
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

