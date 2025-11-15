# Documentation for `modules/dnn/src/layers/group_norm_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/group_norm_layer.cpp`
- **File Name**: `group_norm_layer.cpp`
- **File Size**: 7,542 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/group_norm_layer.cpp](../../../../modules/dnn/src/layers/group_norm_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include <opencv2/dnn/shape_utils.hpp>
#include "./cpu_kernels/fast_norm.hpp"

// CUDA backend
#include "../op_cuda.hpp"
#ifdef HAVE_CUDA
#include "../cuda4dnn/primitives/group_norm.hpp"
using namespace cv::dnn::cuda4dnn;
#endif

// OpenCL backend
#ifdef HAVE_OPENCL
#include "../ocl4dnn/include/math_functions.hpp"
#include "opencl_kernels_dnn.hpp"
#endif

namespace cv {
namespace dnn {

// https://github.com/onnx/onnx/blob/main/docs/Operators.md#GroupNormalization
class GroupNormLayerImpl CV_FINAL : public GroupNormLayer {
public:
    GroupNormLayerImpl(const LayerParams &params) {
        setParamsFrom(params);

        epsilon = params.get<float>("epsilon", 1e-5);
        num_groups = params.get<int>("num_groups");
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE {
        return backendId == DNN_BACKEND_OPENCV ||
               backendId == DNN_BACKEND_CUDA;
    }

    bool getMemoryShapes(const std::vector<MatShape> &inputs,
                         const int requiredOutputs,
                         std::vector<MatShape> &outputs,
                         std::vector<MatShape> &internals) const CV_OVERRIDE {
        const auto &input = inputs[0];
        const auto &scale = inputs[1];
        const auto &bias = inputs[2];
        CV_CheckGE(input.size(), static_cast<size_t>(3), "DNN/GroupNorm: input dimension >= 3 is required");

        int C = input[1];
        int scale_dim = std::accumulate(scale.begin(), scale.end(), 1, std::multiplies<int>());
        CV_CheckEQ(scale_dim, C, "DNN/InstanceNorm: scale must be a 1d tensor and match the channel of input");
        int bias_dim = std::accumulate(bias.begin(), bias.end(), 1, std::multiplies<int>());
        CV_CheckEQ(bias_dim, C, "DNN/InstanceNorm: bias must be a 1d tensor and match the channel of input");

        outputs.assign(1, inputs[0]);
        return false;
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        if (inputs_arr.depth() == CV_16F) {
            forward_fallback(inputs_arr, outputs_arr, internals_arr);
            return;
        }

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        const auto& input = inputs[0];
        const auto& scale = inputs[1];
        const auto& bias = inputs[2];

        fastNormGroup(input, scale, bias, outputs[0], epsilon, num_groups);
    }

#ifdef HAVE_OPENCL
    bool forward_ocl(InputArrayOfArrays inputs_, OutputArrayOfArrays outputs_, OutputArrayOfArrays internals_) {
        std::vector<UMat> inputs;
        std::vector<UMat> outputs;

        inputs_.getUMatVector(inputs);
        outputs_.getUMatVector(outputs);

        const auto &input = inputs[0], &scale = inputs[1], &bias = inputs[2];
        auto &output = outputs[0];

        const auto input_shape = shape(input);
        size_t N = input_shape[0], C = input_shape[1];
        size_t num_groups = this->num_groups;
        size_t channels_per_group = C / num_groups;
        size_t loops = N * num_groups, norm_size = static_cast<size_t>(total(input_shape, 2)) * channels_per_group;
        float inv_norm_size = 1.f / norm_size;

        // no fp16 support
        if (input.depth() == CV_16F) {
            return false;
        }

        String base_opts = format(" -DT=float -DT4=float4 -Dconvert_T=convert_float4");

        // Calculate mean
        UMat one = UMat::ones(norm_size, 1, CV_32F);
        UMat mean = UMat(loops, 1, CV_32F);
        UMat mean_square = UMat(loops, 1, CV_32F);
        UMat tmp = UMat(loops, norm_size, CV_32F);
        bool ret = ocl4dnn::ocl4dnnGEMV<float>(ocl4dnn::CblasNoTrans, loops, norm_size, inv_norm_size,
                                               input, 0, one, 0, 0.f, mean, 0);
        if (!ret) {
            return false;
        }
        // Calculate mean_square
        int num_vector = (norm_size % 8 == 0) ? 8 : ((norm_size % 4 == 0) ? 4 : 1);
        size_t global[] = {loops, static_cast<size_t>(norm_size / num_vector)};
        String build_opt = format(" -DNUM=%d", num_vector) + base_opts;
        String mean_square_kernel_name = format("calc_mean%d", num_vector);
        ocl::Kernel mean_square_kernel(mean_square_kernel_name.c_str(), ocl::dnn::mvn_oclsrc, build_opt + " -DKERNEL_MEAN");
        if (mean_square_kernel.empty()) {
            return false;
        }
        mean_square_kernel.set(0, ocl::KernelArg::PtrReadOnly(input));
        mean_square_kernel.set(1, (int)loops);
        mean_square_kernel.set(2, (int)norm_size);
        mean_square_kernel.set(3, ocl::KernelArg::PtrReadOnly(mean));
        mean_square_kernel.set(4, ocl::KernelArg::PtrWriteOnly(tmp));
        ret = mean_square_kernel.run(2, global, NULL, false);
        if (!ret) {
            return false;
        }
        ret = ocl4dnn::ocl4dnnGEMV<float>(ocl4dnn::CblasNoTrans, loops, norm_size, inv_norm_size,
                                          tmp, 0, one, 0, 0.f, mean_square, 0);
        if (!ret) {
            return false;
        }
        // Calculate group norm: output = scale * (x - mean) / sqrt(var + eps) + bias
        String mvn_group_kernel_name = format("mvn_group%d", num_vector);
        build_opt += " -DNORM_VARIANCE -DKERNEL_MVN_GROUP";
        ocl::Kernel mvn_group_kernel(mvn_group_kernel_name.c_str(), ocl::dnn::mvn_oclsrc, build_opt);
        if (mvn_group_kernel.empty()) {
            return false;
        }
        mvn_group_kernel.set(0, ocl::KernelArg::PtrReadOnly(input));
        mvn_group_kernel.set(1, (int)loops);
        mvn_group_kernel.set(2, (int)norm_size);
        mvn_group_kernel.set(3, (float)epsilon);
        mvn_group_kernel.set(4, ocl::KernelArg::PtrReadOnly(mean));
        mvn_group_kernel.set(5, ocl::KernelArg::PtrReadOnly(mean_square));
        mvn_group_kernel.set(6, ocl::KernelArg::PtrReadOnly(scale));
        mvn_group_kernel.set(7, ocl::KernelArg::PtrReadOnly(bias));
        mvn_group_kernel.set(8, (int)C);
        mvn_group_kernel.set(9, (int)num_groups);
        mvn_group_kernel.set(10, (float)0.f);
        mvn_group_kernel.set(11, ocl::KernelArg::PtrWriteOnly(output));
        ret = mvn_group_kernel.run(2, global, NULL, false);
        if (!ret) {
            return false;
        }

        return true;
        }
#endif

#ifdef HAVE_CUDA
    Ptr<BackendNode> initCUDA(void *context_,
                          const std::vector<Ptr<BackendWrapper>>& inputs,
                          const std::vector<Ptr<BackendWrapper>>& outputs) override {
    auto context = reinterpret_cast<csl::CSLContext*>(context_);

    auto input_wrapper = inputs[0].dynamicCast<CUDABackendWrapper>();
    auto input_shape = input_wrapper->getShape();
    size_t N = input_shape[0];
    size_t num_groups = this->num_groups;
    size_t loops = N * num_groups;

    return make_cuda_node<cuda4dnn::GroupNormOp>(preferableTarget, std::move(context->stream), epsilon, loops, num_groups);
}
#endif // HAVE_CUDA

private:
    float epsilon;
    size_t num_groups;
};

Ptr<GroupNormLayer> GroupNormLayer::create(const LayerParams &params) {
    return Ptr<GroupNormLayer>(new GroupNormLayerImpl(params));
}

}} // cv::dnn
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

- **GroupNormLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../op_cuda.hpp`
- `../ocl4dnn/include/math_functions.hpp`
- `opencl_kernels_dnn.hpp`
- `../precomp.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `../cuda4dnn/primitives/group_norm.hpp`
- `./cpu_kernels/fast_norm.hpp`


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

