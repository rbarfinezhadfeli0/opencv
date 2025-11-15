# Documentation for `modules/core/src/opencl/runtime/generator/template/opencl_gl_wrappers.hpp.in`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/template/opencl_gl_wrappers.hpp.in`
- **File Name**: `opencl_gl_wrappers.hpp.in`
- **File Size**: 165 bytes
- **File Type**: .in
- **Link to Source**: [modules/core/src/opencl/runtime/generator/template/opencl_gl_wrappers.hpp.in](../../../../../../../modules/core/src/opencl/runtime/generator/template/opencl_gl_wrappers.hpp.in)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#ifndef OPENCV_CORE_OCL_RUNTIME_OPENCL_GL_WRAPPERS_HPP
#error "Invalid usage"
#endif

#ifdef cl_khr_gl_sharing

@CL_FN_INLINE_WRAPPERS@

#endif // cl_khr_gl_sharing

```

## General Information

This file is part of the OpenCV repository infrastructure.

