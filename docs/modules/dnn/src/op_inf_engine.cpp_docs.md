# Documentation for `modules/dnn/src/op_inf_engine.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_inf_engine.cpp`
- **File Name**: `op_inf_engine.cpp`
- **File Size**: 11,941 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/op_inf_engine.cpp](../../../modules/dnn/src/op_inf_engine.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2019, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "precomp.hpp"
#include "op_inf_engine.hpp"
#include <opencv2/dnn/shape_utils.hpp>

#ifdef HAVE_INF_ENGINE
#include <openvino/core/extension.hpp>
#elif defined(ENABLE_PLUGINS)
// using plugin API
#include "backend.hpp"
#include "factory.hpp"
#endif

#include <opencv2/core/utils/configuration.private.hpp>
#include <opencv2/core/utils/logger.hpp>

namespace cv { namespace dnn {

#ifdef HAVE_INF_ENGINE

CV__DNN_INLINE_NS_BEGIN

cv::String getInferenceEngineBackendType()
{
    return "NGRAPH";
}
cv::String setInferenceEngineBackendType(const cv::String& newBackendType)
{
    if (newBackendType != "NGRAPH")
        CV_Error(Error::StsNotImplemented, cv::format("DNN/IE: only NGRAPH backend is supported: %s", newBackendType.c_str()));
    return newBackendType;
}

CV__DNN_INLINE_NS_END

Mat infEngineBlobToMat(const ov::Tensor& blob)
{
    std::vector<size_t> dims = blob.get_shape();
    std::vector<int> size(dims.begin(), dims.end());
    auto precision = blob.get_element_type();

    int type = -1;
    switch (precision)
    {
        case ov::element::f32: type = CV_32F; break;
        case ov::element::u8: type = CV_8U; break;
        default:
            CV_Error(Error::StsNotImplemented, "Unsupported blob precision");
    }
    return Mat(size, type, blob.data());
}

void infEngineBlobsToMats(const ov::TensorVector& blobs,
                          std::vector<Mat>& mats)
{
    mats.resize(blobs.size());
    for (int i = 0; i < blobs.size(); ++i)
        mats[i] = infEngineBlobToMat(blobs[i]);
}


static bool init_IE_plugins()
{
    // load and hold IE plugins
    static ov::Core* init_core = new ov::Core();  // 'delete' is never called
    (void)init_core->get_available_devices();
    return true;
}
static ov::Core& retrieveIECore(const std::string& id, std::map<std::string, std::shared_ptr<ov::Core> >& cores)
{
    AutoLock lock(getInitializationMutex());
    std::map<std::string, std::shared_ptr<ov::Core> >::iterator i = cores.find(id);
    if (i == cores.end())
    {
        std::shared_ptr<ov::Core> core = std::make_shared<ov::Core>();
        cores[id] = core;
        return *core.get();
    }
    return *(i->second).get();
}
static ov::Core& create_IE_Core_instance(const std::string& id)
{
    static std::map<std::string, std::shared_ptr<ov::Core> > cores;
    return retrieveIECore(id, cores);
}
static ov::Core& create_IE_Core_pointer(const std::string& id)
{
    // load and hold IE plugins
    static std::map<std::string, std::shared_ptr<ov::Core> >* cores =
            new std::map<std::string, std::shared_ptr<ov::Core> >();
    return retrieveIECore(id, *cores);
}

ov::Core& getCore(const std::string& id)
{
    // to make happy memory leak tools use:
    // - OPENCV_DNN_INFERENCE_ENGINE_HOLD_PLUGINS=0
    // - OPENCV_DNN_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND=0
    static bool param_DNN_INFERENCE_ENGINE_HOLD_PLUGINS = utils::getConfigurationParameterBool("OPENCV_DNN_INFERENCE_ENGINE_HOLD_PLUGINS", true);
    static bool init_IE_plugins_ = param_DNN_INFERENCE_ENGINE_HOLD_PLUGINS && init_IE_plugins(); CV_UNUSED(init_IE_plugins_);

    static bool param_DNN_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND =
            utils::getConfigurationParameterBool("OPENCV_DNN_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND",
#ifdef _WIN32
                true
#else
                false
#endif
            );

    ov::Core& core = param_DNN_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND
            ? create_IE_Core_pointer(id)
            : create_IE_Core_instance(id);
    return core;
}


static bool detectArmPlugin_()
{
    ov::Core& ie = getCore("CPU");
    const std::vector<std::string> devices = ie.get_available_devices();
    for (std::vector<std::string>::const_iterator i = devices.begin(); i != devices.end(); ++i)
    {
        if (i->find("CPU") != std::string::npos)
        {
            const std::string name = ie.get_property(*i, ov::device::full_name);
            CV_LOG_INFO(NULL, "CPU plugin: " << name);
            return name.find("arm_compute::NEON") != std::string::npos;
        }
    }
    return false;
}

#if !defined(OPENCV_DNN_IE_VPU_TYPE_DEFAULT)
static bool detectMyriadX_(const std::string& device)
{
    AutoLock lock(getInitializationMutex());

    // Lightweight detection
    ov::Core& ie = getCore(device);
    const std::vector<std::string> devices = ie.get_available_devices();
    for (std::vector<std::string>::const_iterator i = devices.begin(); i != devices.end(); ++i)
    {
        if (i->find(device) != std::string::npos)
        {
            const std::string name = ie.get_property(*i, ov::device::full_name);
            CV_LOG_INFO(NULL, "Myriad device: " << name);
            return name.find("MyriadX") != std::string::npos || name.find("Myriad X") != std::string::npos || name.find("HDDL") != std::string::npos;
        }
    }
    return false;
}
#endif  // !defined(OPENCV_DNN_IE_VPU_TYPE_DEFAULT)

#endif  // HAVE_INF_ENGINE


CV__DNN_INLINE_NS_BEGIN

void resetMyriadDevice()
{
#ifdef HAVE_INF_ENGINE
    CV_LOG_INFO(NULL, "DNN: Unregistering both 'MYRIAD' and 'HETERO:MYRIAD,CPU' plugins");

    AutoLock lock(getInitializationMutex());

    ov::Core& ie = getCore("MYRIAD");
    try
    {
        ie.unload_plugin("MYRIAD");
        ie.unload_plugin("HETERO");
    }
    catch (...) {}
#endif  // HAVE_INF_ENGINE
}

void releaseHDDLPlugin()
{
#ifdef HAVE_INF_ENGINE
    CV_LOG_INFO(NULL, "DNN: Unregistering both 'HDDL' and 'HETERO:HDDL,CPU' plugins");

    AutoLock lock(getInitializationMutex());

    ov::Core& ie = getCore("HDDL");
    try
    {
        ie.unload_plugin("HDDL");
        ie.unload_plugin("HETERO");
    }
    catch (...) {}
#endif  // HAVE_INF_ENGINE
}

#ifdef HAVE_INF_ENGINE
bool isMyriadX()
{
    static bool myriadX = getInferenceEngineVPUType() == CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_X;
    return myriadX;
}

bool isArmComputePlugin()
{
    static bool armPlugin = getInferenceEngineCPUType() == CV_DNN_INFERENCE_ENGINE_CPU_TYPE_ARM_COMPUTE;
    return armPlugin;
}

static std::string getInferenceEngineVPUType_()
{
    static std::string param_vpu_type = utils::getConfigurationParameterString("OPENCV_DNN_IE_VPU_TYPE", "");
    if (param_vpu_type == "")
    {
#if defined(OPENCV_DNN_IE_VPU_TYPE_DEFAULT)
        param_vpu_type = OPENCV_DNN_IE_VPU_TYPE_DEFAULT;
#else
        CV_LOG_INFO(NULL, "OpenCV-DNN: running Inference Engine VPU autodetection: Myriad2/X or HDDL. In case of other accelerator types specify 'OPENCV_DNN_IE_VPU_TYPE' parameter");
        try {
            bool isMyriadX_ = detectMyriadX_("MYRIAD");
            bool isHDDL_ = detectMyriadX_("HDDL");
            if (isMyriadX_ || isHDDL_)
            {
                param_vpu_type = CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_X;
            }
            else
            {
                param_vpu_type = CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_2;
            }
        }
        catch (...)
        {
            CV_LOG_WARNING(NULL, "OpenCV-DNN: Failed Inference Engine VPU autodetection. Specify 'OPENCV_DNN_IE_VPU_TYPE' parameter.");
            param_vpu_type.clear();
        }
#endif
    }
    CV_LOG_INFO(NULL, "OpenCV-DNN: Inference Engine VPU type='" << param_vpu_type << "'");
    return param_vpu_type;
}

cv::String getInferenceEngineVPUType()
{
    static cv::String vpu_type = getInferenceEngineVPUType_();
    return vpu_type;
}

cv::String getInferenceEngineCPUType()
{
    static cv::String cpu_type = detectArmPlugin_() ?
                                 CV_DNN_INFERENCE_ENGINE_CPU_TYPE_ARM_COMPUTE :
                                 CV_DNN_INFERENCE_ENGINE_CPU_TYPE_X86;
    return cpu_type;
}


namespace openvino {

bool checkTarget(Target target)
{
    // Lightweight detection
    const std::vector<std::string> devices = getCore("").get_available_devices();
    for (std::vector<std::string>::const_iterator i = devices.begin(); i != devices.end(); ++i)
    {
        if (std::string::npos != i->find("MYRIAD") && target == DNN_TARGET_MYRIAD)
            return true;
        if (std::string::npos != i->find("HDDL") && target == DNN_TARGET_HDDL)
            return true;
        else if (std::string::npos != i->find("FPGA") && target == DNN_TARGET_FPGA)
            return true;
        else if (std::string::npos != i->find("CPU") && target == DNN_TARGET_CPU)
            return true;
        else if (std::string::npos != i->find("GPU") && (target == DNN_TARGET_OPENCL || target == DNN_TARGET_OPENCL_FP16))
            return true;
        else if (std::string::npos != i->find("NPU") && target == DNN_TARGET_NPU)
            return true;
    }
    return false;
}

}  // namespace openvino

#else  // HAVE_INF_ENGINE


namespace openvino {

bool checkTarget(Target target)
{
#if defined(ENABLE_PLUGINS)
    try
    {
        auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
        return networkBackend.checkTarget(target);
    }
    catch (const std::exception& e)
    {
        CV_LOG_INFO(NULL, "DNN/OpenVINO: checkTarget failed: " << e.what())
    }
#endif
    return false;
}

}  // namespace openvino


cv::String getInferenceEngineBackendType()
{
#if defined(ENABLE_PLUGINS)
    try
    {
        auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
        CV_UNUSED(networkBackend);
        return CV_DNN_BACKEND_INFERENCE_ENGINE_NGRAPH;
    }
    catch (const std::exception& e)
    {
        CV_LOG_INFO(NULL, "DNN/OpenVINO: plugin is not available: " << e.what())
    }
#endif
    CV_Error(Error::StsNotImplemented, "This OpenCV build doesn't include InferenceEngine support");
}
cv::String setInferenceEngineBackendType(const cv::String& newBackendType)
{
#if defined(ENABLE_PLUGINS)
    try
    {
        auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
        CV_UNUSED(networkBackend);
        CV_Assert(newBackendType == CV_DNN_BACKEND_INFERENCE_ENGINE_NGRAPH);
    }
    catch (const std::exception& e)
    {
        CV_LOG_INFO(NULL, "DNN/OpenVINO: plugin is not available: " << e.what())
    }
#endif
    CV_UNUSED(newBackendType);
    CV_Error(Error::StsNotImplemented, "This OpenCV build doesn't include InferenceEngine support");
}
cv::String getInferenceEngineVPUType()
{
#if defined(ENABLE_PLUGINS)
    try
    {
        auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
        if (networkBackend.checkTarget(DNN_TARGET_MYRIAD))
            return CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_X;  // 2021.4 supports NCS2 only
        CV_Error(Error::StsError, "DNN/OpenVINO: DNN_TARGET_MYRIAD is not available");
    }
    catch (const std::exception& e)
    {
        CV_LOG_INFO(NULL, "DNN/OpenVINO: plugin is not available: " << e.what())
    }
#endif
    CV_Error(Error::StsNotImplemented, "This OpenCV build doesn't include InferenceEngine support");
}

cv::String getInferenceEngineCPUType()
{
#if defined(ENABLE_PLUGINS)
    try
    {
        auto& networkBackend = dnn_backend::createPluginDNNNetworkBackend("openvino");
        CV_UNUSED(networkBackend);
#if defined(__arm__) || defined(__aarch64__) || defined(_M_ARM64) || defined(_M_ARM64EC)
        return CV_DNN_INFERENCE_ENGINE_CPU_TYPE_ARM_COMPUTE;
#else
        return CV_DNN_INFERENCE_ENGINE_CPU_TYPE_X86;
#endif
    }
    catch (const std::exception& e)
    {
        CV_LOG_INFO(NULL, "DNN/OpenVINO: plugin is not available: " << e.what())
    }
#endif
    CV_Error(Error::StsNotImplemented, "This OpenCV build doesn't include InferenceEngine support");
}

#endif  // HAVE_INF_ENGINE


CV__DNN_INLINE_NS_END
}}  // namespace dnn, namespace cv
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

- **_WIN32()**: A function/method defined in this file
- **HAVE_INF_ENGINE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `backend.hpp`
- `factory.hpp`
- `op_inf_engine.hpp`
- `opencv2/core/utils/logger.hpp`
- `opencv2/dnn/shape_utils.hpp`
- `openvino/core/extension.hpp`
- `opencv2/core/utils/configuration.private.hpp`
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

