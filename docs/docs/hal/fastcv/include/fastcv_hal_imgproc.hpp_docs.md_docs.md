# Documentation for `docs/hal/fastcv/include/fastcv_hal_imgproc.hpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/fastcv/include/fastcv_hal_imgproc.hpp_docs.md`
- **File Name**: `fastcv_hal_imgproc.hpp_docs.md`
- **File Size**: 14,554 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/fastcv/include/fastcv_hal_imgproc.hpp_docs.md](../../../../docs/hal/fastcv/include/fastcv_hal_imgproc.hpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/fastcv/include` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/fastcv/include/fastcv_hal_imgproc.hpp`

## File Metadata

- **Full Path**: `hal/fastcv/include/fastcv_hal_imgproc.hpp`
- **File Name**: `fastcv_hal_imgproc.hpp`
- **File Size**: 10,803 bytes
- **File Type**: .hpp
- **Link to Source**: [hal/fastcv/include/fastcv_hal_imgproc.hpp](../../../hal/fastcv/include/fastcv_hal_imgproc.hpp)

## Purpose and Role

This file is located in the `hal/fastcv/include` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Copyright (c) 2024 Qualcomm Innovation Center, Inc. All rights reserved.
 * SPDX-License-Identifier: Apache-2.0
*/

#ifndef OPENCV_FASTCV_HAL_IMGPROC_HPP_INCLUDED
#define OPENCV_FASTCV_HAL_IMGPROC_HPP_INCLUDED

#include <opencv2/core/base.hpp>

#undef  cv_hal_medianBlur
#define cv_hal_medianBlur           fastcv_hal_medianBlur
#undef  cv_hal_sobel
#define cv_hal_sobel                fastcv_hal_sobel
#undef  cv_hal_boxFilter
#define cv_hal_boxFilter            fastcv_hal_boxFilter
#undef  cv_hal_adaptiveThreshold
#define cv_hal_adaptiveThreshold    fastcv_hal_adaptiveThreshold
#undef  cv_hal_gaussianBlurBinomial
#define cv_hal_gaussianBlurBinomial fastcv_hal_gaussianBlurBinomial
#undef  cv_hal_warpPerspective
#define cv_hal_warpPerspective      fastcv_hal_warpPerspective
#undef  cv_hal_pyrdown
#define cv_hal_pyrdown              fastcv_hal_pyrdown
#undef  cv_hal_cvtBGRtoHSV
#define cv_hal_cvtBGRtoHSV          fastcv_hal_cvtBGRtoHSV
#undef  cv_hal_cvtBGRtoYUVApprox
#define cv_hal_cvtBGRtoYUVApprox    fastcv_hal_cvtBGRtoYUVApprox
#undef  cv_hal_canny
#define cv_hal_canny                fastcv_hal_canny
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Calculate medianBlur filter
/// @param src_data Source image data
/// @param src_step Source image step
/// @param dst_data Destination image data
/// @param dst_step Destination image step
/// @param width    Source image width
/// @param height   Source image height
/// @param depth    Depths of source and destination image
/// @param cn       Number of channels
/// @param ksize    Size of kernel
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_medianBlur(
    const uchar*    src_data,
    size_t          src_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             depth,
    int             cn,
    int             ksize);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Computes Sobel derivatives
///
/// @param src_data         Source image data
/// @param src_step         Source image step
/// @param dst_data         Destination image data
/// @param dst_step         Destination image step
/// @param width            Source image width
/// @param height           Source image height
/// @param src_depth        Depth of source image
/// @param dst_depth        Depths of destination image
/// @param cn               Number of channels
/// @param margin_left      Left margins for source image
/// @param margin_top       Top margins for source image
/// @param margin_right     Right margins for source image
/// @param margin_bottom    Bottom margins for source image
/// @param dx               orders of the derivative x
/// @param dy               orders of the derivative y
/// @param ksize            Size of kernel
/// @param scale            Scale factor for the computed derivative values
/// @param delta            Delta value that is added to the results prior to storing them in dst
/// @param border_type      Border type
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_sobel(
    const uchar*    src_data,
    size_t          src_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             src_depth,
    int             dst_depth,
    int             cn,
    int             margin_left,
    int             margin_top,
    int             margin_right,
    int             margin_bottom,
    int             dx,
    int             dy,
    int             ksize,
    double          scale,
    double          delta,
    int             border_type);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

int fastcv_hal_boxFilter(
    const uchar*     src_data,
    size_t           src_step,
    uchar*           dst_data,
    size_t           dst_step,
    int              width,
    int              height,
    int              src_depth,
    int              dst_depth,
    int              cn,
    int              margin_left,
    int              margin_top,
    int              margin_right,
    int              margin_bottom,
    size_t           ksize_width,
    size_t           ksize_height,
    int              anchor_x,
    int              anchor_y,
    bool             normalize,
    int              border_type);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_adaptiveThreshold(
    const uchar*    src_data,
    size_t          src_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    double          maxValue,
    int             adaptiveMethod,
    int             thresholdType,
    int             blockSize,
    double          C);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Blurs an image using a Gaussian filter.
/// @param src_data         Source image data
/// @param src_step         Source image step
/// @param dst_data         Destination image data
/// @param dst_step         Destination image step
/// @param width            Source image width
/// @param height           Source image height
/// @param depth            Depth of source and destination image
/// @param cn               Number of channels
/// @param margin_left      Left margins for source image
/// @param margin_top       Top margins for source image
/// @param margin_right     Right margins for source image
/// @param margin_bottom    Bottom margins for source image
/// @param ksize            Kernel size
/// @param border_type      Border type
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_gaussianBlurBinomial(
    const uchar*    src_data,
    size_t          src_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             depth,
    int             cn,
    size_t          margin_left,
    size_t          margin_top,
    size_t          margin_right,
    size_t          margin_bottom,
    size_t          ksize,
    int             border_type);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Applies a perspective transformation to an image.
///
/// @param src_type         Source and destination image type
/// @param src_data         Source image data
/// @param src_step         Source image step
/// @param src_width        Source image width
/// @param src_height       Source image height
/// @param dst_data         Destination image data
/// @param dst_step         Destination image step
/// @param dst_width        Destination image width
/// @param dst_height       Destination image height
/// @param M                3x3 matrix with transform coefficients
/// @param interpolation    Interpolation mode (CV_HAL_INTER_NEAREST, ...)
/// @param border_type      Border processing mode (CV_HAL_BORDER_REFLECT, ...)
/// @param border_value     Values to use for CV_HAL_BORDER_CONSTANT mode
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_warpPerspective(
    int             src_type,
    const uchar*    src_data,
    size_t          src_step,
    int             src_width,
    int             src_height,
    uchar*          dst_data,
    size_t          dst_step,
    int             dst_width,
    int             dst_height,
    const double    M[9],
    int             interpolation,
    int             border_type,
    const double    border_value[4]);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_pyrdown(
    const uchar*     src_data,
    size_t           src_step,
    int              src_width,
    int              src_height,
    uchar*           dst_data,
    size_t           dst_step,
    int              dst_width,
    int              dst_height,
    int              depth,
    int              cn,
    int              border_type);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_cvtBGRtoHSV(
    const uchar    * src_data,
    size_t          src_step,
    uchar          * dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             depth,
    int             scn,
    bool            swapBlue,
    bool            isFullRange,
    bool            isHSV);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_cvtBGRtoYUVApprox(
    const uchar    * src_data,
    size_t          src_step,
    uchar          * dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             depth,
    int             scn,
    bool            swapBlue,
    bool            isCbCr);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Canny edge detector
/// @param src_data Source image data
/// @param src_step Source image step
/// @param dst_data Destination image data
/// @param dst_step Destination image step
/// @param width Source image width
/// @param height Source image height
/// @param cn Number of channels
/// @param lowThreshold low thresholds value
/// @param highThreshold high thresholds value
/// @param ksize Kernel size for Sobel operator.
/// @param L2gradient Flag, indicating use of L2 or L1 norma.
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_canny(
    const uchar*    src_data,
    size_t          src_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    int             cn,
    double          lowThreshold,
    double          highThreshold,
    int             ksize,
    bool            L2gradient);

#endif
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

- **cv_hal_canny()**: A function/method defined in this file
- **OPENCV_FASTCV_HAL_IMGPROC_HPP_INCLUDED()**: A function/method defined in this file
- **cv_hal_medianBlur()**: A function/method defined in this file
- **cv_hal_cvtBGRtoHSV()**: A function/method defined in this file
- **cv_hal_boxFilter()**: A function/method defined in this file
- **cv_hal_cvtBGRtoYUVApprox()**: A function/method defined in this file
- **cv_hal_gaussianBlurBinomial()**: A function/method defined in this file
- **cv_hal_warpPerspective()**: A function/method defined in this file
- **cv_hal_adaptiveThreshold()**: A function/method defined in this file
- **cv_hal_pyrdown()**: A function/method defined in this file
- **cv_hal_sobel()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/base.hpp`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

