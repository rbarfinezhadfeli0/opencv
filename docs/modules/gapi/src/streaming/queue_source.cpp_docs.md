# Documentation for `modules/gapi/src/streaming/queue_source.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/queue_source.cpp`
- **File Name**: `queue_source.cpp`
- **File Size**: 2,312 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/queue_source.cpp](../../../../modules/gapi/src/streaming/queue_source.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2023 Intel Corporation

#include <chrono>
#include <atomic>

#include <ade/util/zip_range.hpp>

#include <opencv2/gapi/streaming/queue_source.hpp>
#include <opencv2/gapi/streaming/meta.hpp>

#include "executor/conc_queue.hpp"

namespace cv {
namespace gapi {
namespace wip {

class QueueSourceBase::Priv {
public:
    explicit Priv(const cv::GMetaArg &meta) {
        m = meta;
        halted = false;
    }

    cv::GMetaArg m;
    cv::gapi::own::concurrent_bounded_queue<cv::GRunArg> q;
    int64_t c = 0;
    std::atomic<bool> halted;
};

QueueSourceBase::QueueSourceBase(const cv::GMetaArg &m)
    : m_priv(new Priv(m)) {
}

void QueueSourceBase::push(Data &&data) {

    // Tag data with seq_id/ts
    const auto now = std::chrono::system_clock::now();
    const auto dur = std::chrono::duration_cast<std::chrono::microseconds>
        (now.time_since_epoch());
    data.meta[cv::gapi::streaming::meta_tag::timestamp] = int64_t{dur.count()};
    data.meta[cv::gapi::streaming::meta_tag::seq_id]    = int64_t{m_priv->c++};

    m_priv->q.push(data);
}

bool QueueSourceBase::pull(Data &data) {
    m_priv->q.pop(data);

    if (m_priv->halted) {
        return false;
    }
    return true;
}

void QueueSourceBase::halt() {
    m_priv->halted.store(true);
    m_priv->q.push(cv::GRunArg{});
}

cv::GMetaArg QueueSourceBase::descr_of() const {
    return m_priv->m;
}

QueueInput::QueueInput(const cv::GMetaArgs &args) {
    for (auto &&m : args) {
        m_sources.emplace_back(new cv::gapi::wip::QueueSourceBase(m));
    }
}

void QueueInput::push(cv::GRunArgs &&args) {
    GAPI_Assert(m_sources.size() == args.size());
    for (auto && it : ade::util::zip(ade::util::toRange(m_sources),
                                     ade::util::toRange(args)))
    {
        auto &src = std::get<0>(it);
        auto &obj = std::get<1>(it);

        Data d;
        d = obj;
        src->push(std::move(d));
    }
}

QueueInput::operator cv::GRunArgs () {
    cv::GRunArgs args;
    for (auto &&s : m_sources) {
        args.push_back(s->ptr());
    }
    return args;
}

} // wip
} // gapi
} // cv
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

### Classes and Structures

- **QueueSourceBase**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ade/util/zip_range.hpp`
- `atomic`
- `opencv2/gapi/streaming/queue_source.hpp`
- `chrono`
- `opencv2/gapi/streaming/meta.hpp`
- `executor/conc_queue.hpp`


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

