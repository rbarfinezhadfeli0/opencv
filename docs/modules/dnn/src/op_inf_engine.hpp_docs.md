# Documentation for `modules/dnn/src/op_inf_engine.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_inf_engine.hpp`
- **File Name**: `op_inf_engine.hpp`
- **File Size**: 2,950 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/op_inf_engine.hpp](../../../modules/dnn/src/op_inf_engine.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2019, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#ifndef __OPENCV_DNN_OP_INF_ENGINE_HPP__
#define __OPENCV_DNN_OP_INF_ENGINE_HPP__

#include "opencv2/core/cvdef.h"
#include "opencv2/core/cvstd.hpp"
#include "opencv2/dnn.hpp"

#include "opencv2/core/async.hpp"
#include "opencv2/core/detail/async_promise.hpp"

#include "opencv2/dnn/utils/inference_engine.hpp"

#ifdef HAVE_INF_ENGINE

#define INF_ENGINE_RELEASE_2022_1 2022010000
#define INF_ENGINE_RELEASE_2023_0 2023000000
#define INF_ENGINE_RELEASE_2024_0 2024000000

#ifndef INF_ENGINE_RELEASE
#warning("IE version have not been provided via command-line. Using 2022.1 by default")
#define INF_ENGINE_RELEASE INF_ENGINE_RELEASE_2022_1
#endif

#define INF_ENGINE_VER_MAJOR_GT(ver) (((INF_ENGINE_RELEASE) / 10000) > ((ver) / 10000))
#define INF_ENGINE_VER_MAJOR_GE(ver) (((INF_ENGINE_RELEASE) / 10000) >= ((ver) / 10000))
#define INF_ENGINE_VER_MAJOR_LT(ver) (((INF_ENGINE_RELEASE) / 10000) < ((ver) / 10000))
#define INF_ENGINE_VER_MAJOR_LE(ver) (((INF_ENGINE_RELEASE) / 10000) <= ((ver) / 10000))
#define INF_ENGINE_VER_MAJOR_EQ(ver) (((INF_ENGINE_RELEASE) / 10000) == ((ver) / 10000))

#if defined(__GNUC__) && __GNUC__ >= 5
//#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wsuggest-override"
#endif

#include <openvino/openvino.hpp>
#include <openvino/pass/serialize.hpp>
#include <openvino/pass/convert_fp32_to_fp16.hpp>

#if defined(__GNUC__) && __GNUC__ >= 5
//#pragma GCC diagnostic pop
#endif

#endif  // HAVE_INF_ENGINE

#define CV_ERROR_DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_2019 do { CV_Error(Error::StsNotImplemented, "This OpenCV version is built without Inference Engine NN Builder API support (legacy API is not supported anymore)"); } while (0)

namespace cv { namespace dnn {

CV__DNN_INLINE_NS_BEGIN
namespace openvino {

// TODO: use std::string as parameter
bool checkTarget(Target target);

}  // namespace openvino
CV__DNN_INLINE_NS_END

#ifdef HAVE_INF_ENGINE

Backend& getInferenceEngineBackendTypeParam();

Mat infEngineBlobToMat(const ov::Tensor& blob);

void infEngineBlobsToMats(const ov::TensorVector& blobs,
                          std::vector<Mat>& mats);

CV__DNN_INLINE_NS_BEGIN

void switchToOpenVINOBackend(Net& net);

bool isMyriadX();

bool isArmComputePlugin();

CV__DNN_INLINE_NS_END

ov::Core& getCore(const std::string& id);

template<typename T = size_t>
static inline std::vector<T> getShape(const Mat& mat)
{
    std::vector<T> result(mat.dims);
    for (int i = 0; i < mat.dims; i++)
        result[i] = (T)mat.size[i];
    return result;
}

#endif  // HAVE_INF_ENGINE

}}  // namespace dnn, namespace cv

#endif  // __OPENCV_DNN_OP_INF_ENGINE_HPP__
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

- **INF_ENGINE_RELEASE()**: A function/method defined in this file
- **HAVE_INF_ENGINE()**: A function/method defined in this file
- **__OPENCV_DNN_OP_INF_ENGINE_HPP__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/async.hpp`
- `openvino/openvino.hpp`
- `opencv2/core/detail/async_promise.hpp`
- `opencv2/dnn/utils/inference_engine.hpp`
- `openvino/pass/convert_fp32_to_fp16.hpp`
- `openvino/pass/serialize.hpp`
- `opencv2/core/cvstd.hpp`
- `opencv2/dnn.hpp`
- `opencv2/core/cvdef.h`


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

