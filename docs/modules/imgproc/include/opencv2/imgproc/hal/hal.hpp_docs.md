# Documentation for `modules/imgproc/include/opencv2/imgproc/hal/hal.hpp`

## File Metadata

- **Full Path**: `modules/imgproc/include/opencv2/imgproc/hal/hal.hpp`
- **File Name**: `hal.hpp`
- **File Size**: 13,288 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/imgproc/include/opencv2/imgproc/hal/hal.hpp](../../../../../../modules/imgproc/include/opencv2/imgproc/hal/hal.hpp)

## Purpose and Role

This file is located in the `modules/imgproc/include/opencv2/imgproc/hal` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef CV_IMGPROC_HAL_HPP
#define CV_IMGPROC_HAL_HPP

#include "opencv2/core/cvdef.h"
#include "opencv2/core/cvstd.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/core/hal/interface.h"

namespace cv { namespace hal {

//! @addtogroup imgproc_hal_functions
//! @{

//---------------------------
//! @cond IGNORED

struct CV_EXPORTS Filter2D
{
    CV_DEPRECATED static Ptr<hal::Filter2D> create(uchar * , size_t , int ,
                                     int , int ,
                                     int , int ,
                                     int , int ,
                                     int , double ,
                                     int , int ,
                                     bool , bool );
    virtual void apply(uchar * , size_t ,
                       uchar * , size_t ,
                       int , int ,
                       int , int ,
                       int , int ) = 0;
    virtual ~Filter2D() {}
};

struct CV_EXPORTS SepFilter2D
{
    CV_DEPRECATED static Ptr<hal::SepFilter2D> create(int , int , int ,
                                        uchar * , int ,
                                        uchar * , int ,
                                        int , int ,
                                        double , int );
    virtual void apply(uchar * , size_t ,
                       uchar * , size_t ,
                       int , int ,
                       int , int ,
                       int , int ) = 0;
    virtual ~SepFilter2D() {}
};


struct CV_EXPORTS Morph
{
    CV_DEPRECATED static Ptr<hal::Morph> create(int , int , int , int , int ,
                                    int , uchar * , size_t ,
                                    int , int ,
                                    int , int ,
                                    int , const double *,
                                    int , bool , bool );
    virtual void apply(uchar * , size_t , uchar * , size_t , int , int ,
                       int , int , int , int ,
                       int , int , int , int ) = 0;
    virtual ~Morph() {}
};

//! @endcond
//---------------------------

CV_EXPORTS void filter2D(int stype, int dtype, int kernel_type,
                         uchar * src_data, size_t src_step,
                         uchar * dst_data, size_t dst_step,
                         int width, int height,
                         int full_width, int full_height,
                         int offset_x, int offset_y,
                         uchar * kernel_data, size_t kernel_step,
                         int kernel_width, int kernel_height,
                         int anchor_x, int anchor_y,
                         double delta, int borderType,
                         bool isSubmatrix);

CV_EXPORTS void sepFilter2D(int stype, int dtype, int ktype,
                            uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int full_width, int full_height,
                            int offset_x, int offset_y,
                            uchar * kernelx_data, int kernelx_len,
                            uchar * kernely_data, int kernely_len,
                            int anchor_x, int anchor_y,
                            double delta, int borderType);

CV_EXPORTS void morph(int op, int src_type, int dst_type,
                      uchar * src_data, size_t src_step,
                      uchar * dst_data, size_t dst_step,
                      int width, int height,
                      int roi_width, int roi_height, int roi_x, int roi_y,
                      int roi_width2, int roi_height2, int roi_x2, int roi_y2,
                      int kernel_type, uchar * kernel_data, size_t kernel_step,
                      int kernel_width, int kernel_height, int anchor_x, int anchor_y,
                      int borderType, const double borderValue[4],
                      int iterations, bool isSubmatrix);


CV_EXPORTS void resize(int src_type,
                       const uchar * src_data, size_t src_step, int src_width, int src_height,
                       uchar * dst_data, size_t dst_step, int dst_width, int dst_height,
                       double inv_scale_x, double inv_scale_y, int interpolation);

CV_EXPORTS void warpAffine(int src_type,
                           const uchar * src_data, size_t src_step, int src_width, int src_height,
                           uchar * dst_data, size_t dst_step, int dst_width, int dst_height,
                           const double M[6], int interpolation, int borderType, const double borderValue[4]);

CV_EXPORTS void warpAffineBlocklineNN(int *adelta, int *bdelta, short* xy, int X0, int Y0, int bw);

CV_EXPORTS void warpAffineBlockline(int *adelta, int *bdelta, short* xy, short* alpha, int X0, int Y0, int bw);

CV_EXPORTS void warpPerspective(int src_type,
                               const uchar * src_data, size_t src_step, int src_width, int src_height,
                               uchar * dst_data, size_t dst_step, int dst_width, int dst_height,
                               const double M[9], int interpolation, int borderType, const double borderValue[4]);

CV_EXPORTS void warpPerspectiveBlocklineNN(const double *M, short* xy, double X0, double Y0, double W0, int bw);

CV_EXPORTS void warpPerspectiveBlockline(const double *M, short* xy, short* alpha, double X0, double Y0, double W0, int bw);

CV_EXPORTS void cvtBGRtoBGR(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int scn, int dcn, bool swapBlue);

CV_EXPORTS void cvtBGRtoBGR5x5(const uchar * src_data, size_t src_step,
                               uchar * dst_data, size_t dst_step,
                               int width, int height,
                               int scn, bool swapBlue, int greenBits);

CV_EXPORTS void cvtBGR5x5toBGR(const uchar * src_data, size_t src_step,
                               uchar * dst_data, size_t dst_step,
                               int width, int height,
                               int dcn, bool swapBlue, int greenBits);

CV_EXPORTS void cvtBGRtoGray(const uchar * src_data, size_t src_step,
                             uchar * dst_data, size_t dst_step,
                             int width, int height,
                             int depth, int scn, bool swapBlue);

CV_EXPORTS void cvtGraytoBGR(const uchar * src_data, size_t src_step,
                             uchar * dst_data, size_t dst_step,
                             int width, int height,
                             int depth, int dcn);

CV_EXPORTS void cvtBGR5x5toGray(const uchar * src_data, size_t src_step,
                                uchar * dst_data, size_t dst_step,
                                int width, int height,
                                int greenBits);

CV_EXPORTS void cvtGraytoBGR5x5(const uchar * src_data, size_t src_step,
                                uchar * dst_data, size_t dst_step,
                                int width, int height,
                                int greenBits);
CV_EXPORTS void cvtBGRtoYUV(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int scn, bool swapBlue, bool isCbCr,
                            AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtYUVtoBGR(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int dcn, bool swapBlue, bool isCbCr,
                            AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtBGRtoXYZ(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int scn, bool swapBlue);

CV_EXPORTS void cvtXYZtoBGR(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int dcn, bool swapBlue);

CV_EXPORTS void cvtBGRtoHSV(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int scn, bool swapBlue, bool isFullRange, bool isHSV);

CV_EXPORTS void cvtHSVtoBGR(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int dcn, bool swapBlue, bool isFullRange, bool isHSV);

CV_EXPORTS void cvtBGRtoLab(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int scn, bool swapBlue, bool isLab, bool srgb);

CV_EXPORTS void cvtLabtoBGR(const uchar * src_data, size_t src_step,
                            uchar * dst_data, size_t dst_step,
                            int width, int height,
                            int depth, int dcn, bool swapBlue, bool isLab, bool srgb);

CV_EXPORTS void cvtTwoPlaneYUVtoBGR(const uchar * src_data, size_t src_step,
                                    uchar * dst_data, size_t dst_step,
                                    int dst_width, int dst_height,
                                    int dcn, bool swapBlue, int uIdx,
                                    AlgorithmHint hint = ALGO_HINT_DEFAULT);

//! Separate Y and UV planes
CV_EXPORTS void cvtTwoPlaneYUVtoBGR(const uchar * y_data, const uchar * uv_data, size_t src_step,
                                    uchar * dst_data, size_t dst_step,
                                    int dst_width, int dst_height,
                                    int dcn, bool swapBlue, int uIdx,
                                    AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtTwoPlaneYUVtoBGR(const uchar * y_data, size_t y_step, const uchar * uv_data, size_t uv_step,
                                    uchar * dst_data, size_t dst_step,
                                    int dst_width, int dst_height,
                                    int dcn, bool swapBlue, int uIdx,
                                    AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtThreePlaneYUVtoBGR(const uchar * src_data, size_t src_step,
                                      uchar * dst_data, size_t dst_step,
                                      int dst_width, int dst_height,
                                      int dcn, bool swapBlue, int uIdx,
                                      AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtBGRtoThreePlaneYUV(const uchar * src_data, size_t src_step,
                                      uchar * dst_data, size_t dst_step,
                                      int width, int height,
                                      int scn, bool swapBlue, int uIdx,
                                      AlgorithmHint hint = ALGO_HINT_DEFAULT);

//! Separate Y and UV planes
CV_EXPORTS void cvtBGRtoTwoPlaneYUV(const uchar * src_data, size_t src_step,
                                    uchar * y_data, uchar * uv_data, size_t dst_step,
                                    int width, int height,
                                    int scn, bool swapBlue, int uIdx);

CV_EXPORTS void cvtOnePlaneYUVtoBGR(const uchar * src_data, size_t src_step,
                                    uchar * dst_data, size_t dst_step,
                                    int width, int height,
                                    int dcn, bool swapBlue, int uIdx, int ycn,
                                    AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtOnePlaneBGRtoYUV(const uchar * src_data, size_t src_step,
                                    uchar * dst_data, size_t dst_step,
                                    int width, int height,
                                    int scn, bool swapBlue, int uIdx, int ycn,
                                    AlgorithmHint hint = ALGO_HINT_DEFAULT);

CV_EXPORTS void cvtRGBAtoMultipliedRGBA(const uchar * src_data, size_t src_step,
                                        uchar * dst_data, size_t dst_step,
                                        int width, int height);

CV_EXPORTS void cvtMultipliedRGBAtoRGBA(const uchar * src_data, size_t src_step,
                                        uchar * dst_data, size_t dst_step,
                                        int width, int height);

CV_EXPORTS void integral(int depth, int sdepth, int sqdepth,
                         const uchar* src, size_t srcstep,
                         uchar* sum, size_t sumstep,
                         uchar* sqsum, size_t sqsumstep,
                         uchar* tilted, size_t tstep,
                         int width, int height, int cn);

//! @}

}}

#endif // CV_IMGPROC_HAL_HPP
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

- **CV_EXPORTS**: A class/struct defined in this file

### Functions and Methods

- **CV_IMGPROC_HAL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/interface.h`
- `opencv2/core/utility.hpp`
- `opencv2/core/cvstd.hpp`
- `opencv2/core/cvdef.h`


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

