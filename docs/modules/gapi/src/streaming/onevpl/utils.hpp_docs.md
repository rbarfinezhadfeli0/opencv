# Documentation for `modules/gapi/src/streaming/onevpl/utils.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/utils.hpp`
- **File Name**: `utils.hpp`
- **File Size**: 2,941 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/utils.hpp](../../../../../modules/gapi/src/streaming/onevpl/utils.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_ONEVPL_UTILS_HPP
#define GAPI_STREAMING_ONEVPL_ONEVPL_UTILS_HPP

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

#include <map>
#include <memory>
#include <set>
#include <string>

#include <opencv2/gapi/streaming/onevpl/cfg_params.hpp>


namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

// Since ATL headers might not be available on specific MSVS Build Tools
// we use simple `CComPtr` implementation like as `ComPtrGuard`
// which is not supposed to be the full functional replacement of `CComPtr`
// and it uses as RAII to make sure utilization is correct
template <typename COMNonManageableType>
void release(COMNonManageableType *ptr) {
    if (ptr) {
        ptr->Release();
    }
}

template <typename COMNonManageableType>
using ComPtrGuard = std::unique_ptr<COMNonManageableType, decltype(&release<COMNonManageableType>)>;

template <typename COMNonManageableType>
using ComSharedPtrGuard = std::shared_ptr<COMNonManageableType>;

template <typename COMNonManageableType>
ComPtrGuard<COMNonManageableType> createCOMPtrGuard(COMNonManageableType *ptr = nullptr) {
    return ComPtrGuard<COMNonManageableType> {ptr, &release<COMNonManageableType>};
}

template <typename COMNonManageableType>
ComSharedPtrGuard<COMNonManageableType> createCOMSharedPtrGuard(ComPtrGuard<COMNonManageableType>&& unique_guard) {
    return ComSharedPtrGuard<COMNonManageableType>(std::move(unique_guard));
}


const char* mfx_impl_to_cstr(const mfxIMPL impl);

mfxIMPL cstr_to_mfx_impl(const char* cstr);

const char* mfx_accel_mode_to_cstr (const mfxAccelerationMode mode);

mfxAccelerationMode cstr_to_mfx_accel_mode(const char* cstr);

const char* mfx_resource_type_to_cstr (const mfxResourceType type);

mfxResourceType cstr_to_mfx_resource_type(const char* cstr);

mfxU32 cstr_to_mfx_codec_id(const char* cstr);

const char* mfx_codec_id_to_cstr(mfxU32 mfx_id);

const std::set<mfxU32> &get_supported_mfx_codec_ids();

const char* mfx_codec_type_to_cstr(const mfxU32 fourcc, const mfxU32 type);

mfxU32 cstr_to_mfx_version(const char* cstr);

std::string GAPI_EXPORTS mfxstatus_to_string(int64_t err);
std::string GAPI_EXPORTS mfxstatus_to_string(mfxStatus err);

std::string mfx_frame_info_to_string(const mfxFrameInfo &info);
bool operator< (const mfxFrameInfo &lhs, const mfxFrameInfo &rhs);
bool operator== (const mfxFrameInfo &lhs, const mfxFrameInfo &rhs);

std::ostream& operator<< (std::ostream& out, const mfxImplDescription& idesc);

std::string ext_mem_frame_type_to_cstr(int type);
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_ONEVPL_UTILS_HPP
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

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **GAPI_STREAMING_ONEVPL_ONEVPL_UTILS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `set`
- `opencv2/gapi/streaming/onevpl/cfg_params.hpp`
- `streaming/onevpl/onevpl_export.hpp`
- `string`
- `memory`
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

