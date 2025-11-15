# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/surface/surface.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/surface/surface.hpp`
- **File Name**: `surface.hpp`
- **File Size**: 4,126 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/surface/surface.hpp](../../../../../../../modules/gapi/src/streaming/onevpl/accelerators/surface/surface.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/accelerators/surface` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_ACCELERATORS_SURFACE_HPP
#define GAPI_STREAMING_ONEVPL_ACCELERATORS_SURFACE_HPP

#include <atomic>
#include <memory>

#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

/**
 * @brief Inner class for managing oneVPL surface through interface `mfxFrameSurface1`.
 *
 * Surface has no own memory and shares accelerator allocated memory using reference counter semantics.
 * So it lives till a last memory consumer (surface/accelerator/media frame) lives.
 * This approach allows to support different scenarious in releasing allocated memory
 *
 * VPL surface `mfxFrameSurface1` support Lock-Free semantics and application MUST NOT operate with a
 * surface in locked state. But VPL inner counter is not threadsafe so it would be failed in any concurrent scenario.
 * std::atomic counter introduced in a way to overcome this problem.
 * But only few scenarious for concurrency are supported here because it is not assumed to implement entire Surface in
 * for a fully multithread approach.
 * Supported concurrency scenarios deal only with transaction pair: @ref Surface::get_locks_count() against
 * @ref Surface::release_lock() - which may be called from different threads. On the other hand @ref Surface::get_locks_count() against
 * @ref Surface::obtain_lock() happens in single thread only. Surface doesn't support shared ownership that
 * because it doesn't require thread safe guarantee between transactions:
 * - @ref Surface::obtain_lock() against @ref Surface::obtain_lock()
 * - @ref Surface::obtain_lock() against @ref Surface::release_lock()
 * - @ref Surface::release_lock() against @ref Surface::release_lock()
 */
class GAPI_EXPORTS Surface final { // GAPI_EXPORTS for tests
public:
    using handle_t = mfxFrameSurface1;
    using info_t = mfxFrameInfo;
    using data_t = mfxFrameData;


    static std::shared_ptr<Surface> create_surface(std::unique_ptr<handle_t>&& surf,
                                                   std::shared_ptr<void> accociated_memory);
    ~Surface();

    handle_t* get_handle() const;
    const info_t& get_info() const;
    const data_t& get_data() const;
    data_t& get_data();

    /**
     * Extract value thread-safe lock counter (see @ref Surface description).
     * It's usual situation that counter may be instantly decreased in other thread after this method called.
     * We need instantaneous value. This method synchronized in inter-threading way with @ref Surface::release_lock()
     *
     * @return fetched locks count.
     */
    size_t get_locks_count() const;

    /**
     * Atomically increase value of thread-safe lock counter (see @ref Surface description).
     * This method is single-threaded happens-after @ref Surface::get_locks_count() and
     * multi-threaded happens-before @ref Surface::release_lock()
     *
     * @return locks count just before its increasing.
     */
    size_t obtain_lock();

    /**
     * Atomically decrease value of thread-safe lock counter (see @ref Surface description).
     * This method is synchronized with @ref Surface::get_locks_count() and
     * multi-threaded happens-after @ref Surface::obtain_lock()
     *
     * @return locks count just before its decreasing.
     */
     size_t release_lock();
private:
    Surface(std::unique_ptr<handle_t>&& surf, std::shared_ptr<void> accociated_memory);

    std::shared_ptr<void> workspace_memory_ptr;
    std::unique_ptr<handle_t> mfx_surface;
    std::atomic<size_t> mirrored_locked_count;
};

using surface_ptr_t = std::shared_ptr<Surface>;
using surface_weak_ptr_t = std::weak_ptr<Surface>;
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ACCELERATORS_SURFACE_HPP
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
- **for**: A class/struct defined in this file

### Functions and Methods

- **GAPI_STREAMING_ONEVPL_ACCELERATORS_SURFACE_HPP()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/own/exports.hpp`
- `memory`
- `atomic`
- `streaming/onevpl/onevpl_export.hpp`

**Python Imports:**
- `different`


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

