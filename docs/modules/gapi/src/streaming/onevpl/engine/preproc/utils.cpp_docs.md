# Documentation for `modules/gapi/src/streaming/onevpl/engine/preproc/utils.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/engine/preproc/utils.cpp`
- **File Name**: `utils.cpp`
- **File Size**: 2,774 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/engine/preproc/utils.cpp](../../../../../../../modules/gapi/src/streaming/onevpl/engine/preproc/utils.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl/engine/preproc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2022 Intel Corporation

#include <type_traits>

#include "streaming/onevpl/engine/preproc/utils.hpp"

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "logger.hpp"

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {
namespace utils {

cv::MediaFormat fourcc_to_MediaFormat(int value) {
    switch (value)
    {
        case MFX_FOURCC_BGRP:
            return cv::MediaFormat::BGR;
        case MFX_FOURCC_NV12:
            return cv::MediaFormat::NV12;
        default:
            GAPI_LOG_WARNING(nullptr, "Unsupported FourCC format requested: " << value <<
                                     ". Cannot cast to cv::MediaFrame");
            GAPI_Error("Unsupported FOURCC");

    }
}

int MediaFormat_to_fourcc(cv::MediaFormat value) {
    switch (value)
    {
        case cv::MediaFormat::BGR:
            return MFX_FOURCC_BGRP;
        case cv::MediaFormat::NV12:
            return MFX_FOURCC_NV12;
        default:
            GAPI_LOG_WARNING(nullptr, "Unsupported cv::MediaFormat format requested: " <<
                                      static_cast<typename std::underlying_type<cv::MediaFormat>::type>(value) <<
                                     ". Cannot cast to FourCC");
            GAPI_Error("Unsupported cv::MediaFormat");
    }
}
int MediaFormat_to_chroma(cv::MediaFormat value) {
    switch (value)
    {
        case cv::MediaFormat::BGR:
            return MFX_CHROMAFORMAT_MONOCHROME;
        case cv::MediaFormat::NV12:
            return MFX_CHROMAFORMAT_YUV420;
        default:
            GAPI_LOG_WARNING(nullptr, "Unsupported cv::MediaFormat format requested: " <<
                                      static_cast<typename std::underlying_type<cv::MediaFormat>::type>(value) <<
                                     ". Cannot cast to ChromaFormateIdc");
            GAPI_Error("Unsupported cv::MediaFormat");
    }
}

mfxFrameInfo to_mfxFrameInfo(const cv::GFrameDesc& frame_info) {
    mfxFrameInfo ret {0};
    ret.FourCC        = MediaFormat_to_fourcc(frame_info.fmt);
    ret.ChromaFormat  = MediaFormat_to_chroma(frame_info.fmt);
    ret.Width         = frame_info.size.width;
    ret.Height        = frame_info.size.height;
    ret.CropX         = 0;
    ret.CropY         = 0;
    ret.CropW         = 0;
    ret.CropH         = 0;
    ret.PicStruct     = MFX_PICSTRUCT_UNKNOWN;
    ret.FrameRateExtN = 0;
    ret.FrameRateExtD = 0;
    return ret;
}
} // namespace utils
} // namespace cv
} // namespace gapi
} // namespace wip
} // namespace onevpl

#endif // HAVE_ONEVPL
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

- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `type_traits`
- `streaming/onevpl/engine/preproc/utils.hpp`
- `streaming/onevpl/onevpl_export.hpp`
- `logger.hpp`


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

