# Documentation for `modules/dnn/src/legacy_backend.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/legacy_backend.cpp`
- **File Name**: `legacy_backend.cpp`
- **File Size**: 3,578 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/legacy_backend.cpp](../../../modules/dnn/src/legacy_backend.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#include "legacy_backend.hpp"

#include "op_halide.hpp"
#include "op_inf_engine.hpp"
#include "ie_ngraph.hpp"
#include "op_vkcom.hpp"
#include "op_cuda.hpp"
#include "op_webnn.hpp"
#include "op_timvx.hpp"
#include "op_cann.hpp"

namespace cv {
namespace dnn {
CV__DNN_INLINE_NS_BEGIN


BackendNode::BackendNode(int backendId)
    : backendId(backendId)
{}

BackendNode::~BackendNode() {}

BackendWrapper::BackendWrapper(int backendId, int targetId)
    : backendId(backendId)
    , targetId(targetId)
{}

BackendWrapper::BackendWrapper(int targetId, const cv::Mat& m)
{
    CV_Error(Error::StsNotImplemented,
            "Constructor of backend wrapper must be implemented");
}

BackendWrapper::BackendWrapper(const Ptr<BackendWrapper>& base, const MatShape& shape)
{
    CV_Error(Error::StsNotImplemented,
            "Constructor of backend wrapper must be implemented");
}

BackendWrapper::~BackendWrapper() {}



inline namespace detail {


Ptr<BackendWrapper> wrapMat(int backendId, int targetId, cv::Mat& m)
{
    if (backendId == DNN_BACKEND_OPENCV)
    {
        if (targetId == DNN_TARGET_CPU)
            return Ptr<BackendWrapper>();
#ifdef HAVE_OPENCL
        else if (IS_DNN_OPENCL_TARGET(targetId))
            return OpenCLBackendWrapper::create(m);
#endif
        else
            CV_Error(Error::StsNotImplemented, "Unknown/unsupported target identifier");
    }
    else if (backendId == DNN_BACKEND_HALIDE)
    {
        CV_Assert(haveHalide());
#ifdef HAVE_HALIDE
        return Ptr<BackendWrapper>(new HalideBackendWrapper(targetId, m));
#endif  // HAVE_HALIDE
    }
    else if (backendId == DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_2019)
    {
        CV_ERROR_DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_2019;
    }
    else if (backendId == DNN_BACKEND_INFERENCE_ENGINE_NGRAPH)
    {
        CV_Assert(0 && "Internal error: DNN_BACKEND_INFERENCE_ENGINE_NGRAPH must be implemented through inheritance");
    }
    else if (backendId == DNN_BACKEND_WEBNN)
    {
#ifdef HAVE_WEBNN
        return Ptr<BackendWrapper>(new WebnnBackendWrapper(targetId, m));
#else
        CV_Error(Error::StsNotImplemented, "This OpenCV version is built without support of WebNN");
#endif
    }
    else if (backendId == DNN_BACKEND_VKCOM)
    {
        CV_Assert(haveVulkan());
#ifdef HAVE_VULKAN
        return Ptr<BackendWrapper>(new VkComBackendWrapper(m));
#endif  // HAVE_VULKAN
    }
    else if (backendId == DNN_BACKEND_CUDA)
    {
        CV_Assert(haveCUDA());

#ifdef HAVE_CUDA
        switch (targetId)
        {
        case DNN_TARGET_CUDA:
            return CUDABackendWrapperFP32::create(m);
        case DNN_TARGET_CUDA_FP16:
            return CUDABackendWrapperFP16::create(m);
        default:
            CV_Assert(IS_DNN_CUDA_TARGET(targetId));
        }
#endif
    }
    else if (backendId == DNN_BACKEND_TIMVX)
    {
        CV_Assert(haveTimVX());
#ifdef HAVE_TIMVX
        return Ptr<BackendWrapper>(new TimVXBackendWrapper(m));
#endif  // HAVE_TIMVX
    }
    else if (backendId == DNN_BACKEND_CANN)
    {
        CV_Assert(0 && "Internal error: DNN_BACKEND_CANN must be implemented through inheritance");
    }
    else
        CV_Error(Error::StsNotImplemented, "Unknown backend identifier");
    return Ptr<BackendWrapper>();  // TODO Error?
}  // wrapMat()


}  // namespace detail
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

- **HAVE_WEBNN()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file
- **HAVE_HALIDE()**: A function/method defined in this file
- **HAVE_TIMVX()**: A function/method defined in this file
- **HAVE_VULKAN()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `op_cuda.hpp`
- `op_vkcom.hpp`
- `op_cann.hpp`
- `op_inf_engine.hpp`
- `op_webnn.hpp`
- `ie_ngraph.hpp`
- `op_timvx.hpp`
- `op_halide.hpp`
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

