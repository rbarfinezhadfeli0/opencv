# Documentation for `modules/dnn/src/layers/flatten_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/flatten_layer.cpp`
- **File Name**: `flatten_layer.cpp`
- **File Size**: 9,508 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/flatten_layer.cpp](../../../../modules/dnn/src/layers/flatten_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "../precomp.hpp"
#include "layers_common.hpp"
#include "../op_cuda.hpp"
#include "../op_inf_engine.hpp"
#include "../ie_ngraph.hpp"
#include "../op_cann.hpp"

#include <float.h>
#include <algorithm>
#include <opencv2/dnn/shape_utils.hpp>

#ifdef HAVE_CUDA
#include "../cuda4dnn/primitives/reshape.hpp"
using namespace cv::dnn::cuda4dnn;
#endif


namespace cv
{
namespace dnn
{

class FlattenLayerImpl CV_FINAL : public FlattenLayer
{
public:
    FlattenLayerImpl(const LayerParams &params)
    {
        _startAxis = params.get<int>("axis", 1);
        _endAxis = params.get<int>("end_axis", -1);
        setParamsFrom(params);
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
#ifdef HAVE_INF_ENGINE
        if (backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
            return true;
#endif
        return backendId == DNN_BACKEND_OPENCV ||
               backendId == DNN_BACKEND_CUDA ||
               backendId == DNN_BACKEND_CANN;
    }

    bool getMemoryShapes(const std::vector<MatShape> &inputs,
                         const int requiredOutputs,
                         std::vector<MatShape> &outputs,
                         std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        CV_Assert(inputs.size() > 0);
        for (size_t i = 1; i < inputs.size(); i++)
        {
            CV_Assert(inputs[i] == inputs[0]);
        }

        int numAxes = inputs[0].size();
        int startAxis = normalize_axis(_startAxis, numAxes);
        int endAxis = normalize_axis(_endAxis, numAxes);

        CV_Assert(startAxis >= 0);
        CV_Assert(endAxis >= startAxis && endAxis < (int)numAxes);

        size_t flattenedDimensionSize = total(inputs[0], startAxis, endAxis + 1);

        MatShape outputShapeVec;
        for (int i = 0; i < startAxis; i++)
        {
            outputShapeVec.push_back(inputs[0][i]);
        }
        outputShapeVec.push_back(flattenedDimensionSize);
        for (size_t i = endAxis + 1; i < numAxes; i++)
        {
            outputShapeVec.push_back(inputs[0][i]);
        }

        outputs.resize(inputs.size(), outputShapeVec);

        return true;
    }

    void finalize(InputArrayOfArrays inputs_arr, OutputArrayOfArrays) CV_OVERRIDE
    {
        std::vector<Mat> inputs;
        inputs_arr.getMatVector(inputs);

        int numAxes = inputs[0].dims;
        _startAxis = normalize_axis(_startAxis, numAxes);
        _endAxis = normalize_axis(_endAxis, numAxes);
    }

#ifdef HAVE_OPENCL
    bool forward_ocl(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr)
    {
        std::vector<UMat> inpvec;
        std::vector<UMat> outputs;

        inputs_arr.getUMatVector(inpvec);
        outputs_arr.getUMatVector(outputs);

        std::vector<UMat*> inputs(inpvec.size());
        for (int i = 0; i < inpvec.size(); i++)
            inputs[i] = &inpvec[i];

        for (size_t i = 0; i < inputs.size(); i++)
        {
            MatShape outShape = shape(outputs[i]);
            UMat& output = outputs_arr.getUMatRef(i);
            output = inputs[i]->reshape(1, (int)outShape.size(), &outShape[0]);
        }

        return true;
    }
#endif

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        CV_OCL_RUN(IS_DNN_OPENCL_TARGET(preferableTarget) &&
                   outputs_arr.isUMatVector(),
                   forward_ocl(inputs_arr, outputs_arr, internals_arr))

        std::vector<Mat> inputs, outputs;
        inputs_arr.getMatVector(inputs);
        outputs_arr.getMatVector(outputs);

        for (size_t i = 0; i < inputs.size(); i++)
        {
            MatShape outShape = shape(outputs[i]);
            if (inputs[i].data != outputs[i].data)
            {
                inputs[i].reshape(1, (int)outShape.size(), &outShape[0]).copyTo(outputs[i]);
            }
        }
    }

#ifdef HAVE_CANN
    virtual Ptr<BackendNode> initCann(const std::vector<Ptr<BackendWrapper> > &inputs,
                                      const std::vector<Ptr<BackendWrapper> > &outputs,
                                      const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        auto x = inputs[0].dynamicCast<CannBackendWrapper>();
        auto x_desc = x->getTensorDesc();
        auto op_x = nodes[0].dynamicCast<CannBackendNode>()->getOp();
        auto output_desc = std::make_shared<ge::TensorDesc>(ge::Shape(), ge::FORMAT_NCHW, ge::DT_FLOAT);

        auto op = std::make_shared<ge::op::FlattenV2>(name);

        // set attributes
        int num_axes = x->host->dims;
        int start_axis = normalize_axis(_startAxis, num_axes);
        int end_axis = normalize_axis(_endAxis, num_axes);
        op->set_attr_axis(start_axis);
        op->set_attr_end_axis(end_axis);

        // set inputs
        op->set_input_x_by_name(*op_x, x->name.c_str());
        op->update_input_desc_x(*x_desc);
        // set outputs
        op->update_output_desc_y(*output_desc);

        return Ptr<BackendNode>(new CannBackendNode(op));
    }
#endif

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        auto& ieInpNode = nodes[0].dynamicCast<InfEngineNgraphNode>()->node;
        std::vector<size_t> dims = ieInpNode.get_shape();

        int numAxes = dims.size();
        int startAxis = normalize_axis(_startAxis, numAxes);
        int endAxis = normalize_axis(_endAxis, numAxes);

        CV_Assert(startAxis >= 0);
        CV_Assert(endAxis >= startAxis && endAxis < numAxes);
        int64_t flattenedDimensionSize = std::accumulate(dims.begin() + startAxis,
                                         dims.begin() + endAxis + 1, 1, std::multiplies<size_t>());

        std::vector<int64_t> outputShapeVec(dims.begin(), dims.begin() + startAxis);
        outputShapeVec.push_back(flattenedDimensionSize);
        outputShapeVec.insert(outputShapeVec.end(), dims.begin() + endAxis + 1, dims.end());

        auto shape   = std::make_shared<ov::op::v0::Constant>(ov::element::i64,
                       ov::Shape({outputShapeVec.size()}), outputShapeVec.data());
        auto reshape = std::make_shared<ov::op::v1::Reshape>(ieInpNode, shape, true);
        return Ptr<BackendNode>(new InfEngineNgraphNode(reshape));
    }
#endif  // HAVE_DNN_NGRAPH


#ifdef HAVE_CUDA
    Ptr<BackendNode> initCUDA(
        void *context_,
        const std::vector<Ptr<BackendWrapper>>& inputs,
        const std::vector<Ptr<BackendWrapper>>& outputs
    ) override
    {
        auto context = reinterpret_cast<csl::CSLContext*>(context_);
        return make_cuda_node<cuda4dnn::ReshapeOp>(preferableTarget, std::move(context->stream));
    }
#endif

    virtual bool tryQuantize(const std::vector<std::vector<float> > &scales,
                             const std::vector<std::vector<int> > &zeropoints, LayerParams& params) CV_OVERRIDE
    {
        return true;
    }

    int _startAxis;
    int _endAxis;
};

Ptr<FlattenLayer> FlattenLayer::create(const LayerParams& params)
{
    return Ptr<FlattenLayer>(new FlattenLayerImpl(params));
}

}
}
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

- **FlattenLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_INF_ENGINE()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **HAVE_CANN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../op_cann.hpp`
- `float.h`
- `../op_cuda.hpp`
- `../cuda4dnn/primitives/reshape.hpp`
- `../op_inf_engine.hpp`
- `../ie_ngraph.hpp`
- `../precomp.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `layers_common.hpp`
- `algorithm`

**Python Imports:**
- `this`


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

