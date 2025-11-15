# Documentation for `modules/core/src/opencl/runtime/generator/template/opencl_core.hpp.in`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/template/opencl_core.hpp.in`
- **File Name**: `opencl_core.hpp.in`
- **File Size**: 248 bytes
- **File Type**: .in
- **Link to Source**: [modules/core/src/opencl/runtime/generator/template/opencl_core.hpp.in](../../../../../../../modules/core/src/opencl/runtime/generator/template/opencl_core.hpp.in)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#ifndef OPENCV_CORE_OCL_RUNTIME_OPENCL_CORE_HPP
#error "Invalid usage"
#endif

@CL_REMAP_ORIGIN@

#if defined __APPLE__
#define CL_SILENCE_DEPRECATION
#include <OpenCL/cl.h>
#else
#include <CL/cl.h>
#endif

@CL_REMAP_DYNAMIC@

@CL_FN_DECLARATIONS@

```

## General Information

This file is part of the OpenCV repository infrastructure.

