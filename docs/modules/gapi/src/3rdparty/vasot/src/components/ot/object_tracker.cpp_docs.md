# Documentation for `modules/gapi/src/3rdparty/vasot/src/components/ot/object_tracker.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/3rdparty/vasot/src/components/ot/object_tracker.cpp`
- **File Name**: `object_tracker.cpp`
- **File Size**: 12,532 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/3rdparty/vasot/src/components/ot/object_tracker.cpp](../../../../../../../../modules/gapi/src/3rdparty/vasot/src/components/ot/object_tracker.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/3rdparty/vasot/src/components/ot` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*******************************************************************************
 * Copyright (C) 2023 Intel Corporation
 *
 * SPDX-License-Identifier: MIT
 ******************************************************************************/

#include "prof_def.hpp"
#include "tracker.hpp"
#include "../../common/exception.hpp"

#include <vas/ot.hpp>
#include <vas/common.hpp>

#include <memory>

namespace vas {
namespace ot {
const float kDefaultDeltaTime = 0.033f;
const int kDefaultNumThreads = 1;
const char kNameMaxNumThreads[] = "max_num_threads";

vas::Version GetVersion() noexcept {
    vas::Version version(OT_VERSION_MAJOR, OT_VERSION_MINOR, OT_VERSION_PATCH);
    return version;
}

std::ostream &operator<<(std::ostream &os, TrackingStatus ts) {
    if (ts == TrackingStatus::NEW)
        os << "NEW";
    else if (ts == TrackingStatus::TRACKED)
        os << "TRACKED";
    // else if (ts == TrackingStatus::LOST)
    else
        os << "LOST";

    return os;
}

std::ostream &operator<<(std::ostream &os, const Object &object) {
    os << "Object:" << std::endl;
    os << "    rect            -> " << object.rect << std::endl;
    os << "    tracking id     -> " << object.tracking_id << std::endl;
    os << "    class label     -> " << object.class_label << std::endl;
    os << "    tracking status -> " << object.status;

    return os;
}

// Internal implementation: includes OT component
class ObjectTracker::Impl {
  public:
    class InitParameters : public vas::ot::Tracker::InitParameters {
      public:
        TrackingType tracking_type;
        vas::BackendType backend_type;
    };

  public:
    explicit Impl(const InitParameters &param);

    Impl() = delete;
    ~Impl();
    Impl(const Impl &) = delete;
    Impl(Impl &&) = delete;
    Impl &operator=(const Impl &) = delete;
    Impl &operator=(Impl &&) = delete;

  public:
    int32_t GetMaxNumObjects() const noexcept;
    TrackingType GetTrackingType() const noexcept;
    vas::ColorFormat GetInputColorFormat() const noexcept;
    float GetDeltaTime() const noexcept;
    vas::BackendType GetBackendType() const noexcept;
    bool GetTrackingPerClass() const noexcept;
    void SetDeltaTime(float delta_t);
    std::vector<Object> Track(const cv::Mat &frame, const std::vector<DetectedObject> &objects);

  private:
    std::unique_ptr<vas::ot::Tracker> tracker_;
    std::vector<std::shared_ptr<Tracklet>> produced_tracklets_;

    int32_t max_num_objects_;
    float delta_t_;
    TrackingType tracking_type_;
    vas::BackendType backend_type_;
    vas::ColorFormat input_color_format_;
    bool tracking_per_class_;
#ifdef DUMP_OTAV
    Otav otav_;
#endif

    friend class ObjectTracker::Builder;
};

namespace {
void vas_exit() {
}
} // anonymous namespace

ObjectTracker::ObjectTracker(ObjectTracker::Impl *impl) : impl_(impl) {
    atexit(vas_exit);
}

ObjectTracker::~ObjectTracker() = default;

int32_t ObjectTracker::GetMaxNumObjects() const noexcept {
    return impl_->GetMaxNumObjects();
}

TrackingType ObjectTracker::GetTrackingType() const noexcept {
    return impl_->GetTrackingType();
}

vas::ColorFormat ObjectTracker::GetInputColorFormat() const noexcept {
    return impl_->GetInputColorFormat();
}

float ObjectTracker::GetFrameDeltaTime() const noexcept {
    return impl_->GetDeltaTime();
}

vas::BackendType ObjectTracker::GetBackendType() const noexcept {
    return impl_->GetBackendType();
}

bool ObjectTracker::GetTrackingPerClass() const noexcept {
    return impl_->GetTrackingPerClass();
}

void ObjectTracker::SetFrameDeltaTime(float frame_delta_t) {
    impl_->SetDeltaTime(frame_delta_t);
}

std::vector<Object> ObjectTracker::Track(const cv::Mat &frame, const std::vector<DetectedObject> &objects) {
    return impl_->Track(frame, objects);
}

ObjectTracker::Impl::Impl(const InitParameters &param)
    : max_num_objects_(param.max_num_objects), delta_t_(kDefaultDeltaTime), tracking_type_(param.tracking_type),
      backend_type_(param.backend_type), input_color_format_(param.format),
      tracking_per_class_(param.tracking_per_class) {
    PROF_INIT(OT);
    TRACE("BEGIN");
    if ((param.max_num_objects) != -1 && (param.max_num_objects <= 0)) {
        std::cout << "Error: Invalid maximum number of objects: " << param.max_num_objects << std::endl;
        ETHROW(false, invalid_argument, "Invalid maximum number of objects");
    }

    TRACE("tracking_type: %d, backend_type: %d, color_format: %d, max_num_object: %d, tracking_per_class: %d",
          static_cast<int32_t>(tracking_type_), static_cast<int32_t>(backend_type_),
          static_cast<int32_t>(input_color_format_), max_num_objects_, tracking_per_class_);

    if (param.backend_type == vas::BackendType::CPU) {
        tracker_.reset(vas::ot::Tracker::CreateInstance(param));
    } else {
        std::cout << "Error: Unexpected backend type" << std::endl;
        ETHROW(false, invalid_argument, "Unexpected backend type");
    }

    produced_tracklets_.clear();

    TRACE("END");
}

ObjectTracker::Impl::~Impl() {
    PROF_FLUSH(OT);
}

void ObjectTracker::Impl::SetDeltaTime(float delta_t) {
    if (delta_t < 0.005f || delta_t > 0.5f) {
        std::cout << "Error: Invalid argument for SetFrameDeltaTime " << delta_t << std::endl;
        ETHROW(false, invalid_argument, "Invalid argument for SetFrameDeltaTime");
    }

    delta_t_ = delta_t;
    return;
}

int32_t ObjectTracker::Impl::GetMaxNumObjects() const noexcept {
    return max_num_objects_;
}

TrackingType ObjectTracker::Impl::GetTrackingType() const noexcept {
    return tracking_type_;
}

vas::ColorFormat ObjectTracker::Impl::GetInputColorFormat() const noexcept {
    return input_color_format_;
}

float ObjectTracker::Impl::GetDeltaTime() const noexcept {
    return delta_t_;
}

vas::BackendType ObjectTracker::Impl::GetBackendType() const noexcept {
    return backend_type_;
}

bool ObjectTracker::Impl::GetTrackingPerClass() const noexcept {
    return tracking_per_class_;
}

std::vector<Object> ObjectTracker::Impl::Track(const cv::Mat &frame,
                                               const std::vector<DetectedObject> &detected_objects) {
    if (frame.cols <= 0 || frame.rows <= 0) {
        std::cout << "Error: Invalid frame size(" << frame.cols << "x" << frame.rows << ") empty("
                  << frame.empty() << ")" << std::endl;
        ETHROW(false, invalid_argument, "Invalid frame size(%dx%d) empty(%d)\n", frame.cols, frame.rows, frame.empty());
    }
    int32_t frame_w = frame.cols;
    int32_t frmae_h = (input_color_format_ == vas::ColorFormat::NV12) ? frame.rows * 2 / 3 : frame.rows;
    cv::Rect frame_rect(0, 0, frame_w, frmae_h);

    TRACE("START");
    PROF_START(PROF_COMPONENTS_OT_RUN_TRACK);
    std::vector<vas::ot::Detection> detections;

    TRACE("+ Number: Detected objects (%d)", static_cast<int32_t>(detected_objects.size()));
    int32_t index = 0;
    for (const auto &object : detected_objects) {
        vas::ot::Detection detection;

        detection.class_label = object.class_label;
        detection.rect = static_cast<cv::Rect2f>(object.rect);
        detection.index = index;

        detections.emplace_back(detection);
        index++;
    }

    std::vector<Object> objects;
    if (backend_type_ == vas::BackendType::CPU) {
        tracker_->TrackObjects(frame, detections, &produced_tracklets_, delta_t_);
        TRACE("+ Number: Tracking objects (%d)", static_cast<int32_t>(produced_tracklets_.size()));

        for (const auto &tracklet : produced_tracklets_) // result 'Tracklet'
        {
            cv::Rect rect = static_cast<cv::Rect>(tracklet->trajectory_filtered.back());
            if ((rect & frame_rect).area() > 0) {
                Object object;
                // TRACE("     - ID(%d) Status(%d)", tracklet.id, tracklet.status);
                object.rect = static_cast<cv::Rect>(tracklet->trajectory_filtered.back());
                object.tracking_id = tracklet->id;
                object.class_label = tracklet->label;
                object.association_idx = tracklet->association_idx;
                object.status = vas::ot::TrackingStatus::LOST;
                switch (tracklet->status) {
                case ST_NEW:
                    object.status = vas::ot::TrackingStatus::NEW;
                    break;
                case ST_TRACKED:
                    object.status = vas::ot::TrackingStatus::TRACKED;
                    break;
                case ST_LOST:
                default:
                    object.status = vas::ot::TrackingStatus::LOST;
                }
                objects.emplace_back(object);
            } else {
                TRACE("[ %d, %d, %d, %d ] is out of the image bound! -> Filtered out.", rect.x, rect.y, rect.width,
                      rect.height);
            }
        }
    } else {
        ETHROW(false, invalid_argument, "Unexpected input backend type for VAS-OT.")
    }
    TRACE("+ Number: Result objects (%d)", static_cast<int32_t>(objects.size()));

    PROF_END(PROF_COMPONENTS_OT_RUN_TRACK);

#ifdef DUMP_OTAV
    otav_.Dump(frame, detections, produced_tracklets_, tracker_->GetFrameCount() - 1);
#endif

    TRACE("END");
    return objects;
}

ObjectTracker::Builder::Builder()
    : backend_type(vas::BackendType::CPU), max_num_objects(kDefaultMaxNumObjects),
      input_image_format(vas::ColorFormat::BGR), tracking_per_class(true) {
}

ObjectTracker::Builder::~Builder() {
}

std::unique_ptr<ObjectTracker> ObjectTracker::Builder::Build(TrackingType tracking_type) const {
    TRACE("BEGIN");

    ObjectTracker::Impl *ot_impl = nullptr;
    ObjectTracker::Impl::InitParameters param;

    param.max_num_objects = max_num_objects;
    param.format = input_image_format;
    param.backend_type = backend_type;
    param.tracking_type = tracking_type;
    param.tracking_per_class = tracking_per_class;

    if (static_cast<int32_t>(vas::ColorFormat::BGR) > static_cast<int32_t>(input_image_format) ||
        static_cast<int32_t>(vas::ColorFormat::I420) < static_cast<int32_t>(input_image_format)) {
        ETHROW(false, invalid_argument, "Invalid color format(%d)", static_cast<int32_t>(input_image_format));
    }

    switch (tracking_type) {
    case vas::ot::TrackingType::LONG_TERM:
        param.profile = vas::ot::Tracker::PROFILE_LONG_TERM;
        break;
    case vas::ot::TrackingType::SHORT_TERM:
        param.profile = vas::ot::Tracker::PROFILE_SHORT_TERM;
        break;
    case vas::ot::TrackingType::SHORT_TERM_KCFVAR:
        param.profile = vas::ot::Tracker::PROFILE_SHORT_TERM_KCFVAR;
        break;
    case vas::ot::TrackingType::SHORT_TERM_IMAGELESS:
        param.profile = vas::ot::Tracker::PROFILE_SHORT_TERM_IMAGELESS;
        break;
    case vas::ot::TrackingType::ZERO_TERM:
        param.profile = vas::ot::Tracker::PROFILE_ZERO_TERM;
        break;
    case vas::ot::TrackingType::ZERO_TERM_COLOR_HISTOGRAM:
        param.profile = vas::ot::Tracker::PROFILE_ZERO_TERM_COLOR_HISTOGRAM;
        break;
    case vas::ot::TrackingType::ZERO_TERM_IMAGELESS:
        param.profile = vas::ot::Tracker::PROFILE_ZERO_TERM_IMAGELESS;
        break;
    default:
        std::cout << "Error: Invalid tracker type vas::ot::Tracker" << std::endl;
        ETHROW(false, invalid_argument, "Invalid tracker type vas::ot::Tracker");
        return nullptr;
    }

    // Not exposed to external parameter
    param.min_region_ratio_in_boundary =
        kMinRegionRatioInImageBoundary; // Ratio threshold of size: used by zttchist, zttimgless, sttkcfvar, sttimgless

    for (const auto &item : platform_config) {
        (void)item; // resolves ununsed warning when LOG_TRACE is OFF
        TRACE("platform_config[%s] = %s", item.first.c_str(), item.second.c_str());
    }

    int max_num_threads = kDefaultNumThreads;
    auto max_num_threads_iter = platform_config.find(kNameMaxNumThreads);
    if (max_num_threads_iter != platform_config.end()) {
        try {
            max_num_threads = std::stoi(max_num_threads_iter->second);
        } catch (const std::exception &) {
            ETHROW(false, invalid_argument, "max_num_threads should be integer");
        }

        if (max_num_threads == 0 || max_num_threads < -1)
            ETHROW(false, invalid_argument, "max_num_threads cannot be 0 or smaller than -1");
    }
    param.max_num_threads = max_num_threads;

    ot_impl = new ObjectTracker::Impl(param);
    std::unique_ptr<ObjectTracker> ot(new ObjectTracker(ot_impl));

    TRACE("END");
    return ot;
}

}; // namespace ot
}; // namespace vas
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

- **label**: A class/struct defined in this file
- **InitParameters**: A class/struct defined in this file
- **ObjectTracker**: A class/struct defined in this file

### Functions and Methods

- **DUMP_OTAV()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `vas/common.hpp`
- `../../common/exception.hpp`
- `prof_def.hpp`
- `memory`
- `vas/ot.hpp`
- `tracker.hpp`


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

