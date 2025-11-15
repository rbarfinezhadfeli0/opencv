# Documentation for `modules/dnn/src/net_impl.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/net_impl.hpp`
- **File Name**: `net_impl.hpp`
- **File Size**: 9,467 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/net_impl.hpp](../../../modules/dnn/src/net_impl.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef __OPENCV_DNN_SRC_NET_IMPL_HPP__
#define __OPENCV_DNN_SRC_NET_IMPL_HPP__

#include "op_halide.hpp"
#include "op_inf_engine.hpp"
#include "ie_ngraph.hpp"
#include "op_vkcom.hpp"
#include "op_cuda.hpp"
#include "op_webnn.hpp"
#include "op_timvx.hpp"
#include "op_cann.hpp"

#include <opencv2/dnn/shape_utils.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/dnn/layer_reg.private.hpp>

#include <opencv2/core/utils/fp_control_utils.hpp>

#include <opencv2/core/utils/logger.hpp>

#include "layer_internals.hpp"  // LayerPin LayerData DataLayer

#include "legacy_backend.hpp"  // wrapMat BlobManager OpenCLBackendWrapper

namespace cv {
namespace dnn {
CV__DNN_INLINE_NS_BEGIN

using std::make_pair;
using std::string;

// NB: Implementation is divided between of multiple .cpp files
struct Net::Impl : public detail::NetImplBase
{
    typedef std::map<int, LayerShapes> LayersShapesMap;
    typedef std::map<int, LayerData> MapIdToLayerData;

    virtual ~Impl();
    Impl();
    Impl(const Impl&) = delete;

    // Inheritance support
    Ptr<Net::Impl> basePtr_;

    Ptr<DataLayer> netInputLayer;
    std::vector<LayerPin> blobsToKeep;
    MapIdToLayerData layers;
    std::map<String, int> layerNameToId;
    std::map<std::string, int> outputNameToId;  // use registerOutput() to populate outputs
    BlobManager blobManager;
    int preferableBackend;
    int preferableTarget;
    String halideConfigFile;
    bool hasDynamicShapes;
    // Map host data to backend specific wrapper.
    std::map<void*, Ptr<BackendWrapper>> backendWrappers;

    int lastLayerId;

    bool netWasAllocated;
    bool netWasQuantized;
    bool fusion;
    bool isAsync;  // FIXIT: drop
    bool useWinograd;
    std::vector<int64> layersTimings;


    virtual bool empty() const;
    virtual void setPreferableBackend(Net& net, int backendId);
    virtual void setPreferableTarget(int targetId);

    // FIXIT use inheritance
    virtual Ptr<BackendWrapper> wrap(Mat& host);


    virtual void clear();


    virtual void validateBackendAndTarget();

    void setUpNet(const std::vector<LayerPin>& blobsToKeep_ = std::vector<LayerPin>());


    virtual Ptr<Layer> createLayerInstance(const LayerData& ld) const
    {
        return LayerFactory::createLayerInstance(ld.type, const_cast<LayerParams&>(ld.params));
    }
    Ptr<Layer> getLayerInstance(LayerData& ld) const
    {
        CV_TRACE_FUNCTION();
        CV_TRACE_ARG_VALUE(type, "type", ld.type.c_str());

        if (ld.layerInstance)
            return ld.layerInstance;

        ld.layerInstance = createLayerInstance(ld);
        if (!ld.layerInstance && basePtr_)
        {
            ld.layerInstance = basePtr_->createLayerInstance(ld);
            CV_LOG_IF_DEBUG(NULL, ld.layerInstance, "Created layer \"" + ld.name + "\" of type \"" + ld.type + "\" from upstream layers registry");
        }
        if (!ld.layerInstance)
        {
            CV_Error(Error::StsError, "Can't create layer \"" + ld.name + "\" of type \"" + ld.type + "\"");
        }

        return ld.layerInstance;
    }

    Ptr<Layer> getLayer(int layerId) const;
    Ptr<Layer> getLayer(const LayerId& layerId) const;

    int getLayerId(const String& layerName) const;

    int getLayerId(int id) const;

    int getLayerId(DictValue& layerDesc) const;

    String getLayerName(int id) const;

    LayerData& getLayerData(int id) const;

    LayerData& getLayerData(const String& layerName) const;

    LayerData& getLayerData(const DictValue& layerDesc) const;

    static void addLayerInput(LayerData& ld, int inNum, LayerPin from);

    int resolvePinOutputName(LayerData& ld, const String& outName) const;

    LayerPin getPinByAlias(const String& layerName) const;

    std::vector<LayerPin> getLayerOutPins(const String& layerName) const;

    // FIXIT remove dtype
    int addLayer(const String& name, const String& type, const int& dtype, LayerParams& params);

    int addLayerToPrev(const String& name, const String& type, const int& dtype, LayerParams& params);


    void connect(int outLayerId, int outNum, int inLayerId, int inNum);

    int registerOutput(const std::string& outputName, int layerId, int outputPort);

    // FIXIT drop "unconnected" API
    std::vector<int> getUnconnectedOutLayers() const;
    std::vector<String> getUnconnectedOutLayersNames() /*const*/;


    void setInputsNames(const std::vector<String>& inputBlobNames);
    void setInputShape(const String& inputName, const MatShape& shape);
    virtual void setInput(InputArray blob, const String& name, double scalefactor, const Scalar& mean);
    Mat getParam(int layer, int numParam) const;
    void setParam(int layer, int numParam, const Mat& blob);
    std::vector<Ptr<Layer>> getLayerInputs(int layerId) const;
    std::vector<String> getLayerNames() const;


    // TODO drop?
    void getLayerTypes(std::vector<String>& layersTypes) const;
    int getLayersCount(const String& layerType) const;


    virtual void initBackend(const std::vector<LayerPin>& blobsToKeep_);

    void setHalideScheduler(const String& scheduler);
#ifdef HAVE_HALIDE
    void compileHalide();
    void initHalideBackend();
#endif

#ifdef HAVE_WEBNN
    void addWebnnOutputs(LayerData& ld);
    void initWebnnBackend(const std::vector<LayerPin>& blobsToKeep_);
#endif

#ifdef HAVE_VULKAN
    Ptr<vkcom::Context> context;
    void initVkComBackend();
#endif

#ifdef HAVE_TIMVX
    // Create timVxInfo for reserve tvGraphList.
    TimVXInfo timVxInfo = TimVXInfo();
    void tvUpdateConfictMap(int graphIndex, LayerData& ld, std::vector<std::vector<int> >& graphConflictMap);
    void tvConvertToOutputNode(const LayerData& ld, Ptr<TimVXBackendWrapper>& targetWrap);
    void initTimVXBackend();
#endif

#ifdef HAVE_CUDA
    struct CudaInfo_t
    {
        CudaInfo_t(cuda4dnn::csl::CSLContext ctxt, cuda4dnn::csl::Stream d2h_stream_)
            : context(std::move(ctxt))
            , d2h_stream(std::move(d2h_stream_))
        {}
        cuda4dnn::csl::CSLContext context;
        cuda4dnn::csl::Stream d2h_stream;
        cuda4dnn::csl::Workspace workspace;
    };

    std::unique_ptr<CudaInfo_t> cudaInfo;

    void initCUDABackend(const std::vector<LayerPin>& blobsToKeep_);
#endif

    void allocateLayer(int lid, const LayersShapesMap& layersShapes);

    // TODO add getter
    void enableFusion(bool fusion_);

    virtual void fuseLayers(const std::vector<LayerPin>& blobsToKeep_);
    void enableWinograd(bool useWinograd_);

    void allocateLayers(const std::vector<LayerPin>& blobsToKeep_);

    virtual void forwardLayer(LayerData& ld);

    void forwardToLayer(LayerData& ld, bool clearFlags = true);

    Mat forward(const String& outputName);
    AsyncArray forwardAsync(const String& outputName);
    void forward(OutputArrayOfArrays outputBlobs, const String& outputName);
    void forward(OutputArrayOfArrays outputBlobs,
            const std::vector<String>& outBlobNames);
    void forward(std::vector<std::vector<Mat>>& outputBlobs,
            const std::vector<String>& outBlobNames);


    void getLayerShapesRecursively(int id, LayersShapesMap& inOutShapes);

    void getLayersShapes(
            const ShapesVec& netInputShapes,
            std::vector<int>& layersIds,
            std::vector<ShapesVec>& inLayersShapes,
            std::vector<ShapesVec>& outLayersShapes) /*const*/;

    void getLayersShapes(const ShapesVec& netInputShapes,
            LayersShapesMap& inOutShapes);

    void getLayerShapes(const ShapesVec& netInputShapes,
            const int layerId,
            LayerShapes& shapes);

    void updateLayersShapes();

    int64 getFLOPS(const std::vector<MatShape>& netInputShapes) /*const*/;
    int64 getFLOPS(
            const int layerId,
            const std::vector<MatShape>& netInputShapes) /*const*/;

    void getMemoryConsumption(
            const int layerId,
            const std::vector<MatShape>& netInputShapes,
            size_t& weights, size_t& blobs) /*const*/;
    void getMemoryConsumption(
            const std::vector<MatShape>& netInputShapes,
            size_t& weights, size_t& blobs) /*const*/;
    void getMemoryConsumption(
            const std::vector<MatShape>& netInputShapes,
            std::vector<int>& layerIds, std::vector<size_t>& weights,
            std::vector<size_t>& blobs) /*const*/;
    int64 getPerfProfile(std::vector<double>& timings) const;

    // TODO drop
    LayerPin getLatestLayerPin(const std::vector<LayerPin>& pins) const;

    Mat getBlob(const LayerPin& pin) const;

    Mat getBlob(String outputName) const;

    virtual AsyncArray getBlobAsync(const LayerPin& pin);

    AsyncArray getBlobAsync(String outputName);

    string dump(bool forceAllocation = false) const;
    string dumpToPbtxt(bool forceAllocation = false) const;

    void dumpNetworkToFile() const;

    // FIXIT drop from inference API
    Net quantize(Net& net, InputArrayOfArrays calibData, int inputsDtype, int outputsDtype, bool perChannel) /*const*/;
    void getInputDetails(std::vector<float>& scales, std::vector<int>& zeropoints) /*const*/;
    void getOutputDetails(std::vector<float>& scales, std::vector<int>& zeropoints) /*const*/;

};  // Net::Impl


CV__DNN_INLINE_NS_END
}}  // namespace cv::dnn
#endif  // __OPENCV_DNN_SRC_NET_IMPL_HPP__
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

- **Net**: A class/struct defined in this file
- **CudaInfo_t**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **HAVE_WEBNN()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_HALIDE()**: A function/method defined in this file
- **HAVE_TIMVX()**: A function/method defined in this file
- **HAVE_VULKAN()**: A function/method defined in this file
- **__OPENCV_DNN_SRC_NET_IMPL_HPP__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `op_cuda.hpp`
- `opencv2/core/utils/fp_control_utils.hpp`
- `layer_internals.hpp`
- `op_vkcom.hpp`
- `op_cann.hpp`
- `op_inf_engine.hpp`
- `opencv2/core/utils/logger.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `op_webnn.hpp`
- `opencv2/dnn/layer_reg.private.hpp`
- `ie_ngraph.hpp`
- `opencv2/imgproc.hpp`
- `op_timvx.hpp`
- `op_halide.hpp`
- `legacy_backend.hpp`

**Python Imports:**
- `inference`
- `upstream`


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

