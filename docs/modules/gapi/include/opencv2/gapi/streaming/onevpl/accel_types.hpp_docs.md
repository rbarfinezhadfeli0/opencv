# Documentation for `modules/gapi/include/opencv2/gapi/streaming/onevpl/accel_types.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/onevpl/accel_types.hpp`
- **File Name**: `accel_types.hpp`
- **File Size**: 1,927 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/onevpl/accel_types.hpp](../../../../../../../modules/gapi/include/opencv2/gapi/streaming/onevpl/accel_types.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_ACCEL_TYPES_HPP
#define GAPI_STREAMING_ONEVPL_ACCEL_TYPES_HPP

#include <limits>
#include <string>

#include "opencv2/gapi/own/exports.hpp" // GAPI_EXPORTS

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

enum class AccelType: uint8_t {
    HOST,
    DX11,
    VAAPI,

    LAST_VALUE = std::numeric_limits<uint8_t>::max()
};

GAPI_EXPORTS const char* to_cstring(AccelType type);

struct IDeviceSelector;
struct GAPI_EXPORTS Device {
    friend struct IDeviceSelector;
    using Ptr = void*;

    ~Device();
    const std::string& get_name() const;
    Ptr get_ptr() const;
    AccelType get_type() const;
private:
    Device(Ptr device_ptr, const std::string& device_name,
           AccelType device_type);

    std::string name;
    Ptr ptr;
    AccelType type;
};

struct GAPI_EXPORTS Context {
    friend struct IDeviceSelector;
    using Ptr = void*;

    ~Context();
    Ptr get_ptr() const;
    AccelType get_type() const;
private:
    Context(Ptr ctx_ptr, AccelType ctx_type);
    Ptr ptr;
    AccelType type;
};

GAPI_EXPORTS Device create_host_device();
GAPI_EXPORTS Context create_host_context();

GAPI_EXPORTS Device create_dx11_device(Device::Ptr device_ptr,
                                       const std::string& device_name);
GAPI_EXPORTS Context create_dx11_context(Context::Ptr ctx_ptr);

GAPI_EXPORTS Device create_vaapi_device(Device::Ptr device_ptr,
                                        const std::string& device_name);
GAPI_EXPORTS Context create_vaapi_context(Context::Ptr ctx_ptr);
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // GAPI_STREAMING_ONEVPL_ACCEL_TYPES_HPP
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
- **IDeviceSelector**: A class/struct defined in this file
- **AccelType**: A class/struct defined in this file

### Functions and Methods

- **GAPI_STREAMING_ONEVPL_ACCEL_TYPES_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `limits`
- `opencv2/gapi/own/exports.hpp`
- `string`


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

