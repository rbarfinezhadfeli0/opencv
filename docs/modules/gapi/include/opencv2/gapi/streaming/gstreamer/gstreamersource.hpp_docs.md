# Documentation for `modules/gapi/include/opencv2/gapi/streaming/gstreamer/gstreamersource.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/gstreamer/gstreamersource.hpp`
- **File Name**: `gstreamersource.hpp`
- **File Size**: 3,523 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/gstreamer/gstreamersource.hpp](../../../../../../../modules/gapi/include/opencv2/gapi/streaming/gstreamer/gstreamersource.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming/gstreamer` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_GSTREAMER_GSTREAMERSOURCE_HPP
#define OPENCV_GAPI_STREAMING_GSTREAMER_GSTREAMERSOURCE_HPP

#include <opencv2/gapi/streaming/source.hpp>
#include <opencv2/gapi/garg.hpp>

#include <memory>

namespace cv {
namespace gapi {
namespace wip {
namespace gst {

/**
 * @brief OpenCV's GStreamer streaming source.
 *        Streams cv::Mat-s/cv::MediaFrame from passed GStreamer pipeline.
 *
 * This class implements IStreamSource interface.
 *
 * To create GStreamerSource instance you need to pass 'pipeline' and, optionally, 'outputType'
 * arguments into constructor.
 * 'pipeline' should represent GStreamer pipeline in form of textual description.
 * Almost any custom pipeline is supported which can be successfully ran via gst-launch.
 * The only two limitations are:
 *      - there should be __one__ appsink element in the pipeline to pass data to OpenCV app.
 *        Pipeline can actually contain many sink elements, but it must have one and only one
 *        appsink among them.
 *
 *      - data passed to appsink should be video-frame in NV12 or GRAY8 format.
 *
 * 'outputType' is used to select type of output data to produce: 'cv::MediaFrame' or 'cv::Mat'.
 * To produce 'cv::MediaFrame'-s you need to pass 'GStreamerSource::OutputType::FRAME' and,
 * correspondingly, 'GStreamerSource::OutputType::MAT' to produce 'cv::Mat'-s.
 * Please note, that in the last case, output 'cv::Mat' will be of BGR format, internal conversion
 * from NV12 / GRAY8 GStreamer data will happen.
 * Default value for 'outputType' is 'GStreamerSource::OutputType::MAT'.
 *
 * @note Stream sources are passed to G-API via shared pointers, so please use gapi::make_src<>
 *       to create objects and ptr() to pass a GStreamerSource to cv::gin().
 *
 * @note You need to build OpenCV with GStreamer support to use this class.
 */

class GStreamerPipelineFacade;

class GAPI_EXPORTS GStreamerSource : public IStreamSource
{
public:
    class Priv;

    // Indicates what type of data should be produced by GStreamerSource: cv::MediaFrame or cv::Mat
    enum class OutputType {
        FRAME,
        MAT
    };

    GStreamerSource(const std::string& pipeline,
                    const GStreamerSource::OutputType outputType =
                        GStreamerSource::OutputType::MAT);
    GStreamerSource(std::shared_ptr<GStreamerPipelineFacade> pipeline,
                    const std::string& appsinkName,
                    const GStreamerSource::OutputType outputType =
                        GStreamerSource::OutputType::MAT);

    bool pull(cv::gapi::wip::Data& data) override;
    GMetaArg descr_of() const override;
    ~GStreamerSource() override;

protected:
    explicit GStreamerSource(std::unique_ptr<Priv> priv);

    std::unique_ptr<Priv> m_priv;
};

} // namespace gst

using GStreamerSource = gst::GStreamerSource;

// NB: Overload for using from python
GAPI_EXPORTS_W cv::Ptr<IStreamSource>
inline make_gst_src(const std::string& pipeline,
                    const GStreamerSource::OutputType outputType =
                    GStreamerSource::OutputType::MAT)
{
    return make_src<GStreamerSource>(pipeline, outputType);
}
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STREAMING_GSTREAMER_GSTREAMERSOURCE_HPP
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
- **GStreamerPipelineFacade**: A class/struct defined in this file
- **Priv**: A class/struct defined in this file
- **implements**: A class/struct defined in this file
- **OutputType**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_GSTREAMER_GSTREAMERSOURCE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/streaming/source.hpp`
- `memory`
- `opencv2/gapi/garg.hpp`

**Python Imports:**
- `NV12`
- `python`
- `passed`


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

