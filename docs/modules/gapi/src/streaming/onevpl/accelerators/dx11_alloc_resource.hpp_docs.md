# Documentation for `modules/gapi/src/streaming/onevpl/accelerators/dx11_alloc_resource.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/accelerators/dx11_alloc_resource.hpp`
- **File Name**: `dx11_alloc_resource.hpp`
- **File Size**: 4,873 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/accelerators/dx11_alloc_resource.hpp](../../../../../../modules/gapi/src/streaming/onevpl/accelerators/dx11_alloc_resource.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/accelerators` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef GAPI_STREAMING_ONEVPL_ACCEL_DX11_ALLOC_RESOURCE_HPP
#define GAPI_STREAMING_ONEVPL_ACCEL_DX11_ALLOC_RESOURCE_HPP

#include <map>

#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS
#include <opencv2/gapi/util/compiler_hints.hpp>

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/accelerators/utils/elastic_barrier.hpp"
#include "streaming/onevpl/utils.hpp"

#ifdef HAVE_DIRECTX
#ifdef HAVE_D3D11

#define D3D11_NO_HELPERS
#define NOMINMAX
#include <d3d11.h>
#include <d3d11_4.h>
#include <codecvt>
#include "opencv2/core/directx.hpp"
#ifdef HAVE_OPENCL
#include <CL/cl_d3d11.h>
#endif // HAVE_OPENCL
#undef D3D11_NO_HELPERS
#undef NOMINMAX

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

class SharedLock;
// GAPI_EXPORTS for tests
struct GAPI_EXPORTS LockAdapter {
    LockAdapter(mfxFrameAllocator origin_allocator);

    size_t read_lock(mfxMemId mid, mfxFrameData &data);
    size_t unlock_read(mfxMemId mid, mfxFrameData &data);

    void write_lock(mfxMemId mid, mfxFrameData &data);
    bool is_write_acquired();
    void unlock_write(mfxMemId mid, mfxFrameData &data);

    SharedLock* set_adaptee(SharedLock* new_impl);
    SharedLock* get_adaptee();
private:
    LockAdapter(const LockAdapter&) = delete;
    LockAdapter(LockAdapter&&) = delete;
    LockAdapter& operator= (const LockAdapter&) = delete;
    LockAdapter& operator= (LockAdapter&&) = delete;

    mfxFrameAllocator lockable_allocator;
    SharedLock* impl;
};

struct GAPI_EXPORTS NativeHandleAdapter {
    NativeHandleAdapter(mfxFrameAllocator origin_allocator);

    void get_handle(mfxMemId mid, mfxHDL& out);
private:
    mfxFrameAllocator native_handle_getter;
};

struct DX11AllocationRecord;
struct DX11AllocationItem : public LockAdapter,
                            public NativeHandleAdapter,
                            public elastic_barrier<DX11AllocationItem> {
    using subresource_id_t = unsigned int;

    friend struct DX11AllocationRecord;
    friend class elastic_barrier<DX11AllocationItem>;
    ~DX11AllocationItem();

    void release();
    ID3D11Texture2D* get_texture_ptr();
    ID3D11Texture2D* get_staging_texture_ptr();
    DX11AllocationItem::subresource_id_t get_subresource() const;

    ID3D11DeviceContext* get_device_ctx_ptr();

    // public transactional access to resources.
    // implements dispatching through different access acquisition modes.
    // current acquisition mode determined by `LockAdapter` with `is_write_acquired()`
    mfxStatus acquire_access(mfxFrameData *ptr);
    mfxStatus release_access(mfxFrameData *ptr);
private:
    DX11AllocationItem(std::weak_ptr<DX11AllocationRecord> parent,
                       ID3D11DeviceContext* origin_ctx,
                       mfxFrameAllocator origin_allocator,
                       ComSharedPtrGuard<ID3D11Texture2D> texture_ptr,
                       subresource_id_t subresource_id,
                       ComPtrGuard<ID3D11Texture2D>&& staging_tex_ptr);

    // elastic barrier interface impl
    void on_first_in_impl(mfxFrameData *ptr);
    void on_last_out_impl(mfxFrameData *ptr);

    mfxStatus shared_access_acquire_unsafe(mfxFrameData *ptr);
    mfxStatus shared_access_release_unsafe(mfxFrameData *ptr);
    mfxStatus exclusive_access_acquire_unsafe(mfxFrameData *ptr);
    mfxStatus exclusive_access_release_unsafe(mfxFrameData *ptr);

    ID3D11DeviceContext* shared_device_context;

    ComSharedPtrGuard<ID3D11Texture2D> texture_ptr;
    subresource_id_t subresource_id = 0;
    ComPtrGuard<ID3D11Texture2D> staging_texture_ptr;
    std::weak_ptr<DX11AllocationRecord> observer;
};

struct DX11AllocationRecord : public std::enable_shared_from_this<DX11AllocationRecord> {

    using Ptr = std::shared_ptr<DX11AllocationRecord>;

    ~DX11AllocationRecord();

    template<typename... Args>
    static Ptr create(Args&& ...args) {
        std::shared_ptr<DX11AllocationRecord> record(new DX11AllocationRecord);
        record->init(std::forward<Args>(args)...);
        return record;
    }

    Ptr get_ptr();

    // Raw ptr is required as a part of VPL `Mid` c-interface
    // which requires contiguous memory
    using AllocationId = DX11AllocationItem*;
    AllocationId* data();
    size_t size() const;
private:
    DX11AllocationRecord();

    void init(unsigned int items, ID3D11DeviceContext* origin_ctx,
              mfxFrameAllocator origin_allocator,
              std::vector<ComPtrGuard<ID3D11Texture2D>>&& textures, std::vector<ComPtrGuard<ID3D11Texture2D>> &&staging_textures);
    std::vector<AllocationId> resources;
    ComSharedPtrGuard<ID3D11Texture2D> texture_ptr;
};

} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_D3D11
#endif // HAVE_DIRECTX
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ACCEL_DX11_ALLOC_RESOURCE_HPP
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

- **elastic_barrier**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **DX11AllocationRecord**: A class/struct defined in this file
- **DX11AllocationItem**: A class/struct defined in this file
- **impl**: A class/struct defined in this file
- **SharedLock**: A class/struct defined in this file

### Functions and Methods

- **D3D11_NO_HELPERS()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file
- **NOMINMAX()**: A function/method defined in this file
- **HAVE_D3D11()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **HAVE_DIRECTX()**: A function/method defined in this file
- **GAPI_STREAMING_ONEVPL_ACCEL_DX11_ALLOC_RESOURCE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/directx.hpp`
- `d3d11.h`
- `opencv2/gapi/own/exports.hpp`
- `streaming/onevpl/onevpl_export.hpp`
- `codecvt`
- `CL/cl_d3d11.h`
- `streaming/onevpl/accelerators/utils/elastic_barrier.hpp`
- `d3d11_4.h`
- `opencv2/gapi/util/compiler_hints.hpp`
- `streaming/onevpl/utils.hpp`
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

