# Documentation for `modules/gapi/include/opencv2/gapi/streaming/onevpl/source.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/onevpl/source.hpp`
- **File Name**: `source.hpp`
- **File Size**: 2,998 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/onevpl/source.hpp](../../../../../../../modules/gapi/include/opencv2/gapi/streaming/onevpl/source.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_HPP
#define OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_HPP

#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/streaming/meta.hpp>
#include <opencv2/gapi/streaming/source.hpp>
#include <opencv2/gapi/streaming/onevpl/cfg_params.hpp>
#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include <opencv2/gapi/streaming/onevpl/device_selector_interface.hpp>

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
using CfgParams = std::vector<CfgParam>;

/**
 * @brief G-API streaming source based on OneVPL implementation.
 *
 * This class implements IStreamSource interface.
 * Its constructor takes source file path (in usual way) or @ref onevpl::IDataProvider
 * interface implementation (for not file-based sources). It also allows to pass-through
 * oneVPL configuration parameters by using several @ref onevpl::CfgParam.
 *
 * @note stream sources are passed to G-API via shared pointers, so
 *  please gapi::make_onevpl_src<> to create objects and ptr() to pass a
 *  GSource to cv::gin().
 */
class GAPI_EXPORTS GSource : public IStreamSource
{
public:
    struct Priv;

    GSource(const std::string& filePath,
            const CfgParams& cfg_params = CfgParams{});

    GSource(const std::string& filePath,
            const CfgParams& cfg_params,
            const std::string& device_id,
            void* accel_device_ptr,
            void* accel_ctx_ptr);

    GSource(const std::string& filePath,
            const CfgParams& cfg_params,
            const Device &device, const Context &ctx);

    GSource(const std::string& filePath,
            const CfgParams& cfg_params,
            std::shared_ptr<IDeviceSelector> selector);


    GSource(std::shared_ptr<IDataProvider> source,
            const CfgParams& cfg_params = CfgParams{});

    GSource(std::shared_ptr<IDataProvider> source,
            const CfgParams& cfg_params,
            const std::string& device_id,
            void* accel_device_ptr,
            void* accel_ctx_ptr);

    GSource(std::shared_ptr<IDataProvider> source,
            const CfgParams& cfg_params,
            std::shared_ptr<IDeviceSelector> selector);

    ~GSource() override;

    bool pull(cv::gapi::wip::Data& data) override;
    GMetaArg descr_of() const override;

private:
    explicit GSource(std::unique_ptr<Priv>&& impl);
    std::unique_ptr<Priv> m_priv;
};
} // namespace onevpl

using GVPLSource = onevpl::GSource;

template<class... Args>
GAPI_EXPORTS_W cv::Ptr<IStreamSource> inline make_onevpl_src(Args&&... args)
{
    return make_src<onevpl::GSource>(std::forward<Args>(args)...);
}

} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_HPP
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
- **Priv**: A class/struct defined in this file
- **implements**: A class/struct defined in this file
- **implementation**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_ONEVPL_ONEVPL_SOURCE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/streaming/onevpl/cfg_params.hpp`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `opencv2/gapi/streaming/onevpl/device_selector_interface.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi/streaming/meta.hpp`
- `opencv2/gapi/streaming/source.hpp`


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

