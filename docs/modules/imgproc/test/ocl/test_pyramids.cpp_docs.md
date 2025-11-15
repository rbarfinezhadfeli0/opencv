# Documentation for `modules/imgproc/test/ocl/test_pyramids.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/test/ocl/test_pyramids.cpp`
- **File Name**: `test_pyramids.cpp`
- **File Size**: 6,344 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/test/ocl/test_pyramids.cpp](../../../../modules/imgproc/test/ocl/test_pyramids.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/test/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////////////////
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
// Copyright (C) 2010-2012, Institute Of Software Chinese Academy Of Science, all rights reserved.
// Copyright (C) 2010-2012, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// @Authors
//    Yao Wang yao@multicorewareinc.com
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


#include "../test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

PARAM_TEST_CASE(PyrTestBase, MatDepth, Channels, BorderType, bool)
{
    int depth, channels, borderType;
    bool use_roi;

    TEST_DECLARE_INPUT_PARAMETER(src);
    TEST_DECLARE_OUTPUT_PARAMETER(dst);

    virtual void SetUp()
    {
        depth = GET_PARAM(0);
        channels = GET_PARAM(1);
        borderType = GET_PARAM(2);
        use_roi = GET_PARAM(3);
    }

    void generateTestData(Size src_roiSize, Size dst_roiSize)
    {
        Border srcBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(src, src_roi, src_roiSize, srcBorder, CV_MAKETYPE(depth, channels), -MAX_VALUE, MAX_VALUE);

        Border dstBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(dst, dst_roi, dst_roiSize, dstBorder, CV_MAKETYPE(depth, channels), -MAX_VALUE, MAX_VALUE);

        UMAT_UPLOAD_INPUT_PARAMETER(src);
        UMAT_UPLOAD_OUTPUT_PARAMETER(dst);
    }

    void Near(double threshold = 0.0)
    {
        OCL_EXPECT_MATS_NEAR(dst, threshold);
    }
};

/////////////////////// PyrDown //////////////////////////

typedef PyrTestBase PyrDown;

OCL_TEST_P(PyrDown, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        Size src_roiSize = randomSize(1, MAX_VALUE);
        Size dst_roiSize = Size(randomInt((src_roiSize.width - 1) / 2, (src_roiSize.width + 3) / 2),
                                randomInt((src_roiSize.height - 1) / 2, (src_roiSize.height + 3) / 2));
        dst_roiSize = dst_roiSize.empty() ? Size((src_roiSize.width + 1) / 2, (src_roiSize.height + 1) / 2) : dst_roiSize;
        generateTestData(src_roiSize, dst_roiSize);

        OCL_OFF(pyrDown(src_roi, dst_roi, dst_roiSize, borderType));
        OCL_ON(pyrDown(usrc_roi, udst_roi, dst_roiSize, borderType));

        Near(depth == CV_32F ? 1e-4f : 1.0f);
    }
}

OCL_INSTANTIATE_TEST_CASE_P(ImgprocPyr, PyrDown, Combine(
                            Values(CV_8U, CV_16U, CV_16S, CV_32F, CV_64F),
                            Values(1, 2, 3, 4),
                            Values((BorderType)BORDER_REPLICATE,
                            (BorderType)BORDER_REFLECT, (BorderType)BORDER_REFLECT_101),
                            Bool()
                            ));

/////////////////////// PyrUp //////////////////////////

typedef PyrTestBase PyrUp;

OCL_TEST_P(PyrUp, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        Size src_roiSize = randomSize(1, MAX_VALUE);
        Size dst_roiSize = Size(2 * src_roiSize.width, 2 * src_roiSize.height);
        generateTestData(src_roiSize, dst_roiSize);

        OCL_OFF(pyrUp(src_roi, dst_roi, dst_roiSize, borderType));
        OCL_ON(pyrUp(usrc_roi, udst_roi, dst_roiSize, borderType));

        Near(depth == CV_32F ? 1e-4f : 1.0f);
    }
}

typedef PyrTestBase PyrUp_cols2;

OCL_TEST_P(PyrUp_cols2, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        Size src_roiSize = randomSize(1, MAX_VALUE);
        src_roiSize.width += (src_roiSize.width % 2);
        Size dst_roiSize = Size(2 * src_roiSize.width, 2 * src_roiSize.height);
        generateTestData(src_roiSize, dst_roiSize);

        OCL_OFF(pyrUp(src_roi, dst_roi, dst_roiSize, borderType));
        OCL_ON(pyrUp(usrc_roi, udst_roi, dst_roiSize, borderType));

        Near(depth == CV_32F ? 1e-4f : 1.0f);
    }
}

OCL_INSTANTIATE_TEST_CASE_P(ImgprocPyr, PyrUp, Combine(
                            Values(CV_8U, CV_16U, CV_16S, CV_32F, CV_64F),
                            Values(1, 2, 3, 4),
                            Values((BorderType)BORDER_REFLECT_101),
                            Bool()
                            ));

OCL_INSTANTIATE_TEST_CASE_P(ImgprocPyr, PyrUp_cols2, Combine(
                            Values((MatDepth)CV_8U),
                            Values((Channels)1),
                            Values((BorderType)BORDER_REFLECT_101),
                            Bool()
                            ));

} } // namespace opencv_test::ocl

#endif // HAVE_OPENCL
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

- **PyrTestBase()**: A function/method defined in this file
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

