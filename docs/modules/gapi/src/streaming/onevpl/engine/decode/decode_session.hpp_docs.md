# Documentation for `modules/gapi/src/streaming/onevpl/engine/decode/decode_session.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/decode/decode_session.hpp`
- **File Name**: `decode_session.hpp`
- **File Size**: 1,972 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/decode/decode_session.hpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/decode/decode_session.hpp)

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

#ifndef GAPI_STREAMING_ONVPL_ENGINE_DECODE_DECODE_SESSION_HPP
#define GAPI_STREAMING_ONVPL_ENGINE_DECODE_DECODE_SESSION_HPP
#include <stdio.h>
#include <memory>
#include <queue>

#include <opencv2/gapi/streaming/meta.hpp>

#include "streaming/onevpl/engine/engine_session.hpp"
#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
class Surface;
struct VPLAccelerationPolicy;
class VPLLegacyDecodeEngine;

class GAPI_EXPORTS LegacyDecodeSession : public EngineSession {
public:
    friend class VPLLegacyDecodeEngine;
    friend class VPLLegacyTranscodeEngine; //TODO: remove friend add method

    LegacyDecodeSession(mfxSession sess, DecoderParams&& decoder_param, std::shared_ptr<IDataProvider> provider);
    ~LegacyDecodeSession();
    using EngineSession::EngineSession;

    void swap_decode_surface(VPLLegacyDecodeEngine& engine);
    void init_surface_pool(VPLAccelerationPolicy::pool_key_t key);

    Data::Meta generate_frame_meta();
    virtual const mfxFrameInfo& get_video_param() const override;

    IDataProvider::mfx_bitstream *get_mfx_bitstream_ptr();
private:
    mfxVideoParam mfx_decoder_param;
    VPLAccelerationPolicy::pool_key_t decoder_pool_id;

    std::shared_ptr<IDataProvider> data_provider;
    std::shared_ptr<IDataProvider::mfx_bitstream> stream;

protected:
    std::weak_ptr<Surface> processing_surface_ptr;
    using op_handle_t = std::pair<mfxSyncPoint, mfxFrameSurface1*>;
    std::queue<op_handle_t> sync_queue;

    int64_t decoded_frames_count;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONVPL_ENGINE_DECODE_DECODE_SESSION_HPP
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

- **GAPI_EXPORTS**: A class/struct defined in this file
- **VPLLegacyTranscodeEngine**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file
- **Surface**: A class/struct defined in this file
- **VPLLegacyDecodeEngine**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONVPL_ENGINE_DECODE_DECODE_SESSION_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `stdio.h`
- `streaming/onevpl/onevpl_export.hpp`
- `queue`
- `memory`
- `opencv2/gapi/streaming/meta.hpp`
- `streaming/onevpl/engine/engine_session.hpp`


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

