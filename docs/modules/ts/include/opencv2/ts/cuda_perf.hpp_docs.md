# Documentation for `modules/ts/include/opencv2/ts/cuda_perf.hpp`

## File Metadata

- **Full Path**: `modules/ts/include/opencv2/ts/cuda_perf.hpp`
- **File Name**: `cuda_perf.hpp`
- **File Size**: 4,970 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/ts/include/opencv2/ts/cuda_perf.hpp](../../../../../modules/ts/include/opencv2/ts/cuda_perf.hpp)

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
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
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
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_CUDA_PERF_UTILITY_HPP
#define OPENCV_CUDA_PERF_UTILITY_HPP

#include "opencv2/ts.hpp"

#include "opencv2/ts/ts_perf.hpp"

namespace perf
{
    #define ALL_BORDER_MODES BorderMode::all()
    #define ALL_INTERPOLATIONS Interpolation::all()

    CV_ENUM(BorderMode, BORDER_REFLECT101, BORDER_REPLICATE, BORDER_CONSTANT, BORDER_REFLECT, BORDER_WRAP)
    CV_ENUM(Interpolation, INTER_NEAREST, INTER_LINEAR, INTER_CUBIC, INTER_AREA)
    CV_ENUM(NormType, NORM_INF, NORM_L1, NORM_L2, NORM_HAMMING, NORM_MINMAX)

    enum { Gray = 1, TwoChannel = 2, BGR = 3, BGRA = 4 };
    CV_ENUM(MatCn, Gray, TwoChannel, BGR, BGRA)

    #define CUDA_CHANNELS_1_3_4 testing::Values(MatCn(Gray), MatCn(BGR), MatCn(BGRA))
    #define CUDA_CHANNELS_1_3 testing::Values(MatCn(Gray), MatCn(BGR))

    #define GET_PARAM(k) testing::get< k >(GetParam())

    #define DEF_PARAM_TEST(name, ...) typedef ::perf::TestBaseWithParam< testing::tuple< __VA_ARGS__ > > name
    #define DEF_PARAM_TEST_1(name, param_type) typedef ::perf::TestBaseWithParam< param_type > name

    DEF_PARAM_TEST_1(Sz, cv::Size);
    typedef perf::Size_MatType Sz_Type;
    DEF_PARAM_TEST(Sz_Depth, cv::Size, perf::MatDepth);
    DEF_PARAM_TEST(Sz_Depth_Cn, cv::Size, perf::MatDepth, MatCn);

    #define CUDA_TYPICAL_MAT_SIZES testing::Values(perf::sz720p, perf::szSXGA, perf::sz1080p)

    #define FAIL_NO_CPU() FAIL() << "No such CPU implementation analogy"

    #define CUDA_SANITY_CHECK(mat, ...) \
        do{ \
            cv::Mat gpu_##mat(mat); \
            SANITY_CHECK(gpu_##mat, ## __VA_ARGS__); \
        } while(0)

    #define CPU_SANITY_CHECK(mat, ...) \
        do{ \
            cv::Mat cpu_##mat(mat); \
            SANITY_CHECK(cpu_##mat, ## __VA_ARGS__); \
        } while(0)

    cv::Mat readImage(const std::string& fileName, int flags = cv::IMREAD_COLOR);

    struct CvtColorInfo
    {
        int scn;
        int dcn;
        int code;

        CvtColorInfo() {}
        explicit CvtColorInfo(int scn_, int dcn_, int code_) : scn(scn_), dcn(dcn_), code(code_) {}
    };
    void PrintTo(const CvtColorInfo& info, std::ostream* os);

    void printCudaInfo();

    void sortKeyPoints(std::vector<cv::KeyPoint>& keypoints, cv::InputOutputArray _descriptors = cv::noArray());

#ifdef HAVE_CUDA
    #define CV_PERF_TEST_CUDA_MAIN(modulename) \
        int main(int argc, char **argv)\
        {\
            const char * impls[] = { "cuda", "plain" };\
            CV_PERF_TEST_MAIN_INTERNALS(modulename, impls, perf::printCudaInfo())\
        }
#else
    #define CV_PERF_TEST_CUDA_MAIN(modulename) \
        int main(int argc, char **argv)\
        {\
            const char * plain_only[] = { "plain" };\
            CV_PERF_TEST_MAIN_INTERNALS(modulename, plain_only)\
        }
#endif
}

#endif // OPENCV_CUDA_PERF_UTILITY_HPP
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

- **CvtColorInfo**: A class/struct defined in this file

### Functions and Methods

- **HAVE_CUDA()**: A function/method defined in this file
- **perf()**: A function/method defined in this file
- **OPENCV_CUDA_PERF_UTILITY_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts.hpp`
- `opencv2/ts/ts_perf.hpp`

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

