# Documentation for `modules/gapi/src/streaming/onevpl/engine/engine_session.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/engine_session.hpp`
- **File Name**: `engine_session.hpp`
- **File Size**: 1,951 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/engine_session.hpp](../../../../../../modules/gapi/src/streaming/onevpl/engine/engine_session.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/engine` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_ENGINE_ENGINE_SESSION_HPP
#define GAPI_STREAMING_ONEVPL_ENGINE_ENGINE_SESSION_HPP

#include <functional>
#include <map>
#include <memory>
#include <string>
#include <utility>
#include <vector>

#include "opencv2/gapi/util/optional.hpp"
#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS
#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include "streaming/onevpl/data_provider_defines.hpp"
#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

// GAPI_EXPORTS for tests
struct GAPI_EXPORTS DecoderParams {
    std::shared_ptr<IDataProvider::mfx_bitstream> stream;
    mfxVideoParam param;
    cv::util::optional<size_t> preallocated_frames_count;
};

struct GAPI_EXPORTS TranscoderParams {
    mfxVideoParam param;
};

struct GAPI_EXPORTS EngineSession {
    mfxSession session;
    mfxStatus last_status;

    EngineSession(mfxSession sess);
    std::string error_code_to_str() const;
    virtual ~EngineSession();

    virtual const mfxFrameInfo& get_video_param() const = 0;

    static void request_free_surface(mfxSession session,
                                     VPLAccelerationPolicy::pool_key_t key,
                                     VPLAccelerationPolicy &acceleration_policy,
                                     std::weak_ptr<Surface> &surface_to_exchange,
                                     bool reset_if_not_found = false);
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ENGINE_ENGINE_SESSION_HPP
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

### Functions and Methods

- **GAPI_STREAMING_ONEVPL_ENGINE_ENGINE_SESSION_HPP()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `utility`
- `functional`
- `opencv2/gapi/util/optional.hpp`
- `opencv2/gapi/own/exports.hpp`
- `vector`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `streaming/onevpl/data_provider_defines.hpp`
- `streaming/onevpl/onevpl_export.hpp`
- `string`
- `memory`
- `map`


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

