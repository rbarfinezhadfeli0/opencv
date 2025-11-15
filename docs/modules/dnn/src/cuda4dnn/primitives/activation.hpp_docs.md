# Documentation for `modules/dnn/src/cuda4dnn/primitives/activation.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/activation.hpp`
- **File Name**: `activation.hpp`
- **File Size**: 17,926 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/activation.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/activation.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ACTIVATION_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ACTIVATION_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/tensor.hpp"

#include "../kernels/activations.hpp"

#include <opencv2/core.hpp>

#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    template <template<class> class Op, class T>
    struct BaseOp : public CUDABackendNode
    {
    protected:
        using wrapper_type = GetCUDABackendWrapperType<T>;

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

                static_cast<const Op<T>*>(this)->calculate(output, input);
            }
        }
    };

    template <class T>
    class ReLUOp final : public BaseOp<ReLUOp, T> {
    public:
        ReLUOp(csl::Stream stream_, T slope_)
                : stream(std::move(stream_)), slope{ slope_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::relu<T>(stream, output, input, slope);
        }

    private:
        csl::Stream stream;
        const T slope;
    };

    template <class T>
    class ClippedReLUOp final : public BaseOp<ClippedReLUOp, T> {
    public:
        ClippedReLUOp(csl::Stream stream_, T min_, T max_)
            : stream(std::move(stream_)), min{ min_ }, max{ max_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::clipped_relu<T>(stream, output, input, min, max);
        }

    private:
        csl::Stream stream;
        const T min, max;
    };

    template <class T>
    class ChannelwiseReLUOp final : public BaseOp<ChannelwiseReLUOp, T> {
    public:
        ChannelwiseReLUOp(csl::Stream stream_, const Mat& slope)
                : stream(std::move(stream_))
        {
            CV_Assert(!slope.empty());
            slopeTensor = csl::makeTensorHeader<T>(slope);
            csl::copyMatToTensor<T>(slope, slopeTensor, stream);
        }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            CV_Assert(input.get_axis_size(1) == slopeTensor.size());
            std::size_t inner_size = input.size_range(2, input.rank());
            kernels::axiswise_relu<T>(stream, output, input, inner_size, slopeTensor);
        }

    private:
        csl::Stream stream;
        csl::Tensor<T> slopeTensor;
    };

    template <class T>
    class TanHOp final : public BaseOp<TanHOp, T> {
    public:
        TanHOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::tanh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SwishOp final : public BaseOp<SwishOp, T> {
    public:
        SwishOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::swish<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class MishOp final : public BaseOp<MishOp, T> {
    public:
        MishOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::mish<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SigmoidOp final : public BaseOp<SigmoidOp, T> {
    public:
        SigmoidOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::sigmoid<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class ELUOp final : public BaseOp<ELUOp, T> {
    public:
        ELUOp(csl::Stream stream_, T alpha_) : stream(std::move(stream_)), alpha(alpha_) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::elu<T>(stream, output, input, alpha);
        }

    private:
        csl::Stream stream;
        T alpha;
    };

    template <class T>
    class AbsValOp final : public BaseOp<AbsValOp, T> {
    public:
        AbsValOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::abs<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class BNLLOp final : public BaseOp<BNLLOp, T> {
    public:
        BNLLOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::bnll<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class CeilOp final : public BaseOp<CeilOp, T> {
    public:
        CeilOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::ceil<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class FloorOp final : public BaseOp<FloorOp, T> {
    public:
        FloorOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::floor<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class LogOp final : public BaseOp<LogOp, T> {
    public:
        LogOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::log<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class RoundOp final : public BaseOp<RoundOp, T> {
    public:
        RoundOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::rint<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SqrtOp final : public BaseOp<SqrtOp, T> {
    public:
        SqrtOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::sqrt<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class NotOp final : public BaseOp<NotOp, T> {
    public:
        NotOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::not_k<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AcosOp final : public BaseOp<AcosOp, T> {
    public:
        AcosOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::acos<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AcoshOp final : public BaseOp<AcoshOp, T> {
    public:
        AcoshOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::acosh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AsinOp final : public BaseOp<AsinOp, T> {
    public:
        AsinOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::asin<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AsinhOp final : public BaseOp<AsinhOp, T> {
    public:
        AsinhOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::asinh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AtanOp final : public BaseOp<AtanOp, T> {
    public:
        AtanOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::atan<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class AtanhOp final : public BaseOp<AtanhOp, T> {
    public:
        AtanhOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::atanh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class CosOp final : public BaseOp<CosOp, T> {
    public:
        CosOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::cos<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class CoshOp final : public BaseOp<CoshOp, T> {
    public:
        CoshOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::cosh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class ErfOp final : public BaseOp<ErfOp, T> {
    public:
        ErfOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::erf<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class HardSwishOp final : public BaseOp<HardSwishOp, T> {
    public:
        HardSwishOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::hardswish<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SinOp final : public BaseOp<SinOp, T> {
    public:
        SinOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::sin<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SinhOp final : public BaseOp<SinhOp, T> {
    public:
        SinhOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::sinh<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SoftplusOp final : public BaseOp<SoftplusOp, T> {
    public:
        SoftplusOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::softplus<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class SoftsignOp final : public BaseOp<SoftsignOp, T> {
    public:
        SoftsignOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::softsign<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class TanOp final : public BaseOp<TanOp, T> {
    public:
        TanOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::tan<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class CeluOp final : public BaseOp<CeluOp, T> {
    public:
        CeluOp(csl::Stream stream_, T alpha_) : stream(std::move(stream_)), alpha{ alpha_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::celu<T>(stream, output, input, alpha);
        }

    private:
        csl::Stream stream;
        const T alpha;
    };

    template <class T>
    class HardSigmoidOp final : public BaseOp<HardSigmoidOp, T> {
    public:
        HardSigmoidOp(csl::Stream stream_, T alpha_, T beta_)
            : stream(std::move(stream_)), alpha{ alpha_ }, beta{ beta_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::hardsigmoid<T>(stream, output, input, alpha, beta);
        }

    private:
        csl::Stream stream;
        const T alpha, beta;
    };

    template <class T>
    class SeluOp final : public BaseOp<SeluOp, T> {
    public:
        SeluOp(csl::Stream stream_, T alpha_, T gamma_)
            : stream(std::move(stream_)), alpha{ alpha_ }, gamma{ gamma_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::selu<T>(stream, output, input, alpha, gamma);
        }

    private:
        csl::Stream stream;
        const T alpha, gamma;
    };

    template <class T>
    class GeluOp final : public BaseOp<GeluOp, T> {
    public:
        GeluOp(csl::Stream stream_) : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::gelu<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class ThresholdedReluOp final : public BaseOp<ThresholdedReluOp, T> {
    public:
        ThresholdedReluOp(csl::Stream stream_, T alpha_) : stream(std::move(stream_)), alpha{ alpha_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::thresholdedrelu<T>(stream, output, input, alpha);
        }

    private:
        csl::Stream stream;
        const T alpha;
    };

    template <class T>
    class PowerOp final : public BaseOp<PowerOp, T> {
    public:
        PowerOp(csl::Stream stream_, T exp_, T scale_, T shift_)
            : stream(std::move(stream_)), exp{ exp_ }, scale{ scale_ }, shift{ shift_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::power<T>(stream, output, input, exp, scale, shift);
        }

    private:
        csl::Stream stream;
        const T exp, scale, shift;
    };

    template <class T>
    class ExpOp final : public BaseOp<ExpOp, T> {
    public:
        ExpOp(csl::Stream stream_, T nScale_, T nShift_)
            : stream(std::move(stream_)), normScale{ nScale_ }, normShift{ nShift_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::exp<T>(stream, output, input, normScale, normShift);
        }

    private:
        csl::Stream stream;
        const T normScale, normShift;
    };

    template <class T>
    class ShrinkOp final : public BaseOp<ShrinkOp, T> {
    public:
        ShrinkOp(csl::Stream stream_, T bias_, T lambd_)
                : stream(std::move(stream_)), bias{ bias_ }, lambd{ lambd_ } { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::shrink<T>(stream, output, input, bias, lambd);
        }

    private:
        csl::Stream stream;
        const T bias, lambd;
    };

    template <class T>
    class SignOp final : public BaseOp<SignOp, T> {
    public:
        SignOp(csl::Stream stream_)
                : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::sign<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

    template <class T>
    class ReciprocalOp final : public BaseOp<ReciprocalOp, T> {
    public:
        ReciprocalOp(csl::Stream stream_)
                : stream(std::move(stream_)) { }

        void calculate(csl::TensorSpan<T> output, csl::TensorView<T> input) const
        {
            kernels::reciprocal<T>(stream, output, input);
        }

    private:
        csl::Stream stream;
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ACTIVATION_HPP */
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

- **CeilOp**: A class/struct defined in this file
- **MishOp**: A class/struct defined in this file
- **FloorOp**: A class/struct defined in this file
- **AsinOp**: A class/struct defined in this file
- **Op**: A class/struct defined in this file
- **BaseOp**: A class/struct defined in this file
- **AcosOp**: A class/struct defined in this file
- **AsinhOp**: A class/struct defined in this file
- **TanHOp**: A class/struct defined in this file
- **ReLUOp**: A class/struct defined in this file
- **BNLLOp**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **AtanhOp**: A class/struct defined in this file
- **ELUOp**: A class/struct defined in this file
- **SqrtOp**: A class/struct defined in this file
- **NotOp**: A class/struct defined in this file
- **LogOp**: A class/struct defined in this file
- **CosOp**: A class/struct defined in this file
- **SigmoidOp**: A class/struct defined in this file
- **AcoshOp**: A class/struct defined in this file
- **AtanOp**: A class/struct defined in this file
- **RoundOp**: A class/struct defined in this file
- **SwishOp**: A class/struct defined in this file
- **ChannelwiseReLUOp**: A class/struct defined in this file
- **AbsValOp**: A class/struct defined in this file
- **ClippedReLUOp**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_ACTIVATION_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../csl/stream.hpp`
- `../kernels/activations.hpp`
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

