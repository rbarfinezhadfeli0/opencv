# Documentation for `modules/gapi/src/backends/fluid/gfluidimgproc_func.hpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/fluid/gfluidimgproc_func.hpp`
- **File Name**: `gfluidimgproc_func.hpp`
- **File Size**: 4,759 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/src/backends/fluid/gfluidimgproc_func.hpp](../../../../../modules/gapi/src/backends/fluid/gfluidimgproc_func.hpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/fluid` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation

#pragma once

#if !defined(GAPI_STANDALONE)

#include <opencv2/core.hpp>

namespace cv {
namespace gapi {
namespace fluid {

//----------------------------------
//
// Fluid kernels: RGB2Gray, BGR2Gray
//
//----------------------------------

void run_rgb2gray_impl(uchar out[], const uchar in[], int width,
                       float coef_r, float coef_g, float coef_b);

//--------------------------------------
//
// Fluid kernels: RGB-to-HSV
//
//--------------------------------------

void run_rgb2hsv_impl(uchar out[], const uchar in[], const int sdiv_table[],
        const int hdiv_table[], int width);

//--------------------------------------
//
// Fluid kernels: RGB-to-BayerGR
//
//--------------------------------------

void run_bayergr2rgb_bg_impl(uchar out[], const uchar **in, int width);

void run_bayergr2rgb_gr_impl(uchar out[], const uchar **in, int width);

//--------------------------------------
//
// Fluid kernels: RGB-to-YUV,RGB-to-YUV422, YUV-to-RGB
//
//--------------------------------------

void run_rgb2yuv_impl(uchar out[], const uchar in[], int width, const float coef[5]);

void run_yuv2rgb_impl(uchar out[], const uchar in[], int width, const float coef[4]);

void run_rgb2yuv422_impl(uchar out[], const uchar in[], int width);

//-------------------------
//
// Fluid kernels: sepFilter
//
//-------------------------

#define RUN_SEPFILTER3X3_IMPL(DST, SRC)                                     \
void run_sepfilter3x3_impl(DST out[], const SRC *in[], int width, int chan, \
                           const float kx[], const float ky[], int border,  \
                           float scale, float delta,                        \
                           float *buf[], int y, int y0);

RUN_SEPFILTER3X3_IMPL(uchar , uchar )
RUN_SEPFILTER3X3_IMPL( short, uchar )
RUN_SEPFILTER3X3_IMPL( float, uchar )
RUN_SEPFILTER3X3_IMPL(ushort, ushort)
RUN_SEPFILTER3X3_IMPL( short, ushort)
RUN_SEPFILTER3X3_IMPL( float, ushort)
RUN_SEPFILTER3X3_IMPL( short,  short)
RUN_SEPFILTER3X3_IMPL( float,  short)
RUN_SEPFILTER3X3_IMPL( float,  float)

#undef RUN_SEPFILTER3X3_IMPL

#define RUN_SEPFILTER5x5_IMPL(DST, SRC)                                     \
void run_sepfilter5x5_impl(DST out[], const SRC *in[], int width, int chan, \
                           const float kx[], const float ky[], int border,  \
                           float scale, float delta,                        \
                           float *buf[], int y, int y0);


RUN_SEPFILTER5x5_IMPL(uchar, uchar)
RUN_SEPFILTER5x5_IMPL(short, uchar)
RUN_SEPFILTER5x5_IMPL(float, uchar)
RUN_SEPFILTER5x5_IMPL(ushort, ushort)
RUN_SEPFILTER5x5_IMPL(short, ushort)
RUN_SEPFILTER5x5_IMPL(float, ushort)
RUN_SEPFILTER5x5_IMPL(short, short)
RUN_SEPFILTER5x5_IMPL(float, short)
RUN_SEPFILTER5x5_IMPL(float, float)

#undef RUN_SEPFILTER5x5_IMPL

//-------------------------
//
// Fluid kernels: Filter 2D
//
//-------------------------

#define RUN_FILTER2D_3X3_IMPL(DST, SRC)                                     \
void run_filter2d_3x3_impl(DST out[], const SRC *in[], int width, int chan, \
                           const float kernel[], float scale, float delta);

RUN_FILTER2D_3X3_IMPL(uchar , uchar )
RUN_FILTER2D_3X3_IMPL(ushort, ushort)
RUN_FILTER2D_3X3_IMPL( short,  short)
RUN_FILTER2D_3X3_IMPL( float, uchar )
RUN_FILTER2D_3X3_IMPL( float, ushort)
RUN_FILTER2D_3X3_IMPL( float,  short)
RUN_FILTER2D_3X3_IMPL( float,  float)

#undef RUN_FILTER2D_3X3_IMPL

//-----------------------------
//
// Fluid kernels: Erode, Dilate
//
//-----------------------------

enum Morphology { M_ERODE, M_DILATE };

enum MorphShape { M_FULL, M_CROSS, M_UNDEF };

#define RUN_MORPHOLOGY3X3_IMPL(T)                                        \
void run_morphology3x3_impl(T out[], const T *in[], int width, int chan, \
                            const uchar k[], MorphShape k_type,          \
                            Morphology morphology);

RUN_MORPHOLOGY3X3_IMPL(uchar )
RUN_MORPHOLOGY3X3_IMPL(ushort)
RUN_MORPHOLOGY3X3_IMPL( short)
RUN_MORPHOLOGY3X3_IMPL( float)

#undef RUN_MORPHOLOGY3X3_IMPL

//---------------------------
//
// Fluid kernels: Median blur
//
//---------------------------

#define RUN_MEDBLUR3X3_IMPL(T) \
void run_medblur3x3_impl(T out[], const T *in[], int width, int chan);

RUN_MEDBLUR3X3_IMPL(uchar )
RUN_MEDBLUR3X3_IMPL(ushort)
RUN_MEDBLUR3X3_IMPL( short)
RUN_MEDBLUR3X3_IMPL( float)

#undef RUN_MEDBLUR3X3_IMPL

}  // namespace fluid
}  // namespace gapi
}  // namespace cv

#endif // !defined(GAPI_STANDALONE)
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

- **RUN_MORPHOLOGY3X3_IMPL()**: A function/method defined in this file
- **RUN_SEPFILTER5x5_IMPL()**: A function/method defined in this file
- **RUN_SEPFILTER3X3_IMPL()**: A function/method defined in this file
- **RUN_FILTER2D_3X3_IMPL()**: A function/method defined in this file
- **RUN_MEDBLUR3X3_IMPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`


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

