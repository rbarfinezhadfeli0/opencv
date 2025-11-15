# Documentation for `modules/gapi/include/opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- **File Name**: `data_provider_interface.hpp`
- **File Size**: 4,323 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/onevpl/data_provider_interface.hpp](../../../../../../../modules/gapi/include/opencv2/gapi/streaming/onevpl/data_provider_interface.hpp)

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

#ifndef GAPI_STREAMING_ONEVPL_ONEVPL_DATA_PROVIDER_INTERFACE_HPP
#define GAPI_STREAMING_ONEVPL_ONEVPL_DATA_PROVIDER_INTERFACE_HPP
#include <exception>
#include <memory>
#include <string>

#include <opencv2/gapi/own/exports.hpp> // GAPI_EXPORTS
namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

struct GAPI_EXPORTS DataProviderException : public std::exception {
    DataProviderException(const std::string& descr);
    DataProviderException(std::string&& descr);

    virtual ~DataProviderException() = default;
    virtual const char* what() const noexcept override;
private:
    std::string reason;
};

struct GAPI_EXPORTS DataProviderSystemErrorException final : public DataProviderException {
    DataProviderSystemErrorException(int error_code, const std::string& description = std::string());
    ~DataProviderSystemErrorException() = default;
};

struct GAPI_EXPORTS DataProviderUnsupportedException final : public DataProviderException {
    DataProviderUnsupportedException(const std::string& description);
    ~DataProviderUnsupportedException() = default;
};

struct GAPI_EXPORTS DataProviderImplementationException : public DataProviderException {
    DataProviderImplementationException(const std::string& description);
    ~DataProviderImplementationException() = default;
};
/**
 * @brief Public interface allows to customize extraction of video stream data
 * used by onevpl::GSource instead of reading stream from file (by default).
 *
 * Interface implementation constructor MUST provide consistency and creates fully operable object.
 * If error happened implementation MUST throw `DataProviderException` kind exceptions
 *
 * @note Interface implementation MUST manage stream and other constructed resources by itself to avoid any kind of leak.
 * For simple interface implementation example please see `StreamDataProvider` in `tests/streaming/gapi_streaming_tests.cpp`
 */
struct GAPI_EXPORTS IDataProvider {
    using Ptr = std::shared_ptr<IDataProvider>;
    using mfx_codec_id_type = uint32_t;

    /**
     * NB: here is supposed to be forward declaration of mfxBitstream
     * But according to current oneVPL implementation it is impossible to forward
     * declare untagged struct mfxBitstream.
     *
     * IDataProvider makes sense only for HAVE_VPL is ON and to keep IDataProvider
     * interface API/ABI compliant between core library and user application layer
     * let's introduce wrapper mfx_bitstream which inherits mfxBitstream in private
     * G-API code section and declare forward for wrapper mfx_bitstream here
     */
    struct mfx_bitstream;

    virtual ~IDataProvider() = default;

    /**
     * The function is used by onevpl::GSource to extract codec id from data
     *
     */
    virtual mfx_codec_id_type get_mfx_codec_id() const = 0;

    /**
     * The function is used by onevpl::GSource to extract binary data stream from @ref IDataProvider
     * implementation.
     *
     * It MUST throw `DataProviderException` kind exceptions in fail cases.
     * It MUST return MFX_ERR_MORE_DATA in EOF which considered as not-fail case.
     *
     * @param in_out_bitsream the input-output reference on MFX bitstream buffer which MUST be empty at the first request
     * to allow implementation to allocate it by itself and to return back. Subsequent invocation of `fetch_bitstream_data`
     * MUST use the previously used in_out_bitsream to avoid skipping rest of frames which haven't been consumed
     * @return true for fetched data, false on EOF and throws exception on error
     */
    virtual bool fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &in_out_bitsream) = 0;

    /**
     * The function is used by onevpl::GSource to check more binary data availability.
     *
     * It MUST return TRUE in case of EOF and NO_THROW exceptions.
     *
     * @return boolean value which detects end of stream
     */
    virtual bool empty() const = 0;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // GAPI_STREAMING_ONEVPL_ONEVPL_DATA_PROVIDER_INTERFACE_HPP
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
- **allows**: A class/struct defined in this file
- **mfxBitstream**: A class/struct defined in this file
- **implementation**: A class/struct defined in this file
- **mfx_bitstream**: A class/struct defined in this file
- **API**: A class/struct defined in this file

### Functions and Methods

- **is()**: A function/method defined in this file
- **GAPI_STREAMING_ONEVPL_ONEVPL_DATA_PROVIDER_INTERFACE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `exception`
- `string`
- `memory`
- `opencv2/gapi/own/exports.hpp`

**Python Imports:**
- `file`
- `data`


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

