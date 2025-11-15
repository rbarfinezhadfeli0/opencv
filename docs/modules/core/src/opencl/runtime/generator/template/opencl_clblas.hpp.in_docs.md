# Documentation for `modules/core/src/opencl/runtime/generator/template/opencl_clblas.hpp.in`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/template/opencl_clblas.hpp.in`
- **File Name**: `opencl_clblas.hpp.in`
- **File Size**: 179 bytes
- **File Type**: .in
- **Link to Source**: [modules/core/src/opencl/runtime/generator/template/opencl_clblas.hpp.in](../../../../../../../modules/core/src/opencl/runtime/generator/template/opencl_clblas.hpp.in)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#ifndef OPENCV_CORE_OCL_RUNTIME_CLAMDBLAS_HPP
#error "Invalid usage"
#endif

@CLAMDBLAS_REMAP_ORIGIN@

#include <clBLAS.h>

@CLAMDBLAS_REMAP_DYNAMIC@

@CLAMDBLAS_FN_DECLARATIONS@

```

## General Information

This file is part of the OpenCV repository infrastructure.

