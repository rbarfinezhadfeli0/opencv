# Documentation for `modules/gapi/src/streaming/onevpl/file_data_provider.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/file_data_provider.cpp`
- **File Name**: `file_data_provider.cpp`
- **File Size**: 6,124 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/file_data_provider.cpp](../../../../../modules/gapi/src/streaming/onevpl/file_data_provider.cpp)

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

#include <errno.h>

#include "streaming/onevpl/file_data_provider.hpp"
#include "streaming/onevpl/cfg_params_parser.hpp"
#include "streaming/onevpl/utils.hpp"
#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

#ifdef HAVE_ONEVPL
FileDataProvider::FileDataProvider(const std::string& file_path,
                                   const std::vector<CfgParam> &codec_params,
                                   uint32_t bitstream_data_size_value) :
    source_handle(nullptr, &fclose),
    bitstream_data_size(bitstream_data_size_value) {

    GAPI_LOG_DEBUG(nullptr, "[" << this << "] " <<
                            "check codec Id from CfgParam, total param count: " <<
                            codec_params.size());
    auto codec_it =
        std::find_if(codec_params.begin(), codec_params.end(), [] (const CfgParam& value) {
            return value.get_name() == CfgParam::decoder_id_name();
        });
    if (codec_it == codec_params.end())
    {
        GAPI_LOG_WARNING(nullptr, "[" << this << "] " <<
                                  "\"" << CfgParam::decoder_id_name() << "\" "
                                  "is absent, total param count" << codec_params.size());
        throw DataProviderUnsupportedException(std::string("\"") + CfgParam::decoder_id_name() + "\" "
                                               "is required for FileDataProvider");
    }

    codec = cfg_param_to_mfx_variant(*codec_it).Data.U32;

    GAPI_LOG_DEBUG(nullptr, "[" << this << "] " <<
                            "opening file: " << file_path);
    source_handle.reset(fopen(file_path.c_str(), "rb"));
    if (!source_handle) {
        throw DataProviderSystemErrorException(errno,
                                               "FileDataProvider: cannot open source file: " + file_path);
    }

    GAPI_LOG_INFO(nullptr, "[" << this << "] " <<
                            "file: " << file_path << " opened, codec requested: " << mfx_codec_id_to_cstr(codec));
}

FileDataProvider::~FileDataProvider() = default;

IDataProvider::mfx_codec_id_type FileDataProvider::get_mfx_codec_id() const {
    return codec;
}

bool FileDataProvider::fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &out_bitstream) {

    GAPI_LOG_DEBUG(nullptr, "[" << this << "] " <<
                            ", dst: " << out_bitstream.get());
    if (empty()) {
        return false;
    }

    if (!out_bitstream) {
        out_bitstream = std::make_shared<mfx_bitstream>();
        out_bitstream->MaxLength = bitstream_data_size;
        out_bitstream->Data = (mfxU8 *)calloc(out_bitstream->MaxLength, sizeof(mfxU8));
        if(!out_bitstream->Data) {
            throw std::runtime_error("Cannot allocate bitstream.Data bytes: " +
                                     std::to_string(out_bitstream->MaxLength * sizeof(mfxU8)));
        }
        out_bitstream->CodecId = get_mfx_codec_id();
    }
    GAPI_LOG_DEBUG(nullptr, "[" << this << "] " <<
                            "bitstream before fetch, DataOffset: " <<
                            out_bitstream->DataOffset <<
                            ", DataLength: " <<
                            out_bitstream->DataLength);
    mfxU8 *p0 = out_bitstream->Data;
    mfxU8 *p1 = out_bitstream->Data + out_bitstream->DataOffset;
    if (out_bitstream->DataOffset > out_bitstream->MaxLength - 1) {
        throw DataProviderImplementationException(mfxstatus_to_string(MFX_ERR_NOT_ENOUGH_BUFFER));
    }
    if (out_bitstream->DataLength + out_bitstream->DataOffset > out_bitstream->MaxLength) {
        throw DataProviderImplementationException(mfxstatus_to_string(MFX_ERR_NOT_ENOUGH_BUFFER));
    }

    std::copy_n(p1, out_bitstream->DataLength, p0);

    out_bitstream->DataOffset = 0;
    size_t bytes_count = fread(out_bitstream->Data + out_bitstream->DataLength,
                               1, out_bitstream->MaxLength - out_bitstream->DataLength,
                               source_handle.get());
    if (bytes_count == 0) {
        if (feof(source_handle.get())) {
            source_handle.reset();
        } else {
            throw DataProviderSystemErrorException (errno, "FileDataProvider::fetch_bitstream_data error read");
        }
    }
    out_bitstream->DataLength += static_cast<mfxU32>(bytes_count);
    GAPI_LOG_DEBUG(nullptr, "bitstream after fetch, DataOffset: " << out_bitstream->DataOffset <<
                            ", DataLength: " << out_bitstream->DataLength);
    if (out_bitstream->DataLength == 0)
        return false;

    GAPI_LOG_DEBUG(nullptr, "[" << this << "] " <<
                            "buff fetched: " << out_bitstream.get());
    return true;
}

bool FileDataProvider::empty() const {
    return !source_handle;
}

#else // HAVE_ONEVPL

FileDataProvider::FileDataProvider(const std::string&,
                                   const std::vector<CfgParam> &,
                                   uint32_t bitstream_data_size_value) :
    source_handle(nullptr, &fclose),
    codec(std::numeric_limits<mfx_codec_id_type>::max()),
    bitstream_data_size(bitstream_data_size_value) {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

FileDataProvider::~FileDataProvider() = default;

IDataProvider::mfx_codec_id_type FileDataProvider::get_mfx_codec_id() const {
    cv::util::suppress_unused_warning(codec);
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

bool FileDataProvider::fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &) {
    cv::util::suppress_unused_warning(bitstream_data_size);
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}

bool FileDataProvider::empty() const {
    GAPI_Error("Unsupported: G-API compiled without `WITH_GAPI_ONEVPL=ON`");
}
#endif // HAVE_ONEVPL
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `logger.hpp`
- `streaming/onevpl/cfg_params_parser.hpp`
- `streaming/onevpl/file_data_provider.hpp`
- `streaming/onevpl/utils.hpp`
- `errno.h`

**Python Imports:**
- `CfgParam`


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

