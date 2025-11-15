# Documentation for `modules/gapi/src/streaming/onevpl/engine/processing_engine_base.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/processing_engine_base.hpp`
- **File Name**: `processing_engine_base.hpp`
- **File Size**: 3,525 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/processing_engine_base.hpp](../../../../../../modules/gapi/src/streaming/onevpl/engine/processing_engine_base.hpp)

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

#ifndef GAPI_STREAMING_ONEVPL_ENGINE_PROCESSING_ENGINE_BASE_HPP
#define GAPI_STREAMING_ONEVPL_ENGINE_PROCESSING_ENGINE_BASE_HPP

#include <queue>
#include <opencv2/gapi/streaming/onevpl/cfg_params.hpp>
#include <opencv2/gapi/garg.hpp>
#include "streaming/onevpl/engine/engine_session.hpp"
#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS

#ifdef HAVE_ONEVPL

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

struct VPLAccelerationPolicy;
struct IDataProvider;

// GAPI_EXPORTS for tests
class GAPI_EXPORTS ProcessingEngineBase {
public:
    enum class ExecutionStatus {
        Continue,
        Processed,
        SessionNotFound,
        Failed
    };
    struct ExecutionData {
        size_t op_id = 0;
    };

    using file_ptr = std::unique_ptr<FILE, decltype(&fclose)>;

    using session_ptr = std::shared_ptr<EngineSession>;
    using SessionsTable = std::map<mfxSession, session_ptr>;
    using ExecutionDataTable = std::map<mfxSession, ExecutionData>;

    using frame_t = cv::gapi::wip::Data;
    using frames_container_t = std::queue<frame_t>;
    using operation_t = std::function<ExecutionStatus(EngineSession&)>;

    static const char * status_to_string(ExecutionStatus);

    ProcessingEngineBase(std::unique_ptr<VPLAccelerationPolicy>&& accel);
    virtual ~ProcessingEngineBase();

    virtual session_ptr initialize_session(mfxSession mfx_session,
                                           const std::vector<CfgParam>& cfg_params,
                                           std::shared_ptr<IDataProvider> provider) = 0;

    ExecutionStatus process(mfxSession session);
    size_t get_ready_frames_count() const;
    void get_frame(Data &data);

    const VPLAccelerationPolicy* get_accel() const;
    VPLAccelerationPolicy* get_accel();
protected:
    SessionsTable sessions;
    frames_container_t ready_frames;
    ExecutionDataTable execution_table;

    std::vector<operation_t> pipeline;
    std::unique_ptr<VPLAccelerationPolicy> acceleration_policy;
public:
    virtual ExecutionStatus execute_op(operation_t& op, EngineSession& sess);

    template<class ...Ops>
    void create_pipeline(Ops&&...ops)
    {
        std::vector<operation_t>({std::forward<Ops>(ops)...}).swap(pipeline);
    }

    template<class ...Ops>
    void inject_pipeline_operations(size_t in_position, Ops&&...ops)
    {
        GAPI_Assert(pipeline.size() >= in_position &&
                    "Invalid position to inject pipeline operation");
        auto it = pipeline.begin();
        std::advance(it, in_position);
        pipeline.insert(it, {std::forward<Ops>(ops)...});
    }

    template<class SpecificSession, class ...SessionArgs>
    std::shared_ptr<SpecificSession> register_session(mfxSession key,
                                                      SessionArgs&& ...args)
    {
        auto sess_impl = std::make_shared<SpecificSession>(key,
                                                           std::forward<SessionArgs>(args)...);
        sessions.emplace(key, sess_impl);
        execution_table.emplace(key, ExecutionData{});
        return sess_impl;
    }
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ENGINE_PROCESSING_ENGINE_BASE_HPP
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
- **IDataProvider**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file
- **SpecificSession**: A class/struct defined in this file
- **ExecutionData**: A class/struct defined in this file
- **ExecutionStatus**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONEVPL_ENGINE_PROCESSING_ENGINE_BASE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/streaming/onevpl/cfg_params.hpp`
- `opencv2/gapi/own/exports.hpp`
- `opencv2/gapi/garg.hpp`
- `queue`
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

