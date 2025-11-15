# Documentation for `modules/gapi/test/common/gapi_stereo_tests_inl.hpp`

## File Metadata

- **Full Path**: `modules/gapi/test/common/gapi_stereo_tests_inl.hpp`
- **File Name**: `gapi_stereo_tests_inl.hpp`
- **File Size**: 2,474 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/test/common/gapi_stereo_tests_inl.hpp](../../../../modules/gapi/test/common/gapi_stereo_tests_inl.hpp)

## Purpose and Role

This file is located in the `modules/gapi/test/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STEREO_TESTS_INL_HPP
#define OPENCV_GAPI_STEREO_TESTS_INL_HPP


#include <opencv2/gapi/stereo.hpp>
#include <opencv2/gapi/cpu/stereo.hpp>
#include "gapi_stereo_tests.hpp"

#ifdef HAVE_OPENCV_CALIB3D

#include <opencv2/calib3d.hpp>

namespace opencv_test {

TEST_P(TestGAPIStereo, DisparityDepthTest)
{
    using format = cv::gapi::StereoOutputFormat;
    switch(oF) {
        case format::DEPTH_FLOAT16: dtype = CV_16FC1; break;
        case format::DEPTH_FLOAT32: dtype = CV_32FC1; break;
        case format::DISPARITY_FIXED16_12_4: dtype = CV_16SC1; break;
        default: GAPI_Error("Unsupported format in test");
    }
    initOutMats(sz, dtype);

    // G-API
    cv::GMat inL, inR;
    cv::GMat out = cv::gapi::stereo(inL, inR, oF);

    cv::GComputation(cv::GIn(inL, inR), cv::GOut(out))
        .apply(cv::gin(in_mat1, in_mat2), cv::gout(out_mat_gapi),
        cv::compile_args(cv::gapi::calib3d::cpu::kernels(),
                         cv::gapi::calib3d::cpu::StereoInitParam {
                             numDisparities,
                             blockSize,
                             baseline,
                             focus}));

    // OpenCV
    cv::StereoBM::create(numDisparities, blockSize)->compute(in_mat1,
                                                             in_mat2,
                                                             out_mat_ocv);

    static const int DISPARITY_SHIFT_16S = 4;
    switch(oF) {
        case format::DEPTH_FLOAT16:
            out_mat_ocv.convertTo(out_mat_ocv, CV_32FC1, 1./(1 << DISPARITY_SHIFT_16S), 0);
            out_mat_ocv = (focus * baseline) / out_mat_ocv;
            out_mat_ocv.convertTo(out_mat_ocv, CV_16FC1);
            break;
        case format::DEPTH_FLOAT32:
            out_mat_ocv.convertTo(out_mat_ocv, CV_32FC1, 1./(1 << DISPARITY_SHIFT_16S), 0);
            out_mat_ocv = (focus * baseline) / out_mat_ocv;
            break;
        case format::DISPARITY_FIXED16_12_4:
            break;
        default:
            GAPI_Error("Unsupported format in test");
    }

    EXPECT_TRUE(cmpF(out_mat_gapi, out_mat_ocv));
}

} // namespace opencv_test

#endif // HAVE_OPENCV_CALIB3D

#endif // OPENCV_GAPI_STEREO_TESTS_INL_HPP
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

- **HAVE_OPENCV_CALIB3D()**: A function/method defined in this file
- **OPENCV_GAPI_STEREO_TESTS_INL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/cpu/stereo.hpp`
- `opencv2/gapi/stereo.hpp`
- `opencv2/calib3d.hpp`
- `gapi_stereo_tests.hpp`


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

