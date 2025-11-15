# Documentation for `docs/samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 1,420 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt_docs.md](../../../../../../docs/samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/gpu/gpu-thrust-interop` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 293 bytes
- **File Type**: .txt
- **Link to Source**: [samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt](../../../../../samples/cpp/tutorial_code/gpu/gpu-thrust-interop/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/gpu/gpu-thrust-interop` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
CMAKE_MINIMUM_REQUIRED(VERSION 3.5)

FIND_PACKAGE(CUDA REQUIRED)
INCLUDE_DIRECTORIES(${CUDA_INCLUDE_DIRS})

FIND_PACKAGE(OpenCV REQUIRED COMPONENTS core)
INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIRS})

CUDA_ADD_EXECUTABLE(opencv_thrust main.cu)
TARGET_LINK_LIBRARIES(opencv_thrust ${OpenCV_LIBS})
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

