# Documentation for `modules/gapi/src/streaming/gstreamer/gstreamer_pipeline_facade.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/gstreamer/gstreamer_pipeline_facade.cpp`
- **File Name**: `gstreamer_pipeline_facade.cpp`
- **File Size**: 10,077 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/gstreamer/gstreamer_pipeline_facade.cpp](../../../../../modules/gapi/src/streaming/gstreamer/gstreamer_pipeline_facade.cpp)

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

#include "gstreamerenv.hpp"

#include "gstreamer_pipeline_facade.hpp"

#include <opencv2/gapi/streaming/meta.hpp>

#include <logger.hpp>

#include <opencv2/imgproc.hpp>

#ifdef HAVE_GSTREAMER
#include <gst/app/gstappsink.h>
#include <gst/gstbuffer.h>
#include <gst/video/video-frame.h>
#include <gst/pbutils/missing-plugins.h>

namespace cv {
namespace gapi {
namespace wip {
namespace gst {

GStreamerPipelineFacade::GStreamerPipelineFacade():
    m_isPrerolled(false),
    m_isPlaying(false)
    { }

GStreamerPipelineFacade::GStreamerPipelineFacade(const std::string& pipelineDesc):
    GStreamerPipelineFacade()
{
    m_pipelineDesc = pipelineDesc;

    // Initialize GStreamer library:
    GStreamerEnv::init();

    // Create GStreamer pipeline:
    GError* error = NULL;
    // gst_parse_launch [transfer floating]
    m_pipeline = GST_ELEMENT(g_object_ref_sink(
        gst_parse_launch(m_pipelineDesc.c_str(), &error)));

    GStreamerPtr<GError> err(error);

    if (err)
    {
        cv::util::throw_error(
            std::runtime_error("Error in parsing pipeline: " + std::string(err->message)));
    }
}

// The destructors are noexcept by default (since C++11).
GStreamerPipelineFacade::~GStreamerPipelineFacade()
{
    // There is no mutex acquisition here, because we assume that no one will call this method
    // directly.

    // Destructor may be called on empty GStreamerSource object in case if
    // exception is thrown during construction.
    if (m_pipeline && GST_IS_ELEMENT(m_pipeline.get()))
    {
        try
        {
            setState(GST_STATE_NULL);
        }
        catch(...)
        {
            GAPI_LOG_WARNING(NULL, "Unable to stop pipeline in destructor.\n");
        }

        m_pipeline.release();
    }
}

std::vector<GstElement*> GStreamerPipelineFacade::getElementsByFactoryName(
    const std::string& factoryName)
{
    std::vector<GstElement*> outElements = getElements(
        [&factoryName](GstElement* element) {
            GStreamerPtr<gchar> name(
                gst_object_get_name(GST_OBJECT(gst_element_get_factory(element))));
            return name && (0 == strcmp(name, factoryName.c_str()));
        });

    return outElements;
}

GstElement* GStreamerPipelineFacade::getElementByName(const std::string& elementName)
{
    std::vector<GstElement*> outElements = getElements(
    [&elementName](GstElement* element) {
        GStreamerPtr<gchar> name(gst_element_get_name(element));
        return name && (0 == strcmp(name, elementName.c_str()));
    });

    if (outElements.empty())
    {
        return nullptr;
    }
    else
    {
        GAPI_Assert(1ul == outElements.size());
        return outElements[0];
    }
}

void GStreamerPipelineFacade::completePreroll() {
    // FIXME: If there are multiple sources in pipeline and one of them is live, then pipeline
    //        will return GST_STATE_CHANGE_NO_PREROLL while pipeline pausing.
    //        But appsink may not be connected to this live source and only to another,
    //        not-live ones. So, it is not required to start the playback for appsink to complete
    //        the preroll.
    //        Starting of playback for the not-live sources before the first frame pull will lead
    //        to losing of some amount of frames and pulling of the first frame can return frame
    //        which is far from the first.
    //
    //        Need to handle this case or forbid to mix multiples sources of different
    //        categories(live, not-live) in the pipeline explicitly(assert).

    if (!m_isPrerolled.load(std::memory_order_acquire))
    {
        std::lock_guard<std::mutex> lock(m_stateChangeMutex);

        if(!m_isPrerolled.load(std::memory_order_relaxed))
        {
            PipelineState state = queryState();

            // Only move forward in the pipeline's state machine
            GAPI_Assert(state.current != GST_STATE_PLAYING);

            GAPI_Assert(state.pending == GST_STATE_VOID_PENDING);
            GstStateChangeReturn status = gst_element_set_state(m_pipeline, GST_STATE_PAUSED);
            checkBusMessages();
            if (status == GST_STATE_CHANGE_NO_PREROLL)
            {
                status = gst_element_set_state(m_pipeline, GST_STATE_PLAYING);
                m_isPlaying.store(true);
            }
            verifyStateChange(status);

            m_isPrerolled.store(true, std::memory_order_release);
        }
    }
}

void GStreamerPipelineFacade::play()
{
    if (!m_isPlaying.load(std::memory_order_acquire))
    {
        std::lock_guard<std::mutex> lock(m_stateChangeMutex);

        if (!m_isPlaying.load(std::memory_order_relaxed))
        {
            setState(GST_STATE_PLAYING);
            m_isPlaying.store(true, std::memory_order_release);
            m_isPrerolled.store(true);
        }
    }
}

bool GStreamerPipelineFacade::isPlaying() {
    return m_isPlaying.load();
}

std::vector<GstElement*> GStreamerPipelineFacade::getElements(
    std::function<bool(GstElement*)> comparator)
{
    std::vector<GstElement*> outElements;
    GStreamerPtr<GstIterator> it(gst_bin_iterate_elements(GST_BIN(m_pipeline.get())));
    GValue value = G_VALUE_INIT;

    GstIteratorResult status = gst_iterator_next(it, &value);
    while (status != GST_ITERATOR_DONE && status != GST_ITERATOR_ERROR)
    {
        if (status == GST_ITERATOR_OK)
        {
            GstElement* element = GST_ELEMENT(g_value_get_object(&value));
            if (comparator(element))
            {
                outElements.push_back(GST_ELEMENT(element));
            }

            g_value_unset(&value);
        }
        else if (status == GST_ITERATOR_RESYNC)
        {
            gst_iterator_resync(it);
        }

        status = gst_iterator_next(it, &value);
    }

    return outElements;
}

PipelineState GStreamerPipelineFacade::queryState()
{
    GAPI_Assert(m_pipeline && GST_IS_ELEMENT(m_pipeline.get()) &&
                "GStreamer pipeline has not been created!");

    PipelineState state;
    GstClockTime timeout = 5 * GST_SECOND;
    gst_element_get_state(m_pipeline, &state.current, &state.pending, timeout);

    return state;
}

void GStreamerPipelineFacade::setState(GstState newState)
{
    PipelineState state = queryState();
    GAPI_Assert(state.pending == GST_STATE_VOID_PENDING);

    if (state.current != newState)
    {
        GstStateChangeReturn status = gst_element_set_state(m_pipeline, newState);
        verifyStateChange(status);
    }
}

void GStreamerPipelineFacade::verifyStateChange(GstStateChangeReturn status)
{
    if (status == GST_STATE_CHANGE_ASYNC)
    {
        // Wait for status update.
        status = gst_element_get_state(m_pipeline, NULL, NULL, GST_CLOCK_TIME_NONE);
    }

    if (status == GST_STATE_CHANGE_FAILURE)
    {
        checkBusMessages();
        PipelineState state = queryState();
        const gchar* currentState = gst_element_state_get_name(state.current);
        const gchar* pendingState = gst_element_state_get_name(state.pending);
        cv::util::throw_error(
            std::runtime_error(std::string("Unable to change pipeline state from ") +
                               std::string(currentState) + std::string(" to ") +
                               std::string(pendingState)));
    }

    checkBusMessages();
}

// Handles GStreamer bus messages.
// For debugging purposes.
void GStreamerPipelineFacade::checkBusMessages() const
{
    GStreamerPtr<GstBus> bus(gst_element_get_bus(m_pipeline));

    while (gst_bus_have_pending(bus))
    {
        GStreamerPtr<GstMessage> msg(gst_bus_pop(bus));
        if (!msg || !GST_IS_MESSAGE(msg.get()))
        {
            continue;
        }

        if (gst_is_missing_plugin_message(msg))
        {
            GStreamerPtr<gchar> descr(gst_missing_plugin_message_get_description(msg));
            cv::util::throw_error(
                std::runtime_error("Your GStreamer installation is missing a required plugin!"
                                   "Details: " + std::string(descr)));
        }
        else
        {
            switch (GST_MESSAGE_TYPE(msg))
            {
                case GST_MESSAGE_STATE_CHANGED:
                {
                    if (GST_MESSAGE_SRC(msg.get()) == GST_OBJECT(m_pipeline.get()))
                    {
                        GstState oldState = GST_STATE_NULL,
                                 newState = GST_STATE_NULL;
                        gst_message_parse_state_changed(msg, &oldState, &newState, NULL);
                        const gchar* oldStateName = gst_element_state_get_name(oldState);
                        const gchar* newStateName = gst_element_state_get_name(newState);
                        GAPI_LOG_INFO(NULL, "Pipeline state changed from " << oldStateName << " to "
                                            << newStateName);
                    }
                    break;
                }
                case GST_MESSAGE_ERROR:
                {
                    GError* error = NULL;
                    gchar*  debug = NULL;

                    gst_message_parse_error(msg, &error, &debug); // transfer full for out args

                    GStreamerPtr<GError> err(error);
                    GStreamerPtr<gchar> deb(debug);

                    GStreamerPtr<gchar> name(gst_element_get_name(GST_MESSAGE_SRC(msg.get())));
                    GAPI_LOG_WARNING(NULL, "Embedded video playback halted; module " << name.get()
                                           << " reported: " << err->message);
                    GAPI_LOG_WARNING(NULL, "GStreamer debug: " << deb);

                    break;
                }
                default:
                    break;
            }
        }
    }
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
- `gst/gstbuffer.h`
- `gstreamer_pipeline_facade.hpp`
- `gst/pbutils/missing-plugins.h`
- `gst/app/gstappsink.h`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/streaming/meta.hpp`
- `gstreamerenv.hpp`
- `logger.hpp`
- `gst/video/video-frame.h`

**Python Imports:**
- `the`


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

