# Documentation for `modules/imgproc/test/ocl/test_match_template.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/test/ocl/test_match_template.cpp`
- **File Name**: `test_match_template.cpp`
- **File Size**: 5,001 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/test/ocl/test_match_template.cpp](../../../../modules/imgproc/test/ocl/test_match_template.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/test/ocl` directory and serves as part of the OpenCV library infrastructure.

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
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2010-2012, Multicoreware, Inc., all rights reserved.
// Copyright (C) 2010-2012, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
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
// This software is provided by the copyright holders and contributors as is and
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

#include "../test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

///////////////////////////////////////////// matchTemplate //////////////////////////////////////////////////////////

CV_ENUM(MatchTemplType, cv::TM_CCORR, cv::TM_CCORR_NORMED, cv::TM_SQDIFF, cv::TM_SQDIFF_NORMED, cv::TM_CCOEFF, cv::TM_CCOEFF_NORMED)

PARAM_TEST_CASE(MatchTemplate, MatDepth, Channels, MatchTemplType, bool)
{
    int type;
    int depth;
    int method;
    bool use_roi;

    TEST_DECLARE_INPUT_PARAMETER(image);
    TEST_DECLARE_INPUT_PARAMETER(templ);
    TEST_DECLARE_OUTPUT_PARAMETER(result);

    virtual void SetUp()
    {
        type = CV_MAKE_TYPE(GET_PARAM(0), GET_PARAM(1));
        depth = GET_PARAM(0);
        method = GET_PARAM(2);
        use_roi = GET_PARAM(3);
    }

    void generateTestData()
    {
        Size image_roiSize = randomSize(2, 100);
        Size templ_roiSize = Size(randomInt(1, image_roiSize.width), randomInt(1, image_roiSize.height));
        Size result_roiSize = Size(image_roiSize.width - templ_roiSize.width + 1,
                                   image_roiSize.height - templ_roiSize.height + 1);

        const double upValue = 256;

        Border imageBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(image, image_roi, image_roiSize, imageBorder, type, -upValue, upValue);

        Border templBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(templ, templ_roi, templ_roiSize, templBorder, type, -upValue, upValue);

        Border resultBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(result, result_roi, result_roiSize, resultBorder, CV_32FC1, -upValue, upValue);

        UMAT_UPLOAD_INPUT_PARAMETER(image);
        UMAT_UPLOAD_INPUT_PARAMETER(templ);
        UMAT_UPLOAD_OUTPUT_PARAMETER(result);
    }

    void Near()
    {
        bool isNormed =
        method == TM_CCORR_NORMED ||
        method == TM_SQDIFF_NORMED ||
        method == TM_CCOEFF_NORMED;

        if (isNormed)
            OCL_EXPECT_MATS_NEAR(result, 3e-2);
        else
            OCL_EXPECT_MATS_NEAR_RELATIVE_SPARSE(result, 1.5e-2);
    }
};

OCL_TEST_P(MatchTemplate, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        generateTestData();

        OCL_OFF(cv::matchTemplate(image_roi, templ_roi, result_roi, method));
        OCL_ON(cv::matchTemplate(uimage_roi, utempl_roi, uresult_roi, method));

        Near();
    }
}

OCL_INSTANTIATE_TEST_CASE_P(ImageProc, MatchTemplate, Combine(
                                Values(CV_8U, CV_32F),
                                Values(1, 2, 3, 4),
                                MatchTemplType::all(),
                                Bool())
                           );
} } // namespace opencv_test::ocl

#endif
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

- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../test_precomp.hpp`
- `opencv2/ts/ocl_test.hpp`

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

