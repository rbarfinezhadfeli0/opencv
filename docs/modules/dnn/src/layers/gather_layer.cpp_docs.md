# Documentation for `modules/dnn/src/layers/gather_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/gather_layer.cpp`
- **File Name**: `gather_layer.cpp`
- **File Size**: 5,170 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/gather_layer.cpp](../../../../modules/dnn/src/layers/gather_layer.cpp)

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
#include "layers_common.hpp"


namespace cv { namespace dnn {

class GatherLayerImpl CV_FINAL : public GatherLayer
{
public:
    GatherLayerImpl(const LayerParams& params)
    {
        setParamsFrom(params);
        m_axis = params.get<int>("axis", 0);
        m_real_ndims = params.get<int>("real_ndims", -1);
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
        CV_CheckEQ(inputs.size(), 2ull, "");
        MatShape inpShape = inputs[0];
        const int axis = normalize_axis(m_axis, inpShape);

        inpShape.erase(inpShape.begin() + axis);
        auto end = m_real_ndims == -1 ? inputs[1].end() : inputs[1].begin() + m_real_ndims;
        inpShape.insert(inpShape.begin() + axis, inputs[1].begin(), end);

        outputs.assign(1, inpShape);
        return false;
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        // FP16 fallback is not needed as we handle FP16 below

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        CV_CheckEQ(inputs.size(), (size_t)2, "");
        CV_CheckEQ(outputs.size(), (size_t)1, "");

        const Mat& inp = inputs[0];

        int indicesType = inputs[1].type();
        CV_CheckType(indicesType, indicesType == CV_32FC1 || indicesType == CV_16FC1, "");
        Mat indices32S;
        if (indicesType == CV_16F/*FP16*/)
        {
            Mat indicesF32;
            inputs[1].convertTo(indicesF32, CV_32F);
            indicesF32.convertTo(indices32S, CV_32S);
        }
        else
        {
            inputs[1].convertTo(indices32S, CV_32S);
        }
        const size_t indices_total = indices32S.total();
        indices32S = indices32S.reshape(1, indices_total);

        Mat& out = outputs[0];

        CV_CheckTypeEQ(inp.type(), out.type(), "");
        CV_CheckTypeEQ(indices32S.type(), CV_32SC1, "");

        const int axis = normalize_axis(m_axis, shape(inp));

        // FIXIT: why should we work with non-normalized input? it should be handled in importer or layers's output generator
        const int axis_size = (int)inp.size[axis];
        for (size_t j = 0 ; j < indices_total; ++j)
        {
            int& idx = indices32S.at<int>(j);
            idx = normalize_axis(idx, axis_size);  // validate and normalize indices
        }

        const size_t outer_size = axis == 0 ? inp.total() : inp.step1(axis - 1);
        const size_t outer_dims = inp.total() / outer_size;
        const size_t inner_size = inp.step1(axis);

        const int* idx = indices32S.ptr<int>();
        const char* src = inp.ptr<const char>();
        char* dst = out.ptr<char>();
        CV_CheckEQ(out.total(), outer_dims * indices_total * inner_size, "");

        const size_t es = inp.elemSize1();
        // TODO: optimize through switch (inner_size * es)
        const size_t inner_bytes = inner_size * es;
        for (size_t i = 0; i < outer_dims; ++i)
        {
            const size_t src_offset = i * outer_size;
            for (size_t j = 0 ; j < indices_total; ++j)
            {
                const int index = idx[j];
                CV_DbgCheck(index, index >= 0 && index < axis_size, "");
                const size_t new_offset = src_offset + index * inner_size;
                std::memcpy(dst, src + new_offset * es, inner_bytes);
                dst += inner_bytes;
            }
        }
    }

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        auto axisNode = std::make_shared<ov::op::v0::Constant>(ov::element::i32, ov::Shape{}, &m_axis);
        auto gather = std::make_shared<ov::op::v8::Gather>(
            nodes[0].dynamicCast<InfEngineNgraphNode>()->node,
            std::make_shared<ov::op::v0::Convert>(nodes[1].dynamicCast<InfEngineNgraphNode>()->node, ov::element::i32),
            axisNode);
        return Ptr<BackendNode>(new InfEngineNgraphNode(gather));
    }
#endif  // HAVE_DNN_NGRAPH

private:
    // The axis to gather along
    int m_axis;
    int m_real_ndims;
};

Ptr<GatherLayer> GatherLayer::create(const LayerParams& params)
{
    return makePtr<GatherLayerImpl>(params);
}

}}  // namespace cv::dnn
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

- **GatherLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../ie_ngraph.hpp`
- `../op_inf_engine.hpp`
- `../precomp.hpp`
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

