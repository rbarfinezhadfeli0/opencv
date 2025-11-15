# Documentation for `modules/dnn/src/ocl4dnn/src/ocl4dnn_softmax.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/ocl4dnn/src/ocl4dnn_softmax.cpp`
- **File Name**: `ocl4dnn_softmax.cpp`
- **File Size**: 5,831 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/ocl4dnn/src/ocl4dnn_softmax.cpp](../../../../../modules/dnn/src/ocl4dnn/src/ocl4dnn_softmax.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/ocl4dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

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

#include "../../precomp.hpp"
#include <vector>
#include "../include/common.hpp"
#include "../include/ocl4dnn.hpp"
#include "opencl_kernels_dnn.hpp"

namespace cv { namespace dnn { namespace ocl4dnn {
template<typename Dtype>
OCL4DNNSoftmax<Dtype>::OCL4DNNSoftmax(OCL4DNNSoftmaxConfig config)
{
    softmax_axis_ = config.axis;
    channels_ = config.channels;
    log_softmax_ = config.logsoftmax;
    use_half_ = config.use_half;

    inner_num_ = 1;
    outer_num_ = 1;
    count_ = 1;
    int32_t scale_sz = 1;
    for (int32_t i = softmax_axis_ + 1; i < config.in_shape.size(); i++)
        inner_num_ *= config.in_shape[i];
    use_slm_ = (config.in_shape[softmax_axis_] * inner_num_ + inner_num_ * 17) <= 8192;
    for (int32_t i = 0; i < softmax_axis_; i++)
        outer_num_ *= config.in_shape[i];
    count_ = inner_num_ + outer_num_;

    std::vector<int32_t> scale_dims = config.in_shape;
    scale_dims[softmax_axis_] = use_slm_ ? 1 : 17;
    for (int32_t i = 0; i < scale_dims.size(); i++)
        scale_sz *= scale_dims[i];

    scale_data_.create(1, scale_sz, CV_32FC1);
}

template<typename Dtype>
OCL4DNNSoftmax<Dtype>::~OCL4DNNSoftmax()
{
    scale_data_.release();
}

template<typename Dtype>
bool OCL4DNNSoftmax<Dtype>::Forward(const UMat& bottom, UMat& top)
{
    bool ret = false;
    bool intel_subgroup = ocl::Device::getDefault().intelSubgroupsSupport();
    if (intel_subgroup && inner_num_ < 128)
    {
        String opts = clOptionSupport("-cl-no-subgroup-ifp") ? " -cl-no-subgroup-ifp " : "";
        String kname;
        ocl::Kernel oclk_softmax_forward_kernel;

        if (log_softmax_) opts += " -DLOG_SOFTMAX ";
        if (use_slm_)
            kname = "softmax_forward_slm";
        else
            kname = "softmax_forward";

        kname += format("%s", (use_half_) ? "_half" : "_float");
        opts += format(" -D Dtype=%s -D DTYPE_MAX=%s", (use_half_) ? "half" : "float",
                       (use_half_) ? "HALF_MAX" : "FLT_MAX");
        if (!oclk_softmax_forward_kernel.create(kname.c_str(), ocl::dnn::softmax_loss_oclsrc, opts))
            return false;

        size_t global_size[] = { 256, (size_t)outer_num_, 1 };
        size_t local_size[] = { 256, 1, 1 };
        cl_uint argIdx = 0;

        if (use_slm_)
        {
            oclk_softmax_forward_kernel.set(argIdx++, outer_num_);
            oclk_softmax_forward_kernel.set(argIdx++, channels_);
            oclk_softmax_forward_kernel.set(argIdx++, inner_num_);
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrWriteOnly(scale_data_));
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrReadOnly(bottom));
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrWriteOnly(top));
            oclk_softmax_forward_kernel.set(argIdx++, NULL, channels_ * inner_num_* sizeof(Dtype));
            oclk_softmax_forward_kernel.set(argIdx++, NULL, inner_num_* sizeof(Dtype));
            oclk_softmax_forward_kernel.set(argIdx++, NULL, 16 * inner_num_* sizeof(Dtype));
        }
        else
        {
            oclk_softmax_forward_kernel.set(argIdx++, outer_num_);
            oclk_softmax_forward_kernel.set(argIdx++, channels_);
            oclk_softmax_forward_kernel.set(argIdx++, inner_num_);
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrWriteOnly(scale_data_));
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrReadOnly(bottom));
            oclk_softmax_forward_kernel.set(argIdx++, ocl::KernelArg::PtrWriteOnly(top));
        }
        ret = oclk_softmax_forward_kernel.run_(3, global_size, local_size, false);
    }
    return ret;
}

template class OCL4DNNSoftmax<float>;

}}} // namespace cv::dnn::ocl4dnn
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **OCL4DNNSoftmax**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../precomp.hpp`
- `opencl_kernels_dnn.hpp`
- `vector`
- `../include/ocl4dnn.hpp`
- `../include/common.hpp`

**Python Imports:**
- `this`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

