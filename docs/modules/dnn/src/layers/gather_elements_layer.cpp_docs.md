# Documentation for `modules/dnn/src/layers/gather_elements_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/gather_elements_layer.cpp`
- **File Name**: `gather_elements_layer.cpp`
- **File Size**: 6,948 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/gather_elements_layer.cpp](../../../../modules/dnn/src/layers/gather_elements_layer.cpp)

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

static inline int calculateOffset(int outer_dim, const MatShape &shape_indices, int axis_skip, const MatStep &step_data) {
    int offset = 0;
    for (int axis = static_cast<int>(shape_indices.size()) - 2; axis >= 0; axis--) {
        int dim = shape_indices[axis];
        if (axis != axis_skip) {
            offset += (outer_dim % dim) * step_data[axis];
        }
        outer_dim /= dim;
    }
    return offset;
}

class GatherElementsLayerImpl CV_FINAL : public GatherElementsLayer
{
public:
    GatherElementsLayerImpl(const LayerParams& params)
    {
        setParamsFrom(params);
        axis = params.get<int>("axis", 0);
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV ||
               backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH;
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        CV_CheckEQ(inputs.size(), 2ull, "GatherElements: requires two inputs");

        const auto &data = inputs[0];
        const auto &indices = inputs[1];
        CV_CheckEQ(data.size(), indices.size(), "GatherElements: data and indices should have the same dimension");

        int normalized_axis = normalize_axis(axis, static_cast<int>(data.size()));
        CV_CheckGE(normalized_axis, 0, "GatherElements: axis out of range");
        CV_CheckLT(normalized_axis, static_cast<int>(data.size()), "GatherElements: axis out of range");
        for (size_t i = 0; i < data.size(); i++) {
            if (i != normalized_axis) {
                CV_CheckEQ(data[i], indices[i], "GatherElements: shape mismatched");
            }
        }

        outputs.assign(1, inputs[1]); // shape of output is same as indices
        return false;
    }

    virtual void finalize(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr) CV_OVERRIDE {
        std::vector<Mat> inputs;
        inputs_arr.getMatVector(inputs);

        const auto &data = inputs[0];
        axis = normalize_axis(axis, data.dims);
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

        const Mat& data = inputs[0];
        const Mat& indices = inputs[1];
        Mat& out = outputs[0];

        typeDispatch(outputs[0].type(), data, indices, out);
    }

    template <typename T>
    void forward_impl(const Mat& data_, const Mat& indices_,  Mat& out_)
    {
        const auto *ptr_data = data_.ptr<const T>();
        const auto *ptr_indices = indices_.ptr<const T>();
        auto *ptr_out = out_.ptr<T>();

        const auto shape_data = shape(data_);
        const auto &step_data = data_.step;
        const auto shape_indices = shape(indices_);

        int inner_most_dim = shape_indices.back();
        int axis_dim = shape_data[axis];
        size_t axis_step = static_cast<size_t>(step_data[axis] / sizeof(T));

        bool innermost_axis = axis == static_cast<int>(shape_data.size() - 1);

        auto fn = [&](const Range &r) {
            for (int i = r.start; i < r.end; i++) {
                auto *data = ptr_data + static_cast<size_t>(calculateOffset(i, shape_indices, axis, step_data) / sizeof(T));
                auto *indices = ptr_indices + i * inner_most_dim;
                auto *out = ptr_out + i * inner_most_dim;

                if (innermost_axis) {
                    for (int j = 0; j < inner_most_dim; j++) {
                        int index = static_cast<int>((indices[j] + axis_dim)) % axis_dim; // TODO: Check out-of-range index
                        out[j] = data[index];
                    }
                } else {
                    for (int j = 0; j < inner_most_dim; j++) {
                        int index = static_cast<int>(indices[j] + axis_dim) % axis_dim; // TODO: Check out-of-range index
                        out[j] = data[index * axis_step + j];
                    }
                }
            }
        };

        int outer_dims = total(shape_indices, 0, shape_indices.size() - 1);
        double nstripes = static_cast<size_t>(outer_dims * inner_most_dim * (1 / 1024.0));
        parallel_for_(Range(0, outer_dims), fn, nstripes);
    }

    template<typename... Args>
    inline void typeDispatch(const int type, Args&&... args)
    {
        switch (type)
        {
            case CV_8U:
                forward_impl<uint8_t>(std::forward<Args>(args)...);
                break;
            case CV_32S:
                forward_impl<int32_t>(std::forward<Args>(args)...);
                break;
            case CV_32F:
                forward_impl<float>(std::forward<Args>(args)...);
                break;
            default:
                CV_Error(cv::Error::BadDepth, "DNN/GatherElements: Unsupported type.");
        };
    }

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        int32_t indicesBoundValue = nodes[0].dynamicCast<InfEngineNgraphNode>()->node.get_shape()[axis];
        auto indicesBound = std::make_shared<ov::op::v0::Constant>(ov::element::i32, ov::Shape{}, &indicesBoundValue);
        auto indices = std::make_shared<ov::op::v0::Convert>(nodes[1].dynamicCast<InfEngineNgraphNode>()->node, ov::element::i32);
        auto indicesNonNegative = std::make_shared<ov::op::v1::Mod>(
            std::make_shared<ov::op::v1::Add>(indices, indicesBound),
            indicesBound);

        auto gatherElements = std::make_shared<ov::op::v6::GatherElements>(
            nodes[0].dynamicCast<InfEngineNgraphNode>()->node,
            indicesNonNegative,
            axis);
        return Ptr<BackendNode>(new InfEngineNgraphNode(gatherElements));
    }
#endif  // HAVE_DNN_NGRAPH

private:
    int axis;
};

Ptr<GatherElementsLayer> GatherElementsLayer::create(const LayerParams& params)
{
    return makePtr<GatherElementsLayerImpl>(params);
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

- **GatherElementsLayerImpl**: A class/struct defined in this file

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

