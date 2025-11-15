# Documentation for `modules/core/cmake/parallel/init.cmake`

## File Metadata

- **Full Path**: `modules/core/cmake/parallel/init.cmake`
- **File Name**: `init.cmake`
- **File Size**: 263 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/core/cmake/parallel/init.cmake](../../../../modules/core/cmake/parallel/init.cmake)

## Purpose and Role

This file is located in the `modules/core/cmake/parallel` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
macro(ocv_add_core_parallel_backend backend_id cond_var)
  if(${cond_var})
    include("${CMAKE_CURRENT_LIST_DIR}/detect_${backend_id}.cmake")
  endif()
endmacro()

ocv_add_core_parallel_backend("tbb" WITH_TBB)
ocv_add_core_parallel_backend("openmp" WITH_OPENMP)

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

