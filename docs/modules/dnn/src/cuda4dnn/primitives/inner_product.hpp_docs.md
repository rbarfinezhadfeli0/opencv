# Documentation for `modules/dnn/src/cuda4dnn/primitives/inner_product.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/inner_product.hpp`
- **File Name**: `inner_product.hpp`
- **File Size**: 3,448 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/inner_product.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/inner_product.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_INNER_PRODUCT_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_INNER_PRODUCT_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/cublas.hpp"
#include "../csl/tensor.hpp"
#include "../csl/tensor_ops.hpp"

#include "../kernels/scale_shift.hpp"

#include <opencv2/core.hpp>

#include <cstddef>
#include <vector>
#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    template <class T>
    class InnerProductOp final : public CUDABackendNode {
    public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        InnerProductOp(csl::Stream stream_, csl::cublas::Handle handle, std::size_t axis, const Mat& weights, const Mat& bias)
            : stream(std::move(stream_)), cublasHandle(std::move(handle)), axis{ axis }
        {
            weightsTensor = csl::makeTensorHeader<T>(weights);
            CV_Assert(get_effective_rank(weightsTensor) <= 2);
            csl::copyMatToTensor<T>(weights, weightsTensor, stream);

            if (!bias.empty())
            {
                biasTensor = csl::makeTensorHeader<T>(bias);
                csl::copyMatToTensor<T>(bias, biasTensor, stream);
                CV_Assert(weightsTensor.get_axis_size(-2) == biasTensor.size());
            }
        }

        void forward(
            const std::vector<cv::Ptr<BackendWrapper>>& inputs,
            const std::vector<cv::Ptr<BackendWrapper>>& outputs,
            csl::Workspace& workspace) override
        {
            for (int i = 0; i < inputs.size(); i++)
            {
                auto input_wrapper = inputs[i].dynamicCast<wrapper_type>();
                auto input = input_wrapper->getView();

                auto output_wrapper = outputs[i].dynamicCast<wrapper_type>();
                auto output = output_wrapper->getSpan();

                std::size_t batch_size = input.size_range(0, axis);

                auto input_size = input.size() / batch_size;
                CV_Assert(input_size == weightsTensor.get_axis_size(-1));

                auto output_size = output.size() / batch_size;
                CV_Assert(output_size == weightsTensor.get_axis_size(-2));

                /* we treat the input and output as a matrix with dimensions (batch_size, input_size)
                 * and (batch_size, output_size) respectively
                 *
                 * weight matrix dimensions: (output_size, input_size)
                 *
                 * I(W^T) = O
                 * (batch_size, input_size) * (input_size, output_size) = (batch_size, output_size)
                 */
                input.reshape(batch_size, input_size);
                output.reshape(batch_size, output_size);
                csl::tensor_ops::gemm<T>(cublasHandle, 0.0, output, 1.0, false, input, true, weightsTensor);

                if (!biasTensor.empty())
                    kernels::biasN<T>(stream, output, output, 1, biasTensor);
            }
        }

    private:
        csl::Stream stream;
        csl::cublas::Handle cublasHandle;
        csl::Tensor<T> weightsTensor, biasTensor;
        std::size_t axis;
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_INNER_PRODUCT_HPP */
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

- **T**: A class/struct defined in this file
- **InnerProductOp**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_INNER_PRODUCT_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../csl/stream.hpp`
- `../csl/cublas.hpp`
- `vector`
- `../csl/tensor_ops.hpp`
- `cstddef`
- `../kernels/scale_shift.hpp`
- `opencv2/core.hpp`
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

