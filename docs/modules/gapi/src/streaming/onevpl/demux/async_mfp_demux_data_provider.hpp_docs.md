# Documentation for `modules/gapi/src/streaming/onevpl/demux/async_mfp_demux_data_provider.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/demux/async_mfp_demux_data_provider.hpp`
- **File Name**: `async_mfp_demux_data_provider.hpp`
- **File Size**: 4,391 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/demux/async_mfp_demux_data_provider.hpp](../../../../../../modules/gapi/src/streaming/onevpl/demux/async_mfp_demux_data_provider.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/demux` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef GAPI_STREAMING_ONEVPL_DEMUX_ASYNC_MFP_DEMUX_DATA_PROVIDER_HPP
#define GAPI_STREAMING_ONEVPL_DEMUX_ASYNC_MFP_DEMUX_DATA_PROVIDER_HPP

#include <atomic>
#include <condition_variable>
#include <mutex>
#include <queue>

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>

#ifdef HAVE_GAPI_MSMF
#define NOMINMAX
#include <mfapi.h>
#include <mfidl.h>
#include <mfreadwrite.h>
#include <mfobjects.h>
#include <mftransform.h>
#include <mferror.h>
#include <shlwapi.h>
#include <wmcontainer.h>
#include <wmcodecdsp.h>
#undef NOMINMAX

#include "streaming/onevpl/data_provider_defines.hpp"
#include "streaming/onevpl/utils.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
struct GAPI_EXPORTS MFPAsyncDemuxDataProvider : public IDataProvider,
                                                public IMFSourceReaderCallback {
    MFPAsyncDemuxDataProvider(const std::string& file_path,
                              size_t keep_preprocessed_buf_count_value = 3);
    ~MFPAsyncDemuxDataProvider();

    mfx_codec_id_type get_mfx_codec_id() const override;
    bool fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &out_bitsream) override;
    bool empty() const override;

protected: /* For Unit tests only */
    enum class State {
        InProgress,
        Exhausted
    };

    // IUnknown methods forbidden for current implementations
    STDMETHODIMP QueryInterface(REFIID iid, void** ppv) override;
    STDMETHODIMP_(ULONG) AddRef() override;
    STDMETHODIMP_(ULONG) Release() override;

    // IMFSourceReaderCallback methods
    virtual STDMETHODIMP OnReadSample(HRESULT status, DWORD stream_index,
                                      DWORD stream_flag, LONGLONG timestamp,
                                      IMFSample *sample_ptr) override;
    STDMETHODIMP OnEvent(DWORD, IMFMediaEvent *) override;
    STDMETHODIMP OnFlush(DWORD) override;

    // implementation methods
    void flush();
    HRESULT request_next(HRESULT hr, DWORD stream_flag,
                         size_t worker_buffer_count);
    void consume_worker_data();
    virtual size_t produce_worker_data(void *key,
                                       ComPtrGuard<IMFMediaBuffer> &&buffer,
                                       std::shared_ptr<mfx_bitstream> &&staging_stream);
    size_t get_locked_buffer_size() const;

private:
    static bool select_supported_video_stream(ComPtrGuard<IMFPresentationDescriptor> &descriptor,
                                              mfx_codec_id_type &out_codec_id,
                                              void *source_id);
    // members
    size_t keep_preprocessed_buf_count;

    // COM members
    ComPtrGuard<IMFMediaSource> source;
    ComPtrGuard<IMFSourceReader> source_reader;
    std::atomic<ULONG> com_interface_reference_count;

    mfx_codec_id_type codec;

    // worker & processing buffers
    std::map<void*, ComPtrGuard<IMFMediaBuffer>> worker_key_to_buffer_mapping_storage;
    std::map<void*, ComPtrGuard<IMFMediaBuffer>> processing_key_to_buffer_mapping_storage;
    std::queue<std::shared_ptr<mfx_bitstream>> worker_locked_buffer_storage;
    std::queue<std::shared_ptr<mfx_bitstream>> processing_locked_buffer_storage;
    std::condition_variable buffer_storage_non_empty_cond;
    mutable std::mutex buffer_storage_mutex;

    std::atomic_flag submit_read_request;
    std::atomic<State> provider_state;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#else // HAVE_GAPI_MSMF
namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
struct GAPI_EXPORTS MFPAsyncDemuxDataProvider : public IDataProvider {
    explicit MFPAsyncDemuxDataProvider(const std::string&);

    mfx_codec_id_type get_mfx_codec_id() const override;
    bool fetch_bitstream_data(std::shared_ptr<mfx_bitstream> &out_bitsream) override;
    bool empty() const override;
};
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // HAVE_GAPI_MSMF
#endif // HAVE_ONEVPL
#endif // GAPI_STREAMING_ONEVPL_DEMUX_ASYNC_MFP_DEMUX_DATA_PROVIDER_HPP
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
- **State**: A class/struct defined in this file

### Functions and Methods

- **GAPI_STREAMING_ONEVPL_DEMUX_ASYNC_MFP_DEMUX_DATA_PROVIDER_HPP()**: A function/method defined in this file
- **NOMINMAX()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file
- **HAVE_GAPI_MSMF()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `wmcodecdsp.h`
- `mfreadwrite.h`
- `mfidl.h`
- `mfapi.h`
- `streaming/onevpl/onevpl_export.hpp`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `shlwapi.h`
- `atomic`
- `wmcontainer.h`
- `streaming/onevpl/data_provider_defines.hpp`
- `mftransform.h`
- `queue`
- `mfobjects.h`
- `mferror.h`
- `condition_variable`
- `streaming/onevpl/utils.hpp`
- `mutex`


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

