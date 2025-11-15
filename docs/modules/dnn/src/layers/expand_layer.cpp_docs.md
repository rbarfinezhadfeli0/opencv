# Documentation for `modules/dnn/src/layers/expand_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/expand_layer.cpp`
- **File Name**: `expand_layer.cpp`
- **File Size**: 7,234 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/expand_layer.cpp](../../../../modules/dnn/src/layers/expand_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "../op_inf_engine.hpp"
#include "../ie_ngraph.hpp"
#include <opencv2/dnn/shape_utils.hpp>

namespace cv { namespace dnn {

class ExpandLayerImpl CV_FINAL : public ExpandLayer
{
public:
    ExpandLayerImpl(const LayerParams &params) {
        setParamsFrom(params);

        // shape as param
        CV_CheckTrue(params.has("shape"), "DNN/Expand: shape is required in Expand layer initialization");
        DictValue param_shape = params.get("shape");
        int ndims_shape = param_shape.size();
        CV_CheckGT(ndims_shape, 0, "DNN/Expand: ndims of shape must be > 0");
        target_shape.resize(ndims_shape);
        for (int i = 0; i < ndims_shape; i++) {
            target_shape[i] = param_shape.get<int>(i);
        }

        // FIXME: remove when 0d/1d mat is available
        const_input_1d = params.get("const_input_1d", false);
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV ||
               backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH;
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE {
        CV_CheckGE(inputs.size(), static_cast<size_t>(1), "DNN/Expand: one input at least");
        CV_CheckLE(inputs.size(), static_cast<size_t>(2), "DNN/Expand: two input at most");
        CV_CheckFalse(target_shape.empty(), "DNN/Expand: shape must known before memory is set");

        MatShape input_shape = inputs[0]; // 1d tensor is represented as 2d mat, e.g. [3] -> [3, 1]
        if (const_input_1d) {
            input_shape = {inputs[0][0]};
        }

        auto& moreDimension = input_shape.size() > target_shape.size() ? input_shape : target_shape;
        auto& lessDimension = input_shape.size() <= target_shape.size() ? input_shape : target_shape;

        /*  Example:
                             i = 3
                               |
            moreDimension: 1 2 3 4 5, assign non-aligned dimensions to output shape
            lessDimension:     1 1 5, when dimension is aligned, check valid dimension (either equal or one of them is 1) and assign bigger one
                               |
                             j = 0 = i - (moreDimension.size() - lessDimension.size());
        */
        MatShape outputShape(moreDimension.size(), 1);
        for (int i = 0; i < moreDimension.size(); i++) {
            int d = moreDimension[i];
            int j = i - (moreDimension.size() - lessDimension.size());
            if (j >= 0) {
                if (d == 1 || lessDimension[j] == 1 || // broadcast
                    d == lessDimension[j]) {           // plain copy
                    outputShape[i] = std::max(d, lessDimension[j]);
                } else {
                    CV_Error(Error::StsBadSize, cv::format("DNN/Expand: invalid dimension, d (%d) != d (%d)", moreDimension[i], lessDimension[j]));
                }
            } else {
                outputShape[i] = d;
            }
        }
        outputs.assign(1, outputShape);
        return false;
    }

    virtual void finalize(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr) CV_OVERRIDE {
        std::vector<Mat> inputs;
        inputs_arr.getMatVector(inputs);

        const auto &input = inputs[0];
        auto input_shape = shape(input);
        if (const_input_1d) {
            input_shape = {input_shape[0]};
        }

        auto& moreDimension = input_shape.size() > target_shape.size() ? input_shape : target_shape;
        auto& lessDimension = input_shape.size() <= target_shape.size() ? input_shape : target_shape;

        MatShape final_target_shape(moreDimension.size(), 1);
        for (int i = 0; i < moreDimension.size(); i++) {
            int d = moreDimension[i];
            int j = i - (moreDimension.size() - lessDimension.size());
            if (j >= 0) {
                final_target_shape[i] = std::max(lessDimension[j], d);
            } else {
                final_target_shape[i] = d;
            }
        }
        target_shape.clear();
        target_shape = std::move(final_target_shape);
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE {
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

        int target_shape_total = std::accumulate(target_shape.begin(), target_shape.end(), 1, std::multiplies<int>());
        if (target_shape_total == inputs[0].total()) {
            const char *data = inputs[0].ptr<const char>();
            char *output = outputs[0].ptr<char>();
            int step = target_shape_total * outputs[0].elemSize();
            std::memcpy(output, data, step);
            return;
        }

        if (const_input_1d) {
            const char *data = inputs[0].ptr<const char>();
            char *output = outputs[0].ptr<char>();
            int step = target_shape.back() * outputs[0].elemSize();
            int total = std::accumulate(target_shape.begin(), target_shape.end() - 1, 1, std::multiplies<int>());
            for (int i = 0; i < total; i++) {
                std::memcpy(output + i * step, data, step);
            }
        } else {
            cv::broadcast(inputs[0], target_shape, outputs[0]);
        }
    }

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        auto input_shape = nodes[0].dynamicCast<InfEngineNgraphNode>()->node.get_shape();
        CV_CheckGE(target_shape.size(), input_shape.size(), "");

        std::vector<int32_t> output_shape(target_shape.begin(), target_shape.end());
        for (int i = 1; i < input_shape.size() + 1; ++i)
            output_shape[output_shape.size() - i] = std::max(
                (int32_t)input_shape[input_shape.size() - i],
                output_shape[output_shape.size() - i]);

        auto shape_node = std::make_shared<ov::op::v0::Constant>(ov::element::i32, ov::Shape{output_shape.size()}, output_shape.data());
        auto expand = std::make_shared<ov::op::v3::Broadcast>(nodes[0].dynamicCast<InfEngineNgraphNode>()->node, shape_node);
        return Ptr<BackendNode>(new InfEngineNgraphNode(expand));
    }
#endif  // HAVE_DNN_NGRAPH

private:
    MatShape target_shape;
    bool const_input_1d;
};

Ptr<ExpandLayer> ExpandLayer::create(const LayerParams &params) {
    return makePtr<ExpandLayerImpl>(params);
}

}}  // cv::dnn
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

- **ExpandLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../ie_ngraph.hpp`
- `../op_inf_engine.hpp`
- `../precomp.hpp`
- `opencv2/dnn/shape_utils.hpp`


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

