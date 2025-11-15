# Documentation for `docs/3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h_docs.md`
- **File Name**: `cl_va_api_media_sharing_intel.h_docs.md`
- **File Size**: 9,906 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h_docs.md](../../../../../../docs/3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/include/opencl/1.2/CL` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h`

## File Metadata

- **Full Path**: `3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h`
- **File Name**: `cl_va_api_media_sharing_intel.h`
- **File Size**: 6,525 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h](../../../../../3rdparty/include/opencl/1.2/CL/cl_va_api_media_sharing_intel.h)

## Purpose and Role

This file is located in the `3rdparty/include/opencl/1.2/CL` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*******************************************************************************
 * Copyright (c) 2008-2020 The Khronos Group Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 ******************************************************************************/
/*****************************************************************************\

Copyright (c) 2013-2019 Intel Corporation All Rights Reserved.

THESE MATERIALS ARE PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL INTEL OR ITS
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THESE
MATERIALS, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

File Name: cl_va_api_media_sharing_intel.h

Abstract:

Notes:

\*****************************************************************************/


#ifndef __OPENCL_CL_VA_API_MEDIA_SHARING_INTEL_H
#define __OPENCL_CL_VA_API_MEDIA_SHARING_INTEL_H

#include <CL/cl.h>
#include <CL/cl_platform.h>
#include <va/va.h>

#ifdef __cplusplus
extern "C" {
#endif

/******************************************
* cl_intel_va_api_media_sharing extension *
*******************************************/

#define cl_intel_va_api_media_sharing 1

/* error codes */
#define CL_INVALID_VA_API_MEDIA_ADAPTER_INTEL               -1098
#define CL_INVALID_VA_API_MEDIA_SURFACE_INTEL               -1099
#define CL_VA_API_MEDIA_SURFACE_ALREADY_ACQUIRED_INTEL      -1100
#define CL_VA_API_MEDIA_SURFACE_NOT_ACQUIRED_INTEL          -1101

/* cl_va_api_device_source_intel */
#define CL_VA_API_DISPLAY_INTEL                             0x4094

/* cl_va_api_device_set_intel */
#define CL_PREFERRED_DEVICES_FOR_VA_API_INTEL               0x4095
#define CL_ALL_DEVICES_FOR_VA_API_INTEL                     0x4096

/* cl_context_info */
#define CL_CONTEXT_VA_API_DISPLAY_INTEL                     0x4097

/* cl_mem_info */
#define CL_MEM_VA_API_MEDIA_SURFACE_INTEL                   0x4098

/* cl_image_info */
#define CL_IMAGE_VA_API_PLANE_INTEL                         0x4099

/* cl_command_type */
#define CL_COMMAND_ACQUIRE_VA_API_MEDIA_SURFACES_INTEL      0x409A
#define CL_COMMAND_RELEASE_VA_API_MEDIA_SURFACES_INTEL      0x409B

typedef cl_uint cl_va_api_device_source_intel;
typedef cl_uint cl_va_api_device_set_intel;

extern CL_API_ENTRY cl_int CL_API_CALL
clGetDeviceIDsFromVA_APIMediaAdapterINTEL(
    cl_platform_id                platform,
    cl_va_api_device_source_intel media_adapter_type,
    void*                         media_adapter,
    cl_va_api_device_set_intel    media_adapter_set,
    cl_uint                       num_entries,
    cl_device_id*                 devices,
    cl_uint*                      num_devices) CL_EXT_SUFFIX__VERSION_1_2;

typedef CL_API_ENTRY cl_int (CL_API_CALL * clGetDeviceIDsFromVA_APIMediaAdapterINTEL_fn)(
    cl_platform_id                platform,
    cl_va_api_device_source_intel media_adapter_type,
    void*                         media_adapter,
    cl_va_api_device_set_intel    media_adapter_set,
    cl_uint                       num_entries,
    cl_device_id*                 devices,
    cl_uint*                      num_devices) CL_EXT_SUFFIX__VERSION_1_2;

extern CL_API_ENTRY cl_mem CL_API_CALL
clCreateFromVA_APIMediaSurfaceINTEL(
    cl_context                    context,
    cl_mem_flags                  flags,
    VASurfaceID*                  surface,
    cl_uint                       plane,
    cl_int*                       errcode_ret) CL_EXT_SUFFIX__VERSION_1_2;

typedef CL_API_ENTRY cl_mem (CL_API_CALL * clCreateFromVA_APIMediaSurfaceINTEL_fn)(
    cl_context                    context,
    cl_mem_flags                  flags,
    VASurfaceID*                  surface,
    cl_uint                       plane,
    cl_int*                       errcode_ret) CL_EXT_SUFFIX__VERSION_1_2;

extern CL_API_ENTRY cl_int CL_API_CALL
clEnqueueAcquireVA_APIMediaSurfacesINTEL(
    cl_command_queue              command_queue,
    cl_uint                       num_objects,
    const cl_mem*                 mem_objects,
    cl_uint                       num_events_in_wait_list,
    const cl_event*               event_wait_list,
    cl_event*                     event) CL_EXT_SUFFIX__VERSION_1_2;

typedef CL_API_ENTRY cl_int (CL_API_CALL *clEnqueueAcquireVA_APIMediaSurfacesINTEL_fn)(
    cl_command_queue              command_queue,
    cl_uint                       num_objects,
    const cl_mem*                 mem_objects,
    cl_uint                       num_events_in_wait_list,
    const cl_event*               event_wait_list,
    cl_event*                     event) CL_EXT_SUFFIX__VERSION_1_2;

extern CL_API_ENTRY cl_int CL_API_CALL
clEnqueueReleaseVA_APIMediaSurfacesINTEL(
    cl_command_queue              command_queue,
    cl_uint                       num_objects,
    const cl_mem*                 mem_objects,
    cl_uint                       num_events_in_wait_list,
    const cl_event*               event_wait_list,
    cl_event*                     event) CL_EXT_SUFFIX__VERSION_1_2;

typedef CL_API_ENTRY cl_int (CL_API_CALL *clEnqueueReleaseVA_APIMediaSurfacesINTEL_fn)(
    cl_command_queue              command_queue,
    cl_uint                       num_objects,
    const cl_mem*                 mem_objects,
    cl_uint                       num_events_in_wait_list,
    const cl_event*               event_wait_list,
    cl_event*                     event) CL_EXT_SUFFIX__VERSION_1_2;

#ifdef __cplusplus
}
#endif

#endif  /* __OPENCL_CL_VA_API_MEDIA_SHARING_INTEL_H */

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

- **__OPENCL_CL_VA_API_MEDIA_SHARING_INTEL_H()**: A function/method defined in this file
- **CL_API_ENTRY()**: A function/method defined in this file
- **cl_uint()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `CL/cl_platform.h`
- `CL/cl.h`
- `va/va.h`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

