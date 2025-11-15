# Documentation for `modules/dnn/src/op_vkcom.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_vkcom.cpp`
- **File Name**: `op_vkcom.cpp`
- **File Size**: 5,762 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/op_vkcom.cpp](../../../modules/dnn/src/op_vkcom.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "precomp.hpp"
#include <opencv2/dnn/shape_utils.hpp>
#include "op_vkcom.hpp"
#include "net_impl.hpp"

namespace cv
{
namespace dnn
{
#ifdef HAVE_VULKAN

CV__DNN_INLINE_NS_BEGIN

void Net::Impl::initVkComBackend()
{
    CV_TRACE_FUNCTION();
    CV_Assert(preferableBackend == DNN_BACKEND_VKCOM);

    context = vkcom::Context::create();

    for (MapIdToLayerData::iterator it = layers.begin(); it != layers.end(); it++)
    {
        LayerData &ld = it->second;
        Ptr<Layer> layer = ld.layerInstance;
        if (!layer->supportBackend(preferableBackend))
        {
            continue;
        }

        try
        {
            ld.backendNodes[DNN_BACKEND_VKCOM] = layer->initVkCom(ld.inputBlobsWrappers, ld.outputBlobsWrappers);
        }
        catch (const cv::Exception& e)
        {
            CV_LOG_ERROR(NULL, "initVkCom failed, fallback to CPU implementation. " << e.what());
            ld.backendNodes[DNN_BACKEND_VKCOM] = Ptr<BackendNode>();
        }
    }
}

CV__DNN_INLINE_NS_END


///////////////////////////////////////////////////////////////////////////////
int transFusedActivType(Ptr<ActivationLayer> &actLayer)
{
    if (actLayer)
    {
        Ptr<ReLULayer> activ_relu = actLayer.dynamicCast<ReLULayer>();
        Ptr<ReLU6Layer> activ_relu6 = actLayer.dynamicCast<ReLU6Layer>();

        if (!activ_relu.empty())
        {
            if (activ_relu->negativeSlope == 0.0f)
            {
                return 1; // kFusedActivRelu
            }
            else // Leaky ReLU
            {
                return -1; // kFusedActivNone
            }
        }
        else if (!activ_relu6.empty())
        {
            return 2; // kFusedActivRelu6
        }
        else
            return -1; // kFusedActivUnsupport
    }
    else
        return 0; // kFusedActivNone
}

void copyToTensor(vkcom::Tensor &dst, const Mat &src)
{
    CV_Assert(src.isContinuous() && src.type() == CV_32F);

    std::vector<int> mat_shape = shape(src);

    // The following code will copy the src data from CPU Mat to GPU VkBuffer.
    dst.reshape((const char*)src.data, mat_shape);
}

void copyToMat(Mat &dst, vkcom::Tensor &src)
{
    CV_Assert(dst.type() == CV_32F);

    std::vector<int> shape = src.getShape();
    void *data = src.map();
    Mat tmp(shape, CV_32F, data);
    tmp.copyTo(dst);
    src.unMap();
}

vkcom::Tensor VkComTensor(const Ptr<BackendWrapper>& ptr)
{
    CV_Assert(!ptr.empty());
    return ptr.dynamicCast<VkComBackendWrapper>()->getTensor();
}

void setDirty(std::vector<Ptr<BackendWrapper> >& ptrs)
{
    for (const Ptr<BackendWrapper>& ptr : ptrs)
    {
        ptr.dynamicCast<VkComBackendWrapper>()->setDeviceDirty();
    }
}

std::vector<vkcom::Tensor> VkComTensors(const std::vector<Ptr<BackendWrapper> >& ptrs)
{
    std::vector<vkcom::Tensor> vec;
    vec.reserve(ptrs.size());
    for (const Ptr<BackendWrapper>& ptr : ptrs)
    {
        vec.push_back(VkComTensor(ptr));
    }
    return vec;
}

VkComBackendNode::VkComBackendNode(const std::vector<Ptr<BackendWrapper> >& inputsWrapper,
                                   const Ptr<vkcom::OpBase>& op,
                                   const std::vector<Ptr<BackendWrapper> >& outputsWrapper)
                                   : BackendNode(DNN_BACKEND_VKCOM)
{
    operation = op;

    inputsWrapper_ = inputsWrapper;
    ins = VkComTensors(inputsWrapper_);

    outputsWrapper_ = outputsWrapper;
    outs = VkComTensors(outputsWrapper_);
}

bool VkComBackendNode::forward()
{
    for (int i = 0, n = inputsWrapper_.size(); i < n; ++i)
    {
        inputsWrapper_[i].dynamicCast<VkComBackendWrapper>()->copyToDevice();
    }

    return operation->forward(ins, outs);
}

VkComBackendWrapper::VkComBackendWrapper(Mat& m) : BackendWrapper(DNN_BACKEND_VKCOM, DNN_TARGET_VULKAN)
{
    CV_Assert(m.isContinuous());
    copyToTensor(tensor, m);
    host = &m;
    hostDirty = false;
    deviceDirty = false;
}

// Other constructor, need change the logical. The purpose is to decline the data copy.
VkComBackendWrapper::VkComBackendWrapper(const Ptr<BackendWrapper>& baseBuffer, Mat& m)
    : BackendWrapper(DNN_BACKEND_VKCOM, DNN_TARGET_VULKAN)
{
    Ptr<VkComBackendWrapper> base = baseBuffer.dynamicCast<VkComBackendWrapper>();
    CV_Assert(!base.empty());

    host = &m;
    tensor = base->tensor;
    CV_Assert(tensor.count() >= m.total());
    tensor.reshape(0, shape(m));
    hostDirty = false;
    deviceDirty = false;
}

void VkComBackendWrapper::copyToHost()
{
    if (deviceDirty)
        copyToMat(*host, tensor);
}

void VkComBackendWrapper::setHostDirty()
{
    hostDirty = true;
};

void VkComBackendWrapper::setDeviceDirty()
{
    deviceDirty = true;
};

void VkComBackendWrapper::copyToDevice()
{
    if (hostDirty)
    {
        copyToTensor(tensor, *host);
        hostDirty = false;
    }
}

vkcom::Tensor VkComBackendWrapper::getTensor()
{
    return tensor;
}

Mat* VkComBackendWrapper::getMat()
{
    return host;
}

#endif

void forwardVkCom(std::vector<Ptr<BackendWrapper> > &outputs,
                  const Ptr<BackendNode>& node)
{
#ifdef HAVE_VULKAN
    CV_Assert(!node.empty());

    Ptr<VkComBackendNode> node_ = node.dynamicCast<VkComBackendNode>();

    CV_Assert(node_->forward());
    setDirty(outputs);
#endif
}

bool haveVulkan()
{
#ifdef HAVE_VULKAN
    return vkcom::isAvailable();
#else
    return false;
#endif  // HAVE_VULKAN
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

### Functions and Methods

- **HAVE_VULKAN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `op_vkcom.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `net_impl.hpp`

**Python Imports:**
- `CPU`


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

