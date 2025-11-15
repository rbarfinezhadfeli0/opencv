# Documentation for `docs/hal/fastcv/include/fastcv_hal_core.hpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/fastcv/include/fastcv_hal_core.hpp_docs.md`
- **File Name**: `fastcv_hal_core.hpp_docs.md`
- **File Size**: 15,301 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/fastcv/include/fastcv_hal_core.hpp_docs.md](../../../../docs/hal/fastcv/include/fastcv_hal_core.hpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/fastcv/include` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/fastcv/include/fastcv_hal_core.hpp`

## File Metadata

- **Full Path**: `hal/fastcv/include/fastcv_hal_core.hpp`
- **File Name**: `fastcv_hal_core.hpp`
- **File Size**: 11,319 bytes
- **File Type**: .hpp
- **Link to Source**: [hal/fastcv/include/fastcv_hal_core.hpp](../../../hal/fastcv/include/fastcv_hal_core.hpp)

## Purpose and Role

This file is located in the `hal/fastcv/include` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Copyright (c) 2024-2025 Qualcomm Innovation Center, Inc. All rights reserved.
 * SPDX-License-Identifier: Apache-2.0
*/

#ifndef OPENCV_FASTCV_HAL_CORE_HPP_INCLUDED
#define OPENCV_FASTCV_HAL_CORE_HPP_INCLUDED

#include <opencv2/core/base.hpp>

#undef  cv_hal_lut
#define cv_hal_lut                  fastcv_hal_lut
#undef  cv_hal_normHammingDiff8u
#define cv_hal_normHammingDiff8u    fastcv_hal_normHammingDiff8u
#undef  cv_hal_mul8u16u
#define cv_hal_mul8u16u             fastcv_hal_mul8u16u
#undef  cv_hal_sub8u32f
#define cv_hal_sub8u32f             fastcv_hal_sub8u32f
#undef  cv_hal_transpose2d
#define cv_hal_transpose2d          fastcv_hal_transpose2d
#undef  cv_hal_meanStdDev
#define cv_hal_meanStdDev           fastcv_hal_meanStdDev
#undef  cv_hal_flip
#define cv_hal_flip                 fastcv_hal_flip
#undef  cv_hal_rotate90
#define cv_hal_rotate90             fastcv_hal_rotate
#undef  cv_hal_addWeighted8u
#define cv_hal_addWeighted8u        fastcv_hal_addWeighted8u
#undef  cv_hal_mul8u
#define cv_hal_mul8u                fastcv_hal_mul8u
#undef  cv_hal_mul16s
#define cv_hal_mul16s               fastcv_hal_mul16s
#undef  cv_hal_mul32f
#define cv_hal_mul32f               fastcv_hal_mul32f
#undef  cv_hal_SVD32f
#define cv_hal_SVD32f               fastcv_hal_SVD32f
#undef  cv_hal_gemm32f
#define cv_hal_gemm32f              fastcv_hal_gemm32f

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief look-up table transform of an array.
/// @param src_data Source image data
/// @param src_step Source image step
/// @param src_type Source image type
/// @param lut_data Pointer to lookup table
/// @param lut_channel_size Size of each channel in bytes
/// @param lut_channels Number of channels in lookup table
/// @param dst_data Destination data
/// @param dst_step Destination step
/// @param width Width of images
/// @param height Height of images
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_lut(
    const uchar*    src_data,
    size_t          src_step,
    size_t          src_type,
    const uchar*    lut_data,
    size_t          lut_channel_size,
    size_t          lut_channels,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Hamming distance between two vectors
/// @param a pointer to first vector data
/// @param b pointer to second vector data
/// @param n length of vectors
/// @param cellSize how many bits of the vectors will be added and treated as a single bit, can be 1 (standard Hamming distance), 2 or 4
/// @param result pointer to result output
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_normHammingDiff8u(const uchar* a, const uchar* b, int n, int cellSize, int* result);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_mul8u16u(
    const uchar    * src1_data,
    size_t           src1_step,
    const uchar    * src2_data,
    size_t           src2_step,
    ushort         * dst_data,
    size_t           dst_step,
    int              width,
    int              height,
    double           scale);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_sub8u32f(
    const uchar     *src1_data,
    size_t           src1_step,
    const uchar     *src2_data,
    size_t           src2_step,
    float           *dst_data,
    size_t           dst_step,
    int              width,
    int              height);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_transpose2d(
    const uchar*     src_data,
    size_t           src_step,
    uchar*           dst_data,
    size_t           dst_step,
    int              src_width,
    int              src_height,
    int              element_size);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_meanStdDev(
    const uchar     * src_data,
    size_t            src_step,
    int               width,
    int               height,
    int               src_type,
    double          * mean_val,
    double          * stddev_val,
    uchar           * mask,
    size_t            mask_step);


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Flips a 2D array around vertical, horizontal, or both axes
/// @param src_type source and destination image type
/// @param src_data source image data
/// @param src_step source image step
/// @param src_width source and destination image width
/// @param src_height source and destination image height
/// @param dst_data destination image data
/// @param dst_step destination image step
/// @param flip_mode 0 flips around x-axis, 1 around y-axis, -1 both
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_flip(
    int             src_type,
    const uchar*    src_data,
    size_t          src_step,
    int             src_width,
    int             src_height,
    uchar*          dst_data,
    size_t          dst_step,
    int             flip_mode);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief Rotates a 2D array in multiples of 90 degrees.
/// @param src_type source and destination image type
/// @param src_data source image data
/// @param src_step source image step
/// @param src_width source image width
///   @If angle has value [180] it is also destination image width
///   If angle has values [90, 270] it is also destination image height
/// @param src_height source and destination image height (destination image width for angles [90, 270])
///   If angle has value [180] it is also destination image height
///   If angle has values [90, 270] it is also destination image width
/// @param dst_data destination image data
/// @param dst_step destination image step
/// @param angle clockwise angle for rotation in degrees from set [90, 180, 270]
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_rotate(
    int             src_type,
    const uchar*    src_data,
    size_t          src_step,
    int             src_width,
    int             src_height,
    uchar*          dst_data,
    size_t          dst_step,
    int             angle);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// @brief weighted sum of two arrays using formula: dst[i] = a * src1[i] + b * src2[i]
/// @param src1_data first source image data
/// @param src1_step first source image step
/// @param src2_data second source image data
/// @param src2_step second source image step
/// @param dst_data  destination image data
/// @param dst_step  destination image step
/// @param width     width of the images
/// @param height    height of the images
/// @param scalars   numbers a, b, and c
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_addWeighted8u(
    const uchar*    src1_data,
    size_t          src1_step,
    const uchar*    src2_data,
    size_t          src2_step,
    uchar*          dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    const double    scalars[3]);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_mul8u(
    const uchar     *src1_data,
    size_t          src1_step,
    const uchar     *src2_data,
    size_t          src2_step,
    uchar           *dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    double          scale);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_mul16s(
    const short     *src1_data,
    size_t          src1_step,
    const short     *src2_data,
    size_t          src2_step,
    short           *dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    double          scale);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_mul32f(
    const float    *src1_data,
    size_t          src1_step,
    const float    *src2_data,
    size_t          src2_step,
    float          *dst_data,
    size_t          dst_step,
    int             width,
    int             height,
    double          scale);

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// Performs singular value decomposition of \f$M\times N\f$(\f$M>N\f$) matrix \f$A = U*\Sigma*V^T\f$.
///
/// @param src      Pointer to input MxN matrix A stored in column major order.
///                 After finish of work src will be filled with rows of U or not modified (depends of flag CV_HAL_SVD_MODIFY_A).
/// @param src_step Number of bytes between two consequent columns of matrix A.
/// @param w        Pointer to array for singular values of matrix A (i. e. first N diagonal elements of matrix \f$\Sigma\f$).
/// @param u        Pointer to output MxN or MxM matrix U (size depends of flags).
///                 Pointer must be valid if flag CV_HAL_SVD_MODIFY_A not used.
/// @param u_step   Number of bytes between two consequent rows of matrix U.
/// @param vt       Pointer to array for NxN matrix V^T.
/// @param vt_step  Number of bytes between two consequent rows of matrix V^T.
/// @param m        Number fo rows in matrix A.
/// @param n        Number of columns in matrix A.
/// @param flags    Algorithm options (combination of CV_HAL_SVD_FULL_UV, ...).
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
int fastcv_hal_SVD32f(
    float* src,
    size_t src_step,
    float* w,
    float* u,
    size_t u_step,
    float* vt,
    size_t vt_step,
    int    m,
    int    n,
    int    flags);

int fastcv_hal_gemm32f(
    const float*    src1,
    size_t          src1_step,
    const float*    src2,
    size_t          src2_step,
    float           alpha,
    const float*    src3,
    size_t          src3_step,
    float           beta,
    float*          dst,
    size_t          dst_step,
    int             m,
    int             n,
    int             k,
    int             flags);

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

- **cv_hal_gemm32f()**: A function/method defined in this file
- **cv_hal_sub8u32f()**: A function/method defined in this file
- **cv_hal_mul8u()**: A function/method defined in this file
- **cv_hal_mul16s()**: A function/method defined in this file
- **cv_hal_lut()**: A function/method defined in this file
- **cv_hal_transpose2d()**: A function/method defined in this file
- **cv_hal_SVD32f()**: A function/method defined in this file
- **cv_hal_addWeighted8u()**: A function/method defined in this file
- **cv_hal_mul8u16u()**: A function/method defined in this file
- **cv_hal_mul32f()**: A function/method defined in this file
- **cv_hal_meanStdDev()**: A function/method defined in this file
- **cv_hal_normHammingDiff8u()**: A function/method defined in this file
- **cv_hal_flip()**: A function/method defined in this file
- **OPENCV_FASTCV_HAL_CORE_HPP_INCLUDED()**: A function/method defined in this file
- **cv_hal_rotate90()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/base.hpp`

**Python Imports:**
- `set`


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

