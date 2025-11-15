# Documentation for `modules/core/src/opencl/runtime/generator/template/opencl_clblas_impl.hpp.in`

## File Metadata

- **Full Path**: `modules/core/src/opencl/runtime/generator/template/opencl_clblas_impl.hpp.in`
- **File Name**: `opencl_clblas_impl.hpp.in`
- **File Size**: 125 bytes
- **File Type**: .in
- **Link to Source**: [modules/core/src/opencl/runtime/generator/template/opencl_clblas_impl.hpp.in](../../../../../../../modules/core/src/opencl/runtime/generator/template/opencl_clblas_impl.hpp.in)

## Purpose and Role

This file is located in the `modules/core/src/opencl/runtime/generator/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
@CL_FN_ENUMS@

namespace {
@CL_FN_SWITCH@
}

@CL_FN_ENTRY_DEFINITIONS@

@CL_FN_ENTRY_LIST@

@CL_NUMBER_OF_ENABLED_FUNCTIONS@

```

## General Information

This file is part of the OpenCV repository infrastructure.

