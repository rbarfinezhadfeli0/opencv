# Documentation for `modules/core/src/opencl/runtime/generator/template/opencl_gl.hpp.in`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/template/opencl_gl.hpp.in`
- **File Name**: `opencl_gl.hpp.in`
- **File Size**: 276 bytes
- **File Type**: .in
- **Link to Source**: [modules/core/src/opencl/runtime/generator/template/opencl_gl.hpp.in](../../../../../../../modules/core/src/opencl/runtime/generator/template/opencl_gl.hpp.in)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#ifndef OPENCV_CORE_OCL_RUNTIME_OPENCL_GL_HPP
#error "Invalid usage"
#endif

@CL_REMAP_ORIGIN@

#if defined __APPLE__
#include <OpenCL/cl_gl.h>
#else
#include <CL/cl_gl.h>
#endif

@CL_REMAP_DYNAMIC@

#ifdef cl_khr_gl_sharing

@CL_FN_DECLARATIONS@

#endif // cl_khr_gl_sharing

```

## General Information

This file is part of the OpenCV repository infrastructure.

