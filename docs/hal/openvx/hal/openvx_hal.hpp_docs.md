# Documentation for `hal/openvx/hal/openvx_hal.hpp`

## File Metadata

- **Full Path**: `hal/openvx/hal/openvx_hal.hpp`
- **File Name**: `openvx_hal.hpp`
- **File Size**: 10,481 bytes
- **File Type**: .hpp
- **Link to Source**: [hal/openvx/hal/openvx_hal.hpp](../../../hal/openvx/hal/openvx_hal.hpp)

## Purpose and Role

This file is located in the `hal/openvx/hal` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef OPENCV_OPENVX_HAL_HPP_INCLUDED
#define OPENCV_OPENVX_HAL_HPP_INCLUDED

#include "opencv2/core/hal/interface.h"

#include "VX/vx.h"

template <typename T>
int ovx_hal_add(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);
template <typename T>
int ovx_hal_sub(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);

template <typename T>
int ovx_hal_absdiff(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);

template <typename T>
int ovx_hal_and(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);
template <typename T>
int ovx_hal_or(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);
template <typename T>
int ovx_hal_xor(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h);
int ovx_hal_not(const uchar *a, size_t astep, uchar *c, size_t cstep, int w, int h);

template <typename T>
int ovx_hal_mul(const T *a, size_t astep, const T *b, size_t bstep, T *c, size_t cstep, int w, int h, double scale);

int ovx_hal_merge8u(const uchar **src_data, uchar *dst_data, int len, int cn);
int ovx_hal_resize(int atype, const uchar *a, size_t astep, int aw, int ah, uchar *b, size_t bstep, int bw, int bh, double inv_scale_x, double inv_scale_y, int interpolation);
int ovx_hal_warpAffine(int atype, const uchar *a, size_t astep, int aw, int ah, uchar *b, size_t bstep, int bw, int bh, const double M[6], int interpolation, int borderType, const double borderValue[4]);
int ovx_hal_warpPerspective(int atype, const uchar *a, size_t astep, int aw, int ah, uchar *b, size_t bstep, int bw, int bh, const double M[9], int interpolation, int borderType, const double borderValue[4]);

struct cvhalFilter2D;
int ovx_hal_filterInit(cvhalFilter2D **filter_context, uchar *kernel_data, size_t kernel_step, int kernel_type, int kernel_width, int kernel_height,
                       int, int, int src_type, int dst_type, int borderType, double delta, int anchor_x, int anchor_y, bool allowSubmatrix, bool allowInplace);
int ovx_hal_filterFree(cvhalFilter2D *filter_context);
int ovx_hal_filter(cvhalFilter2D *filter_context, uchar *a, size_t astep, uchar *b, size_t bstep, int w, int h, int, int, int, int);
int ovx_hal_sepFilterInit(cvhalFilter2D **filter_context, int src_type, int dst_type,
                          int kernel_type, uchar *kernelx_data, int kernelx_length, uchar *kernely_data, int kernely_length,
                          int anchor_x, int anchor_y, double delta, int borderType);

#if VX_VERSION > VX_VERSION_1_0
int ovx_hal_morphInit(cvhalFilter2D **filter_context, int operation, int src_type, int dst_type, int , int ,
                      int kernel_type, uchar *kernel_data, size_t kernel_step, int kernel_width, int kernel_height, int anchor_x, int anchor_y,
                      int borderType, const double borderValue[4], int iterations, bool allowSubmatrix, bool allowInplace);
int ovx_hal_morphFree(cvhalFilter2D *filter_context);
int ovx_hal_morph(cvhalFilter2D *filter_context, uchar *a, size_t astep, uchar *b, size_t bstep, int w, int h, int , int , int , int , int , int , int , int );
#endif // 1.0 guard

int ovx_hal_cvtBGRtoBGR(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int depth, int acn, int bcn, bool swapBlue);
int ovx_hal_cvtGraytoBGR(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int depth, int bcn);
int ovx_hal_cvtTwoPlaneYUVtoBGR(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int bcn, bool swapBlue, int uIdx);
int ovx_hal_cvtTwoPlaneYUVtoBGREx(const uchar * a, size_t astep, const uchar * b, size_t bstep, uchar * c, size_t cstep, int w, int h, int bcn, bool swapBlue, int uIdx);
int ovx_hal_cvtThreePlaneYUVtoBGR(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int bcn, bool swapBlue, int uIdx);
int ovx_hal_cvtBGRtoThreePlaneYUV(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int acn, bool swapBlue, int uIdx);
int ovx_hal_cvtOnePlaneYUVtoBGR(const uchar * a, size_t astep, uchar * b, size_t bstep, int w, int h, int bcn, bool swapBlue, int uIdx, int ycn);
int ovx_hal_integral(int depth, int sdepth, int, const uchar * a, size_t astep, uchar * b, size_t bstep, uchar * c, size_t, uchar * d, size_t, int w, int h, int cn);
int ovx_hal_meanStdDev(const uchar* src_data, size_t src_step, int width, int height,
                       int src_type, double* mean_val, double* stddev_val, uchar* mask, size_t mask_step);
