# Documentation for `modules/gapi/src/backends/render/grenderocv.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/render/grenderocv.cpp`
- **File Name**: `grenderocv.cpp`
- **File Size**: 7,239 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/render/grenderocv.cpp](../../../../../modules/gapi/src/backends/render/grenderocv.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/render` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/imgproc.hpp>

#include "api/render_ocv.hpp"

#include <opencv2/gapi/cpu/gcpukernel.hpp>
#include <opencv2/gapi/fluid/core.hpp>

struct RenderOCVState
{
    std::shared_ptr<cv::gapi::wip::draw::FTTextRender> ftpr;
};

GAPI_OCV_KERNEL_ST(RenderBGROCVImpl, cv::gapi::wip::draw::GRenderBGR, RenderOCVState)
{
    static void run(const cv::Mat& in,
                    const cv::gapi::wip::draw::Prims& prims,
                    cv::Mat& out,
                    RenderOCVState& state)
    {
        // NB: If in and out cv::Mats are the same object
        // we can avoid copy and render on out cv::Mat
        // It's work if this kernel is last operation in the graph
        if (in.data != out.data) {
            in.copyTo(out);
        }

        cv::gapi::wip::draw::drawPrimitivesOCVBGR(out, prims, state.ftpr);
    }

    static void setup(const cv::GMatDesc& /* in */,
                      const cv::GArrayDesc& /* prims */,
                      std::shared_ptr<RenderOCVState>& state,
                      const cv::GCompileArgs& args)
    {
        using namespace cv::gapi::wip::draw;
        auto opt_freetype_font = cv::gapi::getCompileArg<freetype_font>(args);
        state = std::make_shared<RenderOCVState>();

        if (opt_freetype_font.has_value())
        {
            state->ftpr = std::make_shared<FTTextRender>(opt_freetype_font->path);
        }
    }
};

GAPI_OCV_KERNEL_ST(RenderNV12OCVImpl, cv::gapi::wip::draw::GRenderNV12, RenderOCVState)
{
    static void run(const cv::Mat& in_y,
                    const cv::Mat& in_uv,
                    const cv::gapi::wip::draw::Prims& prims,
                    cv::Mat& out_y,
                    cv::Mat& out_uv,
                    RenderOCVState& state)
    {
        // NB: If in and out cv::Mats are the same object
        // we can avoid copy and render on out cv::Mat
        // It's work if this kernel is last operation in the graph
        if (in_y.data != out_y.data) {
            in_y.copyTo(out_y);
        }

        if (in_uv.data != out_uv.data) {
            in_uv.copyTo(out_uv);
        }

        /* FIXME How to render correctly on NV12 format ?
         *
         * Rendering on NV12 via OpenCV looks like this:
         *
         * y --------> 1)(NV12 -> YUV) -> yuv -> 2)draw -> yuv -> 3)split -------> out_y
         *                  ^                                         |
         *                  |                                         |
         * uv --------------                                          `----------> out_uv
         *
         *
         * 1) Collect yuv mat from two planes, uv plain in two times less than y plane
         *    so, upsample uv in tow times, with bilinear interpolation
         *
         * 2) Render primitives on YUV
         *
         * 3) Convert yuv to NV12 (using bilinear interpolation)
         *
         */

        // NV12 -> YUV
        cv::Mat upsample_uv, yuv;
        cv::resize(in_uv, upsample_uv, in_uv.size() * 2, cv::INTER_LINEAR);
        cv::merge(std::vector<cv::Mat>{in_y, upsample_uv}, yuv);

        cv::gapi::wip::draw::drawPrimitivesOCVYUV(yuv, prims, state.ftpr);

        // YUV -> NV12
        cv::Mat out_u, out_v, uv_plane;
        std::vector<cv::Mat> chs = {out_y, out_u, out_v};
        cv::split(yuv, chs);
        cv::merge(std::vector<cv::Mat>{chs[1], chs[2]}, uv_plane);
        cv::resize(uv_plane, out_uv, uv_plane.size() / 2, cv::INTER_LINEAR);
    }

    static void setup(const cv::GMatDesc&   /* in_y  */,
                      const cv::GMatDesc&   /* in_uv */,
                      const cv::GArrayDesc& /* prims */,
                      std::shared_ptr<RenderOCVState>& state,
                      const cv::GCompileArgs& args)
    {
        using namespace cv::gapi::wip::draw;
        auto has_freetype_font = cv::gapi::getCompileArg<freetype_font>(args);
        state = std::make_shared<RenderOCVState>();

        if (has_freetype_font)
        {
            state->ftpr = std::make_shared<FTTextRender>(has_freetype_font->path);
        }
    }
};

