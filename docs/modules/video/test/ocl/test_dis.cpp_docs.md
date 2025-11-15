# Documentation for `modules/video/test/ocl/test_dis.cpp`

## File Metadata

- **Full Path**: `modules/video/test/ocl/test_dis.cpp`
- **File Name**: `test_dis.cpp`
- **File Size**: 3,339 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/test/ocl/test_dis.cpp](../../../../modules/video/test/ocl/test_dis.cpp)

## Purpose and Role

This file is located in the `modules/video/test/ocl` directory and serves as part of the OpenCV library infrastructure.

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
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
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
//   * The name of Intel Corporation may not be used to endorse or promote products
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

namespace opencv_test { namespace {

CV_ENUM(DIS_TestPresets, DISOpticalFlow::PRESET_ULTRAFAST, DISOpticalFlow::PRESET_FAST, DISOpticalFlow::PRESET_MEDIUM)

typedef ocl::TSTestWithParam<DIS_TestPresets> OCL_DenseOpticalFlow_DIS;

OCL_TEST_P(OCL_DenseOpticalFlow_DIS, Mat)
{
    int preset = (int)GetParam();
    Mat frame1, frame2, GT;

    frame1 = imread(TS::ptr()->get_data_path() + "optflow/RubberWhale1.png");
    frame2 = imread(TS::ptr()->get_data_path() + "optflow/RubberWhale2.png");

    CV_Assert(!frame1.empty() && !frame2.empty());

    cvtColor(frame1, frame1, COLOR_BGR2GRAY);
    cvtColor(frame2, frame2, COLOR_BGR2GRAY);

    {
        Mat flow;
        UMat ocl_flow;

        Ptr<DenseOpticalFlow> algo = DISOpticalFlow::create(preset);
        OCL_OFF(algo->calc(frame1, frame2, flow));
        OCL_ON(algo->calc(frame1, frame2, ocl_flow));
        ASSERT_EQ(flow.rows, ocl_flow.rows);
        ASSERT_EQ(flow.cols, ocl_flow.cols);

        EXPECT_MAT_SIMILAR(flow, ocl_flow, 6e-3);
    }
}

OCL_INSTANTIATE_TEST_CASE_P(Video, OCL_DenseOpticalFlow_DIS,
                            DIS_TestPresets::all());

}} // namespace

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

- **ocl()**: A function/method defined in this file
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

