# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_engine.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc/preproc_engine.hpp`
- **File Name**: `preproc_engine.hpp`
- **File Size**: 2,532 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc/preproc_engine.hpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc/preproc_engine.hpp)

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

#ifndef GAPI_STREAMING_ONVPL_PREPROC_ENGINE_HPP
#define GAPI_STREAMING_ONVPL_PREPROC_ENGINE_HPP
#include <stdio.h>
#include <memory>
#include <unordered_map>

#include "streaming/onevpl/engine/processing_engine_base.hpp"
#include "streaming/onevpl/accelerators/utils/shared_lock.hpp"

#include "streaming/onevpl/engine/preproc_engine_interface.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
// GAPI_EXPORTS for tests
struct GAPI_EXPORTS FrameInfoComparator {
    bool operator()(const mfxFrameInfo& lhs, const mfxFrameInfo& rhs) const;
    static bool equal_to(const mfxFrameInfo& lhs, const mfxFrameInfo& rhs);
};

class VPPPreprocSession;
struct IDataProvider;
struct VPLAccelerationPolicy;

// GAPI_EXPORTS for tests
class GAPI_EXPORTS VPPPreprocEngine final : public ProcessingEngineBase,
                                            public cv::gapi::wip::IPreprocEngine {
public:
    using session_type     = VPPPreprocSession;
    using session_ptr_type = std::shared_ptr<session_type>;

    VPPPreprocEngine(std::unique_ptr<VPLAccelerationPolicy>&& accel);

    cv::util::optional<pp_params> is_applicable(const cv::MediaFrame& in_frame) override;

    pp_session initialize_preproc(const pp_params& initial_frame_param,
                                  const GFrameDesc& required_frame_descr) override;

    cv::MediaFrame run_sync(const pp_session &session_handle,
                            const cv::MediaFrame& in_frame,
                            const cv::util::optional<cv::Rect> &opt_roi) override;

private:
    std::map<mfxFrameInfo, session_ptr_type, FrameInfoComparator> preproc_session_map;
    void on_frame_ready(session_type& sess,
                        mfxFrameSurface1* ready_surface);
    ExecutionStatus process_error(mfxStatus status, session_type& sess);
    session_ptr initialize_session(mfxSession mfx_session,
                                   const std::vector<CfgParam>& cfg_params,
                                   std::shared_ptr<IDataProvider> provider) override;
    size_t preprocessed_frames_count;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONVPL_PREPROC_ENGINE_HPP
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

- **VPPPreprocSession**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file
- **IDataProvider**: A class/struct defined in this file

### Functions and Methods

- **GAPI_STREAMING_ONVPL_PREPROC_ENGINE_HPP()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/engine/preproc_engine_interface.hpp`
- `unordered_map`
- `stdio.h`
- `streaming/onevpl/onevpl_export.hpp`
- `streaming/onevpl/engine/processing_engine_base.hpp`
- `memory`
- `streaming/onevpl/accelerators/utils/shared_lock.hpp`


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

