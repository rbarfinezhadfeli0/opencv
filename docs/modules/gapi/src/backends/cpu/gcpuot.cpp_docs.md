# Documentation for `modules/gapi/src/backends/cpu/gcpuot.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/cpu/gcpuot.cpp`
- **File Name**: `gcpuot.cpp`
- **File Size**: 6,198 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/cpu/gcpuot.cpp](../../../../../modules/gapi/src/backends/cpu/gcpuot.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/cpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2023 Intel Corporation

#include <opencv2/gapi/ot.hpp>
#include <opencv2/gapi/cpu/ot.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>

#include <vas/ot.hpp>

namespace cv
{
namespace gapi
{
namespace ot
{

// Helper functions for OT kernels
namespace {
void GTrackImplSetup(cv::GArrayDesc, cv::GArrayDesc, float,
                     std::shared_ptr<vas::ot::ObjectTracker>& state,
                     const ObjectTrackerParams& params) {
    vas::ot::ObjectTracker::Builder ot_builder;
    ot_builder.max_num_objects = params.max_num_objects;
    ot_builder.input_image_format = vas::ColorFormat(params.input_image_format);
    ot_builder.tracking_per_class = params.tracking_per_class;

    state = ot_builder.Build(vas::ot::TrackingType::ZERO_TERM_IMAGELESS);
}

void GTrackImplPrepare(const std::vector<cv::Rect>& in_rects,
                       const std::vector<int32_t>& in_class_labels,
                       float delta,
                       std::vector<vas::ot::DetectedObject>& detected_objs,
                       vas::ot::ObjectTracker& state)
{
    if (in_rects.size() != in_class_labels.size())
    {
        cv::util::throw_error(std::invalid_argument("Track() implementation run() method: in_rects and in_class_labels "
                                                    "sizes are different."));
    }

    detected_objs.reserve(in_rects.size());

    for (std::size_t i = 0; i < in_rects.size(); ++i)
    {
        detected_objs.emplace_back(in_rects[i], in_class_labels[i]);
    }

    state.SetFrameDeltaTime(delta);
}
} // anonymous namespace

GAPI_OCV_KERNEL_ST(GTrackFromMatImpl, cv::gapi::ot::GTrackFromMat, vas::ot::ObjectTracker)
{
    static void setup(cv::GMatDesc, cv::GArrayDesc rects_desc,
                      cv::GArrayDesc labels_desc, float delta,
                      std::shared_ptr<vas::ot::ObjectTracker>& state,
                      const cv::GCompileArgs& compile_args)
    {
        auto params = cv::gapi::getCompileArg<ObjectTrackerParams>(compile_args)
            .value_or(ObjectTrackerParams{});

        GAPI_Assert(params.input_image_format == 0 && "Only BGR input as cv::GMat is supported for now");
        GTrackImplSetup(rects_desc, labels_desc, delta, state, params);
    }

    static void run(const cv::Mat& in_mat, const std::vector<cv::Rect>& in_rects,
                    const std::vector<int32_t>& in_class_labels, float delta,
                    std::vector<cv::Rect>& out_tr_rects,
                    std::vector<int32_t>& out_rects_classes,
                    std::vector<uint64_t>& out_tr_ids,
                    std::vector<int>& out_tr_statuses,
                    vas::ot::ObjectTracker& state)
    {
        std::vector<vas::ot::DetectedObject> detected_objs;
        GTrackImplPrepare(in_rects, in_class_labels, delta, detected_objs, state);

        GAPI_Assert(in_mat.type() == CV_8UC3 && "Input mat is not in BGR format");

        auto objects = state.Track(in_mat, detected_objs);

        for (auto&& object : objects)
        {
            out_tr_rects.push_back(object.rect);
            out_rects_classes.push_back(object.class_label);
            out_tr_ids.push_back(object.tracking_id);
            out_tr_statuses.push_back(static_cast<int>(object.status));
        }
    }
};

GAPI_OCV_KERNEL_ST(GTrackFromFrameImpl, cv::gapi::ot::GTrackFromFrame, vas::ot::ObjectTracker)
{
    static void setup(cv::GFrameDesc, cv::GArrayDesc rects_desc,
                      cv::GArrayDesc labels_desc, float delta,
                      std::shared_ptr<vas::ot::ObjectTracker>& state,
                      const cv::GCompileArgs& compile_args)
    {
        auto params = cv::gapi::getCompileArg<ObjectTrackerParams>(compile_args)
            .value_or(ObjectTrackerParams{});

        GAPI_Assert(params.input_image_format == 1 && "Only NV12 input as cv::GFrame is supported for now");
        GTrackImplSetup(rects_desc, labels_desc, delta, state, params);
    }

    static void run(const cv::MediaFrame& in_frame, const std::vector<cv::Rect>& in_rects,
                    const std::vector<int32_t>& in_class_labels, float delta,
                    std::vector<cv::Rect>& out_tr_rects,
                    std::vector<int32_t>& out_rects_classes,
                    std::vector<uint64_t>& out_tr_ids,
                    std::vector<int>& out_tr_statuses,
                    vas::ot::ObjectTracker& state)
    {
        std::vector<vas::ot::DetectedObject> detected_objs;
        GTrackImplPrepare(in_rects, in_class_labels, delta, detected_objs, state);

        // Extract metadata from MediaFrame and construct cv::Mat atop of it
        cv::MediaFrame::View view = in_frame.access(cv::MediaFrame::Access::R);
        auto ptrs = view.ptr;
        auto strides = view.stride;
        auto desc = in_frame.desc();

        GAPI_Assert((desc.fmt == cv::MediaFormat::NV12 || desc.fmt == cv::MediaFormat::BGR) \
                    && "Input frame is not in NV12 or BGR format");

        cv::Mat in;
        if (desc.fmt == cv::MediaFormat::NV12) {
            GAPI_Assert(ptrs[0] != nullptr && "Y plane pointer is empty");
            GAPI_Assert(ptrs[1] != nullptr && "UV plane pointer is empty");
            if (strides[0] > 0) {
                in = cv::Mat(desc.size, CV_8UC1, ptrs[0], strides[0]);
            } else {
                in = cv::Mat(desc.size, CV_8UC1, ptrs[0]);
            }
        }

        auto objects = state.Track(in, detected_objs);

        for (auto&& object : objects)
        {
            out_tr_rects.push_back(object.rect);
            out_rects_classes.push_back(object.class_label);
            out_tr_ids.push_back(object.tracking_id);
            out_tr_statuses.push_back(static_cast<int>(object.status));
        }
    }
};

cv::gapi::GKernelPackage cpu::kernels()
{
    return cv::gapi::kernels
        <
          GTrackFromFrameImpl,
          GTrackFromMatImpl
        >();
}

}   // namespace ot
}   // namespace gapi
}   // namespace cv
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

### Classes and Structures

- **cv**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/ot.hpp`
- `opencv2/gapi/cpu/ot.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `vas/ot.hpp`

**Python Imports:**
- `MediaFrame`


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

