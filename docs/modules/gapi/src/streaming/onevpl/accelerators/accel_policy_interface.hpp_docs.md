# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/accel_policy_interface.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/accel_policy_interface.hpp`
- **File Name**: `accel_policy_interface.hpp`
- **File Size**: 2,804 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/accel_policy_interface.hpp](../../../../../../modules/gapi/src/streaming/onevpl/accelerators/accel_policy_interface.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/accelerators` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_INTERFACE_HPP
#define GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_INTERFACE_HPP

#include <functional>
#include <memory>
#include <type_traits>

#include <opencv2/gapi/media.hpp>
#include <opencv2/gapi/streaming/onevpl/device_selector_interface.hpp>

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/accelerators/surface/base_frame_adapter.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
struct VPLAccelerationPolicy
{
    using device_selector_ptr_t = std::shared_ptr<IDeviceSelector>;

    VPLAccelerationPolicy(device_selector_ptr_t selector) : device_selector(selector) {}
    virtual ~VPLAccelerationPolicy() = default;

    using pool_key_t = void*;

    using session_t = mfxSession;
    using surface_t = Surface;
    using surface_ptr_t = std::shared_ptr<surface_t>;
    using surface_weak_ptr_t = std::weak_ptr<surface_t>;
    using surface_ptr_ctr_t = std::function<surface_ptr_t(std::shared_ptr<void> out_buf_ptr,
                                                          size_t out_buf_ptr_offset,
                                                          size_t out_buf_ptr_size)>;

    struct FrameConstructorArgs {
        surface_t::handle_t *assoc_surface;
        session_t assoc_handle;
    };

    device_selector_ptr_t get_device_selector() {
        return device_selector;
    }
    const device_selector_ptr_t get_device_selector() const {
        return device_selector;
    }

    virtual void init(session_t session) = 0;
    virtual void deinit(session_t session) = 0;

    // Limitation: cannot give guarantee in successful memory realloccation
    // for existing workspace in existing pool (see realloc)
    // thus it is not implemented,
    // PLEASE provide initial memory area large enough
    virtual pool_key_t create_surface_pool(const mfxFrameAllocRequest& alloc_request, mfxFrameInfo& info) = 0;

    virtual surface_weak_ptr_t get_free_surface(pool_key_t key) = 0;
    virtual size_t get_free_surface_count(pool_key_t key) const = 0;
    virtual size_t get_surface_count(pool_key_t key) const = 0;

    virtual cv::MediaFrame::AdapterPtr create_frame_adapter(pool_key_t key,
                                                            const FrameConstructorArgs &params) = 0;
    device_selector_ptr_t device_selector;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_INTERFACE_HPP
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

- **FrameConstructorArgs**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_INTERFACE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `functional`
- `streaming/onevpl/onevpl_export.hpp`
- `opencv2/gapi/streaming/onevpl/device_selector_interface.hpp`
- `streaming/onevpl/accelerators/surface/base_frame_adapter.hpp`
- `memory`
- `opencv2/gapi/media.hpp`
- `type_traits`


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

