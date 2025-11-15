# Documentation for `modules/dnn/src/layers/scatterND_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/scatterND_layer.cpp`
- **File Name**: `scatterND_layer.cpp`
- **File Size**: 8,858 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/scatterND_layer.cpp](../../../../modules/dnn/src/layers/scatterND_layer.cpp)

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

#include <algorithm> // for std::max & std::min

namespace cv { namespace dnn {

class ScatterNDLayerImpl CV_FINAL : public ScatterNDLayer
{
public:
    enum class REDUCTION
    {
        NONE = 1,
        ADD,
        MUL,
        MAX,
        MIN
    } reduction;

    ScatterNDLayerImpl(const LayerParams& params)
    {
        setParamsFrom(params);

        String reduction_name = toLowerCase(params.get<String>("reduction", "none"));
        if (reduction_name == "none")
            reduction = REDUCTION::NONE;
        else if (reduction_name == "add")
            reduction = REDUCTION::ADD;
        else if (reduction_name == "mul")
            reduction = REDUCTION::MUL;
        else if (reduction_name == "max")
            reduction = REDUCTION::MAX;
        else if (reduction_name == "min")
            reduction = REDUCTION::MIN;
        else
            CV_Error(cv::Error::StsBadArg, "Unkown reduction \"" + reduction_name + "\"");
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV ||
               (backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH && reduction == REDUCTION::NONE);
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        CV_CheckEQ(inputs.size(), 3ull, "ScatterND: require three inputs.");

        size_t r = inputs[0].size(), q = inputs[1].size(), p = inputs[2].size(), k = inputs[1].back();
        CV_CheckEQ(r + q - inputs[1].back() - 1, p, "ScatterND: updates should have rank of data.dims + indices.dims - indices.size[-1] - 1");
        CV_CheckLE(k, r, "ScatterND: indices.shape[-1] must be less than (or equal to) the rank of input data.");

        for (int i = 0; i < q - 1; i++) // np.ndindex(indices.shape[-1])
        {
            CV_CheckEQ(inputs[2][i], inputs[1][i], "ScatterND: updates.shape[0 : rank(indices)-1] must equal to indices.shape[0 : rank(indices)-1].");
        }
        for (int i = q - 1, j = k, m = 0; i + m < p; m++)
        {
            CV_CheckEQ(inputs[2][i + m], inputs[0][j + m], "ScatterND: updates.shape[rank(indices)-1 : ] must equal to data[indices.shape[-1] : rank(data)-1].");
        }

        outputs.assign(1, inputs[0]);
        return false;
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        if (inputs_arr.depth() == CV_16F) {
            forward_fallback(inputs_arr, outputs_arr, internals_arr);
            return;
        }

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        const Mat& data = inputs[0];
        const Mat& indices = inputs[1];
        const Mat& updates = inputs[2];
        Mat& out = outputs[0];

        typeDispatch(outputs[0].type(), data, indices, updates, out);
    }

