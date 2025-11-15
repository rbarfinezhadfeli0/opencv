# Documentation for `modules/gapi/test/common/gapi_streaming_tests_common.hpp`

## File Metadata

- **Full Path**: `modules/gapi/test/common/gapi_streaming_tests_common.hpp`
- **File Name**: `gapi_streaming_tests_common.hpp`
- **File Size**: 3,393 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/test/common/gapi_streaming_tests_common.hpp](../../../../modules/gapi/test/common/gapi_streaming_tests_common.hpp)

## Purpose and Role

This file is located in the `modules/gapi/test/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_TESTS_COMMON_HPP
#define OPENCV_GAPI_STREAMING_TESTS_COMMON_HPP

#include "gapi_tests_common.hpp"
#include <opencv2/gapi/streaming/onevpl/source.hpp>
#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include "streaming/onevpl/data_provider_defines.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"

namespace opencv_test {
namespace streaming {
namespace onevpl {

struct StreamDataProvider : public cv::gapi::wip::onevpl::IDataProvider {

    StreamDataProvider(std::istream& in) : data_stream (in) {
        EXPECT_TRUE(in);
    }

mfx_codec_id_type get_mfx_codec_id() const override {
        return MFX_CODEC_HEVC;
    }

    bool fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &out_bitstream) override {
        if (empty()) {
            return false;
        }

        if (!out_bitstream) {
            out_bitstream = std::make_shared<mfx_bitstream>();
            out_bitstream->MaxLength = 2000000;
            out_bitstream->Data = (mfxU8 *)calloc(out_bitstream->MaxLength, sizeof(mfxU8));
            if(!out_bitstream->Data) {
                throw std::runtime_error("Cannot allocate bitstream.Data bytes: " +
                                         std::to_string(out_bitstream->MaxLength * sizeof(mfxU8)));
            }
            out_bitstream->CodecId = get_mfx_codec_id();
        }

        mfxU8 *p0 = out_bitstream->Data;
        mfxU8 *p1 = out_bitstream->Data + out_bitstream->DataOffset;
        EXPECT_FALSE(out_bitstream->DataOffset > out_bitstream->MaxLength - 1);
        EXPECT_FALSE(out_bitstream->DataLength + out_bitstream->DataOffset > out_bitstream->MaxLength);

        std::copy_n(p1, out_bitstream->DataLength, p0);

        out_bitstream->DataOffset = 0;
        out_bitstream->DataLength += static_cast<mfxU32>(fetch_data(out_bitstream->MaxLength - out_bitstream->DataLength,
                                                         out_bitstream->Data + out_bitstream->DataLength));
        return out_bitstream->DataLength != 0;
    }

    size_t fetch_data(size_t out_data_size, void* out_data_buf) {
        data_stream.read(reinterpret_cast<char*>(out_data_buf), out_data_size);
        return data_stream.gcount();
    }
    bool empty() const override {
        return data_stream.eof() || data_stream.bad();
    }
private:
    std::istream& data_stream;
};

static const unsigned char hevc_header[] = {
 0x00, 0x00, 0x00, 0x01, 0x40, 0x01, 0x0C, 0x06, 0xFF, 0xFF, 0x01, 0x40, 0x00,
 0x00, 0x03, 0x00, 0x80, 0x00, 0x00, 0x03, 0x00, 0x00, 0x03, 0x00, 0x78, 0x00,
 0x00, 0x04, 0x02, 0x10, 0x30, 0x00, 0x00, 0x03, 0x00, 0x10, 0x00, 0x00, 0x03,
 0x01, 0xE5, 0x00, 0x00, 0x00, 0x01, 0x42, 0x01, 0x06, 0x01, 0x40, 0x00, 0x00,
 0x03, 0x00, 0x80, 0x00, 0x00, 0x03, 0x00, 0x00, 0x03, 0x00, 0x78, 0x00, 0x00,
 0xA0, 0x10, 0x20, 0x61, 0x63, 0x41, 0x00, 0x86, 0x49, 0x1B, 0x2B, 0x20, 0x00,
 0x00, 0x00, 0x01, 0x44, 0x01, 0xC0, 0x71, 0xC0, 0xD9, 0x20, 0x00, 0x00, 0x00,
 0x01, 0x26, 0x01, 0xAF, 0x0C
};
} // namespace onevpl
} // namespace streaming
} // namespace opencv_test
#endif // HAVE_ONEVPL
#endif // OPENCV_GAPI_STREAMING_TESTS_HPP
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

- **StreamDataProvider**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_TESTS_COMMON_HPP()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/onevpl_export.hpp`
- `streaming/onevpl/data_provider_defines.hpp`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `opencv2/gapi/streaming/onevpl/source.hpp`
- `gapi_tests_common.hpp`


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

