# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc_engine_interface.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc_engine_interface.cpp`
- **File Name**: `preproc_engine_interface.cpp`
- **File Size**: 4,549 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc_engine_interface.cpp](../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc_engine_interface.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/engine` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#include <opencv2/gapi/streaming/onevpl/device_selector_interface.hpp>
#include "streaming/onevpl/engine/preproc_engine_interface.hpp"
#include "streaming/onevpl/engine/preproc/preproc_dispatcher.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/engine/preproc/preproc_engine.hpp"

#include "streaming/onevpl/accelerators/accel_policy_dx11.hpp"
#include "streaming/onevpl/accelerators/accel_policy_cpu.hpp"
#include "streaming/onevpl/accelerators/accel_policy_va_api.hpp"
#include "streaming/onevpl/accelerators/surface/surface.hpp"
#include "streaming/onevpl/cfg_param_device_selector.hpp"
#include "streaming/onevpl/cfg_params_parser.hpp"

#endif //HAVE_ONEVPL

#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {

template<typename SpecificPreprocEngine, typename ...PreprocEngineArgs >
std::unique_ptr<SpecificPreprocEngine>
IPreprocEngine::create_preproc_engine_impl(const PreprocEngineArgs& ...) {
    GAPI_Error("Unsupported ");
}

template <>
std::unique_ptr<onevpl::VPPPreprocDispatcher>
IPreprocEngine::create_preproc_engine_impl(const onevpl::Device &device,
                                           const onevpl::Context &context) {
    using namespace onevpl;
    cv::util::suppress_unused_warning(device);
    cv::util::suppress_unused_warning(context);
    std::unique_ptr<VPPPreprocDispatcher> dispatcher(new VPPPreprocDispatcher);
#ifdef HAVE_ONEVPL
    bool pp_is_created = false;
    switch (device.get_type()) {
        case onevpl::AccelType::DX11: {
            GAPI_LOG_INFO(nullptr, "Creating DX11 VPP preprocessing engine");
#ifdef HAVE_DIRECTX
#ifdef HAVE_D3D11
            // create GPU VPP preproc engine
            dispatcher->insert_worker<VPPPreprocEngine>(
                                std::unique_ptr<VPLAccelerationPolicy>{
                                        new VPLDX11AccelerationPolicy(
                                            std::make_shared<CfgParamDeviceSelector>(
                                                    device, context, CfgParams{}))
                                });
            GAPI_LOG_INFO(nullptr, "DX11 VPP preprocessing engine created");
            pp_is_created = true;
#endif
#endif
            break;
        }
        case onevpl::AccelType::VAAPI: {
            GAPI_LOG_INFO(nullptr, "Creating VAAPI VPP preprocessing engine");
#ifdef __linux__
#if defined(HAVE_VA) || defined(HAVE_VA_INTEL)
            // create GPU VPP preproc engine
            dispatcher->insert_worker<VPPPreprocEngine>(
                                std::unique_ptr<VPLAccelerationPolicy>{
                                        new VPLVAAPIAccelerationPolicy(
                                            std::make_shared<CfgParamDeviceSelector>(
                                                    device, context, CfgParams{}))
                                });
            GAPI_LOG_INFO(nullptr, "VAAPI VPP preprocessing engine created");
            pp_is_created = true;
#endif // defined(HAVE_VA) || defined(HAVE_VA_INTEL)
#endif // #ifdef __linux__
            break;
        }
        default: {
            GAPI_LOG_INFO(nullptr, "Creating CPU VPP preprocessing engine");
            dispatcher->insert_worker<VPPPreprocEngine>(
                        std::unique_ptr<VPLAccelerationPolicy>{
                                new VPLCPUAccelerationPolicy(
                                    std::make_shared<CfgParamDeviceSelector>(CfgParams{}))});
            GAPI_LOG_INFO(nullptr, "CPU VPP preprocessing engine created");
            pp_is_created = true;
            break;
        }
    }
    if (!pp_is_created) {
        GAPI_LOG_WARNING(nullptr, "Cannot create VPP preprocessing engine: configuration unsupported");
        GAPI_Error("VPP preproc unsupported");
    }
#endif // HAVE_ONEVPL
    return dispatcher;
}


// Force instantiation
template
std::unique_ptr<onevpl::VPPPreprocDispatcher>
IPreprocEngine::create_preproc_engine_impl<onevpl::VPPPreprocDispatcher,
                                           const onevpl::Device &,const onevpl::Context &>
                                          (const onevpl::Device &device,
                                           const onevpl::Context &ctx);
} // namespace wip
} // namespace gapi
} // namespace cv
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

- **HAVE_ONEVPL()**: A function/method defined in this file
- **__linux__()**: A function/method defined in this file
- **HAVE_D3D11()**: A function/method defined in this file
- **HAVE_DIRECTX()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/engine/preproc_engine_interface.hpp`
- `streaming/onevpl/accelerators/accel_policy_va_api.hpp`
- `streaming/onevpl/cfg_param_device_selector.hpp`
- `streaming/onevpl/onevpl_export.hpp`
- `opencv2/gapi/streaming/onevpl/device_selector_interface.hpp`
- `streaming/onevpl/engine/preproc/preproc_engine.hpp`
- `streaming/onevpl/accelerators/accel_policy_dx11.hpp`
- `streaming/onevpl/accelerators/accel_policy_cpu.hpp`
- `streaming/onevpl/cfg_params_parser.hpp`
- `streaming/onevpl/engine/preproc/preproc_dispatcher.hpp`
- `streaming/onevpl/accelerators/surface/surface.hpp`
- `logger.hpp`


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

