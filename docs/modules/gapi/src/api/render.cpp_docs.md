# Documentation for `modules/gapi/src/api/render.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/render.cpp`
- **File Name**: `render.cpp`
- **File Size**: 3,135 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/render.cpp](../../../../modules/gapi/src/api/render.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "precomp.hpp"

#include <stdexcept>

#include <opencv2/gapi/render/render.hpp>
#include <opencv2/gapi/own/assert.hpp>

#include "api/render_priv.hpp"

void cv::gapi::wip::draw::render(cv::Mat& bgr,
                                 const cv::gapi::wip::draw::Prims& prims,
                                 cv::GCompileArgs&& args)
{
    cv::GMat in;
    cv::GArray<cv::gapi::wip::draw::Prim> arr;

    cv::GComputation comp(cv::GIn(in, arr),
                          cv::GOut(cv::gapi::wip::draw::render3ch(in, arr)));
    comp.apply(cv::gin(bgr, prims), cv::gout(bgr), std::move(args));
}

void cv::gapi::wip::draw::render(cv::Mat& y_plane,
                                 cv::Mat& uv_plane,
                                 const Prims& prims,
                                 cv::GCompileArgs&& args)
{
    cv::GMat y_in, uv_in, y_out, uv_out;
    cv::GArray<cv::gapi::wip::draw::Prim> arr;
    std::tie(y_out, uv_out) = cv::gapi::wip::draw::renderNV12(y_in, uv_in, arr);

    cv::GComputation comp(cv::GIn(y_in, uv_in, arr), cv::GOut(y_out, uv_out));
    comp.apply(cv::gin(y_plane, uv_plane, prims),
               cv::gout(y_plane, uv_plane), std::move(args));
}

void cv::gapi::wip::draw::render(cv::MediaFrame& frame,
                                 const Prims& prims,
                                 cv::GCompileArgs&& args)
{
    cv::GFrame in, out;
    cv::GArray<cv::gapi::wip::draw::Prim> arr;
    out = cv::gapi::wip::draw::renderFrame(in, arr);

    cv::GComputation comp(cv::GIn(in, arr), cv::GOut(out));
    comp.apply(cv::gin(frame, prims),
               cv::gout(frame), std::move(args));
}


void cv::gapi::wip::draw::cvtYUVToNV12(const cv::Mat& yuv,
                                       cv::Mat& y,
                                       cv::Mat& uv)
{
    GAPI_Assert(yuv.size().width  % 2 == 0);
    GAPI_Assert(yuv.size().height % 2 == 0);

    std::vector<cv::Mat> chs(3);
    cv::split(yuv, chs);
    y = chs[0];
    cv::merge(std::vector<cv::Mat>{chs[1], chs[2]}, uv);
    cv::resize(uv, uv, uv.size() / 2, cv::INTER_LINEAR);
}

void cv::gapi::wip::draw::cvtNV12ToYUV(const cv::Mat& y,
                                       const cv::Mat& uv,
                                       cv::Mat& yuv)
{
    cv::Mat upsample_uv;
    cv::resize(uv, upsample_uv, uv.size() * 2, cv::INTER_LINEAR);
    cv::merge(std::vector<cv::Mat>{y, upsample_uv}, yuv);
}

cv::GMat cv::gapi::wip::draw::render3ch(const cv::GMat& src,
                                        const cv::GArray<cv::gapi::wip::draw::Prim>& prims)
{
    return cv::gapi::wip::draw::GRenderBGR::on(src, prims);
}

std::tuple<cv::GMat, cv::GMat>
cv::gapi::wip::draw::renderNV12(const cv::GMat& y,
                                const cv::GMat& uv,
                                const cv::GArray<cv::gapi::wip::draw::Prim>& prims)
{
    return cv::gapi::wip::draw::GRenderNV12::on(y, uv, prims);
}

cv::GFrame cv::gapi::wip::draw::renderFrame(const cv::GFrame& frame,
                                                const cv::GArray<cv::gapi::wip::draw::Prim>& prims)
{
    return cv::gapi::wip::draw::GRenderFrame::on(frame, prims);
}
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
- `opencv2/gapi/render/render.hpp`
- `api/render_priv.hpp`
- `opencv2/gapi/own/assert.hpp`
- `stdexcept`
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

