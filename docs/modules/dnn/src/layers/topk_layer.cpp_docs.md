# Documentation for `modules/dnn/src/layers/topk_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/topk_layer.cpp`
- **File Name**: `topk_layer.cpp`
- **File Size**: 8,267 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/topk_layer.cpp](../../../../modules/dnn/src/layers/topk_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "layers_common.hpp"

#include <opencv2/dnn/shape_utils.hpp>

namespace cv { namespace dnn {

namespace {

template<typename T>
class ComparatorGreater {
public:
    ComparatorGreater(const T* data, size_t step)
        : data_(data), step_(step) {}

    void addOffset(size_t offset) {
        data_ += offset;
    }

    void minusOffset(size_t offset) {
        data_ -= offset;
    }

    bool operator()(const size_t lhs_idx, const size_t rhs_idx) {
        T lhs = *(data_ + lhs_idx * step_),
          rhs = *(data_ + rhs_idx * step_);
        return (lhs > rhs || (lhs == rhs && lhs_idx < rhs_idx));
    }

private:
    const T* data_;
    size_t step_;
};

template<typename T>
class ComparatorLess {
public:
    ComparatorLess(const T* data, size_t step)
        : data_(data), step_(step) {}

    void addOffset(size_t offset) {
        data_ += offset;
    }

    void minusOffset(size_t offset) {
        data_ -= offset;
    }

    bool operator()(const size_t lhs_idx, const size_t rhs_idx) {
        T lhs = *(data_ + lhs_idx * step_),
          rhs = *(data_ + rhs_idx * step_);
        return (lhs < rhs || (lhs == rhs && lhs_idx < rhs_idx));
    }

private:
    const T* data_;
    size_t step_;
};
}

class TopKLayerImpl CV_FINAL : public TopKLayer
{
public:
    TopKLayerImpl(const LayerParams& params)
    {
        setParamsFrom(params);

        axis = params.get<int>("axis", -1);
        largest = params.get<int>("largest", 1) == 1;
        sorted = params.get<int>("sorted", 1) == 1;
        CV_CheckTrue(sorted, "TopK: sorted == false is not supported"); // TODO: support sorted

        CV_CheckTrue(params.has("k"), "TopK: parameter k is required but missing");
        K = params.get<int>("k");
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV;
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        const auto &input_shape = inputs.front();
        int input_dims = input_shape.size();

        // Check if axis is valid
        CV_CheckGE(axis, -input_dims, "TopK: axis is out of range");
        CV_CheckLT(axis, input_dims, "TopK: axis is out of range");
        // Normalize axis
        int axis_normalized = normalize_axis(axis, input_shape.size());

        // Check if K is in range (0, input_shape[axis])
        CV_CheckGT(K, 0, "TopK: K needs to be a positive integer");
        CV_CheckLT(K, input_shape[axis_normalized], "TopK: K is out of range");

        // Assign output shape
        auto output_shape = input_shape;
        output_shape[axis_normalized] = K;
        outputs.assign(1, output_shape);
        outputs.assign(2, output_shape); // TODO: support indices of type CV_32S on 5.x

        return false;
    }

    virtual void finalize(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr) CV_OVERRIDE {
        std::vector<Mat> inputs;
        inputs_arr.getMatVector(inputs);

        // Normalize axis
        auto input_shape = shape(inputs.front());
        axis = normalize_axis(axis, input_shape.size());
    }

    template<class Comparator>
    void FindTopK(const Mat &input, Mat &output_value, Mat &output_index) {
        const auto input_shape = shape(input);
        size_t loops = std::accumulate(input_shape.begin(), input_shape.begin() + axis, 1, std::multiplies<int>());
        size_t step = std::accumulate(input_shape.begin() + axis + 1, input_shape.end(), 1, std::multiplies<int>());
        int dim_axis = input_shape[axis];
        if (loops == 1) {
            auto worker = [&](const Range &r) {
                const auto *input_ptr = input.ptr<const float>();   // TODO: support other input type
                auto *output_value_ptr = output_value.ptr<float>();
                auto *output_index_ptr = output_index.ptr<float>(); // TODO: use CV_32S on 5.x

                Comparator cmp(input_ptr, step);

                AutoBuffer<int> buffer_index(dim_axis);
                auto *buffer_index_ptr = buffer_index.data();
                for (int offset = r.start; offset < r.end; offset++) {
                    const auto *input_offset_ptr = input_ptr + offset;
                    cmp.addOffset(offset);

                    std::iota(buffer_index_ptr, buffer_index_ptr + dim_axis, 0);
                    std::stable_sort(buffer_index_ptr, buffer_index_ptr + dim_axis, cmp);

                    auto *output_value_offset_ptr = output_value_ptr + offset;
                    auto *output_index_offset_ptr = output_index_ptr + offset;
                    for (int i = 0; i < K; i++) {
                        int source_index = buffer_index_ptr[i];
                        output_value_offset_ptr[i * step] = *(input_offset_ptr + source_index * step);
                        output_index_offset_ptr[i * step] = source_index;
                    }
                    cmp.minusOffset(offset);
                }
            };
            parallel_for_(Range(0, step), worker);
        } else {
            auto worker = [&](const Range &r) {
                const auto *input_ptr = input.ptr<const float>();
                auto *output_value_ptr = output_value.ptr<float>();
                auto *output_index_ptr = output_index.ptr<float>();

                Comparator cmp(input_ptr, step);

                AutoBuffer<int> buffer_index(dim_axis);
                auto *buffer_index_ptr = buffer_index.data();
                for (int batch_index = r.start; batch_index < r.end; batch_index++) {
                    for (size_t offset = 0; offset < step; offset++) {
                        const auto *input_offset_ptr = input_ptr + batch_index * dim_axis * step + offset;
                        cmp.addOffset(batch_index * dim_axis * step + offset);

                        std::iota(buffer_index_ptr, buffer_index_ptr + dim_axis, 0);
                        std::stable_sort(buffer_index_ptr, buffer_index_ptr + dim_axis, cmp);

                        auto *output_value_offset_ptr = output_value_ptr + batch_index * K * step + offset;
                        auto *output_index_offset_ptr = output_index_ptr + batch_index * K * step + offset;
                        for (int i = 0; i < K; i++) {
                            int source_index = buffer_index_ptr[i];
                            output_value_offset_ptr[i * step] = *(input_offset_ptr + source_index * step);
                            output_index_offset_ptr[i * step] = source_index;
                        }
                        cmp.minusOffset(batch_index * dim_axis * step + offset);
                    }
                }
            };
            parallel_for_(Range(0, loops), worker);
        }
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        if (inputs_arr.depth() == CV_16F)
        {
            forward_fallback(inputs_arr, outputs_arr, internals_arr);
            return;
        }

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        const auto &input = inputs.front();
        auto &output_value = outputs.front();
        auto &output_index = outputs.back();

        if (largest) {
            FindTopK<ComparatorGreater<float>>(input, output_value, output_index);
        } else {
            FindTopK<ComparatorLess<float>>(input, output_value, output_index);
        }
    }

private:
    int axis;
    bool largest;
    bool sorted;

    int K; // FIXIT: make it layer input once dynamic shape is supported
};

Ptr<TopKLayer> TopKLayer::create(const LayerParams& params)
{
    return makePtr<TopKLayerImpl>(params);
}

}} // namespace cv::dnn
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

- **ComparatorLess**: A class/struct defined in this file
- **Comparator**: A class/struct defined in this file
- **TopKLayerImpl**: A class/struct defined in this file
- **ComparatorGreater**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../precomp.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `layers_common.hpp`


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

