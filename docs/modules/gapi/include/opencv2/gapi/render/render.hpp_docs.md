# Documentation for `modules/gapi/include/opencv2/gapi/render/render.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/render/render.hpp`
- **File Name**: `render.hpp`
- **File Size**: 6,232 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/render/render.hpp](../../../../../../modules/gapi/include/opencv2/gapi/render/render.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/render` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_RENDER_HPP
#define OPENCV_GAPI_RENDER_HPP

#include <opencv2/gapi/render/render_types.hpp>

#include <opencv2/gapi.hpp>

/** \defgroup gapi_draw G-API Drawing and composition functionality
 *  @{
 *
 *  @brief Functions for in-graph drawing.
 *
 *  @note This is a Work in Progress functionality and APIs may
 *  change in the future releases.
 *
 *  G-API can do some in-graph drawing with a generic operations and a
 *  set of [rendering primitives](@ref gapi_draw_prims).
 *  In contrast with traditional OpenCV, in G-API user need to form a
 *  *rendering list* of primitives to draw. This list can be built
 *  manually or generated within a graph. This list is passed to
 *  [special operations or functions](@ref gapi_draw_api) where all
 *  primitives are interpreted and applied to the image.
 *
 *  For example, in a complex pipeline a list of detected objects
 *  can be translated in-graph to a list of cv::gapi::wip::draw::Rect
 *  primitives to highlight those with bounding boxes, or a list of
 *  detected faces can be translated in-graph to a list of
 *  cv::gapi::wip::draw::Mosaic primitives to hide sensitive content
 *  or protect privacy.
 *
 *  Like any other operations, rendering in G-API can be reimplemented
 *  by different backends. Currently only an OpenCV-based backend is
 *  available.
 *
 *  In addition to the graph-level operations, there are also regular
 *  (immediate) OpenCV-like functions are available -- see
 *  cv::gapi::wip::draw::render(). These functions are just wrappers
 *  over regular G-API and build the rendering graphs on the fly, so
 *  take compilation arguments as parameters.
 *
 *  Currently this API is more machine-oriented than human-oriented.
 *  The main purpose is to translate a set of domain-specific objects
 *  to a list of primitives to draw. For example, in order to generate
 *  a picture like this:
 *
 *  ![](modules/gapi/doc/pics/render_example.png)
 *
 *  Rendering list needs to be generated as follows:
 *
 *  @include modules/gapi/samples/draw_example.cpp
 *
 *  @defgroup gapi_draw_prims Drawing primitives
 *  @defgroup gapi_draw_api Drawing operations and functions
 *  @}
 */

namespace cv
{
namespace gapi
{
namespace wip
{
namespace draw
{

using GMat2     = std::tuple<cv::GMat,cv::GMat>;
using GMatDesc2 = std::tuple<cv::GMatDesc,cv::GMatDesc>;

//! @addtogroup gapi_draw_api
//! @{
/** @brief The function renders on the input image passed drawing primitivies

@param bgr input image: 8-bit unsigned 3-channel image @ref CV_8UC3.
@param prims vector of drawing primitivies
@param args graph compile time parameters
*/
void GAPI_EXPORTS_W render(cv::Mat& bgr,
                           const Prims& prims,
                           cv::GCompileArgs&& args = {});

/** @brief The function renders on two NV12 planes passed drawing primitivies

@param y_plane input image: 8-bit unsigned 1-channel image @ref CV_8UC1.
@param uv_plane input image: 8-bit unsigned 2-channel image @ref CV_8UC2.
@param prims vector of drawing primitivies
@param args graph compile time parameters
*/
void GAPI_EXPORTS_W render(cv::Mat& y_plane,
                           cv::Mat& uv_plane,
                           const Prims& prims,
                           cv::GCompileArgs&& args = {});

/** @brief The function renders on the input media frame passed drawing primitivies

@param frame input Media Frame :  @ref cv::MediaFrame.
@param prims vector of drawing primitivies
@param args graph compile time parameters
*/
void GAPI_EXPORTS render(cv::MediaFrame& frame,
                         const Prims& prims,
                         cv::GCompileArgs&& args = {});


G_TYPED_KERNEL_M(GRenderNV12, <GMat2(cv::GMat,cv::GMat,cv::GArray<wip::draw::Prim>)>, "org.opencv.render.nv12")
{
     static GMatDesc2 outMeta(GMatDesc y_plane, GMatDesc uv_plane, GArrayDesc)
     {
         return std::make_tuple(y_plane, uv_plane);
     }
};

G_TYPED_KERNEL(GRenderBGR, <cv::GMat(cv::GMat,cv::GArray<wip::draw::Prim>)>, "org.opencv.render.bgr")
{
     static GMatDesc outMeta(GMatDesc bgr, GArrayDesc)
     {
         return bgr;
     }
};

G_TYPED_KERNEL(GRenderFrame, <cv::GFrame(cv::GFrame, cv::GArray<wip::draw::Prim>)>, "org.opencv.render.frame")
{
    static GFrameDesc outMeta(GFrameDesc desc, GArrayDesc)
    {
        return desc;
    }
};

/** @brief Renders on 3 channels input

Output image must be 8-bit unsigned planar 3-channel image

@param src input image: 8-bit unsigned 3-channel image @ref CV_8UC3
@param prims draw primitives
*/
GAPI_EXPORTS_W GMat render3ch(const GMat& src, const GArray<Prim>& prims);

/** @brief Renders on two planes

Output y image must be 8-bit unsigned planar 1-channel image @ref CV_8UC1
uv image must be 8-bit unsigned planar 2-channel image @ref CV_8UC2

@param y  input image: 8-bit unsigned 1-channel image @ref CV_8UC1
@param uv input image: 8-bit unsigned 2-channel image @ref CV_8UC2
@param prims draw primitives
*/
GAPI_EXPORTS_W GMat2 renderNV12(const GMat& y,
                                const GMat& uv,
                                const GArray<Prim>& prims);

/** @brief Renders Media Frame

Output media frame frame cv::MediaFrame

@param m_frame input image: cv::MediaFrame @ref cv::MediaFrame
@param prims draw primitives
*/
GAPI_EXPORTS GFrame renderFrame(const GFrame& m_frame,
                                const GArray<Prim>& prims);

//! @} gapi_draw_api

} // namespace draw
} // namespace wip

/**
 * @brief This namespace contains G-API CPU rendering backend functions,
 * structures, and symbols. See @ref gapi_draw for details.
 */
namespace render
{
namespace ocv
{
    GAPI_EXPORTS_W cv::GKernelPackage kernels();

} // namespace ocv
} // namespace render
} // namespace gapi

namespace detail
{
    template<> struct CompileArgTag<cv::gapi::wip::draw::freetype_font>
    {
        static const char* tag() { return "gapi.freetype_font"; }
    };
} // namespace detail

} // namespace cv

#endif // OPENCV_GAPI_RENDER_HPP
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

- **CompileArgTag**: A class/struct defined in this file

### Functions and Methods

- **renders()**: A function/method defined in this file
- **OPENCV_GAPI_RENDER_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/render/render_types.hpp`
- `opencv2/gapi.hpp`


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

