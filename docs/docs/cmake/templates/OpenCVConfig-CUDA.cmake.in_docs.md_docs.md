# Documentation for `docs/cmake/templates/OpenCVConfig-CUDA.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/OpenCVConfig-CUDA.cmake.in_docs.md`
- **File Name**: `OpenCVConfig-CUDA.cmake.in_docs.md`
- **File Size**: 2,262 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/OpenCVConfig-CUDA.cmake.in_docs.md](../../../docs/cmake/templates/OpenCVConfig-CUDA.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/OpenCVConfig-CUDA.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-CUDA.cmake.in`
- **File Name**: `OpenCVConfig-CUDA.cmake.in`
- **File Size**: 1,656 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-CUDA.cmake.in](../../cmake/templates/OpenCVConfig-CUDA.cmake.in)

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

if(NOT CUDA_FOUND)
  find_host_package(CUDA ${OpenCV_CUDA_VERSION} EXACT REQUIRED)
else()
  if(NOT CUDA_VERSION_STRING VERSION_EQUAL OpenCV_CUDA_VERSION)
    message(FATAL_ERROR "OpenCV static library was compiled with CUDA ${OpenCV_CUDA_VERSION} support. Please, use the same version or rebuild OpenCV with CUDA ${CUDA_VERSION_STRING}")
  endif()
endif()

set(OpenCV_CUDA_LIBS_ABSPATH ${CUDA_LIBRARIES})

if(CUDA_VERSION VERSION_LESS "5.5")
  list(APPEND OpenCV_CUDA_LIBS_ABSPATH ${CUDA_npp_LIBRARY})
else()
  find_cuda_helper_libs(nppc)
  find_cuda_helper_libs(nppi)
  find_cuda_helper_libs(npps)
  list(APPEND OpenCV_CUDA_LIBS_ABSPATH ${CUDA_nppc_LIBRARY} ${CUDA_nppi_LIBRARY} ${CUDA_npps_LIBRARY})
endif()

if(OpenCV_USE_CUBLAS)
  list(APPEND OpenCV_CUDA_LIBS_ABSPATH ${CUDA_CUBLAS_LIBRARIES})
endif()

if(OpenCV_USE_CUFFT)
  list(APPEND OpenCV_CUDA_LIBS_ABSPATH ${CUDA_CUFFT_LIBRARIES})
endif()

set(OpenCV_CUDA_LIBS_RELPATH "")
foreach(l ${OpenCV_CUDA_LIBS_ABSPATH})
  get_filename_component(_tmp ${l} PATH)
  if(NOT ${_tmp} MATCHES "-Wl.*")
    list(APPEND OpenCV_CUDA_LIBS_RELPATH ${_tmp})
  endif()
endforeach()

list(REMOVE_DUPLICATES OpenCV_CUDA_LIBS_RELPATH)
link_directories(${OpenCV_CUDA_LIBS_RELPATH})

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

