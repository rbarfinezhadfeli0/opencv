# Documentation for `modules/dnn/src/registry.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/registry.cpp`
- **File Name**: `registry.cpp`
- **File Size**: 5,517 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/registry.cpp](../../../modules/dnn/src/registry.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#include "op_halide.hpp"
#include "op_inf_engine.hpp"
#include "ie_ngraph.hpp"
#include "op_vkcom.hpp"
#include "op_cuda.hpp"
#include "op_webnn.hpp"
#include "op_timvx.hpp"
#include "op_cann.hpp"

#include "halide_scheduler.hpp"

#include "backend.hpp"
#include "factory.hpp"

#ifdef HAVE_CUDA
#include "cuda4dnn/init.hpp"
#endif

namespace cv {
namespace dnn {
CV__DNN_INLINE_NS_BEGIN


class BackendRegistry
{
public:
    typedef std::vector< std::pair<Backend, Target> > BackendsList;
    const BackendsList & getBackends() const { return backends; }
    static BackendRegistry & getRegistry()
    {
        static BackendRegistry impl;
        return impl;
    }


private:
    BackendRegistry()
    {
#ifdef HAVE_HALIDE
        backends.push_back(std::make_pair(DNN_BACKEND_HALIDE, DNN_TARGET_CPU));
#ifdef HAVE_OPENCL
        if (cv::ocl::useOpenCL())
            backends.push_back(std::make_pair(DNN_BACKEND_HALIDE, DNN_TARGET_OPENCL));
#endif
#endif  // HAVE_HALIDE

        bool haveBackendOpenVINO = false;
#ifdef HAVE_INF_ENGINE
        haveBackendOpenVINO = true;
#elif defined(ENABLE_PLUGINS)
        {
            auto factory = dnn_backend::createPluginDNNBackendFactory("openvino");
            if (factory)
            {
                auto backend = factory->createNetworkBackend();
                if (backend)
                    haveBackendOpenVINO = true;
            }
        }
#endif

        bool haveBackendCPU_FP16 = false;
#if defined(__arm64__) && __arm64__
        haveBackendCPU_FP16 = true;
#endif

        if (haveBackendOpenVINO && openvino::checkTarget(DNN_TARGET_CPU))
        {
            backends.push_back(std::make_pair(DNN_BACKEND_INFERENCE_ENGINE_NGRAPH, DNN_TARGET_CPU));
        }
        if (haveBackendOpenVINO && openvino::checkTarget(DNN_TARGET_MYRIAD))
        {
            backends.push_back(std::make_pair(DNN_BACKEND_INFERENCE_ENGINE_NGRAPH, DNN_TARGET_MYRIAD));
        }
        if (haveBackendOpenVINO && openvino::checkTarget(DNN_TARGET_HDDL))
        {
            backends.push_back(std::make_pair(DNN_BACKEND_INFERENCE_ENGINE_NGRAPH, DNN_TARGET_HDDL));
        }
#ifdef HAVE_OPENCL
        if (cv::ocl::useOpenCL() && ocl::Device::getDefault().isIntel())
        {
            if (haveBackendOpenVINO && openvino::checkTarget(DNN_TARGET_OPENCL))
            {
                backends.push_back(std::make_pair(DNN_BACKEND_INFERENCE_ENGINE_NGRAPH, DNN_TARGET_OPENCL));
            }
            if (haveBackendOpenVINO && openvino::checkTarget(DNN_TARGET_OPENCL_FP16))
            {
                backends.push_back(std::make_pair(DNN_BACKEND_INFERENCE_ENGINE_NGRAPH, DNN_TARGET_OPENCL_FP16));
            }
        }
#endif  // HAVE_OPENCL

#ifdef HAVE_WEBNN
        if (haveWebnn())
        {
            backends.push_back(std::make_pair(DNN_BACKEND_WEBNN, DNN_TARGET_CPU));
        }
#endif  // HAVE_WEBNN

#ifdef HAVE_OPENCL
        if (cv::ocl::useOpenCL())
        {
            backends.push_back(std::make_pair(DNN_BACKEND_OPENCV, DNN_TARGET_OPENCL));
            backends.push_back(std::make_pair(DNN_BACKEND_OPENCV, DNN_TARGET_OPENCL_FP16));
        }
#endif

        backends.push_back(std::make_pair(DNN_BACKEND_OPENCV, DNN_TARGET_CPU));

        if (haveBackendCPU_FP16)
            backends.push_back(std::make_pair(DNN_BACKEND_OPENCV, DNN_TARGET_CPU_FP16));

#ifdef HAVE_VULKAN
        if (haveVulkan())
            backends.push_back(std::make_pair(DNN_BACKEND_VKCOM, DNN_TARGET_VULKAN));
#endif

#ifdef HAVE_CUDA
        cuda4dnn::checkVersions();

        bool hasCudaCompatible = false;
        bool hasCudaFP16 = false;
        for (int i = 0; i < cuda4dnn::getDeviceCount(); i++)
        {
            if (cuda4dnn::isDeviceCompatible(i))
            {
                hasCudaCompatible = true;
                if (cuda4dnn::doesDeviceSupportFP16(i))
                {
                    hasCudaFP16 = true;
                    break; // we already have all we need here
                }
            }
        }

        if (hasCudaCompatible)
        {
            backends.push_back(std::make_pair(DNN_BACKEND_CUDA, DNN_TARGET_CUDA));
            if (hasCudaFP16)
                backends.push_back(std::make_pair(DNN_BACKEND_CUDA, DNN_TARGET_CUDA_FP16));
        }
#endif

#ifdef HAVE_TIMVX
        if (haveTimVX())
        {
            backends.push_back(std::make_pair(DNN_BACKEND_TIMVX, DNN_TARGET_NPU));
        }
#endif

#ifdef HAVE_CANN
        backends.push_back(std::make_pair(DNN_BACKEND_CANN, DNN_TARGET_NPU));
#endif
    }

    BackendsList backends;
};


std::vector<std::pair<Backend, Target>> getAvailableBackends()
{
    return BackendRegistry::getRegistry().getBackends();
}

std::vector<Target> getAvailableTargets(Backend be)
{
    if (be == DNN_BACKEND_DEFAULT)
        be = (Backend)getParam_DNN_BACKEND_DEFAULT();

    if (be == DNN_BACKEND_INFERENCE_ENGINE)
        be = DNN_BACKEND_INFERENCE_ENGINE_NGRAPH;

    std::vector<Target> result;
    const BackendRegistry::BackendsList all_backends = getAvailableBackends();
    for (BackendRegistry::BackendsList::const_iterator i = all_backends.begin(); i != all_backends.end(); ++i)
    {
        if (i->first == be)
            result.push_back(i->second);
    }
    return result;
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

### Classes and Structures

- **BackendRegistry**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
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
- `op_cuda.hpp`
- `op_vkcom.hpp`
- `op_cann.hpp`
- `cuda4dnn/init.hpp`
- `op_inf_engine.hpp`
- `op_webnn.hpp`
- `ie_ngraph.hpp`
- `halide_scheduler.hpp`
- `op_timvx.hpp`
- `op_halide.hpp`
- `precomp.hpp`


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

