# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/surface/base_frame_adapter.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/surface/base_frame_adapter.cpp`
- **File Name**: `base_frame_adapter.cpp`
- **File Size**: 2,690 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/surface/base_frame_adapter.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/accelerators/surface/base_frame_adapter.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/accelerators/surface` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#include "streaming/onevpl/accelerators/surface/base_frame_adapter.hpp"
#include "streaming/onevpl/accelerators/surface/surface.hpp"
#include "logger.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
BaseFrameAdapter::BaseFrameAdapter(std::shared_ptr<Surface> surface,
                                   SessionHandle assoc_handle,
                                   AccelType accel):
    parent_surface_ptr(surface), parent_handle(assoc_handle),
    acceleration_type(accel) {
    GAPI_Assert(parent_surface_ptr && "Surface is nullptr");
    GAPI_Assert(parent_handle && "mfxSession is nullptr");

    const Surface::info_t& info = parent_surface_ptr->get_info();
    GAPI_LOG_DEBUG(nullptr, "surface: " << parent_surface_ptr->get_handle() <<
                            ", w: " << info.Width << ", h: " << info.Height <<
                            ", p: " << parent_surface_ptr->get_data().Pitch <<
                            ", frame id: " << reinterpret_cast<void*>(this));
    switch(info.FourCC) {
        case MFX_FOURCC_I420:
            throw std::runtime_error("MediaFrame doesn't support I420 type");
            break;
        case MFX_FOURCC_NV12:
            frame_desc.fmt = MediaFormat::NV12;
            break;
        default:
            throw std::runtime_error("MediaFrame unknown 'fmt' type: " + std::to_string(info.FourCC));
    }

    frame_desc.size = cv::Size{info.Width, info.Height};
    parent_surface_ptr->obtain_lock();
}

BaseFrameAdapter::~BaseFrameAdapter() {
    // Each BaseFrameAdapter releases mfx surface counter
    // The last BaseFrameAdapter releases shared Surface pointer
    // The last surface pointer releases workspace memory
    GAPI_LOG_DEBUG(nullptr, "destroy frame id: " << reinterpret_cast<void*>(this));
    parent_surface_ptr->release_lock();
}

const std::shared_ptr<Surface>& BaseFrameAdapter::get_surface() const {
    return parent_surface_ptr;
}

std::shared_ptr<Surface> BaseFrameAdapter::surface() {
    return parent_surface_ptr;
}

BaseFrameAdapter::SessionHandle BaseFrameAdapter::get_session_handle() const {
    return parent_handle;
}

cv::GFrameDesc BaseFrameAdapter::meta() const {
    return frame_desc;
}
AccelType BaseFrameAdapter::accel_type() const {
    return acceleration_type;
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
- `streaming/onevpl/accelerators/surface/surface.hpp`
- `streaming/onevpl/accelerators/surface/base_frame_adapter.hpp`
- `logger.hpp`
- `streaming/onevpl/onevpl_export.hpp`


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

