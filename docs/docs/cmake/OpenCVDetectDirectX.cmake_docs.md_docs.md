# Documentation for `docs/cmake/OpenCVDetectDirectX.cmake_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/OpenCVDetectDirectX.cmake_docs.md`
- **File Name**: `OpenCVDetectDirectX.cmake_docs.md`
- **File Size**: 1,852 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/OpenCVDetectDirectX.cmake_docs.md](../../docs/cmake/OpenCVDetectDirectX.cmake_docs.md)

## Purpose and Role

This file is located in the `docs/cmake` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/OpenCVDetectDirectX.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVDetectDirectX.cmake`
- **File Name**: `OpenCVDetectDirectX.cmake`
- **File Size**: 895 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVDetectDirectX.cmake](../cmake/OpenCVDetectDirectX.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(WIN32)
  try_compile(__VALID_DIRECTX
    "${OpenCV_BINARY_DIR}"
    "${OpenCV_SOURCE_DIR}/cmake/checks/directx.cpp"
    LINK_LIBRARIES d3d11
    OUTPUT_VARIABLE TRY_OUT
  )
  if(NOT __VALID_DIRECTX)
    message(STATUS "No support for DirectX (install Windows 8 SDK)")
    return()
  endif()
  try_compile(__VALID_DIRECTX_NV12
    "${OpenCV_BINARY_DIR}"
    "${OpenCV_SOURCE_DIR}/cmake/checks/directx.cpp"
    COMPILE_DEFINITIONS "-DCHECK_NV12"
    LINK_LIBRARIES d3d11
    OUTPUT_VARIABLE TRY_OUT
  )
  if(__VALID_DIRECTX_NV12)
    set(HAVE_DIRECTX_NV12 ON)
  else()
    message(STATUS "No support for DirectX NV12 format (install Windows 8 SDK)")
  endif()
  set(HAVE_DIRECTX ON)
  set(HAVE_D3D11 ON)
  set(HAVE_D3D10 ON)
  set(HAVE_D3D9 ON)

  if(HAVE_OPENCL AND WITH_OPENCL_D3D11_NV AND EXISTS "${OPENCL_INCLUDE_DIR}/CL/cl_d3d11_ext.h")
    set(HAVE_OPENCL_D3D11_NV ON)
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

