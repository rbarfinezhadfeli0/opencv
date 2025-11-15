# Documentation for `samples/cpp/tutorial_code/gapi/doc_snippets/kernel_api_snippets.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/gapi/doc_snippets/kernel_api_snippets.cpp`
- **File Name**: `kernel_api_snippets.cpp`
- **File Size**: 4,842 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/gapi/doc_snippets/kernel_api_snippets.cpp](../../../../../samples/cpp/tutorial_code/gapi/doc_snippets/kernel_api_snippets.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/gapi/doc_snippets` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// [filter2d_api]
#include <opencv2/gapi.hpp>

G_TYPED_KERNEL(GFilter2D,
               <cv::GMat(cv::GMat,int,cv::Mat,cv::Point,double,int,cv::Scalar)>,
               "org.opencv.imgproc.filters.filter2D")
{
    static cv::GMatDesc                 // outMeta's return value type
    outMeta(cv::GMatDesc    in       ,  // descriptor of input GMat
            int             ddepth   ,  // depth parameter
            cv::Mat      /* coeffs */,  // (unused)
            cv::Point    /* anchor */,  // (unused)
            double       /* scale  */,  // (unused)
            int          /* border */,  // (unused)
            cv::Scalar   /* bvalue */ ) // (unused)
    {
        return in.withDepth(ddepth);
    }
};
// [filter2d_api]

cv::GMat filter2D(cv::GMat  ,
                  int       ,
                  cv::Mat   ,
                  cv::Point ,
                  double    ,
                  int       ,
                  cv::Scalar);

// [filter2d_wrap]
cv::GMat filter2D(cv::GMat   in,
                  int        ddepth,
                  cv::Mat    k,
                  cv::Point  anchor  = cv::Point(-1,-1),
                  double     scale   = 0.,
                  int        border  = cv::BORDER_DEFAULT,
                  cv::Scalar bval    = cv::Scalar(0))
{
    return GFilter2D::on(in, ddepth, k, anchor, scale, border, bval);
}
// [filter2d_wrap]

// [compound]
#include <opencv2/gapi/gcompoundkernel.hpp>       // GAPI_COMPOUND_KERNEL()

using PointArray2f = cv::GArray<cv::Point2f>;

G_TYPED_KERNEL(HarrisCorners,
               <PointArray2f(cv::GMat,int,double,double,int,double)>,
               "org.opencv.imgproc.harris_corner")
{
    static cv::GArrayDesc outMeta(const cv::GMatDesc &,
                                  int,
                                  double,
                                  double,
                                  int,
                                  double)
    {
        // No special metadata for arrays in G-API (yet)
        return cv::empty_array_desc();
    }
};

// Define Fluid-backend-local kernels which form GoodFeatures
G_TYPED_KERNEL(HarrisResponse,
               <cv::GMat(cv::GMat,double,int,double)>,
               "org.opencv.fluid.harris_response")
{
    static cv::GMatDesc outMeta(const cv::GMatDesc &in,
                                double,
                                int,
                                double)
    {
        return in.withType(CV_32F, 1);
    }
};

G_TYPED_KERNEL(ArrayNMS,
               <PointArray2f(cv::GMat,int,double)>,
               "org.opencv.cpu.nms_array")
{
    static cv::GArrayDesc outMeta(const cv::GMatDesc &,
                                  int,
                                  double)
    {
        return cv::empty_array_desc();
    }
};

GAPI_COMPOUND_KERNEL(GFluidHarrisCorners, HarrisCorners)
{
    static PointArray2f
    expand(cv::GMat in,
           int      maxCorners,
           double   quality,
           double   minDist,
           int      blockSize,
           double   k)
    {
        cv::GMat response = HarrisResponse::on(in, quality, blockSize, k);
        return ArrayNMS::on(response, maxCorners, minDist);
    }
};

// Then implement HarrisResponse as Fluid kernel and NMSresponse
// as a generic (OpenCV) kernel
// [compound]

// [filter2d_ocv]
#include <opencv2/gapi/cpu/gcpukernel.hpp>     // GAPI_OCV_KERNEL()
#include <opencv2/imgproc.hpp>                 // cv::filter2D()

GAPI_OCV_KERNEL(GCPUFilter2D, GFilter2D)
{
    static void
    run(const cv::Mat    &in,       // in - derived from GMat
        const int         ddepth,   // opaque (passed as-is)
        const cv::Mat    &k,        // opaque (passed as-is)
        const cv::Point  &anchor,   // opaque (passed as-is)
        const double      delta,    // opaque (passed as-is)
        const int         border,   // opaque (passed as-is)
        const cv::Scalar &,         // opaque (passed as-is)
        cv::Mat          &out)      // out - derived from GMat (retval)
    {
        cv::filter2D(in, out, ddepth, k, anchor, delta, border);
    }
};
// [filter2d_ocv]

int main(int, char *[])
{
    std::cout << "This sample is non-complete. It is used as code snippents in documentation." << std::endl;

cv::Mat conv_kernel_mat;

{
// [filter2d_on]
cv::GMat in;
cv::GMat out = GFilter2D::on(/* GMat    */  in,
                             /* int     */  -1,
                             /* Mat     */  conv_kernel_mat,
                             /* Point   */  cv::Point(-1,-1),
                             /* double  */  0.,
                             /* int     */  cv::BORDER_DEFAULT,
                             /* Scalar  */  cv::Scalar(0));
// [filter2d_on]
}

{
// [filter2d_wrap_call]
cv::GMat in;
cv::GMat out = filter2D(in, -1, conv_kernel_mat);
// [filter2d_wrap_call]
}

return 0;
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
- `opencv2/imgproc.hpp`
- `opencv2/gapi/gcompoundkernel.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `opencv2/gapi.hpp`

**Python Imports:**
- `GMat`


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

