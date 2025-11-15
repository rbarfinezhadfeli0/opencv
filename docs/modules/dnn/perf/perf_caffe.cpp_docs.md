# Documentation for `modules/dnn/perf/perf_caffe.cpp`

## File Metadata

- **Full Path**: `modules/dnn/perf/perf_caffe.cpp`
- **File Name**: `perf_caffe.cpp`
- **File Size**: 3,484 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/perf/perf_caffe.cpp](../../../modules/dnn/perf/perf_caffe.cpp)

## Purpose and Role

This file is located in the `modules/dnn/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

// Recommends run this performance test via
// ./bin/opencv_perf_dnn 2> /dev/null | grep "PERFSTAT" -A 3
// because whole output includes Caffe's logs.
//
// Note: Be sure that interesting version of Caffe was linked.
// Note: There is an impact on Halide performance. Comment this tests if you
//       want to run the last one.
//
// How to build Intel-Caffe with MKLDNN backend
// ============================================
// mkdir build && cd build
// cmake -DCMAKE_BUILD_TYPE=Release \
//       -DUSE_MKLDNN_AS_DEFAULT_ENGINE=ON \
//       -DUSE_MKL2017_AS_DEFAULT_ENGINE=OFF \
//       -DCPU_ONLY=ON \
//       -DCMAKE_INSTALL_PREFIX=/usr/local .. && make -j8
// sudo make install
//
// In case of problems with cublas_v2.h at include/caffe/util/device_alternate.hpp: add line
// #define CPU_ONLY
// before the first line
// #ifdef CPU_ONLY  // CPU-only Caffe.

#if defined(HAVE_CAFFE) || defined(HAVE_CLCAFFE)

#include "perf_precomp.hpp"
#include <iostream>
#include <caffe/caffe.hpp>

namespace opencv_test {

static caffe::Net<float>* initNet(std::string proto, std::string weights)
{
    proto = findDataFile(proto);
    weights = findDataFile(weights, false);

#ifdef HAVE_CLCAFFE
    caffe::Caffe::set_mode(caffe::Caffe::GPU);
    caffe::Caffe::SetDevice(0);

    caffe::Net<float>* net =
        new caffe::Net<float>(proto, caffe::TEST, caffe::Caffe::GetDefaultDevice());
#else
    caffe::Caffe::set_mode(caffe::Caffe::CPU);

    caffe::Net<float>* net = new caffe::Net<float>(proto, caffe::TEST);
#endif

    net->CopyTrainedLayersFrom(weights);

    caffe::Blob<float>* input = net->input_blobs()[0];

    CV_Assert(input->num() == 1);
    CV_Assert(input->channels() == 3);

    Mat inputMat(input->height(), input->width(), CV_32FC3, (char*)input->cpu_data());
    randu(inputMat, 0.0f, 1.0f);

    net->Forward();
    return net;
}

PERF_TEST(AlexNet_caffe, CaffePerfTest)
{
    caffe::Net<float>* net = initNet("dnn/bvlc_alexnet.prototxt",
                                     "dnn/bvlc_alexnet.caffemodel");
    TEST_CYCLE() net->Forward();
    SANITY_CHECK_NOTHING();
}

PERF_TEST(GoogLeNet_caffe, CaffePerfTest)
{
    caffe::Net<float>* net = initNet("dnn/bvlc_googlenet.prototxt",
                                     "dnn/bvlc_googlenet.caffemodel");
    TEST_CYCLE() net->Forward();
    SANITY_CHECK_NOTHING();
}

PERF_TEST(ResNet50_caffe, CaffePerfTest)
{
    caffe::Net<float>* net = initNet("dnn/ResNet-50-deploy.prototxt",
                                     "dnn/ResNet-50-model.caffemodel");
    TEST_CYCLE() net->Forward();
    SANITY_CHECK_NOTHING();
}

PERF_TEST(SqueezeNet_v1_1_caffe, CaffePerfTest)
{
    caffe::Net<float>* net = initNet("dnn/squeezenet_v1.1.prototxt",
                                     "dnn/squeezenet_v1.1.caffemodel");
    TEST_CYCLE() net->Forward();
    SANITY_CHECK_NOTHING();
}

PERF_TEST(MobileNet_SSD, CaffePerfTest)
{
    caffe::Net<float>* net = initNet("dnn/MobileNetSSD_deploy_19e3ec3.prototxt",
                                     "dnn/MobileNetSSD_deploy_19e3ec3.caffemodel");
    TEST_CYCLE() net->Forward();
    SANITY_CHECK_NOTHING();
}

} // namespace
#endif  // HAVE_CAFFE
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

- **CPU_ONLY()**: A function/method defined in this file
- **HAVE_CLCAFFE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iostream`
- `caffe/caffe.hpp`
- `perf_precomp.hpp`


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

