# Documentation for `modules/gapi/src/streaming/onevpl/engine/decode/decode_engine_legacy.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/decode/decode_engine_legacy.hpp`
- **File Name**: `decode_engine_legacy.hpp`
- **File Size**: 2,008 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/decode/decode_engine_legacy.hpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/decode/decode_engine_legacy.hpp)

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

#ifndef GAPI_STREAMING_ONVPL_DECODE_ENGINE_LEGACY_HPP
#define GAPI_STREAMING_ONVPL_DECODE_ENGINE_LEGACY_HPP
#include <stdio.h>
#include <memory>

#include "streaming/onevpl/engine/processing_engine_base.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

class LegacyDecodeSession;
struct DecoderParams;
struct IDataProvider;
struct VPLAccelerationPolicy;

class GAPI_EXPORTS VPLLegacyDecodeEngine : public ProcessingEngineBase {
public:

    VPLLegacyDecodeEngine(std::unique_ptr<VPLAccelerationPolicy>&& accel);
    virtual session_ptr initialize_session(mfxSession mfx_session,
                                           const std::vector<CfgParam>& cfg_params,
                                           std::shared_ptr<IDataProvider> provider) override;
protected:
    struct SessionParam {
        void* decode_pool_key;
        DecoderParams decoder_params;
    };

    SessionParam prepare_session_param(mfxSession mfx_session,
                                       const std::vector<CfgParam>& cfg_params,
                                       std::shared_ptr<IDataProvider> provider);

    ExecutionStatus process_error(mfxStatus status, LegacyDecodeSession& sess);

    void on_frame_ready(LegacyDecodeSession& sess,
                        mfxFrameSurface1* ready_surface);
    static void try_modify_pool_size_request_param(const char* param_name,
                                                   size_t new_frames_count,
                                                   mfxFrameAllocRequest& request);
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONVPL_DECODE_ENGINE_LEGACY_HPP
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
- **LegacyDecodeSession**: A class/struct defined in this file
- **IDataProvider**: A class/struct defined in this file
- **DecoderParams**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file
- **SessionParam**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONVPL_DECODE_ENGINE_LEGACY_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/engine/processing_engine_base.hpp`
- `stdio.h`
- `streaming/onevpl/onevpl_export.hpp`
- `memory`


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

