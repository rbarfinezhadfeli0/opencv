# Documentation for `modules/dnn/src/cuda4dnn/primitives/layer_norm.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/layer_norm.hpp`
- **File Name**: `layer_norm.hpp`
- **File Size**: 4,459 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/layer_norm.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/layer_norm.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_LAYER_NORM_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_LAYER_NORM_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/span.hpp"
#include "../csl/tensor.hpp"
#include "../csl/workspace.hpp"

#include "../kernels/fill_copy.hpp"
#include "../kernels/mvn.hpp"

#include <opencv2/core.hpp>

#include <cstddef>
#include <vector>
#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    template <class T>
    class LayerNormOp final : public CUDABackendNode {
     public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        LayerNormOp(csl::Stream stream_, const Mat &scale, const Mat &bias, int normalized_axis, float epsilon_, size_t loops)
            : stream(std::move(stream_)), epsilon(epsilon_) {
            CV_CheckGE(normalized_axis, 0, "LayerNorm/CUDA: axis needs to be normalized");
            axis = static_cast<size_t>(normalized_axis);

            if (!scale.empty()) {
                input_scale_tensor = csl::makeTensorHeader<T>(scale);
                csl::copyMatToTensor<T>(scale, input_scale_tensor, stream);
            }
            if (!bias.empty()) {
                input_bias_tensor = csl::makeTensorHeader<T>(bias);
                csl::copyMatToTensor<T>(bias, input_bias_tensor, stream);
            }

            csl::WorkspaceBuilder builder;
            builder.require<float>(loops);
            builder.require<float>(loops);
            scratch_mem_in_bytes = builder.required_workspace_size();
        }

        void forward(const std::vector<cv::Ptr<BackendWrapper>>& inputs,
                     const std::vector<cv::Ptr<BackendWrapper>>& outputs,
                     csl::Workspace& workspace) override {
            auto input_wrapper = inputs[0].dynamicCast<wrapper_type>();
            auto input = input_wrapper->getView();

            csl::TensorView<T> scale;
            if (input_scale_tensor.empty()) {
                auto scale_wrapper = inputs[1].dynamicCast<wrapper_type>();
                scale = scale_wrapper->getView();
            } else {
                scale = csl::TensorView<T>(input_scale_tensor);
            }

            csl::TensorView<T> bias;
            if (input_bias_tensor.empty()) {
                if (inputs.size() >= 3) {
                    auto bias_wrapper = inputs[2].dynamicCast<wrapper_type>();
                    bias = bias_wrapper->getView();
                }
            } else {
                bias = csl::TensorView<T>(input_bias_tensor);
            }

            auto output_wrapper = outputs[0].dynamicCast<wrapper_type>();
            auto output = output_wrapper->getSpan();

            auto loops = input.size_range(0, axis);
            auto norm_size = input.size_range(axis, input.rank());
            if (norm_size == 1) {
                kernels::fill<T>(stream, output, 0.f);
                return;
            } else {
                auto ws_allocator = csl::WorkspaceAllocator(workspace);

                auto mean = ws_allocator.get_span<float>(loops);
                kernels::fill<float>(stream, mean, 0.f);

                auto inv_stddev = ws_allocator.get_span<float>(loops);
                kernels::fill<float>(stream, inv_stddev, 0.f);

                kernels::reduce_mean_sqr_sum<T>(stream, mean, inv_stddev, input, norm_size);
                kernels::compute_normalization_scale(stream, inv_stddev, mean, inv_stddev, norm_size, epsilon);
                if (!bias.empty()) {
                    kernels::normalize_mean_variance_layernorm<T>(stream, output, input, scale, bias, mean, inv_stddev, norm_size);
                } else {
                    kernels::normalize_mean_variance_layernorm<T>(stream, output, input, scale, mean, inv_stddev, norm_size);
                }
            }
        }

        std::size_t get_workspace_memory_in_bytes() const noexcept override { return scratch_mem_in_bytes; }

     private:
        csl::Stream stream;
        csl::Tensor<T> input_scale_tensor;
        csl::Tensor<T> input_bias_tensor;

        float epsilon;
        size_t axis;

        std::size_t scratch_mem_in_bytes;
    };

}}} // cv::dnn::cuda4dnn

#endif // OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_LAYER_NORM_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **LayerNormOp**: A class/struct defined in this file
- **T**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_LAYER_NORM_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../csl/stream.hpp`
- `../csl/workspace.hpp`
- `vector`
- `../kernels/mvn.hpp`
- `cstddef`
- `../kernels/fill_copy.hpp`
- `opencv2/core.hpp`
- `../csl/span.hpp`
- `../csl/tensor.hpp`


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

