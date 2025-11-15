# Documentation for `modules/gapi/include/opencv2/gapi/infer/parsers.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/infer/parsers.hpp`
- **File Name**: `parsers.hpp`
- **File Size**: 6,282 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/infer/parsers.hpp](../../../../../../modules/gapi/include/opencv2/gapi/infer/parsers.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/infer` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation


#ifndef OPENCV_GAPI_PARSERS_HPP
#define OPENCV_GAPI_PARSERS_HPP

#include <utility> // std::tuple

#include <opencv2/gapi/gmat.hpp>
#include <opencv2/gapi/gkernel.hpp>

namespace cv { namespace gapi {
namespace nn {
namespace parsers {
    using GRects      = GArray<Rect>;
    using GDetections = std::tuple<GArray<Rect>, GArray<int>>;

    G_TYPED_KERNEL(GParseSSDBL, <GDetections(GMat, GOpaque<Size>, float, int)>,
                   "org.opencv.nn.parsers.parseSSD_BL") {
        static std::tuple<GArrayDesc,GArrayDesc> outMeta(const GMatDesc&, const GOpaqueDesc&, float, int) {
            return std::make_tuple(empty_array_desc(), empty_array_desc());
        }
    };

    G_TYPED_KERNEL(GParseSSD, <GRects(GMat, GOpaque<Size>, float, bool, bool)>,
                   "org.opencv.nn.parsers.parseSSD") {
        static GArrayDesc outMeta(const GMatDesc&, const GOpaqueDesc&, float, bool, bool) {
            return empty_array_desc();
        }
    };

    G_TYPED_KERNEL(GParseYolo, <GDetections(GMat, GOpaque<Size>, float, float, std::vector<float>)>,
                   "org.opencv.nn.parsers.parseYolo") {
        static std::tuple<GArrayDesc, GArrayDesc> outMeta(const GMatDesc&, const GOpaqueDesc&,
                                                          float, float, const std::vector<float>&) {
            return std::make_tuple(empty_array_desc(), empty_array_desc());
        }
        static const std::vector<float>& defaultAnchors() {
            static std::vector<float> anchors {
                0.57273f, 0.677385f, 1.87446f, 2.06253f, 3.33843f, 5.47434f, 7.88282f, 3.52778f, 9.77052f, 9.16828f
            };
            return anchors;
        }
    };
} // namespace parsers
} // namespace nn

/** @brief Parses output of SSD network.

Extracts detection information (box, confidence, label) from SSD output and
filters it by given confidence and label.

@note Function textual ID is "org.opencv.nn.parsers.parseSSD_BL"

@param in Input CV_32F tensor with {1,1,N,7} dimensions.
@param inSz Size to project detected boxes to (size of the input image).
@param confidenceThreshold If confidence of the
detection is smaller than confidence threshold, detection is rejected.
@param filterLabel If provided (!= -1), only detections with
given label will get to the output.
@return a tuple with a vector of detected boxes and a vector of appropriate labels.
*/
GAPI_EXPORTS_W std::tuple<GArray<Rect>, GArray<int>> parseSSD(const GMat& in,
                                                              const GOpaque<Size>& inSz,
                                                              const float confidenceThreshold = 0.5f,
                                                              const int   filterLabel = -1);

/** @brief Parses output of SSD network.

Extracts detection information (box, confidence) from SSD output and
filters it by given confidence and by going out of bounds.

@note Function textual ID is "org.opencv.nn.parsers.parseSSD"

@param in Input CV_32F tensor with {1,1,N,7} dimensions.
@param inSz Size to project detected boxes to (size of the input image).
@param confidenceThreshold If confidence of the
detection is smaller than confidence threshold, detection is rejected.
@param alignmentToSquare If provided true, bounding boxes are extended to squares.
The center of the rectangle remains unchanged, the side of the square is
the larger side of the rectangle.
@param filterOutOfBounds If provided true, out-of-frame boxes are filtered.
@return a vector of detected bounding boxes.
*/
GAPI_EXPORTS_W GArray<Rect> parseSSD(const GMat& in,
                                     const GOpaque<Size>& inSz,
                                     const float confidenceThreshold,
                                     const bool alignmentToSquare,
                                     const bool filterOutOfBounds);

/** @brief Parses output of Yolo network.

Extracts detection information (box, confidence, label) from Yolo output,
filters it by given confidence and performs non-maximum suppression for overlapping boxes.

@note Function textual ID is "org.opencv.nn.parsers.parseYolo"

@param in Input CV_32F tensor with {1,13,13,N} dimensions, N should satisfy:
\f[\texttt{N} = (\texttt{num_classes} + \texttt{5}) * \texttt{5},\f]
where num_classes - a number of classes Yolo network was trained with.
@param inSz Size to project detected boxes to (size of the input image).
@param confidenceThreshold If confidence of the
detection is smaller than confidence threshold, detection is rejected.
@param nmsThreshold Non-maximum suppression threshold which controls minimum
relative box intersection area required for rejecting the box with a smaller confidence.
If 1.f, nms is not performed and no boxes are rejected.
@param anchors Anchors Yolo network was trained with.
@note The default anchor values are specified for YOLO v2 Tiny as described in Intel Open Model Zoo
<a href="https://github.com/openvinotoolkit/open_model_zoo/blob/master/models/public/yolo-v2-tiny-tf/yolo-v2-tiny-tf.md">documentation</a>.
@return a tuple with a vector of detected boxes and a vector of appropriate labels.
*/
GAPI_EXPORTS_W std::tuple<GArray<Rect>, GArray<int>> parseYolo(const GMat& in,
                                                               const GOpaque<Size>& inSz,
                                                               const float confidenceThreshold = 0.5f,
                                                               const float nmsThreshold = 0.5f,
                                                               const std::vector<float>& anchors
                                                                   = nn::parsers::GParseYolo::defaultAnchors());

} // namespace gapi
} // namespace cv

// Reimport parseSSD & parseYolo under their initial namespace
namespace cv {
namespace gapi {
namespace streaming {

using cv::gapi::parseSSD;
using cv::gapi::parseYolo;

} // namespace streaming
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_PARSERS_HPP
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

### Functions and Methods

- **OPENCV_GAPI_PARSERS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/gmat.hpp`

**Python Imports:**
- `SSD`
- `parseSSD`
- `Yolo`


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

