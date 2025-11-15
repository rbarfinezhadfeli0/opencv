# Documentation for `modules/ts/include/opencv2/ts/ocl_perf.hpp`

## File Metadata

- **Full Path**: `modules/ts/include/opencv2/ts/ocl_perf.hpp`
- **File Name**: `ocl_perf.hpp`
- **File Size**: 5,362 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/ts/include/opencv2/ts/ocl_perf.hpp](../../../../../modules/ts/include/opencv2/ts/ocl_perf.hpp)

## Purpose and Role

This file is located in the `modules/ts/include/opencv2/ts` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2010-2013, Advanced Micro Devices, Inc., all rights reserved.
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
// In no event shall the OpenCV Foundation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_TS_OCL_PERF_HPP
#define OPENCV_TS_OCL_PERF_HPP

#include "opencv2/ts.hpp"

#include "ocl_test.hpp"
#include "ts_perf.hpp"

namespace cvtest {
namespace ocl {

using namespace perf;

#define OCL_PERF_STRATEGY PERF_STRATEGY_SIMPLE

#define OCL_PERF_TEST(fixture, name) SIMPLE_PERF_TEST(fixture, name)
#define OCL_PERF_TEST_P(fixture, name, params) SIMPLE_PERF_TEST_P(fixture, name, params)

#define SIMPLE_PERF_TEST(fixture, name) \
    class OCL##_##fixture##_##name : \
        public ::perf::TestBase \
    { \
    public: \
        OCL##_##fixture##_##name() { } \
    protected: \
        virtual void PerfTestBody() CV_OVERRIDE; \
    }; \
    TEST_F(OCL##_##fixture##_##name, name) { CV_TRACE_REGION("PERF_TEST: " #fixture "_" #name); declare.strategy(OCL_PERF_STRATEGY); RunPerfTestBody(); } \
    void OCL##_##fixture##_##name::PerfTestBody()

#define SIMPLE_PERF_TEST_P(fixture, name, params) \
    class OCL##_##fixture##_##name : \
        public fixture \
    { \
    public: \
        OCL##_##fixture##_##name() { } \
    protected: \
        virtual void PerfTestBody() CV_OVERRIDE; \
    }; \
    TEST_P(OCL##_##fixture##_##name, name) { CV_TRACE_REGION("PERF_TEST_P: " #fixture "_" #name); declare.strategy(OCL_PERF_STRATEGY); RunPerfTestBody(); } \
    INSTANTIATE_TEST_CASE_P(/*none*/, OCL##_##fixture##_##name, params); \
    void OCL##_##fixture##_##name::PerfTestBody()

#define OCL_SIZE_1 szVGA
#define OCL_SIZE_2 sz720p
#define OCL_SIZE_3 sz1080p
#define OCL_SIZE_4 sz2160p

#define OCL_TEST_SIZES ::testing::Values(OCL_SIZE_1, OCL_SIZE_2, OCL_SIZE_3, OCL_SIZE_4)
#define OCL_TEST_TYPES ::testing::Values(CV_8UC1, CV_32FC1, CV_8UC4, CV_32FC4)
#define OCL_TEST_TYPES_14 OCL_TEST_TYPES
#define OCL_TEST_TYPES_134 ::testing::Values(CV_8UC1, CV_32FC1, CV_8UC3, CV_32FC3, CV_8UC4, CV_32FC4)

#define OCL_PERF_ENUM ::testing::Values

//! deprecated
#define OCL_TEST_CYCLE() \
    for (cvtest::ocl::perf::safeFinish(); next() && startTimer(); cvtest::ocl::perf::safeFinish(), stopTimer())
//! deprecated
#define OCL_TEST_CYCLE_N(n) \
    for (declare.iterations(n), cvtest::ocl::perf::safeFinish(); next() && startTimer(); cvtest::ocl::perf::safeFinish(), stopTimer())
//! deprecated
#define OCL_TEST_CYCLE_MULTIRUN(runsNum) \
    for (declare.runs(runsNum), cvtest::ocl::perf::safeFinish(); next() && startTimer(); cvtest::ocl::perf::safeFinish(), stopTimer()) \
        for (int r = 0; r < runsNum; cvtest::ocl::perf::safeFinish(), ++r)

#undef PERF_SAMPLE_BEGIN
#undef PERF_SAMPLE_END
#define PERF_SAMPLE_BEGIN() \
    cvtest::ocl::perf::safeFinish(); \
    for(; next() && startTimer(); cvtest::ocl::perf::safeFinish(), stopTimer()) \
    { \
        CV_TRACE_REGION("iteration");
#define PERF_SAMPLE_END() \
    }


namespace perf {

// Check for current device limitation
void checkDeviceMaxMemoryAllocSize(const Size& size, int type, int factor = 1);

// Initialize Mat with random numbers. Range is depends on the data type.
// TODO Parameter type is actually OutputArray
void randu(InputOutputArray dst);

inline void safeFinish()
{
    if (cv::ocl::useOpenCL())
        cv::ocl::finish();
}

} // namespace perf
using namespace perf;

} // namespace cvtest::ocl
} // namespace cvtest

#endif // OPENCV_TS_OCL_PERF_HPP
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

- **OCL**: A class/struct defined in this file

### Functions and Methods

- **PERF_SAMPLE_BEGIN()**: A function/method defined in this file
- **PERF_SAMPLE_END()**: A function/method defined in this file
- **OPENCV_TS_OCL_PERF_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ocl_test.hpp`
- `opencv2/ts.hpp`
- `ts_perf.hpp`

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

