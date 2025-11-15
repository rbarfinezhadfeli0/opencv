# Documentation for `modules/calib3d/src/precomp.hpp`

## File Metadata

- **Full Path**: `modules/calib3d/src/precomp.hpp`
- **File Name**: `precomp.hpp`
- **File Size**: 6,323 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/calib3d/src/precomp.hpp](../../../modules/calib3d/src/precomp.hpp)

## Purpose and Role

This file is located in the `modules/calib3d/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                          License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/
#ifndef __OPENCV_PRECOMP_H__
#define __OPENCV_PRECOMP_H__

#include "opencv2/core/utility.hpp"

#include "opencv2/core/private.hpp"

#include "opencv2/calib3d.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/features2d.hpp"


#include "opencv2/core/ocl.hpp"

#include <set>

#define GET_OPTIMIZED(func) (func)


namespace cv
{

/**
 * Compute the number of iterations given the confidence, outlier ratio, number
 * of model points and the maximum iteration number.
 *
 * @param p confidence value
 * @param ep outlier ratio
 * @param modelPoints number of model points required for estimation
 * @param maxIters maximum number of iterations
 * @return The number of iterations according to the formula
 * \f[
 * \frac{\ln(1-p)}{\ln\left(1-(1-ep)^\mathrm{modelPoints}\right)}
 * \f]
 *
 * If the computed number of iterations is larger than maxIters, then maxIters is returned.
 */
int RANSACUpdateNumIters( double p, double ep, int modelPoints, int maxIters );

class CV_EXPORTS PointSetRegistrator : public Algorithm
{
public:
    class CV_EXPORTS Callback
    {
    public:
        virtual ~Callback() {}
        virtual int runKernel(InputArray m1, InputArray m2, OutputArray model) const = 0;
        virtual void computeError(InputArray m1, InputArray m2, InputArray model, OutputArray err) const = 0;
        virtual bool checkSubset(InputArray, InputArray, int) const { return true; }
    };

    virtual void setCallback(const Ptr<PointSetRegistrator::Callback>& cb) = 0;
    virtual bool run(InputArray m1, InputArray m2, OutputArray model, OutputArray mask) const = 0;
};

CV_EXPORTS Ptr<PointSetRegistrator> createRANSACPointSetRegistrator(const Ptr<PointSetRegistrator::Callback>& cb,
                                                                    int modelPoints, double threshold,
                                                                    double confidence=0.99, int maxIters=1000 );

CV_EXPORTS Ptr<PointSetRegistrator> createLMeDSPointSetRegistrator(const Ptr<PointSetRegistrator::Callback>& cb,
                                                                   int modelPoints, double confidence=0.99, int maxIters=1000 );

template<typename T> inline int compressElems( T* ptr, const uchar* mask, int mstep, int count )
{
    int i, j;
    for( i = j = 0; i < count; i++ )
        if( mask[i*mstep] )
        {
            if( i > j )
                ptr[j] = ptr[i];
            j++;
        }
    return j;
}

static inline bool haveCollinearPoints( const Mat& m, int count )
{
    int j, k, i = count-1;
    const Point2f* ptr = m.ptr<Point2f>();

    // check that the i-th selected point does not belong
    // to a line connecting some previously selected points
    // also checks that points are not too close to each other
    for( j = 0; j < i; j++ )
    {
        double dx1 = ptr[j].x - ptr[i].x;
        double dy1 = ptr[j].y - ptr[i].y;
        for( k = 0; k < j; k++ )
        {
            double dx2 = ptr[k].x - ptr[i].x;
            double dy2 = ptr[k].y - ptr[i].y;
            if( fabs(dx2*dy1 - dy2*dx1) <= FLT_EPSILON*(fabs(dx1) + fabs(dy1) + fabs(dx2) + fabs(dy2)))
                return true;
        }
    }
    return false;
}

void findExtrinsicCameraParams2( const Mat& objectPoints,
                  const Mat& imagePoints, const Mat& A,
                  const Mat& distCoeffs, Mat& rvec, Mat& tvec,
                  int useExtrinsicGuess );

void projectPoints( InputArray objectPoints,
                    InputArray rvec, InputArray tvec,
                    InputArray cameraMatrix, InputArray distCoeffs,
                    OutputArray imagePoints, OutputArray dpdr,
                    OutputArray dpdt, OutputArray dpdf=noArray(),
                    OutputArray dpdc=noArray(), OutputArray dpdk=noArray(),
                    OutputArray dpdo=noArray(), double aspectRatio=0.);

void getUndistortRectangles(InputArray _cameraMatrix, InputArray _distCoeffs,
              InputArray R, InputArray newCameraMatrix, Size imgSize,
              Rect_<double>& inner, Rect_<double>& outer );

} // namespace cv

int checkChessboardBinary(const cv::Mat & img, const cv::Size & size);

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

- **CV_EXPORTS**: A class/struct defined in this file

### Functions and Methods

- **__OPENCV_PRECOMP_H__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `set`
- `opencv2/features2d.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/core/utility.hpp`
- `opencv2/core/private.hpp`
- `opencv2/core/ocl.hpp`
- `opencv2/calib3d.hpp`

**Python Imports:**
- `this`


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

