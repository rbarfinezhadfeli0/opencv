# Documentation for `modules/dnn/test/test_googlenet.cpp`

## File Metadata

- **Full Path**: `modules/dnn/test/test_googlenet.cpp`
- **File Name**: `test_googlenet.cpp`
- **File Size**: 6,411 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/test/test_googlenet.cpp](../../../modules/dnn/test/test_googlenet.cpp)

## Purpose and Role

This file is located in the `modules/dnn/test` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
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

#include "test_precomp.hpp"
#include "npy_blob.hpp"
#include <opencv2/core/ocl.hpp>
#include <opencv2/ts/ocl_test.hpp>

namespace opencv_test { namespace {

template<typename TString>
static std::string _tf(TString filename)
{
    return (getOpenCVExtraDir() + "/dnn/") + filename;
}

typedef testing::TestWithParam<Target> Reproducibility_GoogLeNet;
TEST_P(Reproducibility_GoogLeNet, Batching)
{
    const int targetId = GetParam();
    if (targetId == DNN_TARGET_OPENCL_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_OPENCL_FP16);
    if (targetId == DNN_TARGET_CPU_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_CPU_FP16);
    Net net = readNetFromCaffe(findDataFile("dnn/bvlc_googlenet.prototxt"),
                               findDataFile("dnn/bvlc_googlenet.caffemodel", false));
    net.setPreferableBackend(DNN_BACKEND_OPENCV);
    net.setPreferableTarget(targetId);

    if (targetId == DNN_TARGET_OPENCL)
    {
        // Initialize network for a single image in the batch but test with batch size=2.
        Mat inp = Mat(224, 224, CV_8UC3);
        randu(inp, -1, 1);
        net.setInput(blobFromImage(inp));
        net.forward();
    }

    std::vector<Mat> inpMats;
    inpMats.push_back( imread(_tf("googlenet_0.png")) );
    inpMats.push_back( imread(_tf("googlenet_1.png")) );
    ASSERT_TRUE(!inpMats[0].empty() && !inpMats[1].empty());

    net.setInput(blobFromImages(inpMats, 1.0f, Size(), Scalar(), false), "data");
    Mat out = net.forward("prob");

    Mat ref = blobFromNPY(_tf("googlenet_prob.npy"));
    normAssert(out, ref);
}

TEST_P(Reproducibility_GoogLeNet, IntermediateBlobs)
{
    const int targetId = GetParam();
    if (targetId == DNN_TARGET_OPENCL_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_OPENCL_FP16);
    if (targetId == DNN_TARGET_CPU_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_CPU_FP16);
    Net net = readNetFromCaffe(findDataFile("dnn/bvlc_googlenet.prototxt"),
                               findDataFile("dnn/bvlc_googlenet.caffemodel", false));
    net.setPreferableBackend(DNN_BACKEND_OPENCV);
    net.setPreferableTarget(targetId);

    std::vector<String> blobsNames;
    blobsNames.push_back("conv1/7x7_s2");
    blobsNames.push_back("conv1/relu_7x7");
    blobsNames.push_back("inception_4c/1x1");
    blobsNames.push_back("inception_4c/relu_1x1");
    std::vector<Mat> outs;
    Mat in = blobFromImage(imread(_tf("googlenet_0.png")), 1.0f, Size(), Scalar(), false);
    net.setInput(in, "data");
    net.forward(outs, blobsNames);
    CV_Assert(outs.size() == blobsNames.size());

    for (size_t i = 0; i < blobsNames.size(); i++)
    {
        std::string filename = blobsNames[i];
        std::replace( filename.begin(), filename.end(), '/', '#');
        Mat ref = blobFromNPY(_tf("googlenet_" + filename + ".npy"));

        normAssert(outs[i], ref, "", 1E-4, 1E-2);
    }
}

TEST_P(Reproducibility_GoogLeNet, SeveralCalls)
{
    const int targetId = GetParam();
    if (targetId == DNN_TARGET_OPENCL_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_OPENCL_FP16);
    if (targetId == DNN_TARGET_CPU_FP16)
        applyTestTag(CV_TEST_TAG_DNN_SKIP_CPU_FP16);
    Net net = readNetFromCaffe(findDataFile("dnn/bvlc_googlenet.prototxt"),
                               findDataFile("dnn/bvlc_googlenet.caffemodel", false));
    net.setPreferableBackend(DNN_BACKEND_OPENCV);
    net.setPreferableTarget(targetId);

    std::vector<Mat> inpMats;
    inpMats.push_back( imread(_tf("googlenet_0.png")) );
    inpMats.push_back( imread(_tf("googlenet_1.png")) );
    ASSERT_TRUE(!inpMats[0].empty() && !inpMats[1].empty());

    net.setInput(blobFromImages(inpMats, 1.0f, Size(), Scalar(), false), "data");
    Mat out = net.forward();

    Mat ref = blobFromNPY(_tf("googlenet_prob.npy"));
    normAssert(out, ref);

    std::vector<String> blobsNames;
    blobsNames.push_back("conv1/7x7_s2");
    std::vector<Mat> outs;
    Mat in = blobFromImage(inpMats[0], 1.0f, Size(), Scalar(), false);
    net.setInput(in, "data");
    net.forward(outs, blobsNames);
    CV_Assert(outs.size() == blobsNames.size());

    ref = blobFromNPY(_tf("googlenet_conv1#7x7_s2.npy"));

    normAssert(outs[0], ref, "", 1E-4, 1E-2);
}

INSTANTIATE_TEST_CASE_P(/**/, Reproducibility_GoogLeNet,
    testing::ValuesIn(getAvailableTargets(DNN_BACKEND_OPENCV)));

}} // namespace
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

- **testing()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `npy_blob.hpp`
- `opencv2/ts/ocl_test.hpp`
- `opencv2/core/ocl.hpp`
- `test_precomp.hpp`

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