int ovx_hal_lut(const uchar *src_data, size_t src_step, size_t src_type, const uchar* lut_data, size_t lut_channel_size, size_t lut_channels, uchar *dst_data, size_t dst_step, int width, int height);
int ovx_hal_minMaxIdxMaskStep(const uchar* src_data, size_t src_step, int width, int height, int depth,
                              double* minVal, double* maxVal, int* minIdx, int* maxIdx, uchar* mask, size_t mask_step);
int ovx_hal_medianBlur(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int width, int height, int depth, int cn, int ksize);
int ovx_hal_sobel(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int width, int height, int src_depth, int dst_depth, int cn, int margin_left, int margin_top, int margin_right, int margin_bottom, int dx, int dy, int ksize, double scale, double delta, int border_type);
int ovx_hal_canny(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step,
                  int width, int height, int cn, double lowThreshold, double highThreshold, int ksize, bool L2gradient);
int ovx_hal_pyrdown(const uchar* src_data, size_t src_step, int src_width, int src_height,
                    uchar* dst_data, size_t dst_step, int dst_width, int dst_height, int depth, int cn, int border_type);
int ovx_hal_boxFilter(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step,
                      int width, int height, int src_depth, int dst_depth, int cn,
                      int margin_left, int margin_top, int margin_right, int margin_bottom,
                      size_t ksize_width, size_t ksize_height, int anchor_x, int anchor_y, bool normalize, int border_type);
int ovx_hal_equalize_hist(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step, int width, int height);
int ovx_hal_gaussianBlur(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step,  int width, int height,
                        int depth, int cn, size_t margin_left, size_t margin_top, size_t margin_right, size_t margin_bottom,
                        size_t ksize_width, size_t ksize_height, double sigmaX, double sigmaY, int border_type);
int ovx_hal_remap32f(int src_type, const uchar *src_data, size_t src_step, int src_width, int src_height,
                     uchar *dst_data, size_t dst_step, int dst_width, int dst_height,
                     float* mapx, size_t mapx_step, float* mapy, size_t mapy_step,
                     int interpolation, int border_type, const double border_value[4]);
int ovx_hal_threshold(const uchar* src_data, size_t src_step, uchar* dst_data, size_t dst_step,
                      int width, int height, int depth, int cn, double thresh, double maxValue, int thresholdType);
int ovx_hal_FAST(const uchar* src_data, size_t src_step, int width, int height, uchar* keypoints_data, size_t* keypoints_count,
                 int threshold, bool nonmax_suppression, int /*cv::FastFeatureDetector::DetectorType*/ dtype);

//==================================================================================================
// functions redefinition
// ...

#undef cv_hal_add8u
#define cv_hal_add8u ovx_hal_add<uchar>
#undef cv_hal_add16s
#define cv_hal_add16s ovx_hal_add<short>
#undef cv_hal_sub8u
#define cv_hal_sub8u ovx_hal_sub<uchar>
#undef cv_hal_sub16s
#define cv_hal_sub16s ovx_hal_sub<short>

#undef cv_hal_absdiff8u
#define cv_hal_absdiff8u ovx_hal_absdiff<uchar>
#undef cv_hal_absdiff16s
#define cv_hal_absdiff16s ovx_hal_absdiff<short>

#undef cv_hal_and8u
#define cv_hal_and8u ovx_hal_and<uchar>
#undef cv_hal_or8u
#define cv_hal_or8u ovx_hal_or<uchar>
#undef cv_hal_xor8u
#define cv_hal_xor8u ovx_hal_xor<uchar>
#undef cv_hal_not8u
#define cv_hal_not8u ovx_hal_not

#undef cv_hal_mul8u
#define cv_hal_mul8u ovx_hal_mul<uchar>
#undef cv_hal_mul16s
#define cv_hal_mul16s ovx_hal_mul<short>

#undef cv_hal_merge8u
#define cv_hal_merge8u ovx_hal_merge8u

//#undef cv_hal_resize
//#define cv_hal_resize ovx_hal_resize

//OpenVX warps use round to zero policy at least in sample implementation
//while OpenCV require round to nearest
//#undef cv_hal_warpAffine
//#define cv_hal_warpAffine ovx_hal_warpAffine
//#undef cv_hal_warpPerspective
//#define cv_hal_warpPerspective ovx_hal_warpPerspective

#undef cv_hal_filterInit
#define cv_hal_filterInit ovx_hal_filterInit
#undef cv_hal_filter
#define cv_hal_filter ovx_hal_filter
#undef cv_hal_filterFree
#define cv_hal_filterFree ovx_hal_filterFree

//#undef cv_hal_sepFilterInit
//#define cv_hal_sepFilterInit ovx_hal_sepFilterInit
//#undef cv_hal_sepFilter
//#define cv_hal_sepFilter ovx_hal_filter
//#undef cv_hal_sepFilterFree
//#define cv_hal_sepFilterFree ovx_hal_filterFree

