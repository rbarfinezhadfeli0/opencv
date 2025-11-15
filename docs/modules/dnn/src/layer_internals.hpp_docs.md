# Documentation for `modules/dnn/src/layer_internals.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layer_internals.hpp`
- **File Name**: `layer_internals.hpp`
- **File Size**: 10,247 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/layer_internals.hpp](../../../modules/dnn/src/layer_internals.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef __OPENCV_DNN_SRC_LAYER_INTERNALS_HPP__
#define __OPENCV_DNN_SRC_LAYER_INTERNALS_HPP__

namespace cv { namespace dnn {
CV__DNN_INLINE_NS_BEGIN
inline namespace detail {

struct LayerPin
{
    int lid;
    int oid;

    LayerPin(int layerId = -1, int outputId = -1)
        : lid(layerId)
        , oid(outputId)
    {}

    bool valid() const
    {
        return (lid >= 0 && oid >= 0);
    }

    bool equal(const LayerPin& r) const
    {
        return (lid == r.lid && oid == r.oid);
    }

    bool operator<(const LayerPin& r) const
    {
        return lid < r.lid || (lid == r.lid && oid < r.oid);
    }

    bool operator==(const LayerPin& r) const
    {
        return lid == r.lid && oid == r.oid;
    }
};

struct LayerData
{
    LayerData()
        : id(-1)
        , dtype(CV_32F)
        , skip(false)
        , flag(0)
    {}
    LayerData(int _id, const String& _name, const String& _type, const int& _dtype, LayerParams& _params)
        : id(_id)
        , name(_name)
        , type(_type)
        , dtype(_dtype)
        , params(_params)
        , skip(false)
        , flag(0)
    {
        CV_TRACE_FUNCTION();

        // add logging info
        params.name = name;
        params.type = type;
    }

    int id;
    String name;
    String type;
    int dtype;  // Datatype of output blobs.
    LayerParams params;

    std::vector<LayerPin> inputBlobsId;
    std::set<int> inputLayersId;
    std::set<int> requiredOutputs;
    std::vector<LayerPin> consumers;
    std::vector<Ptr<BackendWrapper>> outputBlobsWrappers;
    std::vector<Ptr<BackendWrapper>> inputBlobsWrappers;
    std::vector<Ptr<BackendWrapper>> internalBlobsWrappers;

#ifdef HAVE_CUDA
    /* output ids which must be transferred to the host in the background
     * after the completion of the forward pass of the layer
     */
    std::vector<int> cudaD2HBackgroundTransfers;
#endif

    Ptr<Layer> layerInstance;
    std::vector<Mat> outputBlobs;
    std::vector<Mat*> inputBlobs;
    std::vector<Mat> internals;
    // Computation nodes of implemented backends (except DEFAULT).
    std::map<int, Ptr<BackendNode>> backendNodes;
    // Flag for skip layer computation for specific backend.
    bool skip;

    int flag;


    void resetAllocation()
    {
        if (id == 0)
            return;  // skip "input" layer (assertion in Net::Impl::allocateLayers)

        layerInstance.release();
        outputBlobs.clear();
        inputBlobs.clear();
        internals.clear();

        outputBlobsWrappers.clear();
        inputBlobsWrappers.clear();
        internalBlobsWrappers.clear();

        backendNodes.clear();

        skip = false;
        flag = 0;

#ifdef HAVE_CUDA
        cudaD2HBackgroundTransfers.clear();
#endif
    }
};


// fake layer containing network input blobs
struct DataLayer : public Layer
{
    DataLayer()
        : Layer()
    {
        skip = false;
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        return backendId == DNN_BACKEND_OPENCV;
    }

    void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(name, "name", name.c_str());

        // FIXIT: add wrapper without exception suppression
        CV_OCL_RUN(IS_DNN_OPENCL_TARGET(preferableTarget),
                forward_ocl(inputs_arr, outputs_arr, internals_arr))

        bool isFP16 = outputs_arr.depth() == CV_16F;

        std::vector<Mat> outputs, internals;
        outputs_arr.getMatVector(outputs);
        internals_arr.getMatVector(internals);

        for (int i = 0; i < inputsData.size(); ++i)
        {
            double scale = scaleFactors[i];
            Scalar& mean = means[i];

            CV_Assert(mean == Scalar() || inputsData[i].size[1] <= 4);
            if (isFP16)
                CV_CheckTypeEQ(outputs[i].type(), CV_16FC1, "");
            else
                CV_CheckTypeEQ(outputs[i].type(), CV_32FC1, "");

            bool singleMean = true;
            for (int j = 1; j < std::min(4, inputsData[i].size[1]) && singleMean; ++j)
            {
                singleMean = mean[j] == mean[j - 1];
            }

            if (singleMean)
            {
                if (isFP16)
                {
                    Mat input_f32;
                    inputsData[i].convertTo(input_f32, CV_32F, scale, -mean[0] * scale);
                    input_f32.convertTo(outputs[i], CV_16F);
                }
                else
                {
                    inputsData[i].convertTo(outputs[i], CV_32F, scale, -mean[0] * scale);
                }
            }
            else
            {
                for (int n = 0; n < inputsData[i].size[0]; ++n)
                {
                    for (int c = 0; c < inputsData[i].size[1]; ++c)
                    {
                        Mat inp = getPlane(inputsData[i], n, c);
                        Mat out = getPlane(outputs[i], n, c);
                        if (isFP16)
                        {
                            Mat input_f32;
                            inp.convertTo(input_f32, CV_32F, scale, -mean[c] * scale);
                            input_f32.convertTo(out, CV_16F);
                        }
                        else
                        {
                            inp.convertTo(out, CV_32F, scale, -mean[c] * scale);
                        }
                    }
                }
            }
        }
    }

#ifdef HAVE_OPENCL
    bool forward_ocl(InputArrayOfArrays, OutputArrayOfArrays outputs_, OutputArrayOfArrays internals_)
    {
        bool isFP16 = outputs_.depth() == CV_16F;

        std::vector<UMat> outputs;
        outputs_.getUMatVector(outputs);

        for (int i = 0; i < inputsData.size(); ++i)
        {
            Mat inputData = inputsData[i];

            double scale = scaleFactors[i];
            Scalar& mean = means[i];

            CV_Assert(mean == Scalar() || inputData.size[1] <= 4);
            if (isFP16)
                CV_CheckTypeEQ(outputs[i].type(), CV_16FC1, "");
            else
                CV_CheckTypeEQ(outputs[i].type(), CV_32FC1, "");

            bool singleMean = true;
            for (int j = 1; j < std::min(4, inputData.size[1]) && singleMean; ++j)
            {
                singleMean = mean[j] == mean[j - 1];
            }

            if (singleMean)
            {
                if (isFP16)
                {
                    UMat input_i;
                    inputData.convertTo(input_i, CV_32F, scale, -mean[0] * scale);
                    input_i.convertTo(outputs[i], CV_16F);
                }
                else
                {
                    inputData.convertTo(outputs[i], CV_32F, scale, -mean[0] * scale);
                }
            }
            else
            {
                for (int n = 0; n < inputData.size[0]; ++n)
                {
                    for (int c = 0; c < inputData.size[1]; ++c)
                    {
                        Mat inp = getPlane(inputData, n, c);

                        std::vector<cv::Range> plane(4, Range::all());
                        plane[0] = Range(n, n + 1);
                        plane[1] = Range(c, c + 1);
                        UMat out = outputs[i](plane).reshape(1, inp.dims, inp.size);

                        if (isFP16)
                        {
                            UMat input_i;
                            inp.convertTo(input_i, CV_32F, scale, -mean[c] * scale);
                            input_i.convertTo(out, CV_16F);
                        }
                        else
                        {
                            inp.convertTo(out, CV_32F, scale, -mean[c] * scale);
                        }
                    }
                }
            }
        }
        return true;
    }
#endif

    int outputNameToIndex(const String& tgtName) CV_OVERRIDE
    {
        int idx = (int)(std::find(outNames.begin(), outNames.end(), tgtName) - outNames.begin());
        return (idx < (int)outNames.size()) ? idx : -1;
    }

    void setNames(const std::vector<String>& names)
    {
        outNames.assign(names.begin(), names.end());
        shapes.clear();
        shapes.resize(outNames.size());
    }

    void setInputShape(const String& tgtName, const MatShape& shape)
    {
        std::vector<String>::const_iterator it = std::find(outNames.begin(), outNames.end(), tgtName);
        CV_Check(tgtName, it != outNames.end(), "Unknown input");
        int idx = (int)(it - outNames.begin());

        CV_Assert(idx < (int)shapes.size());
        CV_Check(tgtName, shapes[idx].empty(), "Input shape redefinition is not allowed");
        shapes[idx] = shape;
    }

    bool getMemoryShapes(const std::vector<MatShape>& inputs,
            const int requiredOutputs,
            std::vector<MatShape>& outputs,
            std::vector<MatShape>& internals) const CV_OVERRIDE
    {
        CV_Assert(inputs.size() == requiredOutputs);
        outputs.assign(inputs.begin(), inputs.end());
        return false;
    }

    virtual void finalize(InputArrayOfArrays, OutputArrayOfArrays outputs_arr) CV_OVERRIDE
    {
        std::vector<Mat> outputs;
        outputs_arr.getMatVector(outputs);

        CV_Assert_N(outputs.size() == scaleFactors.size(), outputs.size() == means.size(),
                inputsData.size() == outputs.size());
        skip = true;
        for (int i = 0; skip && i < inputsData.size(); ++i)
        {
            if (inputsData[i].data != outputs[i].data || scaleFactors[i] != 1.0 || means[i] != Scalar())
                skip = false;
        }
    }


    std::vector<String> outNames;
    std::vector<MatShape> shapes;
    // Preprocessing parameters for each network's input.
    std::vector<double> scaleFactors;
    std::vector<Scalar> means;
    std::vector<Mat> inputsData;
    bool skip;
};  // DataLayer


}  // namespace detail
CV__DNN_INLINE_NS_END
}}  // namespace cv::dnn
#endif  // __OPENCV_DNN_SRC_LAYER_INTERNALS_HPP__
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

- **LayerPin**: A class/struct defined in this file
- **LayerData**: A class/struct defined in this file
- **DataLayer**: A class/struct defined in this file

### Functions and Methods

- **__OPENCV_DNN_SRC_LAYER_INTERNALS_HPP__()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies


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

