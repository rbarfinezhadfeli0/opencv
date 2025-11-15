# Documentation for `docs/samples/semihosting/norm/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/semihosting/norm/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 1,789 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/semihosting/norm/CMakeLists.txt_docs.md](../../../../docs/samples/semihosting/norm/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/semihosting/norm` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/semihosting/norm/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/semihosting/norm/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 788 bytes
- **File Type**: .txt
- **Link to Source**: [samples/semihosting/norm/CMakeLists.txt](../../../samples/semihosting/norm/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/semihosting/norm` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# This file is part of OpenCV project.
# It is subject to the license terms in the LICENSE file found in the top-level directory
# of this distribution and at http://opencv.org/license.html

set(PROJECT_NAME norm)
project(${PROJECT_NAME})

ocv_install_example_src(norm *.cpp *.hpp CMakeLists.txt)

set(LOCAL_DEPS
  opencv_core
  ${OPENCV_MODULES_PUBLIC}
  ${OpenCV_LIB_COMPONENTS})
ocv_check_dependencies(${LOCAL_DEPS})

if(NOT OCV_DEPENDENCIES_FOUND)
  return()
endif()

ocv_define_sample(norm norm.cpp ${SEMIHOSTING_SUFFIX})
ocv_include_modules_recurse(${LOCAL_DEPS})
target_include_directories(${norm} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(${norm} PRIVATE ${RAW_PIXEL_INCLUDE})
ocv_target_link_libraries(${norm} PRIVATE ${OPENCV_LINKER_LIBS}
  ${LOCAL_DEPS})

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

