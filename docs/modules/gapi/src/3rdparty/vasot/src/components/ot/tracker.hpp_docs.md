# Documentation for `modules/gapi/src/3rdparty/vasot/src/components/ot/tracker.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/3rdparty/vasot/src/components/ot/tracker.hpp`
- **File Name**: `tracker.hpp`
- **File Size**: 3,985 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/3rdparty/vasot/src/components/ot/tracker.hpp](../../../../../../../../modules/gapi/src/3rdparty/vasot/src/components/ot/tracker.hpp)

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

#ifndef VAS_OT_TRACKER_HPP
#define VAS_OT_TRACKER_HPP

#include "mtt/objects_associator.hpp"
#include "tracklet.hpp"

#include <vas/common.hpp>

#include <cstdint>
#include <deque>

namespace vas {
namespace ot {

const int32_t kDefaultMaxNumObjects = -1;
const float kMaxTargetAreaFactor = 0.8f;
const float kMinRegionRatioInImageBoundary = 0.75f; // MIN_REGION_RATIO_IN_IMAGE_BOUNDARY

class Tracker {
  public:
    enum Profile {
        PROFILE_LONG_TERM = 0,        // for long-term tracking usage
        PROFILE_SHORT_TERM,           // for short-term tracking usage (suitable for using with an object detector)
        PROFILE_SHORT_TERM_KCFVAR,    // alias of 'PROFILE_SHORT_TERM'. 'PROFILE_SHORT_TERM' will be deprecated
        PROFILE_SHORT_TERM_IMAGELESS, // for short-term tracking usage with kalman tracking
        PROFILE_ZERO_TERM, // for zero-term tracking usage (only works with object association algorithm, not tracking)
        PROFILE_ZERO_TERM_IMAGELESS,       // for zero-term tracking usage with kalman tracking
        PROFILE_ZERO_TERM_COLOR_HISTOGRAM, // alias of 'PROFILE_ZERO_TERM'. 'PROFILE_ZERO_TERM' will be deprecated
    };

    class InitParameters {
      public:
        Profile profile; // tracking type
        int32_t max_num_objects;
        int32_t max_num_threads; // for Parallelization
        vas::ColorFormat format;
        bool tracking_per_class;

        // Won't be exposed to the external
        float min_region_ratio_in_boundary; // For ST, ZT
    };

  public:
    virtual ~Tracker();

    /**
     * create new object tracker instance
     * @param InitParameters
     */
    static Tracker *CreateInstance(InitParameters init_parameters);

    /**
     * perform tracking
     *
     * @param[in] mat Input frame
     * @param[in] detection Newly detected object data vector which will be added to the tracker. put zero length vector
     *            if there is no new object in the frame.
     * @param[in] delta_t Time passed after the latest call to TrackObjects() in seconds. Use 1.0/FPS in case of
     * constant frame rate
     * @param[out] tracklets Tracked object data vector.
     * @return 0 for success. negative value for failure
     */
    virtual int32_t TrackObjects(const cv::Mat &mat, const std::vector<Detection> &detections,
                                 std::vector<std::shared_ptr<Tracklet>> *tracklets, float delta_t = 0.033f) = 0;

    /**
     * remove object
     *
     * @param[in] id Object id for removing. it should be the 'id' value of the Tracklet
     * @return 0 for success. negative value for failure.
     */
    int32_t RemoveObject(const int32_t id);

    /**
     * reset all internal state to its initial.
     *
     * @return 0 for success. negative value for failure.
     */
    void Reset(void);

    /**
     * get cumulated frame number
     *
     * @return 0
     */
    int32_t GetFrameCount(void) const;

  protected:
    explicit Tracker(int32_t max_objects, float min_region_ratio_in_boundary, vas::ColorFormat format,
                     bool class_per_class = true);
    Tracker() = delete;

    int32_t GetNextTrackingID();
    void IncreaseFrameCount();

    void ComputeOcclusion();

    void RemoveOutOfBoundTracklets(int32_t input_width, int32_t input_height, bool is_filtered = false);
    void RemoveDeadTracklets();
    bool RemoveOneLostTracklet();

  protected:
    int32_t max_objects_; // -1 means no limitation
    int32_t next_id_;
    int32_t frame_count_;

    float min_region_ratio_in_boundary_;
    vas::ColorFormat input_image_format_;

    ObjectsAssociator associator_;
    std::vector<std::shared_ptr<Tracklet>> tracklets_;
};

}; // namespace ot
}; // namespace vas

#endif // VAS_OT_TRACKER_HPP
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

- **Tracker**: A class/struct defined in this file
- **InitParameters**: A class/struct defined in this file

### Functions and Methods

- **VAS_OT_TRACKER_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cstdint`
- `vas/common.hpp`
- `deque`
- `tracklet.hpp`
- `mtt/objects_associator.hpp`


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