#if VX_VERSION > VX_VERSION_1_0

#undef cv_hal_morphInit
#define cv_hal_morphInit ovx_hal_morphInit
#undef cv_hal_morph
#define cv_hal_morph ovx_hal_morph
#undef cv_hal_morphFree
#define cv_hal_morphFree ovx_hal_morphFree

#endif // 1.0 guard

#undef cv_hal_cvtBGRtoBGR
#define cv_hal_cvtBGRtoBGR ovx_hal_cvtBGRtoBGR
#undef cv_hal_cvtGraytoBGR
#define cv_hal_cvtGraytoBGR ovx_hal_cvtGraytoBGR
#undef cv_hal_cvtTwoPlaneYUVtoBGR
#define cv_hal_cvtTwoPlaneYUVtoBGR ovx_hal_cvtTwoPlaneYUVtoBGR
#undef cv_hal_cvtTwoPlaneYUVtoBGREx
#define cv_hal_cvtTwoPlaneYUVtoBGREx ovx_hal_cvtTwoPlaneYUVtoBGREx
#undef cv_hal_cvtThreePlaneYUVtoBGR
#define cv_hal_cvtThreePlaneYUVtoBGR ovx_hal_cvtThreePlaneYUVtoBGR
#undef cv_hal_cvtBGRtoThreePlaneYUV
#define cv_hal_cvtBGRtoThreePlaneYUV ovx_hal_cvtBGRtoThreePlaneYUV
#undef cv_hal_cvtOnePlaneYUVtoBGR
#define cv_hal_cvtOnePlaneYUVtoBGR ovx_hal_cvtOnePlaneYUVtoBGR
#undef cv_hal_integral
#define cv_hal_integral ovx_hal_integral
#undef cv_hal_meanStdDev
#define cv_hal_meanStdDev ovx_hal_meanStdDev
#undef cv_hal_lut
#define cv_hal_lut ovx_hal_lut
#undef cv_hal_minMaxIdxMaskStep
#define cv_hal_minMaxIdxMaskStep ovx_hal_minMaxIdxMaskStep

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

### Classes and Structures

- **cvhalFilter2D**: A class/struct defined in this file

### Functions and Methods

- **cv_hal_resize()**: A function/method defined in this file
- **cv_hal_xor8u()**: A function/method defined in this file
- **cv_hal_meanStdDev()**: A function/method defined in this file
- **cv_hal_cvtTwoPlaneYUVtoBGREx()**: A function/method defined in this file
- **cv_hal_cvtBGRtoBGR()**: A function/method defined in this file
- **cv_hal_warpPerspective()**: A function/method defined in this file
- **cv_hal_filterFree()**: A function/method defined in this file
- **OPENCV_OPENVX_HAL_HPP_INCLUDED()**: A function/method defined in this file
- **cv_hal_mul16s()**: A function/method defined in this file
- **cv_hal_merge8u()**: A function/method defined in this file
- **cv_hal_lut()**: A function/method defined in this file
- **cv_hal_sepFilter()**: A function/method defined in this file
- **cv_hal_add16s()**: A function/method defined in this file
- **cv_hal_cvtTwoPlaneYUVtoBGR()**: A function/method defined in this file
- **cv_hal_minMaxIdxMaskStep()**: A function/method defined in this file
- **cv_hal_sub16s()**: A function/method defined in this file
- **cv_hal_and8u()**: A function/method defined in this file
- **cv_hal_cvtOnePlaneYUVtoBGR()**: A function/method defined in this file
- **cv_hal_integral()**: A function/method defined in this file
- **cv_hal_filterInit()**: A function/method defined in this file
- **cv_hal_cvtThreePlaneYUVtoBGR()**: A function/method defined in this file
- **cv_hal_sub8u()**: A function/method defined in this file
- **cv_hal_cvtBGRtoThreePlaneYUV()**: A function/method defined in this file
- **cv_hal_or8u()**: A function/method defined in this file
- **cv_hal_not8u()**: A function/method defined in this file
- **cv_hal_sepFilterFree()**: A function/method defined in this file
- **cv_hal_cvtGraytoBGR()**: A function/method defined in this file
- **cv_hal_warpAffine()**: A function/method defined in this file
- **cv_hal_mul8u()**: A function/method defined in this file
- **cv_hal_absdiff16s()**: A function/method defined in this file
- **cv_hal_add8u()**: A function/method defined in this file
- **cv_hal_morphFree()**: A function/method defined in this file
- **cv_hal_filter()**: A function/method defined in this file
- **cv_hal_morphInit()**: A function/method defined in this file
- **cv_hal_absdiff8u()**: A function/method defined in this file
- **cv_hal_sepFilterInit()**: A function/method defined in this file
- **cv_hal_morph()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/interface.h`
- `VX/vx.h`


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

