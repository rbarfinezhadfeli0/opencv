# Documentation for `docs/cmake/templates/OpenCVConfig-CUDALanguage.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/OpenCVConfig-CUDALanguage.cmake.in_docs.md`
- **File Name**: `OpenCVConfig-CUDALanguage.cmake.in_docs.md`
- **File Size**: 2,005 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/OpenCVConfig-CUDALanguage.cmake.in_docs.md](../../../docs/cmake/templates/OpenCVConfig-CUDALanguage.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/OpenCVConfig-CUDALanguage.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-CUDALanguage.cmake.in`
- **File Name**: `OpenCVConfig-CUDALanguage.cmake.in`
- **File Size**: 1,359 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-CUDALanguage.cmake.in](../../cmake/templates/OpenCVConfig-CUDALanguage.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Version Compute Capability from which OpenCV has been compiled is remembered
set(OpenCV_COMPUTE_CAPABILITIES "@OpenCV_CUDA_CC@")

set(OpenCV_CUDA_VERSION "@CUDA_VERSION_STRING@")
set(OpenCV_USE_CUBLAS   "@HAVE_CUBLAS@")
set(OpenCV_USE_CUFFT    "@HAVE_CUFFT@")
set(OpenCV_USE_NVCUVID  "@HAVE_NVCUVID@")
set(OpenCV_USE_NVCUVENC "@HAVE_NVCUVENC@")
set(OpenCV_CUDNN_VERSION    "@CUDNN_VERSION@")
set(OpenCV_USE_CUDNN        "@HAVE_CUDNN@")
set(ENABLE_CUDA_FIRST_CLASS_LANGUAGE  ON)

if(NOT CUDAToolkit_FOUND)
  if(NOT CMAKE_VERSION VERSION_LESS 3.18)
    if(UNIX AND NOT CMAKE_CUDA_COMPILER AND NOT CUDAToolkit_ROOT)
      message(STATUS "Checking for CUDAToolkit in default location (/usr/local/cuda)")
      set(CUDA_PATH "/usr/local/cuda" CACHE INTERNAL "")
      set(ENV{CUDA_PATH} ${CUDA_PATH})
    endif()
    find_package(CUDAToolkit ${OpenCV_CUDA_VERSION} EXACT REQUIRED)
  else()
    message(FATAL_ERROR "Using OpenCV compiled with CUDA as first class language requires CMake \>= 3.18.")
  endif()
else()
  if(CUDAToolkit_FOUND)
    set(CUDA_VERSION_STRING ${CUDAToolkit_VERSION})
  endif()
  if(NOT CUDA_VERSION_STRING VERSION_EQUAL OpenCV_CUDA_VERSION)
      message(FATAL_ERROR "OpenCV library was compiled with CUDA ${OpenCV_CUDA_VERSION} support. Please, use the same version or rebuild OpenCV with CUDA ${CUDA_VERSION_STRING}")
  endif()
endif()

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

