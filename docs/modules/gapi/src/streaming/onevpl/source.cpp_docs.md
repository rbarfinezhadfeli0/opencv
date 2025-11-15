# Documentation for `modules/gapi/src/streaming/onevpl/source.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/source.cpp`
- **File Name**: `source.cpp`
- **File Size**: 4,486 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/source.cpp](../../../../../modules/gapi/src/streaming/onevpl/source.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include <opencv2/gapi/streaming/onevpl/source.hpp>

#include "streaming/onevpl/source_priv.hpp"
#include "streaming/onevpl/data_provider_dispatcher.hpp"
#include "streaming/onevpl/cfg_param_device_selector.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

#ifdef HAVE_ONEVPL
GSource::GSource(const std::string& filePath, const CfgParams& cfg_params) :
    GSource(filePath, cfg_params, std::make_shared<CfgParamDeviceSelector>(cfg_params)) {
    if (filePath.empty()) {
        util::throw_error(std::logic_error("Cannot create 'GSource' on empty source file name"));
    }
}

GSource::GSource(const std::string& filePath,
                 const CfgParams& cfg_params,
                 const std::string& device_id,
                 void* accel_device_ptr,
                 void* accel_ctx_ptr) :
    GSource(filePath, cfg_params,
            std::make_shared<CfgParamDeviceSelector>(accel_device_ptr, device_id,
                                                     accel_ctx_ptr, cfg_params)) {
}

GSource::GSource(const std::string& filePath,
                 const CfgParams& cfg_params,
                 const Device &device, const Context &ctx) :
    GSource(filePath, update_param_with_accel_type(CfgParams{cfg_params}, device.get_type()),
            std::make_shared<CfgParamDeviceSelector>(device, ctx, cfg_params)) {
}

GSource::GSource(const std::string& filePath,
                 const CfgParams& cfg_params,
                 std::shared_ptr<IDeviceSelector> selector) :
    GSource(DataProviderDispatcher::create(filePath, cfg_params), cfg_params, selector) {
    if (filePath.empty()) {
        util::throw_error(std::logic_error("Cannot create 'GSource' on empty source file name"));
    }
}

GSource::GSource(std::shared_ptr<IDataProvider> source, const CfgParams& cfg_params) :
    GSource(source, cfg_params,
            std::make_shared<CfgParamDeviceSelector>(cfg_params)) {
}

GSource::GSource(std::shared_ptr<IDataProvider> source,
                 const CfgParams& cfg_params,
                 const std::string& device_id,
                 void* accel_device_ptr,
                 void* accel_ctx_ptr) :
    GSource(source, cfg_params,
            std::make_shared<CfgParamDeviceSelector>(accel_device_ptr, device_id,
                                                     accel_ctx_ptr, cfg_params)) {
}

// common delegating parameters c-tor
GSource::GSource(std::shared_ptr<IDataProvider> source,
                 const CfgParams& cfg_params,
                 std::shared_ptr<IDeviceSelector> selector) :
    GSource(std::unique_ptr<Priv>(new GSource::Priv(source, cfg_params, selector))) {
}

#else
GSource::GSource(const std::string&, const CfgParams&) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(const std::string&, const CfgParams&, const std::string&,
                 void*, void*) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(const std::string&, const CfgParams&, const Device &, const Context &) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(const std::string&, const CfgParams&, std::shared_ptr<IDeviceSelector>) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(std::shared_ptr<IDataProvider>, const CfgParams&) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(std::shared_ptr<IDataProvider>, const CfgParams&,
                 const std::string&, void*, void*) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

GSource::GSource(std::shared_ptr<IDataProvider>, const CfgParams&, std::shared_ptr<IDeviceSelector>) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}
#endif

// final delegating c-tor
GSource::GSource(std::unique_ptr<Priv>&& impl) :
    IStreamSource(),
    m_priv(std::move(impl)) {
}

GSource::~GSource() = default;

bool GSource::pull(cv::gapi::wip::Data& data)
{
    return m_priv->pull(data);
}

GMetaArg GSource::descr_of() const
{
    return m_priv->descr_of();
}
} // namespace onevpl
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/cfg_param_device_selector.hpp`
- `opencv2/gapi/streaming/onevpl/source.hpp`
- `streaming/onevpl/data_provider_dispatcher.hpp`
- `streaming/onevpl/source_priv.hpp`


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

