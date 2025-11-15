# Documentation for `modules/gapi/src/backends/ocl/goclimgproc.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/ocl/goclimgproc.cpp`
- **File Name**: `goclimgproc.cpp`
- **File Size**: 9,975 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/ocl/goclimgproc.cpp](../../../../../modules/gapi/src/backends/ocl/goclimgproc.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "precomp.hpp"

#include <opencv2/gapi/imgproc.hpp>
#include <opencv2/gapi/ocl/imgproc.hpp>
#include "backends/ocl/goclimgproc.hpp"

GAPI_OCL_KERNEL(GOCLResize, cv::gapi::imgproc::GResize)
{
    static void run(const cv::UMat& in, cv::Size sz, double fx, double fy, int interp, cv::UMat &out)
    {
        cv::resize(in, out, sz, fx, fy, interp);
    }
};

GAPI_OCL_KERNEL(GOCLSepFilter, cv::gapi::imgproc::GSepFilter)
{
    static void run(const cv::UMat& in, int ddepth, const cv::Mat& kernX, const cv::Mat& kernY, const cv::Point& anchor, const cv::Scalar& delta,
                    int border, const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( border == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int width_add = (kernY.cols - 1) / 2;
            int height_add =  (kernX.rows - 1) / 2;
            cv::copyMakeBorder(in, temp_in, height_add, height_add, width_add, width_add, border, bordVal);
            cv::Rect rect = cv::Rect(height_add, width_add, in.cols, in.rows);
            cv::sepFilter2D(temp_in(rect), out, ddepth, kernX, kernY, anchor, delta.val[0], border);
        }
        else
            cv::sepFilter2D(in, out, ddepth, kernX, kernY, anchor, delta.val[0], border);
    }
};

GAPI_OCL_KERNEL(GOCLBoxFilter, cv::gapi::imgproc::GBoxFilter)
{
    static void run(const cv::UMat& in, int ddepth, const cv::Size& ksize, const cv::Point& anchor, bool normalize, int borderType, const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( borderType == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int width_add = (ksize.width - 1) / 2;
            int height_add =  (ksize.height - 1) / 2;
            cv::copyMakeBorder(in, temp_in, height_add, height_add, width_add, width_add, borderType, bordVal);
            cv::Rect rect = cv::Rect(height_add, width_add, in.cols, in.rows);
            cv::boxFilter(temp_in(rect), out, ddepth, ksize, anchor, normalize, borderType);
        }
        else
            cv::boxFilter(in, out, ddepth, ksize, anchor, normalize, borderType);
    }
};

GAPI_OCL_KERNEL(GOCLBlur, cv::gapi::imgproc::GBlur)
{
    static void run(const cv::UMat& in, const cv::Size& ksize, const cv::Point& anchor, int borderType, const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( borderType == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int width_add = (ksize.width - 1) / 2;
            int height_add =  (ksize.height - 1) / 2;
            cv::copyMakeBorder(in, temp_in, height_add, height_add, width_add, width_add, borderType, bordVal);
            cv::Rect rect = cv::Rect(height_add, width_add, in.cols, in.rows);
            cv::blur(temp_in(rect), out, ksize, anchor, borderType);
        }
        else
            cv::blur(in, out, ksize, anchor, borderType);
    }
};


GAPI_OCL_KERNEL(GOCLFilter2D, cv::gapi::imgproc::GFilter2D)
{
    static void run(const cv::UMat& in, int ddepth, const cv::Mat& k, const cv::Point& anchor, const cv::Scalar& delta, int border,
                    const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( border == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int width_add = (k.cols - 1) / 2;
            int height_add =  (k.rows - 1) / 2;
            cv::copyMakeBorder(in, temp_in, height_add, height_add, width_add, width_add, border, bordVal );
            cv::Rect rect = cv::Rect(height_add, width_add, in.cols, in.rows);
            cv::filter2D(temp_in(rect), out, ddepth, k, anchor, delta.val[0], border);
        }
        else
            cv::filter2D(in, out, ddepth, k, anchor, delta.val[0], border);
    }
};

GAPI_OCL_KERNEL(GOCLGaussBlur, cv::gapi::imgproc::GGaussBlur)
{
    static void run(const cv::UMat& in, const cv::Size& ksize, double sigmaX, double sigmaY, int borderType, const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( borderType == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int width_add = (ksize.width - 1) / 2;
            int height_add =  (ksize.height - 1) / 2;
            cv::copyMakeBorder(in, temp_in, height_add, height_add, width_add, width_add, borderType, bordVal );
            cv::Rect rect = cv::Rect(height_add, width_add, in.cols, in.rows);
            cv::GaussianBlur(temp_in(rect), out, ksize, sigmaX, sigmaY, borderType);
        }
        else
            cv::GaussianBlur(in, out, ksize, sigmaX, sigmaY, borderType);
    }
};

GAPI_OCL_KERNEL(GOCLMedianBlur, cv::gapi::imgproc::GMedianBlur)
{
    static void run(const cv::UMat& in, int ksize, cv::UMat &out)
    {
        cv::medianBlur(in, out, ksize);
    }
};

GAPI_OCL_KERNEL(GOCLErode, cv::gapi::imgproc::GErode)
{
    static void run(const cv::UMat& in, const cv::Mat& kernel, const cv::Point& anchor, int iterations, int borderType, const cv::Scalar& borderValue, cv::UMat &out)
    {
        cv::erode(in, out, kernel, anchor, iterations, borderType, borderValue);
    }
};

GAPI_OCL_KERNEL(GOCLDilate, cv::gapi::imgproc::GDilate)
{
    static void run(const cv::UMat& in, const cv::Mat& kernel, const cv::Point& anchor, int iterations, int borderType, const cv::Scalar& borderValue, cv::UMat &out)
    {
        cv::dilate(in, out, kernel, anchor, iterations, borderType, borderValue);
    }
};

GAPI_OCL_KERNEL(GOCLSobel, cv::gapi::imgproc::GSobel)
{
    static void run(const cv::UMat& in, int ddepth, int dx, int dy, int ksize, double scale, double delta, int borderType,
                    const cv::Scalar& bordVal, cv::UMat &out)
    {
        if( borderType == cv::BORDER_CONSTANT )
        {
            cv::UMat temp_in;
            int add = (ksize - 1) / 2;
            cv::copyMakeBorder(in, temp_in, add, add, add, add, borderType, bordVal );
            cv::Rect rect = cv::Rect(add, add, in.cols, in.rows);
            cv::Sobel(temp_in(rect), out, ddepth, dx, dy, ksize, scale, delta, borderType);
        }
        else
        cv::Sobel(in, out, ddepth, dx, dy, ksize, scale, delta, borderType);
    }
};

GAPI_OCL_KERNEL(GOCLLaplacian, cv::gapi::imgproc::GLaplacian)
{
    static void run(const cv::UMat& in, int ddepth, int ksize, double scale,
                    double delta, int borderType, cv::UMat &out)
    {
        cv::Laplacian(in, out, ddepth, ksize, scale, delta, borderType);
    }
};

GAPI_OCL_KERNEL(GOCLBilateralFilter, cv::gapi::imgproc::GBilateralFilter)
{
    static void run(const cv::UMat& in, int ddepth, double sigmaColor,
                    double sigmaSpace, int borderType, cv::UMat &out)
    {
        cv::bilateralFilter(in, out, ddepth, sigmaColor, sigmaSpace, borderType);
    }
};

GAPI_OCL_KERNEL(GOCLEqualizeHist, cv::gapi::imgproc::GEqHist)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::equalizeHist(in, out);
    }
};

