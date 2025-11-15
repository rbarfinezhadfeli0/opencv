# Documentation for `modules/gapi/include/opencv2/gapi/streaming/cap.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/cap.hpp`
- **File Name**: `cap.hpp`
- **File Size**: 4,529 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/cap.hpp](../../../../../../modules/gapi/include/opencv2/gapi/streaming/cap.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_CAP_HPP
#define OPENCV_GAPI_STREAMING_CAP_HPP

/**
 * YOUR ATTENTION PLEASE!
 *
 * This is a header-only implementation of cv::VideoCapture-based
 * Stream source.  It is not built by default with G-API as G-API
 * doesn't depend on videoio module.
 *
 * If you want to use it in your application, please make sure
 * videioio is available in your OpenCV package and is linked to your
 * application.
 *
 * Note for developers: please don't put videoio dependency in G-API
 * because of this file.
 */
#include <chrono>
#include <map>

#include <opencv2/videoio.hpp>
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/streaming/meta.hpp>

namespace cv {
namespace gapi {
namespace wip {

/**
 * @brief OpenCV's VideoCapture-based streaming source.
 *
 * This class implements IStreamSource interface.
 * Its constructor takes the same parameters as cv::VideoCapture does.
 *
 * Please make sure that videoio OpenCV module is available before using
 * this in your application (G-API doesn't depend on it directly).
 *
 * @note stream sources are passed to G-API via shared pointers, so
 *  please gapi::make_src<> to create objects and ptr() to pass a
 *  GCaptureSource to cv::gin().
 */
class GCaptureSource: public IStreamSource
{
public:
    explicit GCaptureSource(int id, const std::map<int, double> &properties = {})
        : cap(id) { prep(properties); }

    explicit GCaptureSource(const std::string &path,
                            const std::map<int, double> &properties = {})
        : cap(path) { prep(properties); }

    void set(int propid, double value) {
        cap.set(propid, value);
    }

    // TODO: Add more constructor overloads to make it
    // fully compatible with VideoCapture's interface.

protected:
    cv::VideoCapture cap;
    cv::Mat first;
    bool first_pulled = false;
    int64_t counter = 0;

    void prep(const std::map<int, double> &properties)
    {
        for (const auto &it : properties) {
            cap.set(it.first, it.second);
        }

        // Prepare first frame to report its meta to engine
        // when needed
        GAPI_Assert(first.empty());
        cv::Mat tmp;
        if (!cap.read(tmp))
        {
            GAPI_Error("Couldn't grab the very first frame");
        }
        // NOTE: Some decode/media VideoCapture backends continue
        // owning the video buffer under cv::Mat so in order to
        // process it safely in a highly concurrent pipeline, clone()
        // is the only right way.
        first = tmp.clone();
    }

    virtual bool pull(cv::gapi::wip::Data &data) override
    {
        if (!first_pulled)
        {
            GAPI_Assert(!first.empty());
            first_pulled = true;
            data = first; // no need to clone here since it was cloned already
        }
        else
        {
            if (!cap.isOpened()) return false;

            cv::Mat frame;
            if (!cap.read(frame))
            {
                // end-of-stream happened
                return false;
            }
            // Same reason to clone as in prep()
            data = frame.clone();
        }
        // Tag data with seq_id/ts
        const auto now = std::chrono::system_clock::now();
        const auto dur = std::chrono::duration_cast<std::chrono::microseconds>
            (now.time_since_epoch());
        data.meta[cv::gapi::streaming::meta_tag::timestamp] = int64_t{dur.count()};
        data.meta[cv::gapi::streaming::meta_tag::seq_id]    = int64_t{counter++};
        return true;
    }

    virtual GMetaArg descr_of() const override
    {
        GAPI_Assert(!first.empty());
        return cv::GMetaArg{cv::descr_of(first)};
    }
};

// NB: Overload for using from python
GAPI_EXPORTS_W cv::Ptr<IStreamSource>
inline make_capture_src(const std::string& path,
                        const std::map<int, double>& properties = {})
{
    return make_src<GCaptureSource>(path, properties);
}

// NB: Overload for using from python
GAPI_EXPORTS_W cv::Ptr<IStreamSource>
inline make_capture_src(const int id,
                        const std::map<int, double>& properties = {})
{
    return make_src<GCaptureSource>(id, properties);
}

} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STREAMING_CAP_HPP
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

- **implements**: A class/struct defined in this file
- **GCaptureSource**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_CAP_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/garg.hpp`
- `chrono`
- `opencv2/gapi/streaming/meta.hpp`
- `opencv2/videoio.hpp`
- `map`

**Python Imports:**
- `python`


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

