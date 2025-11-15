# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_dispatcher.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_dispatcher.cpp`
- **File Name**: `preproc_dispatcher.cpp`
- **File Size**: 4,721 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc/preproc_dispatcher.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc/preproc_dispatcher.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/engine/preproc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#include <algorithm>
#include <exception>

#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include "streaming/onevpl/engine/preproc/preproc_dispatcher.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/engine/preproc/preproc_engine.hpp"
#include "streaming/onevpl/engine/preproc/preproc_session.hpp"

#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"
#include "streaming/onevpl/accelerators/surface/surface.hpp"
#include "streaming/onevpl/cfg_params_parser.hpp"
#endif // HAVE_ONEVPL

#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
#ifdef HAVE_ONEVPL
cv::util::optional<pp_params> VPPPreprocDispatcher::is_applicable(const cv::MediaFrame& in_frame) {
    cv::util::optional<pp_params> param;
    GAPI_LOG_DEBUG(nullptr, "workers: " << workers.size());
    bool worker_found = false;
    for (const auto &w : workers) {
        param = w->is_applicable(in_frame);
        if (param.has_value()) {
            auto &vpp_param = param.value().get<vpp_pp_params>();
            BaseFrameAdapter* adapter = reinterpret_cast<BaseFrameAdapter*>(vpp_param.reserved);
            const IDeviceSelector::DeviceScoreTable &devs =
                            (std::static_pointer_cast<VPPPreprocEngine>(w))->get_accel()->get_device_selector()->select_devices();
            GAPI_DbgAssert(devs.size() >= 1 && "Invalid device selector");
            auto worker_accel_type = std::get<1>(*devs.begin()).get_type();
            GAPI_LOG_DEBUG(nullptr, "acceleration types for frame: " << to_cstring(adapter->accel_type()) <<
                           ", for worker: " << to_cstring(worker_accel_type));
            if (worker_accel_type == adapter->accel_type()){
                vpp_param.reserved = reinterpret_cast<void *>(w.get());
                GAPI_LOG_DEBUG(nullptr, "selected worker: " << vpp_param.reserved);
                worker_found = true;
                break;
            }
        }
    }
    return worker_found ? param : cv::util::optional<pp_params>{};
}

pp_session VPPPreprocDispatcher::initialize_preproc(const pp_params& initial_frame_param,
                                                    const GFrameDesc& required_frame_descr) {
    const auto &vpp_param = initial_frame_param.get<vpp_pp_params>();
    GAPI_LOG_DEBUG(nullptr, "workers: " << workers.size());
    for (auto &w : workers) {
        if (reinterpret_cast<void*>(w.get()) == vpp_param.reserved) {
            pp_session sess = w->initialize_preproc(initial_frame_param, required_frame_descr);
            vpp_pp_session &vpp_sess = sess.get<vpp_pp_session>();
            vpp_sess.reserved = reinterpret_cast<void *>(w.get());
            GAPI_LOG_DEBUG(nullptr, "initialized session preproc for worker: " << vpp_sess.reserved);
            return sess;
        }
    }
    GAPI_Error("Cannot initialize VPP preproc in dispatcher, no suitable worker");
}

cv::MediaFrame VPPPreprocDispatcher::run_sync(const pp_session &session_handle,
                                              const cv::MediaFrame& in_frame,
                                              const cv::util::optional<cv::Rect> &opt_roi) {
    const auto &vpp_sess = session_handle.get<vpp_pp_session>();
    GAPI_LOG_DEBUG(nullptr, "workers: " << workers.size());
    for (auto &w : workers) {
        if (reinterpret_cast<void*>(w.get()) == vpp_sess.reserved) {
            GAPI_LOG_DEBUG(nullptr, "trigger execution on worker: " << vpp_sess.reserved);
            return w->run_sync(session_handle, in_frame, opt_roi);
        }
    }
    GAPI_Error("Cannot invoke VPP preproc in dispatcher, no suitable worker");
}

#else // HAVE_ONEVPL
cv::util::optional<pp_params> VPPPreprocDispatcher::is_applicable(const cv::MediaFrame&) {
    return cv::util::optional<pp_params>{};
}

pp_session VPPPreprocDispatcher::initialize_preproc(const pp_params&,
                                                    const GFrameDesc&) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

cv::MediaFrame VPPPreprocDispatcher::run_sync(const pp_session &,
                                              const cv::MediaFrame&,
                                              const cv::util::optional<cv::Rect> &) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}
#endif // HAVE_ONEVPL
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
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `streaming/onevpl/engine/preproc/preproc_session.hpp`
- `exception`
- `streaming/onevpl/onevpl_export.hpp`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `streaming/onevpl/engine/preproc/preproc_engine.hpp`
- `streaming/onevpl/cfg_params_parser.hpp`
- `streaming/onevpl/engine/preproc/preproc_dispatcher.hpp`
- `streaming/onevpl/accelerators/surface/surface.hpp`
- `algorithm`
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

