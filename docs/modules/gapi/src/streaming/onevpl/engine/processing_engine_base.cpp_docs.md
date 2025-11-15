# Documentation for `modules/gapi/src/streaming/onevpl/engine/processing_engine_base.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/processing_engine_base.cpp`
- **File Name**: `processing_engine_base.cpp`
- **File Size**: 3,646 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/processing_engine_base.cpp](../../../../../../modules/gapi/src/streaming/onevpl/engine/processing_engine_base.cpp)

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

#ifdef HAVE_ONEVPL

#include <algorithm>

#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include "streaming/onevpl/engine/processing_engine_base.hpp"
#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"
#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

ProcessingEngineBase::ProcessingEngineBase(std::unique_ptr<VPLAccelerationPolicy>&& accel) :
    acceleration_policy(std::move(accel)) {
}

ProcessingEngineBase::~ProcessingEngineBase() {
    GAPI_LOG_INFO(nullptr, "destroyed, elapsed sessions count: " << sessions.size());
    sessions.clear();
}

ProcessingEngineBase::ExecutionStatus ProcessingEngineBase::process(mfxSession session) {
    auto sess_it = sessions.find(session);
    if (sess_it == sessions.end()) {
        return ExecutionStatus::SessionNotFound;
    }

    session_ptr processing_session = sess_it->second;
    ExecutionData& exec_data = execution_table[session];

    GAPI_LOG_DEBUG(nullptr, "[" << session << "] start op id: " << exec_data.op_id);
    ExecutionStatus status = execute_op(pipeline.at(exec_data.op_id), *processing_session);
    size_t old_op_id = exec_data.op_id++;
    if (exec_data.op_id == pipeline.size())
    {
        exec_data.op_id = 0;
    }
    cv::util::suppress_unused_warning(old_op_id);
    GAPI_LOG_DEBUG(nullptr, "[" << session << "] finish op id: " << old_op_id <<
                            ", " << processing_session->error_code_to_str() <<
                            ", " << ProcessingEngineBase::status_to_string(status) <<
                            ", next op id: " << exec_data.op_id);

    if (status == ExecutionStatus::Failed) {

        GAPI_LOG_WARNING(nullptr, "Operation for session: " << session <<
                                  ", " << ProcessingEngineBase::status_to_string(status) <<
                                  " - remove it");
        sessions.erase(sess_it);
        execution_table.erase(session);
    }

    if (status == ExecutionStatus::Processed) {
        GAPI_LOG_INFO(nullptr, "Processed [" << session << "]");
        sessions.erase(sess_it);
        execution_table.erase(session);
    }

    return status;
}

const char* ProcessingEngineBase::status_to_string(ExecutionStatus status)
{
    switch(status) {
        case ExecutionStatus::Continue: return "CONTINUE";
        case ExecutionStatus::Processed: return "PROCESSED";
        case ExecutionStatus::SessionNotFound: return "NOT_FOUND_SESSION";
        case ExecutionStatus::Failed: return "FAILED";
        default:
            return "UNKNOWN";
    }
}

ProcessingEngineBase::ExecutionStatus ProcessingEngineBase::execute_op(operation_t& op, EngineSession& sess)
{
    return op(sess);
}

size_t ProcessingEngineBase::get_ready_frames_count() const
{
    return ready_frames.size();
}

void ProcessingEngineBase::get_frame(Data &data)
{
    data = ready_frames.front();
    ready_frames.pop();
    GAPI_LOG_DEBUG(nullptr, " elapsed ready frames count: " << ready_frames.size());
}

const VPLAccelerationPolicy* ProcessingEngineBase::get_accel() const {
    return acceleration_policy.get();
}

VPLAccelerationPolicy* ProcessingEngineBase::get_accel() {
    return const_cast<VPLAccelerationPolicy*>(static_cast<const ProcessingEngineBase*>(this)->get_accel());
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
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `streaming/onevpl/engine/processing_engine_base.hpp`
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

