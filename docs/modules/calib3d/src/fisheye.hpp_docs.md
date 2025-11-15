# Documentation for `modules/calib3d/src/fisheye.hpp`

## File Metadata

- **Full Path**: `modules/calib3d/src/fisheye.hpp`
- **File Name**: `fisheye.hpp`
- **File Size**: 2,572 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/calib3d/src/fisheye.hpp](../../../modules/calib3d/src/fisheye.hpp)

## Purpose and Role

This file is located in the `modules/calib3d/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef FISHEYE_INTERNAL_H
#define FISHEYE_INTERNAL_H
#include "precomp.hpp"

namespace cv { namespace internal {

struct CV_EXPORTS IntrinsicParams
{
    Vec2d f;
    Vec2d c;
    Vec4d k;
    double alpha;
    std::vector<uchar> isEstimate;

    IntrinsicParams();
    IntrinsicParams(Vec2d f, Vec2d c, Vec4d k, double alpha = 0);
    IntrinsicParams operator+(const Mat& a);
    IntrinsicParams& operator =(const Mat& a);
    void Init(const cv::Vec2d& f, const cv::Vec2d& c, const cv::Vec4d& k = Vec4d(0,0,0,0), const double& alpha = 0);
};

void projectPoints(cv::InputArray objectPoints, cv::OutputArray imagePoints,
                   cv::InputArray _rvec,cv::InputArray _tvec,
                   const IntrinsicParams& param, cv::OutputArray jacobian);

void ComputeExtrinsicRefine(const Mat& imagePoints, const Mat& objectPoints, Mat& rvec,
                            Mat&  tvec, Mat& J, const int MaxIter,
                            const IntrinsicParams& param, const double thresh_cond);
CV_EXPORTS Mat ComputeHomography(Mat m, Mat M);

CV_EXPORTS Mat NormalizePixels(const Mat& imagePoints, const IntrinsicParams& param);

void InitExtrinsics(const Mat& _imagePoints, const Mat& _objectPoints, const IntrinsicParams& param, Mat& omckk, Mat& Tckk);

void CalibrateExtrinsics(InputArrayOfArrays objectPoints, InputArrayOfArrays imagePoints,
                         const IntrinsicParams& param, const int check_cond,
                         const double thresh_cond, InputOutputArray omc, InputOutputArray Tc);

void ComputeJacobians(InputArrayOfArrays objectPoints, InputArrayOfArrays imagePoints,
                      const IntrinsicParams& param,  InputArray omc, InputArray Tc,
                      const int& check_cond, const double& thresh_cond, Mat& JJ2_inv, Mat& ex3);

CV_EXPORTS void  EstimateUncertainties(InputArrayOfArrays objectPoints, InputArrayOfArrays imagePoints,
                           const IntrinsicParams& params, InputArray omc, InputArray Tc,
                           IntrinsicParams& errors, Vec2d& std_err, double thresh_cond, int check_cond, double& rms);

void dAB(cv::InputArray A, InputArray B, OutputArray dABdA, OutputArray dABdB);

void JRodriguesMatlab(const Mat& src, Mat& dst);

void compose_motion(InputArray _om1, InputArray _T1, InputArray _om2, InputArray _T2,
                    Mat& om3, Mat& T3, Mat& dom3dom1, Mat& dom3dT1, Mat& dom3dom2,
                    Mat& dom3dT2, Mat& dT3dom1, Mat& dT3dT1, Mat& dT3dom2, Mat& dT3dT2);

double median(const Mat& row);

Vec3d median3d(InputArray m);

}}

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

- **FISHEYE_INTERNAL_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`


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

