# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/utils/shared_lock.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/utils/shared_lock.cpp`
- **File Name**: `shared_lock.cpp`
- **File Size**: 2,487 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/utils/shared_lock.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/accelerators/utils/shared_lock.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/accelerators/utils` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include <thread>
#include "streaming/onevpl/accelerators/utils/shared_lock.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

SharedLock::SharedLock() {
    exclusive_lock.store(false);
    shared_counter.store(0);
}

size_t SharedLock::shared_lock() {
    size_t prev = 0;
    bool in_progress = false;
    bool pred_excl = exclusive_lock.load();
    do {
        if (!pred_excl) {
            // if no exclusive lock then start shared lock transaction
            prev = shared_counter.fetch_add(1);
            in_progress = true; // transaction is in progress
        } else {
            if (in_progress) {
                in_progress = false;
                shared_counter.fetch_sub(1);
            }
            std::this_thread::yield();
        }

        // test if exclusive lock happened before
        pred_excl = exclusive_lock.load();
    } while (pred_excl || !in_progress);

    return prev;
}

size_t SharedLock::unlock_shared() {
    return shared_counter.fetch_sub(1);
}

void SharedLock::lock() {
    bool in_progress = false;
    size_t prev_shared = shared_counter.load();
    do {
        if (prev_shared == 0) {
            bool expected = false;
            while (!exclusive_lock.compare_exchange_strong(expected, true)) {
                expected = false;
                std::this_thread::yield();
            }
            in_progress = true;
        } else {
            if (in_progress) {
                in_progress = false;
                exclusive_lock.store(false);
            }
            std::this_thread::yield();
        }
        prev_shared = shared_counter.load();
    } while (prev_shared != 0 || !in_progress);
}

bool SharedLock::try_lock() {
    if (shared_counter.load() != 0) {
        return false;
    }

    bool expected = false;
    if (exclusive_lock.compare_exchange_strong(expected, true)) {
        if (shared_counter.load() == 0) {
            return true;
        } else {
            exclusive_lock.store(false);
        }
    }
    return false;
}

void SharedLock::unlock() {
    exclusive_lock.store(false);
}
bool SharedLock::owns() const {
    return exclusive_lock.load();
}
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
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
- `streaming/onevpl/accelerators/utils/shared_lock.hpp`
- `thread`


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

