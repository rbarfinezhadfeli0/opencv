# Documentation for `modules/gapi/src/streaming/onevpl/source_priv.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/source_priv.hpp`
- **File Name**: `source_priv.hpp`
- **File Size**: 2,185 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/source_priv.hpp](../../../../../modules/gapi/src/streaming/onevpl/source_priv.hpp)

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

#ifndef OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_PRIV_HPP
#define OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_PRIV_HPP

#include <stdio.h>

#include <memory>
#include <string>

#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/streaming/meta.hpp>
#include <opencv2/gapi/streaming/onevpl/source.hpp>

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/engine/processing_engine_base.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

struct VPLAccelerationPolicy;
class ProcessingEngineBase;

struct GSource::Priv
{
    explicit Priv(std::shared_ptr<IDataProvider> provider,
                  const std::vector<CfgParam>& params,
                  std::shared_ptr<IDeviceSelector> selector);
    ~Priv();

    static const std::vector<CfgParam>& getDefaultCfgParams();
    const std::vector<CfgParam>& getCfgParams() const;

    bool pull(cv::gapi::wip::Data& data);
    GMetaArg descr_of() const;
private:
    Priv();
    std::unique_ptr<VPLAccelerationPolicy> initializeHWAccel(std::shared_ptr<IDeviceSelector> selector);

    // TODO not it is global variable. Waiting for FIX issue with CloneSession
    // mfxLoader mfx_handle;
    mfxImplDescription *mfx_impl_description;
    std::vector<mfxConfig> mfx_handle_configs;
    std::vector<CfgParam> cfg_params;

    mfxSession mfx_session;

    cv::GFrameDesc description;
    bool description_is_valid;

    std::unique_ptr<ProcessingEngineBase> engine;

    size_t consumed_frames_count;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#else // HAVE_ONEVPL

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
struct GSource::Priv final
{
    bool pull(cv::gapi::wip::Data&);
    GMetaArg descr_of() const;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_ONEVPL
#endif // OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_PRIV_HPP
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

- **ProcessingEngineBase**: A class/struct defined in this file
- **VPLAccelerationPolicy**: A class/struct defined in this file
- **GSource**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file
- **OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_PRIV_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `stdio.h`
- `streaming/onevpl/onevpl_export.hpp`
- `opencv2/gapi/garg.hpp`
- `streaming/onevpl/engine/processing_engine_base.hpp`
- `opencv2/gapi/streaming/meta.hpp`
- `string`
- `memory`
- `opencv2/gapi/streaming/onevpl/source.hpp`


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

