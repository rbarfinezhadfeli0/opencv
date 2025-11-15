# Documentation for `modules/gapi/samples/pipeline_modeling_tool/dummy_source.hpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/pipeline_modeling_tool/dummy_source.hpp`
- **File Name**: `dummy_source.hpp`
- **File Size**: 3,895 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/samples/pipeline_modeling_tool/dummy_source.hpp](../../../../modules/gapi/samples/pipeline_modeling_tool/dummy_source.hpp)

## Purpose and Role

This file is located in the `modules/gapi/samples/pipeline_modeling_tool` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef OPENCV_GAPI_PIPELINE_MODELING_TOOL_DUMMY_SOURCE_HPP
#define OPENCV_GAPI_PIPELINE_MODELING_TOOL_DUMMY_SOURCE_HPP

#include <thread>
#include <memory>
#include <chrono>

#include <opencv2/gapi.hpp>
#include <opencv2/gapi/streaming/cap.hpp> // cv::gapi::wip::IStreamSource

#include "utils.hpp"

class DummySource final: public cv::gapi::wip::IStreamSource {
public:
    using WaitStrategy = std::function<void(std::chrono::microseconds)>;
    using Ptr = std::shared_ptr<DummySource>;
    using ts_t = std::chrono::microseconds;

    template <typename DurationT>
    DummySource(const DurationT    latency,
                const OutputDescr& output,
                const bool         drop_frames,
                WaitStrategy&&     wait);

    bool pull(cv::gapi::wip::Data& data) override;
    cv::GMetaArg descr_of() const override;

private:
    int64_t       m_latency;
    cv::Mat       m_mat;
    bool          m_drop_frames;
    int64_t       m_next_tick_ts = -1;
    int64_t       m_curr_seq_id  = 0;
    WaitStrategy  m_wait;
};

template <typename DurationT>
DummySource::DummySource(const DurationT    latency,
                         const OutputDescr& output,
                         const bool         drop_frames,
                         WaitStrategy&&     wait)
    : m_latency(std::chrono::duration_cast<ts_t>(latency).count()),
      m_drop_frames(drop_frames),
      m_wait(std::move(wait)) {
    utils::createNDMat(m_mat, output.dims, output.precision);
    utils::generateRandom(m_mat);
}

bool DummySource::pull(cv::gapi::wip::Data& data) {
    using namespace std::chrono;
    using namespace cv::gapi::streaming;

    // NB: Wait m_latency before return the first frame.
    if (m_next_tick_ts == -1) {
        m_next_tick_ts = utils::timestamp<ts_t>() + m_latency;
    }

    int64_t curr_ts = utils::timestamp<ts_t>();
    if (curr_ts < m_next_tick_ts) {
        /*
         *            curr_ts
         *               |
         *    ------|----*-----|------->
         *                     ^
         *               m_next_tick_ts
         *
         *
         * NB: New frame will be produced at the m_next_tick_ts point.
         */
        m_wait(ts_t{m_next_tick_ts - curr_ts});
    } else if (m_latency != 0) {
        /*
         *                                       curr_ts
         *                         +1         +2    |
         *    |----------|----------|----------|----*-----|------->
         *               ^                     ^
         *         m_next_tick_ts ------------->
         *
         */

        // NB: Count how many frames have been produced since last pull (m_next_tick_ts).
        int64_t num_frames =
            static_cast<int64_t>((curr_ts - m_next_tick_ts) / m_latency);
        // NB: Shift m_next_tick_ts to the nearest tick before curr_ts.
        m_next_tick_ts += num_frames * m_latency;
        // NB: if drop_frames is enabled, update current seq_id and wait for the next tick, otherwise
        // return last written frame (+2 at the picture above) immediately.
        if (m_drop_frames) {
            // NB: Shift tick to the next frame.
            m_next_tick_ts += m_latency;
            // NB: Wait for the next frame.
            m_wait(ts_t{m_next_tick_ts - curr_ts});
            // NB: Drop already produced frames + update seq_id for the current.
            m_curr_seq_id += num_frames + 1;
        }
    }
    // NB: Just increase reference counter not to release mat memory
    // after assigning it to the data.
    cv::Mat mat = m_mat;
    data.meta[meta_tag::timestamp] = utils::timestamp<ts_t>();
    data.meta[meta_tag::seq_id] = m_curr_seq_id++;
    data = mat;
    m_next_tick_ts += m_latency;

    return true;
}

cv::GMetaArg DummySource::descr_of() const {
    return cv::GMetaArg{cv::descr_of(m_mat)};
}

#endif // OPENCV_GAPI_PIPELINE_MODELING_TOOL_DUMMY_SOURCE_HPP
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

- **DummySource**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_PIPELINE_MODELING_TOOL_DUMMY_SOURCE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/streaming/cap.hpp`
- `chrono`
- `memory`
- `thread`
- `utils.hpp`
- `opencv2/gapi.hpp`


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

