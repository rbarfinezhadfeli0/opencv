# Documentation for `modules/dnn/src/cuda4dnn/primitives/eltwise.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/eltwise.hpp`
- **File Name**: `eltwise.hpp`
- **File Size**: 6,097 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/eltwise.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/eltwise.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ELTWISE_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ELTWISE_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/tensor.hpp"
#include "../csl/tensor_ops.hpp"

#include "../kernels/eltwise_ops.hpp"

#include <opencv2/core.hpp>

#include <cstddef>
#include <vector>
#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    enum class EltwiseOpType {
        MAX,
        SUM,
        PRODUCT,
        DIV,
        MIN,
        SUB,
        MOD,
        FMOD,
        POW,
    };

    class EltwiseOpBase : public CUDABackendNode {
    public:
        EltwiseOpBase(csl::Stream stream_, EltwiseOpType op_, std::vector<float> coeffs_)
            : stream(std::move(stream_)), op(op_), coeffs(std::move(coeffs_))
        {
        }

    protected:
        csl::Stream stream;

    public:
        EltwiseOpType op;
        std::vector<float> coeffs;
    };

    template <class T>
    class EltwiseOp final : public EltwiseOpBase {
    public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        EltwiseOp(csl::Stream stream_, EltwiseOpType op_, std::vector<float> coeffs_)
            : EltwiseOpBase(std::move(stream_), op_, std::move(coeffs_))
        {
        }

        void forward(
            const std::vector<cv::Ptr<BackendWrapper>>& inputs,
            const std::vector<cv::Ptr<BackendWrapper>>& outputs,
            csl::Workspace& workspace) override
        {
            CV_Assert(outputs.size() == 1);

            CV_Assert(coeffs.size() == 0 || op == EltwiseOpType::SUM);
            CV_Assert(coeffs.size() == 0 || inputs.size() == coeffs.size());

            auto output_wrapper = outputs[0].dynamicCast<wrapper_type>();
            auto output = output_wrapper->getSpan();

            if (inputs.size() == 2)
            {
                auto input_wrapper_x = inputs[0].dynamicCast<wrapper_type>();
                auto input_x = input_wrapper_x->getView();

                auto input_wrapper_y = inputs[1].dynamicCast<wrapper_type>();
                auto input_y = input_wrapper_y->getView();

                switch (op)
                {
                case EltwiseOpType::MAX: kernels::eltwise_max_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::MIN: kernels::eltwise_min_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::PRODUCT: kernels::eltwise_prod_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::DIV: kernels::eltwise_div_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::SUM:
                    if (coeffs.empty() || (coeffs[0] == 1 && coeffs[1] == 1))
                        kernels::eltwise_sum_2<T>(stream, output, input_x, input_y);
                    else
                        kernels::eltwise_sum_coeff_2<T>(stream, output, coeffs[0], input_x, coeffs[1], input_y);
                    break;
                case EltwiseOpType::SUB: kernels::eltwise_sub_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::MOD: kernels::eltwise_mod_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::FMOD: kernels::eltwise_fmod_2<T>(stream, output, input_x, input_y); break;
                case EltwiseOpType::POW: kernels::eltwise_pow_2<T>(stream, output, input_x, input_y); break;
                }
            } else if (inputs.size() == 1) {
                auto input_wrapper_0 = inputs[0].dynamicCast<wrapper_type>();
                auto input_0 = input_wrapper_0->getView();
                csl::tensor_ops::copy(stream, output, input_0);
            } else
            {
                auto input_wrapper_0 = inputs[0].dynamicCast<wrapper_type>();
                auto input_0 = input_wrapper_0->getView();

                /* we first make a copy and then apply EltwiseOp cumulatively */
                csl::tensor_ops::copy(stream, output, input_0);

                for (int i = 1; i < inputs.size(); i++)
                {
                    auto input_wrapper = inputs[i].dynamicCast<wrapper_type>();
                    auto input = input_wrapper->getView();

                    switch (op)
                    {
                    case EltwiseOpType::MAX: kernels::eltwise_max_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::MIN: kernels::eltwise_min_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::PRODUCT: kernels::eltwise_prod_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::DIV: kernels::eltwise_div_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::SUM:
                        if (coeffs.empty() || coeffs[i] == 1)
                            kernels::eltwise_sum_2<T>(stream, output, output, input);
                        else
                        {
                            /* if this is the first op, we must scale output too */
                            T coeff_x = (i == 1) ? coeffs[0] : 1.0;
                            kernels::eltwise_sum_coeff_2<T>(stream, output, coeff_x, output, coeffs[i], input);
                        }
                        break;
                    case EltwiseOpType::SUB: kernels::eltwise_sub_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::MOD: kernels::eltwise_mod_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::FMOD: kernels::eltwise_fmod_2<T>(stream, output, output, input); break;
                    case EltwiseOpType::POW: kernels::eltwise_pow_2<T>(stream, output, output, input); break;
                    }
                }
            }
        }
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ELTWISE_HPP */
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

- **EltwiseOp**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **EltwiseOpType**: A class/struct defined in this file
- **EltwiseOpBase**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ELTWISE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../kernels/eltwise_ops.hpp`
- `../csl/stream.hpp`
- `vector`
- `../csl/tensor_ops.hpp`
- `cstddef`
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

