# Documentation for `docs/samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 2,272 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt_docs.md](../../../../../../docs/samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/core/parallel_backend` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,148 bytes
- **File Type**: .txt
- **Link to Source**: [samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt](../../../../../samples/cpp/tutorial_code/core/parallel_backend/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/core/parallel_backend` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
cmake_minimum_required(VERSION 3.9)

find_package(OpenCV REQUIRED COMPONENTS opencv_core)

if(NOT OPENCV_EXAMPLES_SKIP_PARALLEL_BACKEND_OPENMP
    AND NOT OPENCV_EXAMPLES_SKIP_OPENMP
)
  project(opencv_example_openmp_backend)
  find_package(OpenMP)
  if(OpenMP_FOUND)
    add_executable(opencv_example_openmp_backend example-openmp.cpp)
    target_link_libraries(opencv_example_openmp_backend PRIVATE
        opencv_core
        OpenMP::OpenMP_CXX
    )
  endif()
endif()

if(NOT OPENCV_EXAMPLES_SKIP_PARALLEL_BACKEND_TBB
    AND NOT OPENCV_EXAMPLES_SKIP_TBB
    AND NOT OPENCV_EXAMPLE_SKIP_TBB  # deprecated (to be removed in OpenCV 5.0)
)
  project(opencv_example_tbb_backend)
  find_package(TBB QUIET)
  if(NOT TBB_FOUND)
    find_path(TBB_INCLUDE_DIR NAMES "tbb/tbb.h")
    find_library(TBB_LIBRARY NAMES "tbb")
  endif()
  if(TBB_INCLUDE_DIR AND TBB_LIBRARY)
    add_executable(opencv_example_tbb_backend example-tbb.cpp)
    target_include_directories(opencv_example_tbb_backend SYSTEM PRIVATE ${TBB_INCLUDE_DIR})
    target_link_libraries(opencv_example_tbb_backend PRIVATE
        opencv_core
        ${TBB_LIBRARY}
    )
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

