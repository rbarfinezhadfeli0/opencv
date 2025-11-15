# Documentation for `modules/gapi/include/opencv2/gapi/gframe.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gframe.hpp`
- **File Name**: `gframe.hpp`
- **File Size**: 3,476 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gframe.hpp](../../../../../modules/gapi/include/opencv2/gapi/gframe.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation


#ifndef OPENCV_GAPI_GFRAME_HPP
#define OPENCV_GAPI_GFRAME_HPP

#include <ostream>
#include <memory>                 // std::shared_ptr

#include <opencv2/gapi/opencv_includes.hpp>
#include <opencv2/gapi/gcommon.hpp> // GShape

#include <opencv2/gapi/gmat.hpp>
#include <opencv2/gapi/own/assert.hpp>

// TODO GAPI_EXPORTS or so
namespace cv
{
// Forward declaration; GNode and GOrigin are an internal
// (user-inaccessible) classes.
class GNode;
struct GOrigin;

/** \addtogroup gapi_data_objects
 * @{
 */
/**
 * @brief GFrame class represents an image or media frame in the graph.
 *
 * GFrame doesn't store any data itself, instead it describes a
 * functional relationship between operations consuming and producing
 * GFrame objects.
 *
 * GFrame is introduced to handle various media formats (e.g., NV12 or
 * I420) under the same type. Various image formats may differ in the
 * number of planes (e.g. two for NV12, three for I420) and the pixel
 * layout inside. GFrame type allows to handle these media formats in
 * the graph uniformly -- the graph structure will not change if the
 * media format changes, e.g. a different camera or decoder is used
 * with the same graph. G-API provides a number of operations which
 * operate directly on GFrame, like `infer<>()` or
 * renderFrame(); these operations are expected to handle different
 * media formats inside. There is also a number of accessor
 * operations like BGR(), Y(), UV() -- these operations provide
 * access to frame's data in the familiar cv::GMat form, which can be
 * used with the majority of the existing G-API operations. These
 * accessor functions may perform color space conversion on the fly if
 * the image format of the GFrame they are applied to differs from the
 * operation's semantic (e.g. the BGR() accessor is called on an NV12
 * image frame).
 *
 * GFrame is a virtual counterpart of cv::MediaFrame.
 *
 * @sa cv::MediaFrame, cv::GFrameDesc, BGR(), Y(), UV(), infer<>().
 */
class GAPI_EXPORTS_W_SIMPLE GFrame
{
public:
    /**
     * @brief Constructs an empty GFrame
     *
     * Normally, empty G-API data objects denote a starting point of
     * the graph. When an empty GFrame is assigned to a result of some
     * operation, it obtains a functional link to this operation (and
     * is not empty anymore).
     */
    GAPI_WRAP GFrame();                      // Empty constructor

    /// @private
    GFrame(const GNode &n, std::size_t out); // Operation result constructor
    /// @private
    GOrigin& priv();                         // Internal use only
    /// @private
    const GOrigin& priv()  const;            // Internal use only

private:
    std::shared_ptr<GOrigin> m_priv;
};
/** @} */

enum class MediaFormat: int
{
    BGR = 0,
    NV12,
    GRAY,
};

/**
 * \addtogroup gapi_meta_args
 * @{
 */
struct GAPI_EXPORTS GFrameDesc
{
    MediaFormat fmt;
    cv::Size size;

    bool operator== (const GFrameDesc &) const;
};
static inline GFrameDesc empty_gframe_desc() { return GFrameDesc{}; }
/** @} */

class MediaFrame;
GAPI_EXPORTS GFrameDesc descr_of(const MediaFrame &frame);

GAPI_EXPORTS std::ostream& operator<<(std::ostream& os, const cv::GFrameDesc &desc);

} // namespace cv

#endif // OPENCV_GAPI_GFRAME_HPP
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

- **represents**: A class/struct defined in this file
- **GNode**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **GOrigin**: A class/struct defined in this file
- **MediaFormat**: A class/struct defined in this file
- **MediaFrame**: A class/struct defined in this file
- **GAPI_EXPORTS_W_SIMPLE**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GFRAME_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/gmat.hpp`
- `memory`
- `opencv2/gapi/gcommon.hpp`
- `opencv2/gapi/own/assert.hpp`
- `ostream`

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

