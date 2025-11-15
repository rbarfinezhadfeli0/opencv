# Documentation for `modules/videoio/src/backend_static.cpp`

## File Metadata

- **Full Path**: `modules/videoio/src/backend_static.cpp`
- **File Name**: `backend_static.cpp`
- **File Size**: 8,243 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/videoio/src/backend_static.cpp](../../../modules/videoio/src/backend_static.cpp)

## Purpose and Role

This file is located in the `modules/videoio/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#include "backend.hpp"

namespace cv {


void applyParametersFallback(const Ptr<IVideoCapture>& cap, const VideoCaptureParameters& params)
{
    std::vector<int> props = params.getUnused();
    CV_LOG_INFO(NULL, "VIDEOIO: Backend '" << videoio_registry::getBackendName((VideoCaptureAPIs)cap->getCaptureDomain()) <<
                      "' implementation doesn't support parameters in .open(). Applying " <<
                      props.size() << " properties through .setProperty()");
    for (int prop : props)
    {
        double value = params.get<double>(prop, -1);
        CV_LOG_INFO(NULL, "VIDEOIO: apply parameter: [" << prop << "]=" <<
                          cv::format("%g / %lld / 0x%016llx", value, (long long)value, (long long)value));
        if (!cap->setProperty(prop, value))
        {
            if (prop != CAP_PROP_HW_ACCELERATION && prop != CAP_PROP_HW_DEVICE) { // optional parameters
                CV_Error_(cv::Error::StsNotImplemented, ("VIDEOIO: Failed to apply invalid or unsupported parameter: [%d]=%g / %lld / 0x%08llx", prop, value, (long long)value, (long long)value));
            }
        }
    }
    // NB: there is no dedicated "commit" parameters event, implementations should commit after each property automatically
}

// Legacy API. Modern API with parameters is below
class StaticBackend: public IBackend
{
public:
    FN_createCaptureFile fn_createCaptureFile_;
    FN_createCaptureCamera fn_createCaptureCamera_;
    FN_createCaptureStream fn_createCaptureStream_;
    FN_createWriter fn_createWriter_;

    StaticBackend(FN_createCaptureFile fn_createCaptureFile, FN_createCaptureCamera fn_createCaptureCamera, FN_createCaptureStream fn_createCaptureStream, FN_createWriter fn_createWriter)
        : fn_createCaptureFile_(fn_createCaptureFile), fn_createCaptureCamera_(fn_createCaptureCamera), fn_createCaptureStream_(fn_createCaptureStream), fn_createWriter_(fn_createWriter)
    {
        // nothing
    }

    ~StaticBackend() CV_OVERRIDE {}

    Ptr<IVideoCapture> createCapture(int camera, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureCamera_)
        {
            Ptr<IVideoCapture> cap = fn_createCaptureCamera_(camera);
            if (cap && !params.empty())
            {
                applyParametersFallback(cap, params);
            }
            return cap;
        }
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoCapture> createCapture(const std::string &filename, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureFile_)
        {
            Ptr<IVideoCapture> cap = fn_createCaptureFile_(filename);
            if (cap && !params.empty())
            {
                applyParametersFallback(cap, params);
            }
            return cap;
        }
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoCapture> createCapture(const Ptr<IStreamReader>& stream, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureStream_)
        {
            Ptr<IVideoCapture> cap = fn_createCaptureStream_(stream);
            if (cap && !params.empty())
            {
                applyParametersFallback(cap, params);
            }
            return cap;
        }
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoWriter> createWriter(const std::string& filename, int fourcc, double fps,
                                   const cv::Size& sz, const VideoWriterParameters& params) const CV_OVERRIDE
    {
        if (fn_createWriter_)
            return fn_createWriter_(filename, fourcc, fps, sz, params);
        return Ptr<IVideoWriter>();
    }
}; // StaticBackend

class StaticBackendFactory : public IBackendFactory
{
protected:
    Ptr<StaticBackend> backend;

public:
    StaticBackendFactory(FN_createCaptureFile createCaptureFile, FN_createCaptureCamera createCaptureCamera, FN_createCaptureStream createCaptureStream, FN_createWriter createWriter)
        : backend(makePtr<StaticBackend>(createCaptureFile, createCaptureCamera, createCaptureStream, createWriter))
    {
        // nothing
    }

    ~StaticBackendFactory() CV_OVERRIDE {}

    Ptr<IBackend> getBackend() const CV_OVERRIDE
    {
        return backend.staticCast<IBackend>();
    }

    bool isBuiltIn() const CV_OVERRIDE { return true; }
};


Ptr<IBackendFactory> createBackendFactory(FN_createCaptureFile createCaptureFile,
                                          FN_createCaptureCamera createCaptureCamera,
                                          FN_createCaptureStream createCaptureStream,
                                          FN_createWriter createWriter)
{
    return makePtr<StaticBackendFactory>(createCaptureFile, createCaptureCamera, createCaptureStream, createWriter).staticCast<IBackendFactory>();
}



class StaticBackendWithParams: public IBackend
{
public:
    FN_createCaptureFileWithParams fn_createCaptureFile_;
    FN_createCaptureCameraWithParams fn_createCaptureCamera_;
    FN_createCaptureStreamWithParams fn_createCaptureStream_;
    FN_createWriter fn_createWriter_;

    StaticBackendWithParams(FN_createCaptureFileWithParams fn_createCaptureFile, FN_createCaptureCameraWithParams fn_createCaptureCamera, FN_createCaptureStreamWithParams fn_createCaptureStream, FN_createWriter fn_createWriter)
        : fn_createCaptureFile_(fn_createCaptureFile), fn_createCaptureCamera_(fn_createCaptureCamera), fn_createCaptureStream_(fn_createCaptureStream), fn_createWriter_(fn_createWriter)
    {
        // nothing
    }

    ~StaticBackendWithParams() CV_OVERRIDE {}

    Ptr<IVideoCapture> createCapture(int camera, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureCamera_)
            return fn_createCaptureCamera_(camera, params);
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoCapture> createCapture(const std::string &filename, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureFile_)
            return fn_createCaptureFile_(filename, params);
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoCapture> createCapture(const Ptr<IStreamReader>& stream, const VideoCaptureParameters& params) const CV_OVERRIDE
    {
        if (fn_createCaptureStream_)
            return fn_createCaptureStream_(stream, params);
        return Ptr<IVideoCapture>();
    }
    Ptr<IVideoWriter> createWriter(const std::string& filename, int fourcc, double fps,
                                   const cv::Size& sz, const VideoWriterParameters& params) const CV_OVERRIDE
    {
        if (fn_createWriter_)
            return fn_createWriter_(filename, fourcc, fps, sz, params);
        return Ptr<IVideoWriter>();
    }
}; // StaticBackendWithParams

class StaticBackendWithParamsFactory : public IBackendFactory
{
protected:
    Ptr<StaticBackendWithParams> backend;

public:
    StaticBackendWithParamsFactory(FN_createCaptureFileWithParams createCaptureFile, FN_createCaptureCameraWithParams createCaptureCamera, FN_createCaptureStreamWithParams createCaptureStream, FN_createWriter createWriter)
        : backend(makePtr<StaticBackendWithParams>(createCaptureFile, createCaptureCamera, createCaptureStream, createWriter))
    {
        // nothing
    }

    ~StaticBackendWithParamsFactory() CV_OVERRIDE {}

    Ptr<IBackend> getBackend() const CV_OVERRIDE
    {
        return backend.staticCast<IBackend>();
    }

    bool isBuiltIn() const CV_OVERRIDE { return true; }
};


Ptr<IBackendFactory> createBackendFactory(FN_createCaptureFileWithParams createCaptureFile,
                                          FN_createCaptureCameraWithParams createCaptureCamera,
                                          FN_createCaptureStreamWithParams createCaptureStream,
                                          FN_createWriter createWriter)
{
    return makePtr<StaticBackendWithParamsFactory>(createCaptureFile, createCaptureCamera, createCaptureStream, createWriter).staticCast<IBackendFactory>();
}


} // namespace
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

- **StaticBackend**: A class/struct defined in this file
- **StaticBackendWithParamsFactory**: A class/struct defined in this file
- **StaticBackendWithParams**: A class/struct defined in this file
- **StaticBackendFactory**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `backend.hpp`
- `precomp.hpp`


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

