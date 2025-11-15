# Documentation for `modules/gapi/src/streaming/onevpl/engine/decode/decode_session.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/decode/decode_session.cpp`
- **File Name**: `decode_session.cpp`
- **File Size**: 2,593 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/decode/decode_session.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/decode/decode_session.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/engine/decode` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifdef HAVE_ONEVPL

#include <chrono>
#include <exception>

#include "streaming/onevpl/engine/decode/decode_session.hpp"
#include "streaming/onevpl/engine/decode/decode_engine_legacy.hpp"
#include "streaming/onevpl/accelerators/surface/surface.hpp"
#include "streaming/onevpl/utils.hpp"

#include "logger.hpp"
namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
LegacyDecodeSession::LegacyDecodeSession(mfxSession sess,
                                         DecoderParams&& decoder_param,
                                         std::shared_ptr<IDataProvider> provider) :
    EngineSession(sess),
    mfx_decoder_param(std::move(decoder_param.param)),
    data_provider(std::move(provider)),
    stream(std::move(decoder_param.stream)),
    processing_surface_ptr(),
    sync_queue(),
    decoded_frames_count()
{
}

LegacyDecodeSession::~LegacyDecodeSession()
{
    GAPI_LOG_INFO(nullptr, "Close Decode for session: " << session);
    MFXVideoDECODE_Close(session);
}

void LegacyDecodeSession::swap_decode_surface(VPLLegacyDecodeEngine& engine) {
    VPLAccelerationPolicy* acceleration_policy = engine.get_accel();
    GAPI_Assert(acceleration_policy && "Empty acceleration_policy");
    request_free_surface(session, decoder_pool_id, *acceleration_policy, processing_surface_ptr);
}

void LegacyDecodeSession::init_surface_pool(VPLAccelerationPolicy::pool_key_t key) {
    GAPI_Assert(key && "Init decode pull with empty key");
    decoder_pool_id = key;
}

Data::Meta LegacyDecodeSession::generate_frame_meta() {
    const auto now = std::chrono::system_clock::now();
    const auto dur = std::chrono::duration_cast<std::chrono::microseconds>
                (now.time_since_epoch());
    Data::Meta meta {
                        {cv::gapi::streaming::meta_tag::timestamp, int64_t{dur.count()} },
                        {cv::gapi::streaming::meta_tag::seq_id, int64_t{decoded_frames_count++}}
                    };
    return meta;
}

const mfxFrameInfo& LegacyDecodeSession::get_video_param() const {
    return mfx_decoder_param.mfx.FrameInfo;
}

IDataProvider::mfx_bitstream *LegacyDecodeSession::get_mfx_bitstream_ptr() {
    return (data_provider || (stream && stream->DataLength)) ?
            stream.get() : nullptr;
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
- `streaming/onevpl/engine/decode/decode_session.hpp`
- `exception`
- `chrono`
- `streaming/onevpl/engine/decode/decode_engine_legacy.hpp`
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

