# Documentation for `samples/cpp/tutorial_code/gapi/porting_anisotropic_image_segmentation/porting_anisotropic_image_segmentation_gapi.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/gapi/porting_anisotropic_image_segmentation/porting_anisotropic_image_segmentation_gapi.cpp`
- **File Name**: `porting_anisotropic_image_segmentation_gapi.cpp`
- **File Size**: 3,645 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/gapi/porting_anisotropic_image_segmentation/porting_anisotropic_image_segmentation_gapi.cpp](../../../../../samples/cpp/tutorial_code/gapi/porting_anisotropic_image_segmentation/porting_anisotropic_image_segmentation_gapi.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/gapi/porting_anisotropic_image_segmentation` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/**
* @brief You will learn how port an existing algorithm to G-API
* @author Dmitry Matveev, dmitry.matveev@intel.com, based
*    on sample by Karpushin Vladislav, karpushin@ngs.ru
*/
#include "opencv2/opencv_modules.hpp"
#ifdef HAVE_OPENCV_GAPI

//! [full_sample]
#include <iostream>
#include <utility>

#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/gapi.hpp"
#include "opencv2/gapi/core.hpp"
#include "opencv2/gapi/imgproc.hpp"

//! [calcGST_proto]
void calcGST(const cv::GMat& inputImg, cv::GMat& imgCoherencyOut, cv::GMat& imgOrientationOut, int w);
//! [calcGST_proto]

int main()
{
    int W = 52;             // window size is WxW
    double C_Thr = 0.43;    // threshold for coherency
    int LowThr = 35;        // threshold1 for orientation, it ranges from 0 to 180
    int HighThr = 57;       // threshold2 for orientation, it ranges from 0 to 180

    cv::Mat imgIn = cv::imread("input.jpg", cv::IMREAD_GRAYSCALE);
    if (imgIn.empty()) //check whether the image is loaded or not
    {
        std::cout << "ERROR : Image cannot be loaded..!!" << std::endl;
        return -1;
    }

    //! [main]
    // Calculate Gradient Structure Tensor and post-process it for output with G-API
    cv::GMat in;
    cv::GMat imgCoherency, imgOrientation;
    calcGST(in, imgCoherency, imgOrientation, W);

    cv::GMat imgCoherencyBin = imgCoherency > C_Thr;
    cv::GMat imgOrientationBin = cv::gapi::inRange(imgOrientation, LowThr, HighThr);
    cv::GMat imgBin = imgCoherencyBin & imgOrientationBin;
    cv::GMat out = cv::gapi::addWeighted(in, 0.5, imgBin, 0.5, 0.0);

    // Normalize extra outputs
    cv::GMat imgCoherencyNorm = cv::gapi::normalize(imgCoherency, 0, 255, cv::NORM_MINMAX);
    cv::GMat imgOrientationNorm = cv::gapi::normalize(imgOrientation, 0, 255, cv::NORM_MINMAX);

    // Capture the graph into object segm
    cv::GComputation segm(cv::GIn(in), cv::GOut(out, imgCoherencyNorm, imgOrientationNorm));

    // Define cv::Mats for output data
    cv::Mat imgOut, imgOutCoherency, imgOutOrientation;

    // Run the graph
    segm.apply(cv::gin(imgIn), cv::gout(imgOut, imgOutCoherency, imgOutOrientation));

    cv::imwrite("result.jpg", imgOut);
    cv::imwrite("Coherency.jpg", imgOutCoherency);
    cv::imwrite("Orientation.jpg", imgOutOrientation);
    //! [main]

    return 0;
}
//! [calcGST]
//! [calcGST_header]
void calcGST(const cv::GMat& inputImg, cv::GMat& imgCoherencyOut, cv::GMat& imgOrientationOut, int w)
{
    auto img = cv::gapi::convertTo(inputImg, CV_32F);
    auto imgDiffX = cv::gapi::Sobel(img, CV_32F, 1, 0, 3);
    auto imgDiffY = cv::gapi::Sobel(img, CV_32F, 0, 1, 3);
    auto imgDiffXY = cv::gapi::mul(imgDiffX, imgDiffY);
    //! [calcGST_header]

    auto imgDiffXX = cv::gapi::mul(imgDiffX, imgDiffX);
    auto imgDiffYY = cv::gapi::mul(imgDiffY, imgDiffY);

    auto J11 = cv::gapi::boxFilter(imgDiffXX, CV_32F, cv::Size(w, w));
    auto J22 = cv::gapi::boxFilter(imgDiffYY, CV_32F, cv::Size(w, w));
    auto J12 = cv::gapi::boxFilter(imgDiffXY, CV_32F, cv::Size(w, w));

    auto tmp1 = J11 + J22;
    auto tmp2 = J11 - J22;
    auto tmp22 = cv::gapi::mul(tmp2, tmp2);
    auto tmp3 = cv::gapi::mul(J12, J12);
    auto tmp4 = cv::gapi::sqrt(tmp22 + 4.0*tmp3);

    auto lambda1 = tmp1 + tmp4;
    auto lambda2 = tmp1 - tmp4;

    imgCoherencyOut = (lambda1 - lambda2) / (lambda1 + lambda2);
    imgOrientationOut = 0.5*cv::gapi::phase(J22 - J11, 2.0*J12, true);
}
//! [calcGST]

//! [full_sample]

#else
#include <iostream>
int main()
{
    std::cerr << "This tutorial code requires G-API module to run" << std::endl;
}
#endif  // HAVE_OPECV_GAPI
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

### Functions and Methods

- **HAVE_OPENCV_GAPI()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `opencv2/gapi/imgproc.hpp`
- `opencv2/imgcodecs.hpp`
- `opencv2/opencv_modules.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/core.hpp`
- `opencv2/gapi.hpp`

**Python Imports:**
- `0`


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