    // NOTE: This impl does not check whether indices have duplicate entries.
    //       The last duplicate entry will overwrite the previous.
    template<typename T, typename Functor>
    void forward_impl(const Functor &reduce_operation, const Mat &input_mat, const Mat &indices_mat, const Mat &updates_mat, Mat& output_mat) {
        input_mat.copyTo(output_mat);

        const auto &input_mat_shape = shape(input_mat);
        std::vector<size_t> input_mat_step(input_mat_shape.size());
        for (int i = 0; i < input_mat.dims; i++) {
            input_mat_step[i] = static_cast<size_t>(input_mat.step.p[i] / sizeof(T));
        }

        const int indices_mat_ndims = indices_mat.dims;
        const auto &indices_mat_shape = shape(indices_mat);

        const int updates_mat_ndims = updates_mat.dims;
        const auto &updates_mat_shape = shape(updates_mat);

        int indices_last_dim = indices_mat_shape[indices_mat_ndims - 1]; // last dim of indices

        size_t updates_size = 1;
        for (int i = indices_mat_ndims - 1; i < updates_mat_ndims; i++)
            updates_size *= updates_mat_shape[i];

        auto fn = [&](const Range &r) {
            size_t input_offset = 0,
                   indices_offset = r.start * indices_last_dim,
                   updates_offset = r.start * updates_size;
            for (int i = r.start; i < r.end; i++) {
                const T* indices = indices_mat.ptr<const T>();
                const T* updates = updates_mat.ptr<const T>();
                T* output = output_mat.ptr<T>();

                input_offset = 0;
                indices += indices_offset;
                for (int j = 0; j < indices_last_dim; j++) {
                    int index = static_cast<int>(*(indices + j));
                    index = (index + input_mat_shape[j]) % input_mat_shape[j];
                    CV_Assert(index < input_mat_shape[j] && index >= 0);
                    input_offset += index * input_mat_step[j];
                }

                updates += updates_offset;
                output += input_offset;
                for (int j = 0; j < updates_size; j++) {
                    output[j] = reduce_operation(output[j], updates[j]);
                }

                indices_offset += indices_last_dim;
                updates_offset += updates_size;
            }
        };

        size_t total = (size_t)(indices_mat.total() / indices_last_dim);
        double nstripes = (size_t)total * (indices_last_dim + updates_size) * (1 / 1024.0);
        parallel_for_(Range(0, total), fn, nstripes);
    }

    template<typename... Args>
    inline void typeDispatch(const int type, Args&&... args)
    {
        switch (type)
        {
            case CV_8U:
                reductionDispatch<uint8_t>(std::forward<Args>(args)...);
                break;
            case CV_32S:
                reductionDispatch<int32_t>(std::forward<Args>(args)...);
                break;
            case CV_32F:
                reductionDispatch<float>(std::forward<Args>(args)...);
                break;
            default:
                CV_Error(cv::Error::BadDepth, "Unsupported type.");
        };
    }

    template<typename T, typename... Args>
    inline void reductionDispatch(Args&&... args)
    {
        switch (reduction)
        {
            case REDUCTION::NONE:
            {
                auto rd = [](const T& a, const T& b) { return b; }; // a from input data, b from updates
                forward_impl<T>(rd, std::forward<Args>(args)...);
                break;
            }
            case REDUCTION::ADD:
            {
                auto rd = [](const T& a, const T& b) { return a + b; };
                forward_impl<T>(rd, std::forward<Args>(args)...);
                break;
            }
            case REDUCTION::MUL:
            {
                auto rd = [](const T& a, const T& b) { return a * b; };
                forward_impl<T>(rd, std::forward<Args>(args)...);
                break;
            }
            case REDUCTION::MAX:
            {
                auto rd = [](const T& a, const T& b) { return std::max(a, b); };
                forward_impl<T>(rd, std::forward<Args>(args)...);
                break;
            }
            case REDUCTION::MIN:
            {
                auto rd = [](const T& a, const T& b) { return std::min(a, b); };
                forward_impl<T>(rd, std::forward<Args>(args)...);
                break;
            }
            default:
                CV_Error(Error::StsBadArg, "Unsupported reduction.");
        };
    }

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        auto scatterND = std::make_shared<ov::op::v3::ScatterNDUpdate>(
            nodes[0].dynamicCast<InfEngineNgraphNode>()->node,
            std::make_shared<ov::op::v0::Convert>(nodes[1].dynamicCast<InfEngineNgraphNode>()->node, ov::element::i32),
            nodes[2].dynamicCast<InfEngineNgraphNode>()->node);
        return Ptr<BackendNode>(new InfEngineNgraphNode(scatterND));
    }
#endif  // HAVE_DNN_NGRAPH
};

Ptr<ScatterNDLayer> ScatterNDLayer::create(const LayerParams& params)
{
    return makePtr<ScatterNDLayerImpl>(params);
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

- **REDUCTION**: A class/struct defined in this file
- **ScatterNDLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../op_inf_engine.hpp`
- `../ie_ngraph.hpp`
- `../precomp.hpp`
- `layers_common.hpp`
- `algorithm`

**Python Imports:**
- `updates`
- `input`


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

