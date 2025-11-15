# Documentation for `modules/dnn/src/cuda4dnn/primitives/matmul.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/matmul.hpp`
- **File Name**: `matmul.hpp`
- **File Size**: 5,140 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/matmul.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/matmul.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_MATMUL_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_MATMUL_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/cublas.hpp"
#include "../csl/tensor.hpp"
#include "../csl/tensor_ops.hpp"

#include "../kernels/scale_shift.hpp"

#include <opencv2/core.hpp>

#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    template <class T>
    class MatMulOp final : public CUDABackendNode {
    public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        MatMulOp(csl::Stream stream_, csl::cublas::Handle handle, const Mat& constInp, const Mat& bias, bool _transA, bool _transB)
            : stream(std::move(stream_)), cublasHandle(std::move(handle))
        {
            if (!constInp.empty())
            {
                constTensor = csl::makeTensorHeader<T>(constInp);
                csl::copyMatToTensor<T>(constInp, constTensor, stream);
            }

            if (!bias.empty())
            {
                biasTensor = csl::makeTensorHeader<T>(bias);
                csl::copyMatToTensor<T>(bias, biasTensor, stream);
            }

            transA = _transA;
            transB = _transB;
        }

        void forward(
            const std::vector<cv::Ptr<BackendWrapper>>& inputs,
            const std::vector<cv::Ptr<BackendWrapper>>& outputs,
            csl::Workspace& workspace) override
        {
            CV_Assert(((inputs.size() == 2 && constTensor.empty()) ||
                       (inputs.size() == 1 && !constTensor.empty())) && outputs.size() == 1);

            auto input1_wrapper = inputs[0].dynamicCast<wrapper_type>();
            auto input1 = input1_wrapper->getView();

            csl::TensorView<T> input2;
            if (constTensor.empty())
            {
                auto input2_wrapper = inputs[1].dynamicCast<wrapper_type>();
                input2 = input2_wrapper->getView();
            }
            else
                input2 = csl::TensorView<T>(constTensor);

            auto output_wrapper = outputs[0].dynamicCast<wrapper_type>();
            auto output = output_wrapper->getSpan();

            auto rank = output.rank();
            CV_Assert(rank == input1.rank());
            CV_Assert(rank == input2.rank());
            CV_Assert(rank >= 2); // 1D MatMul not supported

            for (int i = 0; i < rank - 2; i++)
            {
                // broadcasting not supported
                auto size = output.get_axis_size(i);
                CV_Assert(input1.get_axis_size(i) == size);
                CV_Assert(input2.get_axis_size(i) == size);
            }

            int m1, n1, b1, m2, n2, b2;
            if (transA)
            {
                m1 = input1.get_axis_size(-1);
                n1 = input1.get_axis_size(-2);
            }
            else
            {
                m1 = input1.get_axis_size(-2);
                n1 = input1.get_axis_size(-1);
            }

            if (transB)
            {
                m2 = input2.get_axis_size(-1);
                n2 = input2.get_axis_size(-2);
            }
            else
            {
                m2 = input2.get_axis_size(-2);
                n2 = input2.get_axis_size(-1);
            }

            b1 = input1.size() / m1 / n1;
            b2 = input2.size() / m2 / n2;
            CV_Assert(b1 == b2);
            CV_Assert(n1 == m2);
            CV_Assert(output.get_axis_size(-2) == m1);
            CV_Assert(output.get_axis_size(-1) == n2);

            if (get_effective_rank(output) <= 2)
            {
                CV_Assert(b2 == 1);
                CV_Assert(get_effective_rank(input1) <= 2);
                CV_Assert(get_effective_rank(input2) <= 2);
                csl::tensor_ops::gemm<T>(cublasHandle, 0.0, output, 1.0, transA, input1, transB, input2);
                // used for GEMM
                if (!biasTensor.empty())
                    kernels::biasN<T>(stream, output, output, 1, biasTensor);
            }
            else
            {
                CV_Assert(rank >= 3);
                if (transA)
                    input1.reshape(b1, n1, m1);
                else
                    input1.reshape(b1, m1, n1);

                if (transB)
                    input2.reshape(b2, n2, m2);
                else
                    input2.reshape(b2, m2, n2);

                output.reshape(b1, m1, n2);
                input1.squeeze_to(3);
                input2.squeeze_to(3);
                output.squeeze_to(3);
                csl::tensor_ops::gemmStridedBatched<T>(cublasHandle, 0.0, output, 1.0, transA, input1, transB, input2);
            }
        }

    private:
        csl::Stream stream;
        csl::cublas::Handle cublasHandle;
        csl::Tensor<T> constTensor, biasTensor;
        bool transA, transB;
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_MATMUL_HPP */
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

- **MatMulOp**: A class/struct defined in this file
- **T**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_MATMUL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../csl/stream.hpp`
- `../csl/cublas.hpp`
- `../csl/tensor_ops.hpp`
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

