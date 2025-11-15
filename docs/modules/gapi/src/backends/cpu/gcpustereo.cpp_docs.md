# Documentation for `modules/gapi/src/backends/cpu/gcpustereo.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/cpu/gcpustereo.cpp`
- **File Name**: `gcpustereo.cpp`
- **File Size**: 3,001 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/cpu/gcpustereo.cpp](../../../../../modules/gapi/src/backends/cpu/gcpustereo.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/cpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include <opencv2/gapi/stereo.hpp>
#include <opencv2/gapi/cpu/stereo.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>

#ifdef HAVE_OPENCV_CALIB3D
#include <opencv2/calib3d.hpp>
#endif // HAVE_OPENCV_CALIB3D

#ifdef HAVE_OPENCV_CALIB3D

/** @brief Structure for the Stereo operation setup parameters.*/
struct GAPI_EXPORTS StereoSetup {
    double baseline;
    double focus;
    cv::Ptr<cv::StereoBM> stereoBM;
};

namespace {
cv::Mat calcDepth(const cv::Mat &left, const cv::Mat &right,
                  const StereoSetup &ss) {
    constexpr int DISPARITY_SHIFT_16S = 4;
    cv::Mat disp;
    ss.stereoBM->compute(left, right, disp);
    disp.convertTo(disp, CV_32FC1, 1./(1 << DISPARITY_SHIFT_16S), 0);
    return (ss.focus * ss.baseline) / disp;
}
} // anonymous namespace

GAPI_OCV_KERNEL_ST(GCPUStereo, cv::gapi::calib3d::GStereo, StereoSetup)
{
    static void setup(const cv::GMatDesc&, const cv::GMatDesc&,
                      const cv::gapi::StereoOutputFormat,
                      std::shared_ptr<StereoSetup> &stereoSetup,
                      const cv::GCompileArgs &compileArgs) {
        auto stereoInit = cv::gapi::getCompileArg<cv::gapi::calib3d::cpu::StereoInitParam>(compileArgs)
            .value_or(cv::gapi::calib3d::cpu::StereoInitParam{});

        StereoSetup ss{stereoInit.baseline,
                       stereoInit.focus,
                       cv::StereoBM::create(stereoInit.numDisparities,
                       stereoInit.blockSize)};
        stereoSetup = std::make_shared<StereoSetup>(ss);
    }
    static void run(const cv::Mat& left,
                    const cv::Mat& right,
                    const cv::gapi::StereoOutputFormat oF,
                    cv::Mat& out_mat,
                    const StereoSetup &stereoSetup) {
        switch(oF){
            case cv::gapi::StereoOutputFormat::DEPTH_FLOAT16:
                calcDepth(left, right, stereoSetup).convertTo(out_mat, CV_16FC1);
                break;
            case cv::gapi::StereoOutputFormat::DEPTH_FLOAT32:
                calcDepth(left, right, stereoSetup).copyTo(out_mat);
                break;
            case cv::gapi::StereoOutputFormat::DISPARITY_FIXED16_12_4:
                stereoSetup.stereoBM->compute(left, right, out_mat);
                break;
            case cv::gapi::StereoOutputFormat::DISPARITY_FIXED16_11_5:
                GAPI_Error("This case may be supported in future.");
            default:
                GAPI_Error("Unknown output format!");
        }
    }
};

cv::GKernelPackage cv::gapi::calib3d::cpu::kernels() {
    static auto pkg = cv::gapi::kernels<GCPUStereo>();
    return pkg;
}

#else

cv::GKernelPackage cv::gapi::calib3d::cpu::kernels()
{
    return GKernelPackage();
}

#endif // HAVE_OPENCV_CALIB3D
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

### Classes and Structures

- **GAPI_EXPORTS**: A class/struct defined in this file

### Functions and Methods

- **HAVE_OPENCV_CALIB3D()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/calib3d.hpp`
- `opencv2/gapi/cpu/stereo.hpp`
- `opencv2/gapi/stereo.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`


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

