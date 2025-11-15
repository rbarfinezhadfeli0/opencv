# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/accel_policy_dx11.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/accel_policy_dx11.hpp`
- **File Name**: `accel_policy_dx11.hpp`
- **File Size**: 3,378 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/accel_policy_dx11.hpp](../../../../../../modules/gapi/src/streaming/onevpl/accelerators/accel_policy_dx11.hpp)

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

#ifndef GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_DX11_HPP
#define GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_DX11_HPP
#include <map>

#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"
#include "streaming/onevpl/accelerators/surface/surface_pool.hpp"
#include "streaming/onevpl/accelerators/dx11_alloc_resource.hpp"

#if defined(HAVE_DIRECTX) && defined(HAVE_D3D11)
#define D3D11_NO_HELPERS
#define NOMINMAX
#include <d3d11.h>
#include <codecvt>
#include "opencv2/core/directx.hpp"
#ifdef HAVE_OPENCL
#include <CL/cl_d3d11.h>
#endif // HAVE_OPENCL
#undef NOMINMAX
#endif // HAVE_DIRECTX && HAVE_D3D11

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

// GAPI_EXPORTS for tests
struct GAPI_EXPORTS VPLDX11AccelerationPolicy final: public VPLAccelerationPolicy
{
    // GAPI_EXPORTS for tests
    VPLDX11AccelerationPolicy(device_selector_ptr_t selector);
    ~VPLDX11AccelerationPolicy();

    using pool_t = CachedPool;

    void init(session_t session) override;
    void deinit(session_t session) override;
    pool_key_t create_surface_pool(const mfxFrameAllocRequest& alloc_request,
                                   mfxFrameInfo& info) override;
    surface_weak_ptr_t get_free_surface(pool_key_t key) override;
    size_t get_free_surface_count(pool_key_t key) const override;
    size_t get_surface_count(pool_key_t key) const override;

    cv::MediaFrame::AdapterPtr create_frame_adapter(pool_key_t key,
                                                    const FrameConstructorArgs &params) override;
private:
#ifdef HAVE_DIRECTX
#ifdef HAVE_D3D11
    ID3D11Device *hw_handle;
    ID3D11DeviceContext* device_context;

    mfxFrameAllocator allocator;
    static mfxStatus MFX_CDECL alloc_cb(mfxHDL pthis,
                                        mfxFrameAllocRequest *request,
                                        mfxFrameAllocResponse *response);
    static mfxStatus MFX_CDECL lock_cb(mfxHDL pthis, mfxMemId mid, mfxFrameData *ptr);
    static mfxStatus MFX_CDECL unlock_cb(mfxHDL pthis, mfxMemId mid, mfxFrameData *ptr);
    static mfxStatus MFX_CDECL get_hdl_cb(mfxHDL pthis, mfxMemId mid, mfxHDL *handle);
    static mfxStatus MFX_CDECL free_cb(mfxHDL pthis, mfxFrameAllocResponse *response);

    virtual mfxStatus on_alloc(const mfxFrameAllocRequest *request,
                               mfxFrameAllocResponse *response);
    static mfxStatus on_lock(mfxMemId mid, mfxFrameData *ptr);
    static mfxStatus on_unlock(mfxMemId mid, mfxFrameData *ptr);
    static mfxStatus on_get_hdl(mfxMemId mid, mfxHDL *handle);
    virtual mfxStatus on_free(mfxFrameAllocResponse *response);

    using alloc_id_t = mfxU32;
    using allocation_t = std::shared_ptr<DX11AllocationRecord>;
    std::map<alloc_id_t, allocation_t> allocation_table;

    std::map<pool_key_t, pool_t> pool_table;
#endif // HAVE_D3D11
#endif // HAVE_DIRECTX
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_DX11_HPP
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

### Functions and Methods

- **GAPI_STREAMING_ONEVPL_ACCELERATORS_ACCEL_POLICY_DX11_HPP()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file
- **NOMINMAX()**: A function/method defined in this file
- **HAVE_D3D11()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **HAVE_DIRECTX()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `opencv2/core/directx.hpp`
- `d3d11.h`
- `opencv2/gapi/own/exports.hpp`
- `CL/cl_d3d11.h`
- `streaming/onevpl/accelerators/dx11_alloc_resource.hpp`
- `streaming/onevpl/accelerators/surface/surface_pool.hpp`
- `codecvt`
- `map`


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

