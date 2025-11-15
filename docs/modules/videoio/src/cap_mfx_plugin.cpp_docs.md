# Documentation for `modules/videoio/src/cap_mfx_plugin.cpp`

## File Metadata

- **Full Path**: `modules/videoio/src/cap_mfx_plugin.cpp`
- **File Name**: `cap_mfx_plugin.cpp`
- **File Size**: 7,035 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/videoio/src/cap_mfx_plugin.cpp](../../../modules/videoio/src/cap_mfx_plugin.cpp)

## Purpose and Role

This file is located in the `modules/videoio/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#if defined(BUILD_PLUGIN)

#include <string>
#include "cap_mfx_reader.hpp"
#include "cap_mfx_writer.hpp"

#define ABI_VERSION 0
#define API_VERSION 0
#include "plugin_api.hpp"

using namespace std;

namespace cv {

static
CvResult CV_API_CALL cv_capture_open(const char* filename, int, CV_OUT CvPluginCapture* handle)
{
    if (!handle)
        return CV_ERROR_FAIL;
    *handle = NULL;
    if (!filename)
        return CV_ERROR_FAIL;
    VideoCapture_IntelMFX *cap = 0;
    try
    {
        if (filename)
        {
            cap = new VideoCapture_IntelMFX(string(filename));
            if (cap->isOpened())
            {
                *handle = (CvPluginCapture)cap;
                return CV_ERROR_OK;
            }
        }
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
    }
    if (cap)
        delete cap;
    return CV_ERROR_FAIL;
}

static
CvResult CV_API_CALL cv_capture_release(CvPluginCapture handle)
{
    if (!handle)
        return CV_ERROR_FAIL;
    VideoCapture_IntelMFX* instance = (VideoCapture_IntelMFX*)handle;
    delete instance;
    return CV_ERROR_OK;
}


static
CvResult CV_API_CALL cv_capture_get_prop(CvPluginCapture handle, int prop, CV_OUT double* val)
{
    if (!handle)
        return CV_ERROR_FAIL;
    if (!val)
        return CV_ERROR_FAIL;
    try
    {
        VideoCapture_IntelMFX* instance = (VideoCapture_IntelMFX*)handle;
        *val = instance->getProperty(prop);
        return CV_ERROR_OK;
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
        return CV_ERROR_FAIL;
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
        return CV_ERROR_FAIL;
    }
}

static
CvResult CV_API_CALL cv_capture_set_prop(CvPluginCapture handle, int prop, double val)
{
    if (!handle)
        return CV_ERROR_FAIL;
    try
    {
        VideoCapture_IntelMFX* instance = (VideoCapture_IntelMFX*)handle;
        return instance->setProperty(prop, val) ? CV_ERROR_OK : CV_ERROR_FAIL;
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
        return CV_ERROR_FAIL;
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
        return CV_ERROR_FAIL;
    }
}

static
CvResult CV_API_CALL cv_capture_grab(CvPluginCapture handle)
{
    if (!handle)
        return CV_ERROR_FAIL;
    try
    {
        VideoCapture_IntelMFX* instance = (VideoCapture_IntelMFX*)handle;
        return instance->grabFrame() ? CV_ERROR_OK : CV_ERROR_FAIL;
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
        return CV_ERROR_FAIL;
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
        return CV_ERROR_FAIL;
    }
}

static
CvResult CV_API_CALL cv_capture_retrieve(CvPluginCapture handle, int stream_idx, cv_videoio_retrieve_cb_t callback, void* userdata)
{
    if (!handle)
        return CV_ERROR_FAIL;
    try
    {
        VideoCapture_IntelMFX* instance = (VideoCapture_IntelMFX*)handle;
        Mat img;
        if (instance->retrieveFrame(stream_idx, img))
            return callback(stream_idx, img.data, (int)img.step, img.cols, img.rows, img.channels(), userdata);
        return CV_ERROR_FAIL;
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
        return CV_ERROR_FAIL;
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
        return CV_ERROR_FAIL;
    }
}

static
CvResult CV_API_CALL cv_writer_open(const char* filename, int fourcc, double fps, int width, int height, int isColor,
                                    CV_OUT CvPluginWriter* handle)
{
    VideoWriter_IntelMFX* wrt = 0;
    try
    {
        wrt = new VideoWriter_IntelMFX(filename, fourcc, fps, Size(width, height), isColor);
        if(wrt->isOpened())
        {
            *handle = (CvPluginWriter)wrt;
            return CV_ERROR_OK;
        }
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
    }
    if (wrt)
        delete wrt;
    return CV_ERROR_FAIL;
}

static
CvResult CV_API_CALL cv_writer_release(CvPluginWriter handle)
{
    if (!handle)
        return CV_ERROR_FAIL;
    VideoWriter_IntelMFX* instance = (VideoWriter_IntelMFX*)handle;
    delete instance;
    return CV_ERROR_OK;
}

static
CvResult CV_API_CALL cv_writer_get_prop(CvPluginWriter /*handle*/, int /*prop*/, CV_OUT double* /*val*/)
{
    return CV_ERROR_FAIL;
}

static
CvResult CV_API_CALL cv_writer_set_prop(CvPluginWriter /*handle*/, int /*prop*/, double /*val*/)
{
    return CV_ERROR_FAIL;
}

static
CvResult CV_API_CALL cv_writer_write(CvPluginWriter handle, const unsigned char *data, int step, int width, int height, int cn)
{
    if (!handle)
        return CV_ERROR_FAIL;
    try
    {
        VideoWriter_IntelMFX* instance = (VideoWriter_IntelMFX*)handle;
        Mat img(Size(width, height), CV_MAKETYPE(CV_8U, cn), const_cast<uchar*>(data), step);
        instance->write(img);
        return CV_ERROR_OK;
    }
    catch (const std::exception& e)
    {
        CV_LOG_WARNING(NULL, "MFX: Exception is raised: " << e.what());
        return CV_ERROR_FAIL;
    }
    catch (...)
    {
        CV_LOG_WARNING(NULL, "MFX: Unknown C++ exception is raised");
        return CV_ERROR_FAIL;
    }
}

static const OpenCV_VideoIO_Plugin_API_preview plugin_api =
{
    {
        sizeof(OpenCV_VideoIO_Plugin_API_preview), ABI_VERSION, API_VERSION,
        CV_VERSION_MAJOR, CV_VERSION_MINOR, CV_VERSION_REVISION, CV_VERSION_STATUS,
        "MediaSDK OpenCV Video I/O plugin"
    },
    {
        /*  1*/CAP_INTEL_MFX,
        /*  2*/cv_capture_open,
        /*  3*/cv_capture_release,
        /*  4*/cv_capture_get_prop,
        /*  5*/cv_capture_set_prop,
        /*  6*/cv_capture_grab,
        /*  7*/cv_capture_retrieve,
        /*  8*/cv_writer_open,
        /*  9*/cv_writer_release,
        /* 10*/cv_writer_get_prop,
        /* 11*/cv_writer_set_prop,
        /* 12*/cv_writer_write
    }
};

} // namespace

const OpenCV_VideoIO_Plugin_API_preview* opencv_videoio_plugin_init_v0(int requested_abi_version, int requested_api_version, void* /*reserved=NULL*/) CV_NOEXCEPT
{
    if (requested_abi_version == ABI_VERSION && requested_api_version <= API_VERSION)
        return &cv::plugin_api;
    return NULL;
}

#endif // BUILD_PLUGIN
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `string`
- `cap_mfx_writer.hpp`
- `plugin_api.hpp`
- `cap_mfx_reader.hpp`


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

