# Documentation for `modules/gapi/src/streaming/gstreamer/gstreamer_media_adapter.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/gstreamer/gstreamer_media_adapter.cpp`
- **File Name**: `gstreamer_media_adapter.cpp`
- **File Size**: 6,794 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/gstreamer/gstreamer_media_adapter.cpp](../../../../../modules/gapi/src/streaming/gstreamer/gstreamer_media_adapter.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/gstreamer` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include "gstreamer_media_adapter.hpp"
#include "gstreamer_buffer_utils.hpp"

#ifdef HAVE_GSTREAMER
namespace cv {
namespace gapi {
namespace wip {
namespace gst {

GStreamerMediaAdapter::GStreamerMediaAdapter(const cv::GFrameDesc& frameDesc,
                                             GstVideoInfo* videoInfo,
                                             GstBuffer* buffer) :
    m_frameDesc(frameDesc),
    m_videoInfo(gst_video_info_copy(videoInfo)),
    m_buffer(gst_buffer_ref(buffer)),
    m_isMapped(false)
{
#if GST_VERSION_MINOR >= 10
    // Check that GstBuffer has mono-view, so we can retrieve only one video-meta
    GAPI_Assert((gst_buffer_get_flags(m_buffer) & static_cast<GstBufferFlags>(GST_VIDEO_BUFFER_FLAG_MULTIPLE_VIEW)) == 0);
#endif // GST_VERSION_MINOR >= 10

    GstVideoMeta* videoMeta = gst_buffer_get_video_meta(m_buffer);
    if (videoMeta != nullptr) {
        switch (m_frameDesc.fmt) {
            case cv::MediaFormat::NV12: {
                m_strides = { videoMeta->stride[0], videoMeta->stride[1] };
                m_offsets = { videoMeta->offset[0], videoMeta->offset[1] };
                break;
            }
            case cv::MediaFormat::GRAY: {
                m_strides = { videoMeta->stride[0]};
                m_offsets = { videoMeta->offset[0]};
                break;
            }
            default: {
                GAPI_Error("Non NV12 or GRAY Media format is not expected here");
                break;
            }
        }
    } else {
        switch (m_frameDesc.fmt) {
            case cv::MediaFormat::NV12: {
                m_strides = { GST_VIDEO_INFO_PLANE_STRIDE(m_videoInfo.get(), 0),
                              GST_VIDEO_INFO_PLANE_STRIDE(m_videoInfo.get(), 1) };
                m_offsets = { GST_VIDEO_INFO_PLANE_OFFSET(m_videoInfo.get(), 0),
                              GST_VIDEO_INFO_PLANE_OFFSET(m_videoInfo.get(), 1) };
                break;
            }
            case cv::MediaFormat::GRAY: {
                m_strides = { GST_VIDEO_INFO_PLANE_STRIDE(m_videoInfo.get(), 0)};
                m_offsets = { GST_VIDEO_INFO_PLANE_OFFSET(m_videoInfo.get(), 0)};
                break;
            }
            default: {
                GAPI_Error("Non NV12 or GRAY Media format is not expected here");
                break;
            }
        }
    }
}

GStreamerMediaAdapter::~GStreamerMediaAdapter() {
    if (m_isMapped.load(std::memory_order_acquire)) {
        gst_video_frame_unmap(&m_videoFrame);
        m_isMapped.store(false, std::memory_order_release);
        m_mappedForWrite.store(false);
    }
}

cv::GFrameDesc GStreamerMediaAdapter::meta() const {
    return m_frameDesc;
}

cv::MediaFrame::View GStreamerMediaAdapter::access(cv::MediaFrame::Access access) {
    GAPI_Assert(access == cv::MediaFrame::Access::R ||
                access == cv::MediaFrame::Access::W);
    static std::atomic<size_t> thread_counters { };
    ++thread_counters;

    // NOTE: Framework guarantees that there should be no parallel accesses to the frame
    //       memory if is accessing for write.
    if (access == cv::MediaFrame::Access::W && !m_mappedForWrite.load(std::memory_order_acquire)) {
        GAPI_Assert(thread_counters > 1 &&
                    "Multiple access to view during mapping for write detected!");
        gst_video_frame_unmap(&m_videoFrame);
        m_isMapped.store(false);
    }

    if (!m_isMapped.load(std::memory_order_acquire)) {

        std::lock_guard<std::mutex> lock(m_mutex);

        if(!m_isMapped.load(std::memory_order_relaxed)) {

            GAPI_Assert(GST_VIDEO_INFO_N_PLANES(m_videoInfo.get()) == 2 ||
                        GST_VIDEO_INFO_N_PLANES(m_videoInfo.get()) == 1);
            GAPI_Assert(GST_VIDEO_INFO_FORMAT(m_videoInfo.get()) == GST_VIDEO_FORMAT_NV12 ||
                        GST_VIDEO_INFO_FORMAT(m_videoInfo.get()) == GST_VIDEO_FORMAT_GRAY8);

            // TODO: Use RAII for map/unmap
            if (access == cv::MediaFrame::Access::W) {
                gstreamer_utils::mapBufferToFrame(*m_buffer, *m_videoInfo, m_videoFrame,
                                                  GST_MAP_WRITE);
                m_mappedForWrite.store(true, std::memory_order_release);
            } else {
                gstreamer_utils::mapBufferToFrame(*m_buffer, *m_videoInfo, m_videoFrame,
                                                  GST_MAP_READ);
            }

            GAPI_Assert(GST_VIDEO_FRAME_PLANE_STRIDE(&m_videoFrame, 0) == m_strides[0]);
            GAPI_Assert(GST_VIDEO_FRAME_PLANE_OFFSET(&m_videoFrame, 0) == m_offsets[0]);
            if (m_frameDesc.fmt == cv::MediaFormat::NV12) {
                GAPI_Assert(GST_VIDEO_FRAME_PLANE_STRIDE(&m_videoFrame, 1) == m_strides[1]);
                GAPI_Assert(GST_VIDEO_FRAME_PLANE_OFFSET(&m_videoFrame, 1) == m_offsets[1]);
            }

            m_isMapped.store(true, std::memory_order_release);
        }
    }

    cv::MediaFrame::View::Ptrs ps;
    cv::MediaFrame::View::Strides ss;

    switch (m_frameDesc.fmt) {
        case cv::MediaFormat::NV12: {
            ps = {
                static_cast<uint8_t*>(GST_VIDEO_FRAME_PLANE_DATA(&m_videoFrame, 0)) + m_offsets[0], // Y-plane
                static_cast<uint8_t*>(GST_VIDEO_FRAME_PLANE_DATA(&m_videoFrame, 0)) + m_offsets[1], // UV-plane
                nullptr,
                nullptr
            };
            ss = {
                static_cast<std::size_t>(m_strides[0]), // Y-plane stride
                static_cast<std::size_t>(m_strides[1]), // UV-plane stride
                0u,
                0u
            };
            break;
        }
        case cv::MediaFormat::GRAY: {
            ps = {
                static_cast<uint8_t*>(GST_VIDEO_FRAME_PLANE_DATA(&m_videoFrame, 0)) + m_offsets[0], // Y-plane
                nullptr,
                nullptr,
                nullptr
            };
            ss = {
                static_cast<std::size_t>(m_strides[0]), // Y-plane stride
                0u,
                0u,
                0u
            };
            break;
        }
        default: {
            GAPI_Error("Non NV12 or GRAY Media format is not expected here");
            break;
        }
    }


    --thread_counters;
    return cv::MediaFrame::View(std::move(ps), std::move(ss));
}

cv::util::any GStreamerMediaAdapter::blobParams() const {
    GAPI_Error("No implementation for GStreamerMediaAdapter::blobParams()");
}

} // namespace gst
} // namespace wip
} // namespace gapi
} // namespace cv
#endif // HAVE_GSTREAMER
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

- **HAVE_GSTREAMER()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `gstreamer_media_adapter.hpp`
- `gstreamer_buffer_utils.hpp`


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

