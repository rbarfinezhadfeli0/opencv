# Documentation for `modules/features2d/test/ocl/test_brute_force_matcher.cpp`

## File Metadata

- **Full Path**: `modules/features2d/test/ocl/test_brute_force_matcher.cpp`
- **File Name**: `test_brute_force_matcher.cpp`
- **File Size**: 7,095 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/features2d/test/ocl/test_brute_force_matcher.cpp](../../../../modules/features2d/test/ocl/test_brute_force_matcher.cpp)

## Purpose and Role

This file is located in the `modules/features2d/test/ocl` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2010-2012, Institute Of Software Chinese Academy Of Science, all rights reserved.
// Copyright (C) 2010-2012, Advanced Micro Devices, Inc., all rights reserved.
// Copyright (C) 2010-2012, Multicoreware, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// @Authors
//    Niko Li, newlife20080214@gmail.com
//    Jia Haipeng, jiahaipeng95@gmail.com
//    Zero Lin, Zero.Lin@amd.com
//    Zhang Ying, zhangying913@gmail.com
//    Yao Wang, bitwangyaoyao@gmail.com
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
#include "cvconfig.h"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {
PARAM_TEST_CASE(BruteForceMatcher, int, int)
{
    int distType;
    int dim;

    int queryDescCount;
    int countFactor;

    Mat query, train;
    UMat uquery, utrain;

    virtual void SetUp()
    {
        distType = GET_PARAM(0);
        dim = GET_PARAM(1);

        queryDescCount = 300; // must be even number because we split train data in some cases in two
        countFactor = 4; // do not change it

        cv::Mat queryBuf, trainBuf;

        // Generate query descriptors randomly.
        // Descriptor vector elements are integer values.
        queryBuf.create(queryDescCount, dim, CV_32SC1);
        rng.fill(queryBuf, cv::RNG::UNIFORM, cv::Scalar::all(0), cv::Scalar::all(3));
        queryBuf.convertTo(queryBuf, CV_32FC1);

        // Generate train descriptors as follows:
        // copy each query descriptor to train set countFactor times
        // and perturb some one element of the copied descriptors in
        // in ascending order. General boundaries of the perturbation
        // are (0.f, 1.f).
        trainBuf.create(queryDescCount * countFactor, dim, CV_32FC1);
        float step = 1.f / countFactor;
        for (int qIdx = 0; qIdx < queryDescCount; qIdx++)
        {
            cv::Mat queryDescriptor = queryBuf.row(qIdx);
            for (int c = 0; c < countFactor; c++)
            {
                int tIdx = qIdx * countFactor + c;
                cv::Mat trainDescriptor = trainBuf.row(tIdx);
                queryDescriptor.copyTo(trainDescriptor);
                int elem = rng(dim);
                float diff = rng.uniform(step * c, step * (c + 1));
                trainDescriptor.at<float>(0, elem) += diff;
            }
        }

        queryBuf.convertTo(query, CV_32F);
        trainBuf.convertTo(train, CV_32F);
        query.copyTo(uquery);
        train.copyTo(utrain);
    }
};

#ifdef __ANDROID__
OCL_TEST_P(BruteForceMatcher, DISABLED_Match_Single)
#else
OCL_TEST_P(BruteForceMatcher, Match_Single)
#endif
{
    BFMatcher matcher(distType);

    std::vector<cv::DMatch> matches;
    matcher.match(uquery, utrain,  matches);

    ASSERT_EQ(static_cast<size_t>(queryDescCount), matches.size());

    int badCount = 0;
    for (size_t i = 0; i < matches.size(); i++)
    {
        cv::DMatch match = matches[i];
        if ((match.queryIdx != (int)i) || (match.trainIdx != (int)i * countFactor) || (match.imgIdx != 0))
            badCount++;
    }

    ASSERT_EQ(0, badCount);
}

#ifdef __ANDROID__
OCL_TEST_P(BruteForceMatcher, DISABLED_KnnMatch_2_Single)
#else
OCL_TEST_P(BruteForceMatcher, KnnMatch_2_Single)
#endif
{
    const int knn = 2;

    BFMatcher matcher(distType);

    std::vector< std::vector<cv::DMatch> > matches;
    matcher.knnMatch(uquery, utrain, matches, knn);

    ASSERT_EQ(static_cast<size_t>(queryDescCount), matches.size());

    int badCount = 0;
    for (size_t i = 0; i < matches.size(); i++)
    {
        if ((int)matches[i].size() != knn)
            badCount++;
        else
        {
            int localBadCount = 0;
            for (int k = 0; k < knn; k++)
            {
                cv::DMatch match = matches[i][k];
                if ((match.queryIdx != (int)i) || (match.trainIdx != (int)i * countFactor + k) || (match.imgIdx != 0))
                    localBadCount++;
            }
            badCount += localBadCount > 0 ? 1 : 0;
        }
    }

    ASSERT_EQ(0, badCount);
}

#ifdef __ANDROID__
OCL_TEST_P(BruteForceMatcher, DISABLED_RadiusMatch_Single)
#else
OCL_TEST_P(BruteForceMatcher, RadiusMatch_Single)
#endif
{
    float radius = 1.f / countFactor;

    BFMatcher matcher(distType);

    std::vector< std::vector<cv::DMatch> > matches;
    matcher.radiusMatch(uquery, utrain, matches, radius);

    ASSERT_EQ(static_cast<size_t>(queryDescCount), matches.size());

    int badCount = 0;
    for (size_t i = 0; i < matches.size(); i++)
    {
        if ((int)matches[i].size() != 1)
        {
            badCount++;
        }
        else
        {
            cv::DMatch match = matches[i][0];
            if ((match.queryIdx != (int)i) || (match.trainIdx != (int)i * countFactor) || (match.imgIdx != 0))
                badCount++;
        }
    }

    ASSERT_EQ(0, badCount);
}

OCL_INSTANTIATE_TEST_CASE_P(Matcher, BruteForceMatcher, Combine( Values((int)NORM_L1, (int)NORM_L2),
                                                                Values(57, 64, 83, 128, 179, 256, 304) ) );

}//ocl
}//cvtest

#endif //HAVE_OPENCL
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

- **__ANDROID__()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cvconfig.h`
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

