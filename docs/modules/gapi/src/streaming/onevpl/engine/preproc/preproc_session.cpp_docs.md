# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.cpp`
- **File Name**: `preproc_session.cpp`
- **File Size**: 2,988 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.cpp)

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

#ifdef HAVE_ONEVPL

#include <chrono>
#include <exception>

#include "streaming/onevpl/engine/preproc/preproc_session.hpp"
#include "streaming/onevpl/engine/preproc/preproc_engine.hpp"
#include "streaming/onevpl/accelerators/surface/surface.hpp"
#include "streaming/onevpl/utils.hpp"
#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
VPPPreprocSession::VPPPreprocSession(mfxSession sess, const mfxVideoParam& vpp_out_param) :
    EngineSession(sess),
    mfx_vpp_out_param(vpp_out_param),
    processing_surface_ptr(),
    sync_in_queue(),
    vpp_out_queue(),
    preprocessed_frames_count()
{
}

VPPPreprocSession::~VPPPreprocSession() {
    GAPI_LOG_INFO(nullptr, "Close VPP for session: " << session);
    MFXVideoVPP_Close(session);
}

Data::Meta VPPPreprocSession::generate_frame_meta() {
    const auto now = std::chrono::system_clock::now();
    const auto dur = std::chrono::duration_cast<std::chrono::microseconds>
                (now.time_since_epoch());
    Data::Meta meta {
                        {cv::gapi::streaming::meta_tag::timestamp, int64_t{dur.count()} },
                        {cv::gapi::streaming::meta_tag::seq_id, int64_t{preprocessed_frames_count++}}
                    };
    return meta;
}

void VPPPreprocSession::swap_surface(VPPPreprocEngine& engine) {
    VPLAccelerationPolicy* acceleration_policy = engine.get_accel();
    GAPI_Assert(acceleration_policy && "Empty acceleration_policy");
    request_free_surface(session, vpp_pool_id, *acceleration_policy,
                         processing_surface_ptr, true);
}

void VPPPreprocSession::init_surface_pool(VPLAccelerationPolicy::pool_key_t key) {
    GAPI_Assert(key && "Init preproc pull with empty key");
    vpp_pool_id = key;
}

const mfxFrameInfo& VPPPreprocSession::get_video_param() const {
    return mfx_vpp_out_param.vpp.Out;
}

VPPPreprocSession::outgoing_task::outgoing_task(mfxSyncPoint acquired_sync_handle,
                                                mfxFrameSurface1* acquired_surface_ptr,
                                                VPPPreprocSession::incoming_task &&in) :
    sync_handle(acquired_sync_handle),
    vpp_surface_ptr(acquired_surface_ptr),
    original_surface_ptr(in.decoded_surface_ptr),
    original_frame_info(std::move(in.decoded_frame_info)),
    original_frame(in.decoded_frame_copy) {
}

void VPPPreprocSession::outgoing_task::release_frame() {
    // restore initial surface params
    memcpy(&(original_surface_ptr->Info),
           &original_frame_info, sizeof(Surface::info_t));
    // release references on frame adapter
    original_frame = cv::MediaFrame();
}
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
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
- `streaming/onevpl/engine/preproc/preproc_session.hpp`
- `exception`
- `chrono`
- `streaming/onevpl/engine/preproc/preproc_engine.hpp`
- `streaming/onevpl/accelerators/surface/surface.hpp`
- `logger.hpp`
- `streaming/onevpl/utils.hpp`


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