GAPI_OCV_KERNEL_ST(RenderFrameOCVImpl, cv::gapi::wip::draw::GRenderFrame, RenderOCVState)
{
    static void run(const cv::MediaFrame & in,
                    const cv::gapi::wip::draw::Prims & prims,
                    cv::MediaFrame & out,
                    RenderOCVState & state)
    {
        GAPI_Assert(in.desc().fmt == cv::MediaFormat::NV12);

        // FIXME: consider a better approach (aka native inplace operation)
        // Non-intuitive logic with shared_ptr Priv class
        out = in;

        auto desc = out.desc();
        cv::Mat upsample_uv, yuv;
        {
            auto r_in = in.access(cv::MediaFrame::Access::R);

            auto in_y = cv::Mat(desc.size, CV_8UC1, r_in.ptr[0], r_in.stride[0]);
            auto in_uv = cv::Mat(desc.size / 2, CV_8UC2, r_in.ptr[1], r_in.stride[1]);

        /* FIXME How to render correctly on NV12 format ?
         *
         * Rendering on NV12 via OpenCV looks like this:
         *
         * y --------> 1)(NV12 -> YUV) -> yuv -> 2)draw -> yuv -> 3)split -------> out_y
         *                  ^                                         |
         *                  |                                         |
         * uv --------------                                          `----------> out_uv
         *
         *
         * 1) Collect yuv mat from two planes, uv plain in two times less than y plane
         *    so, upsample uv in two times, with bilinear interpolation
         *
         * 2) Render primitives on YUV
         *
         * 3) Convert yuv to NV12 (using bilinear interpolation)
         *
         */

            // NV12 -> YUV
            cv::resize(in_uv, upsample_uv, in_uv.size() * 2, cv::INTER_LINEAR);
            cv::merge(std::vector<cv::Mat>{in_y, upsample_uv}, yuv);
        }

        cv::gapi::wip::draw::drawPrimitivesOCVYUV(yuv, prims, state.ftpr);

        // YUV -> NV12
        {
            auto w_out = out.access(cv::MediaFrame::Access::W);

            auto out_y = cv::Mat(desc.size, CV_8UC1, w_out.ptr[0], w_out.stride[0]);
            auto out_uv = cv::Mat(desc.size / 2, CV_8UC2, w_out.ptr[1], w_out.stride[1]);

            cv::Mat out_u, out_v, uv_plane;
            std::vector<cv::Mat> chs = { out_y, out_u, out_v };
            cv::split(yuv, chs);
            cv::merge(std::vector<cv::Mat>{chs[1], chs[2]}, uv_plane);
            cv::resize(uv_plane, out_uv, uv_plane.size() / 2, cv::INTER_LINEAR);
        }
    }

    static void setup(const cv::GFrameDesc&   /* in_nv12  */,
        const cv::GArrayDesc& /* prims */,
        std::shared_ptr<RenderOCVState>&state,
        const cv::GCompileArgs & args)
    {
        using namespace cv::gapi::wip::draw;
        auto has_freetype_font = cv::gapi::getCompileArg<freetype_font>(args);
        state = std::make_shared<RenderOCVState>();

        if (has_freetype_font)
        {
            state->ftpr = std::make_shared<FTTextRender>(has_freetype_font->path);
        }
    }
};


cv::GKernelPackage cv::gapi::render::ocv::kernels()
{
    const static auto pkg = cv::gapi::kernels<RenderBGROCVImpl, RenderNV12OCVImpl, RenderFrameOCVImpl>();
    return pkg;
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

### Classes and Structures

- **out**: A class/struct defined in this file
- **RenderOCVState**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/fluid/core.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `api/render_ocv.hpp`

**Python Imports:**
- `two`


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

