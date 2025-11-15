# Documentation for `modules/dnn/src/layers/crop_and_resize_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/crop_and_resize_layer.cpp`
- **File Name**: `crop_and_resize_layer.cpp`
- **File Size**: 7,713 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/crop_and_resize_layer.cpp](../../../../modules/dnn/src/layers/crop_and_resize_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2018, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.
#include "../precomp.hpp"
#include "../ie_ngraph.hpp"
#include "layers_common.hpp"

#ifdef HAVE_CUDA
#include "../cuda4dnn/primitives/crop_and_resize.hpp"
using namespace cv::dnn::cuda4dnn;
#endif

namespace cv { namespace dnn {

class CropAndResizeLayerImpl CV_FINAL : public CropAndResizeLayer
{
public:
    CropAndResizeLayerImpl(const LayerParams& params)
    {
        setParamsFrom(params);
        CV_Assert_N(params.has("width"), params.has("height"));
        outWidth = params.get<float>("width");
        outHeight = params.get<float>("height");
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV
               || backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH
               || backendId == DNN_BACKEND_CUDA
        ;
    }

    bool getMemoryShapes(const std::vector<MatShape> &inputs,
                         const int requiredOutputs,
                         std::vector<MatShape> &outputs,
                         std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        CV_Assert_N(inputs.size() == 2, inputs[0].size() == 4);
        if (inputs[0][0] != 1)
            CV_Error(Error::StsNotImplemented, "");
        outputs.resize(1, MatShape(4));
        outputs[0][0] = inputs[1][2];  // Number of bounding boxes.
        outputs[0][1] = inputs[0][1];  // Number of channels.
        outputs[0][2] = outHeight;
        outputs[0][3] = outWidth;
        return false;
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

        Mat& inp = inputs[0];
        Mat& out = outputs[0];
        Mat boxes = inputs[1].reshape(1, inputs[1].total() / 7);
        const int numChannels = inp.size[1];
        const int inpHeight = inp.size[2];
        const int inpWidth = inp.size[3];
        const int inpSpatialSize = inpHeight * inpWidth;
        const int outSpatialSize = outHeight * outWidth;
        CV_Assert_N(inp.isContinuous(), out.isContinuous());

        for (int b = 0; b < boxes.rows; ++b)
        {
            float* outDataBox = out.ptr<float>(b);
            float left = boxes.at<float>(b, 3);
            float top = boxes.at<float>(b, 4);
            float right = boxes.at<float>(b, 5);
            float bottom = boxes.at<float>(b, 6);
            float boxWidth = right - left;
            float boxHeight = bottom - top;

            float heightScale = boxHeight * static_cast<float>(inpHeight - 1) / (outHeight - 1);
            float widthScale = boxWidth * static_cast<float>(inpWidth - 1) / (outWidth - 1);
            for (int y = 0; y < outHeight; ++y)
            {
                float input_y = top * (inpHeight - 1) + y * heightScale;
                int y0 = static_cast<int>(input_y);
                const float* inpData_row0 = inp.ptr<float>(0, 0, y0);
                const float* inpData_row1 = (y0 + 1 < inpHeight) ? (inpData_row0 + inpWidth) : inpData_row0;
                for (int x = 0; x < outWidth; ++x)
                {
                    float input_x = left * (inpWidth - 1) + x * widthScale;
                    int x0 = static_cast<int>(input_x);
                    int x1 = std::min(x0 + 1, inpWidth - 1);

                    float* outData = outDataBox + y * outWidth + x;
                    const float* inpData_row0_c = inpData_row0;
                    const float* inpData_row1_c = inpData_row1;
                    for (int c = 0; c < numChannels; ++c)
                    {
                        *outData = inpData_row0_c[x0] +
                            (input_y - y0) * (inpData_row1_c[x0] - inpData_row0_c[x0]) +
                            (input_x - x0) * (inpData_row0_c[x1] - inpData_row0_c[x0] +
                            (input_y - y0) * (inpData_row1_c[x1] - inpData_row0_c[x1] - inpData_row1_c[x0] + inpData_row0_c[x0]));

                        inpData_row0_c += inpSpatialSize;
                        inpData_row1_c += inpSpatialSize;
                        outData += outSpatialSize;
                    }
                }
            }
        }
        if (boxes.rows < out.size[0])
        {
            // left = top = right = bottom = 0
            std::vector<cv::Range> dstRanges(4, Range::all());
            dstRanges[0] = Range(boxes.rows, out.size[0]);
            out(dstRanges).setTo(inp.ptr<float>(0, 0, 0)[0]);
        }
    }

#ifdef HAVE_DNN_NGRAPH
    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> >& inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        // Slice second input: from 1x1xNx7 to 1x1xNx5
        auto input = nodes[0].dynamicCast<InfEngineNgraphNode>()->node;
        auto rois = nodes[1].dynamicCast<InfEngineNgraphNode>()->node;

        auto rois_shape = rois.get_shape();
        std::vector<int64_t> dims(rois_shape.begin(), rois_shape.end()), offsets(4, 0);
        offsets[3] = 2;
        dims[3] = 7;

        auto lower_bounds = std::make_shared<ov::op::v0::Constant>(ov::element::i64,
                                             ov::Shape{offsets.size()}, offsets.data());
        auto upper_bounds = std::make_shared<ov::op::v0::Constant>(ov::element::i64,
                                             ov::Shape{dims.size()}, dims.data());
        auto strides = std::make_shared<ov::op::v0::Constant>(ov::element::i64,
                                        ov::Shape{dims.size()}, std::vector<int64_t>((int64_t)dims.size(), 1));
        auto slice = std::make_shared<ov::op::v1::StridedSlice>(rois,
                                      lower_bounds, upper_bounds, strides, std::vector<int64_t>{}, std::vector<int64_t>{});

        // Reshape rois from 4D to 2D
        std::vector<int64_t> shapeData = {dims[2], 5};
        auto shape = std::make_shared<ov::op::v0::Constant>(ov::element::i64, ov::Shape{2}, shapeData.data());
        auto reshape = std::make_shared<ov::op::v1::Reshape>(slice, shape, true);

        auto roiPooling =
            std::make_shared<ov::op::v0::ROIPooling>(input, reshape,
                                                         ov::Shape{(size_t)outHeight, (size_t)outWidth},
                                                         1.0f, "bilinear");

        return Ptr<BackendNode>(new InfEngineNgraphNode(roiPooling));
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
        return make_cuda_node<cuda4dnn::CropAndResizeOp>(preferableTarget, std::move(context->stream));
    }
#endif

private:
    int outWidth, outHeight;
};

Ptr<Layer> CropAndResizeLayer::create(const LayerParams& params)
{
    return Ptr<CropAndResizeLayer>(new CropAndResizeLayerImpl(params));
}

}  // namespace dnn
}  // namespace cv
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

- **CropAndResizeLayerImpl**: A class/struct defined in this file

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../cuda4dnn/primitives/crop_and_resize.hpp`
- `../ie_ngraph.hpp`
- `../precomp.hpp`
- `layers_common.hpp`

**Python Imports:**
- `4D`
- `1x1xNx7`


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

