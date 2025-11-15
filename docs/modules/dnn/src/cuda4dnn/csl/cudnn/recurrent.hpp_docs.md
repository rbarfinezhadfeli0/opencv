# Documentation for `modules/dnn/src/cuda4dnn/csl/cudnn/recurrent.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/csl/cudnn/recurrent.hpp`
- **File Name**: `recurrent.hpp`
- **File Size**: 8,602 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/csl/cudnn/recurrent.hpp](../../../../../../modules/dnn/src/cuda4dnn/csl/cudnn/recurrent.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/csl/cudnn` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_CUDA4DNN_CSL_CUDNN_RECURRENT_HPP
#define OPENCV_DNN_CUDA4DNN_CSL_CUDNN_RECURRENT_HPP

#include "cudnn.hpp"
#include <cudnn.h>


namespace cv { namespace dnn { namespace cuda4dnn { namespace csl { namespace cudnn {

/**
 */
class DropoutDescriptor
{
public:
    DropoutDescriptor() noexcept = default;
    DropoutDescriptor(const DropoutDescriptor &) = delete;
    DropoutDescriptor(DropoutDescriptor &&other) noexcept : descriptor{other.descriptor}
    {
        states = std::move(other.states);
        other.descriptor = nullptr;
    }

    /**
     */
    DropoutDescriptor(const Handle &handle, float dropout)
    {
        CUDA4DNN_CHECK_CUDNN(cudnnCreateDropoutDescriptor(&descriptor));

        // we need additional memory for dropout descriptor
        size_t stateSize;
        CUDA4DNN_CHECK_CUDNN(cudnnDropoutGetStatesSize(handle.get(), &stateSize));
        states.reset(stateSize);

        try
        {
            auto seed = 1234ull; // Pick a seed.
            CUDA4DNN_CHECK_CUDNN(cudnnSetDropoutDescriptor(descriptor, handle.get(), dropout,
                                                           states.get().get(), stateSize, seed));
        }
        catch (...)
        {
            CUDA4DNN_CHECK_CUDNN(cudnnDestroyDropoutDescriptor(descriptor));
            throw;
        }
    }

    ~DropoutDescriptor() noexcept
    {
        if (descriptor)
        {
            CUDA4DNN_CHECK_CUDNN(cudnnDestroyDropoutDescriptor(descriptor));
        }
    }

    DropoutDescriptor &operator=(const DropoutDescriptor &) = delete;
    DropoutDescriptor &operator=(DropoutDescriptor &&other) noexcept
    {
        descriptor = other.descriptor;
        states = std::move(other.states);
        other.descriptor = nullptr;
        return *this;
    };

    cudnnDropoutDescriptor_t get() const noexcept { return descriptor; }

private:
    cudnnDropoutDescriptor_t descriptor{nullptr};

    using value_type = typename ManagedPtr<char>::element_type;
    ManagedPtr<value_type> states;
};

/**
 */
template<class T>
class RNNDescriptor
{
public:
    enum class RNNMode
    {
        RNN_RELU,
        RNN_TANH,
        LSTM,
        GRU
    };

    RNNDescriptor() noexcept = default;
    RNNDescriptor(const RNNDescriptor &) = delete;
    RNNDescriptor(RNNDescriptor &&other) noexcept : descriptor{other.descriptor}
    {
        other.descriptor = nullptr;
    }

    /**
    */
    RNNDescriptor(const Handle &handle, RNNMode mode, int input_size, int hidden_size, int num_layers,
                  bool bidirectional, const DropoutDescriptor &dropoutDesc)
    {
        CUDA4DNN_CHECK_CUDNN(cudnnCreateRNNDescriptor(&descriptor));
        const auto rnn_mode = [mode] {
            switch (mode)
            {
            case RNNMode::RNN_RELU:
                return CUDNN_RNN_RELU;
            case RNNMode::RNN_TANH:
                return CUDNN_RNN_TANH;
            case RNNMode::LSTM:
                return CUDNN_LSTM;
            case RNNMode::GRU:
                return CUDNN_GRU;
            default:
                return CUDNN_LSTM;
            }
        }();

        try
        {
#if CUDNN_MAJOR >= 9
            CUDA4DNN_CHECK_CUDNN(cudnnSetRNNDescriptor_v8(
                                    descriptor,
                                    algo,
                                    rnn_mode,
                                    CUDNN_RNN_DOUBLE_BIAS,
                                    bidirectional ? CUDNN_BIDIRECTIONAL : CUDNN_UNIDIRECTIONAL,
                                    CUDNN_LINEAR_INPUT, detail::get_data_type<T>(),
                                    detail::get_data_type<T>(),
                                    detail::get_data_type<T>() == CUDNN_DATA_HALF ? CUDNN_TENSOR_OP_MATH : CUDNN_DEFAULT_MATH,
                                    input_size,
                                    hidden_size,
                                    hidden_size,
                                    num_layers,
                                    dropoutDesc.get(),
                                    0)); // What other flags do we might want here?
#else
            CUDA4DNN_CHECK_CUDNN(cudnnSetRNNDescriptor_v6(
                                    handle.get(),
                                    descriptor,
                                    hidden_size,
                                    num_layers,
                                    dropoutDesc.get(),
                                    CUDNN_LINEAR_INPUT,
                                    bidirectional ? CUDNN_BIDIRECTIONAL : CUDNN_UNIDIRECTIONAL,
                                    rnn_mode,
                                    algo,
                                    detail::get_data_type<T>()));
#endif
        }
        catch (...)
        {
            CUDA4DNN_CHECK_CUDNN(cudnnDestroyRNNDescriptor(descriptor));
            throw;
        }
    }

    ~RNNDescriptor() noexcept
    {
        if (descriptor)
        {
            CUDA4DNN_CHECK_CUDNN(cudnnDestroyRNNDescriptor(descriptor));
        }
    }

    RNNDescriptor &operator=(const RNNDescriptor &) = delete;
    RNNDescriptor &operator=(RNNDescriptor &&other) noexcept
    {
        descriptor = other.descriptor;
        other.descriptor = nullptr;
        return *this;
    };

    cudnnRNNDescriptor_t get() const noexcept { return descriptor; }

private:
    cudnnRNNDescriptor_t descriptor{nullptr};
    cudnnRNNMode_t mode{CUDNN_LSTM};
    // support only one algo for a while
    cudnnRNNAlgo_t algo{CUDNN_RNN_ALGO_STANDARD};
};

#if CUDNN_MAJOR >= 9
template <class T>
void LSTMForward(const Handle &handle, const RNNDescriptor<T> &rnnDesc,
                 cudnnRNNDataDescriptor_t xDesc, DevicePtr<const T> x,
                 cudnnRNNDataDescriptor_t yDesc, DevicePtr<T> y,
                 cudnnTensorDescriptor_t hDesc, DevicePtr<const T> hx, DevicePtr<T> hy,
                 cudnnTensorDescriptor_t cDesc, DevicePtr<const T> cx, DevicePtr<T> cy,
                 size_t weightSpaceSize, DevicePtr<const T> weightSpace,
                 size_t cudnn_WorkspaceSize, DevicePtr<T> cudnn_Workspace,
                 size_t reserveSpaceSize, DevicePtr<T> reserveSpace)
{
    CV_Assert(handle);

    std::cout << "cudnn_WorkspaceSize: " << cudnn_WorkspaceSize << std::endl;
    std::cout << "reserveSpaceSize: " << reserveSpaceSize << std::endl;

    CUDA4DNN_CHECK_CUDNN(cudnnRNNForward(
        handle.get(), rnnDesc.get(), CUDNN_FWD_MODE_INFERENCE,
        nullptr, // docs say use this as null on >= 8.9.1
        xDesc, x.get(), yDesc, y.get(),
        hDesc, hx.get(), hy.get(),
        cDesc, cx.get(), cy.get(),
        weightSpaceSize, weightSpace.get(),
        cudnn_WorkspaceSize, cudnn_Workspace.get(),
        reserveSpaceSize, reserveSpace.get()));
}

#else
template<class T>
void LSTMForward(const Handle &handle, const RNNDescriptor<T> &rnnDesc,
                 const FilterDescriptor<T> &filterDesc, DevicePtr<const T> filterPtr,
                 const TensorDescriptorsArray<T> &inputDesc, DevicePtr<const T> inputPtr,
                 const TensorDescriptor<T> &initialHDesc, DevicePtr<const T> initialH,
                 const TensorDescriptor<T> &initialCDesc, DevicePtr<const T> initialC,
                 const int seqLength, const TensorDescriptorsArray<T> &outputDesc,
                 DevicePtr<T> yOutputPtr, DevicePtr<T> ycOutputPtr, WorkspaceInstance workspace)
{
    CV_Assert(handle);

    CUDA4DNN_CHECK_CUDNN(cudnnRNNForwardInference(handle.get(), rnnDesc.get(), seqLength,
                                                  inputDesc.get().data(), inputPtr.get(), // input sequence
                                                  initialHDesc.get(), initialH.get(),
                                                  initialCDesc.get(), initialC.get(), // hidden
                                                  filterDesc.get(), filterPtr.get(), // weights
                                                  outputDesc.get().data(), yOutputPtr.get(), // output
                                                  nullptr, nullptr,
                                                  initialCDesc.get(), ycOutputPtr.get(),
                                                  static_cast<void*>(workspace.get()), workspace.size_in_bytes()));
}
#endif

}}}}} /* namespace cv::dnn::cuda4dnn::csl::cudnn */

#endif //OPENCV_DNN_CUDA4DNN_CSL_CUDNN_RECURRENT_HPP
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

- **RNNDescriptor**: A class/struct defined in this file
- **RNNMode**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **DropoutDescriptor**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_CUDA4DNN_CSL_CUDNN_RECURRENT_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cudnn.h`
- `cudnn.hpp`


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

