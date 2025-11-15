# Documentation for `modules/dnn/src/opencl/math.cl`

## File Metadata

- **Full Path**: `modules/dnn/src/opencl/math.cl`
- **File Name**: `math.cl`
- **File Size**: 2,776 bytes
- **File Type**: .cl
- **Link to Source**: [modules/dnn/src/opencl/math.cl](../../../../modules/dnn/src/opencl/math.cl)

## Purpose and Role

This file is located in the `modules/dnn/src/opencl` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Copyright (c) 2016-2017 Fabian David Tschopp, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#if defined(cl_khr_fp16)
#pragma OPENCL EXTENSION cl_khr_fp16 : enable
#endif

#define CONCAT(A,B) A##_##B
#define TEMPLATE(name,type) CONCAT(name,type)
#define KERNEL_ARG_DTYPE float

__kernel void TEMPLATE(axpy,Dtype)(const int n, const KERNEL_ARG_DTYPE alpha, __global const Dtype* x,
                                   const int offx, __global Dtype* y,
                                   const int offy) {
  for (int index = get_global_id(0); index < n; index += get_global_size(0)) {
    Dtype src = x[offx + index];
    Dtype dst = y[offy + index];
    y[offy + index] = convert_Dtype(alpha) * src + dst;
  }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