GAPI_OCL_KERNEL(GOCLCanny, cv::gapi::imgproc::GCanny)
{
    static void run(const cv::UMat& in, double thr1, double thr2, int apSize, bool l2gradient, cv::UMat &out)
    {
        cv::Canny(in, out, thr1, thr2, apSize, l2gradient);
    }
};

GAPI_OCL_KERNEL(GOCLRGB2YUV, cv::gapi::imgproc::GRGB2YUV)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_RGB2YUV);
    }
};

GAPI_OCL_KERNEL(GOCLYUV2RGB, cv::gapi::imgproc::GYUV2RGB)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_YUV2RGB);
    }
};

GAPI_OCL_KERNEL(GOCLRGB2Lab, cv::gapi::imgproc::GRGB2Lab)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_RGB2Lab);
    }
};

GAPI_OCL_KERNEL(GOCLBGR2LUV, cv::gapi::imgproc::GBGR2LUV)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_BGR2Luv);
    }
};

GAPI_OCL_KERNEL(GOCLBGR2YUV, cv::gapi::imgproc::GBGR2YUV)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_BGR2YUV);
    }
};

GAPI_OCL_KERNEL(GOCLLUV2BGR, cv::gapi::imgproc::GLUV2BGR)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_Luv2BGR);
    }
};

GAPI_OCL_KERNEL(GOCLYUV2BGR, cv::gapi::imgproc::GYUV2BGR)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_YUV2BGR);
    }
};

GAPI_OCL_KERNEL(GOCLRGB2Gray, cv::gapi::imgproc::GRGB2Gray)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_RGB2GRAY);
    }
};

GAPI_OCL_KERNEL(GOCLBGR2Gray, cv::gapi::imgproc::GBGR2Gray)
{
    static void run(const cv::UMat& in, cv::UMat &out)
    {
        cv::cvtColor(in, out, cv::COLOR_BGR2GRAY);
    }
};

GAPI_OCL_KERNEL(GOCLRGB2GrayCustom, cv::gapi::imgproc::GRGB2GrayCustom)
{
    //TODO: avoid copy
    static void run(const cv::UMat& in, float rY, float bY, float gY, cv::UMat &out)
    {
        cv::Mat planes[3];
        cv::split(in.getMat(cv::ACCESS_READ), planes);
        cv::Mat tmp_out = (planes[0]*rY + planes[1]*bY + planes[2]*gY);
        tmp_out.copyTo(out);
    }
};


cv::GKernelPackage cv::gapi::imgproc::ocl::kernels()
{
    static auto pkg = cv::gapi::kernels
        < GOCLFilter2D
        , GOCLResize
        , GOCLSepFilter
        , GOCLBoxFilter
        , GOCLBlur
        , GOCLGaussBlur
        , GOCLMedianBlur
        , GOCLErode
        , GOCLDilate
        , GOCLSobel
        , GOCLLaplacian
        , GOCLBilateralFilter
        , GOCLCanny
        , GOCLEqualizeHist
        , GOCLRGB2YUV
        , GOCLYUV2RGB
        , GOCLRGB2Lab
        , GOCLBGR2LUV
        , GOCLBGR2YUV
        , GOCLYUV2BGR
        , GOCLLUV2BGR
        , GOCLBGR2Gray
        , GOCLRGB2Gray
        , GOCLRGB2GrayCustom
        >();
    return pkg;
}
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/imgproc.hpp`
- `precomp.hpp`
- `opencv2/gapi/ocl/imgproc.hpp`
- `backends/ocl/goclimgproc.hpp`


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

