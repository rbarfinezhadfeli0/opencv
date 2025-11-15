# Documentation for `modules/dnn/src/net_impl_backend.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/net_impl_backend.cpp`
- **File Name**: `net_impl_backend.cpp`
- **File Size**: 9,045 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/net_impl_backend.cpp](../../../modules/dnn/src/net_impl_backend.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#include "net_impl.hpp"
#include "legacy_backend.hpp"

#include "backend.hpp"
#include "factory.hpp"

#ifdef HAVE_CUDA
#include "cuda4dnn/init.hpp"
#endif

namespace cv {
namespace dnn {
CV__DNN_INLINE_NS_BEGIN


Ptr<BackendWrapper> Net::Impl::wrap(Mat& host)
{
    if (preferableBackend == DNN_BACKEND_OPENCV &&
            (preferableTarget == DNN_TARGET_CPU || preferableTarget == DNN_TARGET_CPU_FP16))
        return Ptr<BackendWrapper>();

    MatShape shape(host.dims);
    for (int i = 0; i < host.dims; ++i)
        shape[i] = host.size[i];

    void* data = host.data;
    if (backendWrappers.find(data) != backendWrappers.end())
    {
        Ptr<BackendWrapper> baseBuffer = backendWrappers[data];
        if (preferableBackend == DNN_BACKEND_OPENCV)
        {
#ifdef HAVE_OPENCL
            CV_Assert(IS_DNN_OPENCL_TARGET(preferableTarget));
            return OpenCLBackendWrapper::create(baseBuffer, host);
#else
            CV_Error(Error::StsInternal, "");
#endif
        }
        else if (preferableBackend == DNN_BACKEND_HALIDE)
        {
            CV_Assert(haveHalide());
#ifdef HAVE_HALIDE
            return Ptr<BackendWrapper>(new HalideBackendWrapper(baseBuffer, shape));
#endif
        }
        else if (preferableBackend == DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_2019)
        {
            CV_ERROR_DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_2019;
        }
        else if (preferableBackend == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
        {
            return wrapMat(preferableBackend, preferableTarget, host);
        }
        else if (preferableBackend == DNN_BACKEND_WEBNN)
        {
#ifdef HAVE_WEBNN
            return wrapMat(preferableBackend, preferableTarget, host);
#endif
        }
        else if (preferableBackend == DNN_BACKEND_VKCOM)
        {
#ifdef HAVE_VULKAN
            return Ptr<BackendWrapper>(new VkComBackendWrapper(baseBuffer, host));
#endif
        }
        else if (preferableBackend == DNN_BACKEND_CUDA)
        {
            CV_Assert(haveCUDA());
#ifdef HAVE_CUDA
            switch (preferableTarget)
            {
            case DNN_TARGET_CUDA:
                return CUDABackendWrapperFP32::create(baseBuffer, shape);
            case DNN_TARGET_CUDA_FP16:
                return CUDABackendWrapperFP16::create(baseBuffer, shape);
            default:
                CV_Assert(IS_DNN_CUDA_TARGET(preferableTarget));
            }
#endif
        }
        else if (preferableBackend == DNN_BACKEND_TIMVX)
        {
#ifdef HAVE_TIMVX
            return Ptr<BackendWrapper>(new TimVXBackendWrapper(baseBuffer, host));
#endif
        }
        else if (preferableBackend == DNN_BACKEND_CANN)
        {
            CV_Assert(0 && "Internal error: DNN_BACKEND_CANN must be implemented through inheritance");
        }
        else
            CV_Error(Error::StsNotImplemented, "Unknown backend identifier");
    }

    Ptr<BackendWrapper> wrapper = wrapMat(preferableBackend, preferableTarget, host);
    backendWrappers[data] = wrapper;
    return wrapper;
}


void Net::Impl::initBackend(const std::vector<LayerPin>& blobsToKeep_)
{
    CV_TRACE_FUNCTION();
    if (preferableBackend == DNN_BACKEND_OPENCV)
    {
        CV_Assert(preferableTarget == DNN_TARGET_CPU || preferableTarget == DNN_TARGET_CPU_FP16 || IS_DNN_OPENCL_TARGET(preferableTarget));
    }
    else if (preferableBackend == DNN_BACKEND_HALIDE)
    {
#ifdef HAVE_HALIDE
        initHalideBackend();
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of Halide");
#endif
    }
    else if (preferableBackend == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
    {
        CV_Assert(0 && "Inheritance must be used with OpenVINO backend");
    }
    else if (preferableBackend == DNN_BACKEND_WEBNN)
    {
#ifdef HAVE_WEBNN
        initWebnnBackend(blobsToKeep_);
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of WebNN");
#endif
    }
    else if (preferableBackend == DNN_BACKEND_VKCOM)
    {
#ifdef HAVE_VULKAN
        initVkComBackend();
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of Vulkan");
#endif
    }
    else if (preferableBackend == DNN_BACKEND_CUDA)
    {
#ifdef HAVE_CUDA
        initCUDABackend(blobsToKeep_);
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of CUDA/CUDNN");
#endif
    }
    else if (preferableBackend == DNN_BACKEND_TIMVX)
    {
#ifdef HAVE_TIMVX
        initTimVXBackend();
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of TimVX");
#endif
    }
    else if (preferableBackend == DNN_BACKEND_CANN)
    {
        CV_Assert(0 && "Internal error: DNN_BACKEND_CANN must be implemented through inheritance");
    }
    else
    {
        CV_Error(Error::StsNotImplemented, cv::format("Unknown backend identifier: %d", preferableBackend));
    }
}


void Net::Impl::setPreferableBackend(Net& net, int backendId)
{
    if (backendId == DNN_BACKEND_DEFAULT)
        backendId = (Backend)getParam_DNN_BACKEND_DEFAULT();

    if (backendId == DNN_BACKEND_INFERENCE_ENGINE)
        backendId = DNN_BACKEND_INFERENCE_ENGINE_NGRAPH;  // = getInferenceEngineBackendTypeParam();

    if (netWasQuantized && backendId != DNN_BACKEND_OPENCV && backendId != DNN_BACKEND_TIMVX &&
        backendId != DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
    {
        CV_LOG_WARNING(NULL, "DNN: Only default, TIMVX and OpenVINO backends support quantized networks");
        backendId = DNN_BACKEND_OPENCV;
    }
#ifdef HAVE_DNN_NGRAPH
    if (netWasQuantized && backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH && INF_ENGINE_VER_MAJOR_LT(INF_ENGINE_RELEASE_2023_0))
    {
        CV_LOG_WARNING(NULL, "DNN: OpenVINO 2023.0 and higher is required to supports quantized networks");
        backendId = DNN_BACKEND_OPENCV;
    }
#endif

    if (preferableBackend != backendId)
    {
        clear();
        if (backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
        {
#if defined(HAVE_INF_ENGINE)
            switchToOpenVINOBackend(net);
#elif defined(ENABLE_PLUGINS)
            auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
            networkBackend.switchBackend(net);
#else
            CV_Error(Error::StsNotImplemented, "OpenVINO backend is not available in the current OpenCV build");
#endif
        }
        else if (backendId == DNN_BACKEND_CANN)
        {
#ifdef HAVE_CANN
            switchToCannBackend(net);
#else
            CV_Error(Error::StsNotImplemented, "CANN backend is not availlable in the current OpenCV build");
#endif
        }
        else
        {
            preferableBackend = backendId;
        }
    }
}

void Net::Impl::setPreferableTarget(int targetId)
{
    if (netWasQuantized && targetId != DNN_TARGET_CPU &&
        targetId != DNN_TARGET_OPENCL && targetId != DNN_TARGET_OPENCL_FP16 && targetId != DNN_TARGET_NPU)
    {
        CV_LOG_WARNING(NULL, "DNN: Only CPU, OpenCL/OpenCL FP16 and NPU targets are supported by quantized networks");
        targetId = DNN_TARGET_CPU;
    }

    if (preferableTarget != targetId)
    {
        preferableTarget = targetId;
        if (IS_DNN_OPENCL_TARGET(targetId))
        {
#ifndef HAVE_OPENCL
#ifdef HAVE_INF_ENGINE
            if (preferableBackend == DNN_BACKEND_OPENCV)
#else
            if (preferableBackend == DNN_BACKEND_DEFAULT ||
                preferableBackend == DNN_BACKEND_OPENCV)
#endif  // HAVE_INF_ENGINE
                preferableTarget = DNN_TARGET_CPU;
#else
            bool fp16 = ocl::Device::getDefault().isExtensionSupported("cl_khr_fp16");
            if (!fp16 && targetId == DNN_TARGET_OPENCL_FP16)
                preferableTarget = DNN_TARGET_OPENCL;
#endif
        }

        if (IS_DNN_CUDA_TARGET(targetId))
        {
            preferableTarget = DNN_TARGET_CPU;
#ifdef HAVE_CUDA
            if (cuda4dnn::doesDeviceSupportFP16() && targetId == DNN_TARGET_CUDA_FP16)
                preferableTarget = DNN_TARGET_CUDA_FP16;
            else
                preferableTarget = DNN_TARGET_CUDA;
#endif
        }
#if !defined(__arm64__) || !__arm64__
        if (targetId == DNN_TARGET_CPU_FP16)
        {
            CV_LOG_WARNING(NULL, "DNN: fall back to DNN_TARGET_CPU. Only ARM v8 CPU is supported by DNN_TARGET_CPU_FP16.");
            targetId = DNN_TARGET_CPU;
        }
#endif

        clear();

        if (targetId == DNN_TARGET_CPU_FP16)
        {
            if (useWinograd) {
                CV_LOG_INFO(NULL, "DNN: DNN_TARGET_CPU_FP16 is set => Winograd convolution is disabled by default to preserve accuracy. If needed, enable it explicitly using enableWinograd(true).");
                enableWinograd(false);
            }
        }
    }
}


CV__DNN_INLINE_NS_END
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

### Functions and Methods

- **HAVE_DNN_NGRAPH()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_WEBNN()**: A function/method defined in this file
- **HAVE_HALIDE()**: A function/method defined in this file
- **HAVE_TIMVX()**: A function/method defined in this file
- **HAVE_VULKAN()**: A function/method defined in this file
- **HAVE_INF_ENGINE()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **HAVE_CANN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `backend.hpp`
- `factory.hpp`
- `net_impl.hpp`
- `cuda4dnn/init.hpp`
- `precomp.hpp`
- `legacy_backend.hpp`


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

