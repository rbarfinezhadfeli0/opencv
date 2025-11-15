# Documentation for `docs/samples/android/video-recorder/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/video-recorder/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 1,414 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/video-recorder/CMakeLists.txt_docs.md](../../../../docs/samples/android/video-recorder/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/video-recorder` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/video-recorder/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/android/video-recorder/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 383 bytes
- **File Type**: .txt
- **Link to Source**: [samples/android/video-recorder/CMakeLists.txt](../../../samples/android/video-recorder/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/android/video-recorder` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(sample example-video-recorder)

if(BUILD_FAT_JAVA_LIB)
  set(native_deps opencv_java)
else()
  set(native_deps videoio)
endif()

add_android_project(${sample} "${CMAKE_CURRENT_SOURCE_DIR}" LIBRARY_DEPS "${OPENCV_ANDROID_LIB_DIR}" SDK_TARGET 11 "${ANDROID_SDK_TARGET}" NATIVE_DEPS ${native_deps})
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

