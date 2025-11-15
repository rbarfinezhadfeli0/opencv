# Documentation for `modules/gapi/src/api/kernels_streaming.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/kernels_streaming.cpp`
- **File Name**: `kernels_streaming.cpp`
- **File Size**: 3,933 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/kernels_streaming.cpp](../../../../modules/gapi/src/api/kernels_streaming.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020-2021 Intel Corporation

#include "precomp.hpp"

#include <opencv2/gapi/streaming/desync.hpp>
#include <opencv2/gapi/streaming/format.hpp>

#include <opencv2/gapi/core.hpp>

cv::GMat cv::gapi::streaming::desync(const cv::GMat &g) {
    // FIXME: this is a limited implementation of desync
    // The real implementation must be generic (template) and
    // reside in desync.hpp (and it is detail::desync<>())

    // FIXME: Put a copy here to solve the below problem
    // FIXME: Because of the copy, the desync functionality is limited
    // to GMat only (we don't have generic copy kernel for other
    // object types)
    return cv::gapi::copy(detail::desync(g));

    // FIXME
    //
    // If consumed by multiple different islands (OCV and Fluid by
    // example, an object needs to be desynchronized individually
    // for every path.
    //
    // This is a limitation of the current implementation. It works
    // this way: every "desync" link from the main path to a new
    // desync path gets its "DesyncQueue" object which stores only the
    // last value written before of the desync object (DO) it consumes
    // (the container of type "last written value" or LWV.
    //
    //                         LWV
    // [Sync path] -> desync() - - > DO -> [ISL0 @ Desync path #1]
    //
    // At the same time, generally, every island in the streaming
    // graph gets its individual input as a queue (so normally, a
    // writer pushes the same output MULTIPLE TIMES if it has multiple
    // readers):
    //
    //                         LWV
    // [Sync path] -> desync() - - > DO1 -> [ISL0 @ Desync path #1]
    //                       : LWV
    //                       ' - - > DO2 -> [ISL1 @ Desync path #1]
    //
    // For users, it may seem legit to use desync here only once, and
    // it MUST BE legit once the problem is fixed.
    // But the problem with the current implementation is that islands
    // on the same desync path get different desync queues and in fact
    // stay desynchronized between each other. One shouldn't consider
    // this as a single desync path anymore.
    // If these two ISLs are then merged e.g. with add(a,b), the
    // results will be inconsistent, given that the latency of ISL0
    // and ISL1 may be different. This is not the same frame anymore
    // coming as `a` and `b` to add(a,b) because of it.
    //
    // To make things clear, we forbid this now and ask to call
    // desync one more time to allow that. It is bad since the graph
    // structure and island layout depends on kernel packages used,
    // not on the sole GComputation structure. This needs to be fixed!
    // Here's the working configuration:
    //
    //                         LWV
    // [Sync path] -> desync() - - > DO1 -> [ISL0 @ Desync path #1]
    //            :            LWV
    //            '-> desync() - - > DO2 -> [ISL1 @ Desync path #2] <-(!)
    //
    // Put an operation right after desync() is a quick workaround to
    // this synchronization problem. There will be one "last_written_value"
    // connected to a desynchronized data object, and this sole last_written_value
    // object will feed both branches of the streaming executable.
}

// All notes from the above desync(GMat) are also applicable here
cv::GFrame cv::gapi::streaming::desync(const cv::GFrame &f) {
    return cv::gapi::copy(detail::desync(f));
}

cv::GMat cv::gapi::streaming::BGR(const cv::GFrame& in) {
    return cv::gapi::streaming::GBGR::on(in);
}

cv::GMat cv::gapi::streaming::Y(const cv::GFrame& in){
    return cv::gapi::streaming::GY::on(in);
}

cv::GMat cv::gapi::streaming::UV(const cv::GFrame& in){
    return cv::gapi::streaming::GUV::on(in);
}
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `opencv2/gapi/streaming/desync.hpp`
- `opencv2/gapi/core.hpp`
- `opencv2/gapi/streaming/format.hpp`

**Python Imports:**
- `the`


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

