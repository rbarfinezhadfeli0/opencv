# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.hpp`
- **File Name**: `preproc_session.hpp`
- **File Size**: 2,296 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.hpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc/preproc_session.hpp)

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

#ifndef GAPI_STREAMING_ONVPL_PREPROC_SESSION_HPP
#define GAPI_STREAMING_ONVPL_PREPROC_SESSION_HPP
#include <memory>
#include <queue>

#include <opencv2/gapi/streaming/meta.hpp>
#include "streaming/onevpl/engine/engine_session.hpp"
#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"
#include "streaming/onevpl/engine/preproc/vpp_preproc_defines.hpp"

#ifdef HAVE_ONEVPL

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
class VPPPreprocEngine;

class VPPPreprocSession : public EngineSession {
public:
    friend class VPPPreprocEngine;
    VPPPreprocSession(mfxSession sess, const mfxVideoParam &vpp_out_param);
    ~VPPPreprocSession();

    Data::Meta generate_frame_meta();
    void swap_surface(VPPPreprocEngine& engine);
    void init_surface_pool(VPLAccelerationPolicy::pool_key_t key);

    virtual const mfxFrameInfo& get_video_param() const override;
private:
    mfxVideoParam mfx_vpp_out_param;
    VPLAccelerationPolicy::pool_key_t vpp_pool_id;
    std::weak_ptr<Surface> processing_surface_ptr;

    struct incoming_task {
        mfxSyncPoint sync_handle;
        mfxFrameSurface1* decoded_surface_ptr;
        Surface::info_t decoded_frame_info;
        cv::MediaFrame decoded_frame_copy;
        cv::util::optional<cv::Rect> roi;
    };

    struct outgoing_task {
        outgoing_task() = default;
        outgoing_task(mfxSyncPoint acquired_sync_handle,
                      mfxFrameSurface1* acquired_surface_ptr,
                      incoming_task &&in);
        mfxSyncPoint sync_handle;
        mfxFrameSurface1* vpp_surface_ptr;

        mfxFrameSurface1* original_surface_ptr;
        void release_frame();
    private:
        Surface::info_t original_frame_info;
        cv::MediaFrame original_frame;
    };

    std::queue<incoming_task> sync_in_queue;
    std::queue<outgoing_task> vpp_out_queue;
    int64_t preprocessed_frames_count;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONVPL_PREPROC_SESSION_HPP
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

- **outgoing_task**: A class/struct defined in this file
- **VPPPreprocSession**: A class/struct defined in this file
- **incoming_task**: A class/struct defined in this file
- **VPPPreprocEngine**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONVPL_PREPROC_SESSION_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `queue`
- `memory`
- `opencv2/gapi/streaming/meta.hpp`
- `streaming/onevpl/engine/engine_session.hpp`
- `streaming/onevpl/engine/preproc/vpp_preproc_defines.hpp`


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

