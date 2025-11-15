# Documentation for `modules/dnn/include/opencv2/dnn/utils/inference_engine.hpp`

## File Metadata

- **Full Path**: `modules/dnn/include/opencv2/dnn/utils/inference_engine.hpp`
- **File Name**: `inference_engine.hpp`
- **File Size**: 2,640 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/include/opencv2/dnn/utils/inference_engine.hpp](../../../../../../modules/dnn/include/opencv2/dnn/utils/inference_engine.hpp)

## Purpose and Role

This file is located in the `modules/dnn/include/opencv2/dnn/utils` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2019, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#ifndef OPENCV_DNN_UTILS_INF_ENGINE_HPP
#define OPENCV_DNN_UTILS_INF_ENGINE_HPP

#include "../dnn.hpp"

namespace cv { namespace dnn {
CV__DNN_INLINE_NS_BEGIN


/* Values for 'OPENCV_DNN_BACKEND_INFERENCE_ENGINE_TYPE' parameter */
/// @deprecated
#define CV_DNN_BACKEND_INFERENCE_ENGINE_NN_BUILDER_API     "NN_BUILDER"
/// @deprecated
#define CV_DNN_BACKEND_INFERENCE_ENGINE_NGRAPH             "NGRAPH"

/** @brief Returns Inference Engine internal backend API.
 *
 * See values of `CV_DNN_BACKEND_INFERENCE_ENGINE_*` macros.
 *
 * `OPENCV_DNN_BACKEND_INFERENCE_ENGINE_TYPE` runtime parameter (environment variable) is ignored since 4.6.0.
 *
 * @deprecated
 */
CV_EXPORTS_W cv::String getInferenceEngineBackendType();

/** @brief Specify Inference Engine internal backend API.
 *
 * See values of `CV_DNN_BACKEND_INFERENCE_ENGINE_*` macros.
 *
 * @returns previous value of internal backend API
 *
 * @deprecated
 */
CV_EXPORTS_W cv::String setInferenceEngineBackendType(const cv::String& newBackendType);


/** @brief Release a Myriad device (binded by OpenCV).
 *
 * Single Myriad device cannot be shared across multiple processes which uses
 * Inference Engine's Myriad plugin.
 */
CV_EXPORTS_W void resetMyriadDevice();


/* Values for 'OPENCV_DNN_IE_VPU_TYPE' parameter */
#define CV_DNN_INFERENCE_ENGINE_VPU_TYPE_UNSPECIFIED ""
/// Intel(R) Movidius(TM) Neural Compute Stick, NCS (USB 03e7:2150), Myriad2 (https://software.intel.com/en-us/movidius-ncs)
#define CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_2 "Myriad2"
/// Intel(R) Neural Compute Stick 2, NCS2 (USB 03e7:2485), MyriadX (https://software.intel.com/ru-ru/neural-compute-stick)
#define CV_DNN_INFERENCE_ENGINE_VPU_TYPE_MYRIAD_X "MyriadX"
#define CV_DNN_INFERENCE_ENGINE_CPU_TYPE_ARM_COMPUTE "ARM_COMPUTE"
#define CV_DNN_INFERENCE_ENGINE_CPU_TYPE_X86         "X86"


/** @brief Returns Inference Engine VPU type.
 *
 * See values of `CV_DNN_INFERENCE_ENGINE_VPU_TYPE_*` macros.
 */
CV_EXPORTS_W cv::String getInferenceEngineVPUType();

/** @brief Returns Inference Engine CPU type.
 *
 * Specify OpenVINO plugin: CPU or ARM.
 */
CV_EXPORTS_W cv::String getInferenceEngineCPUType();

/** @brief Release a HDDL plugin.
 */
CV_EXPORTS_W void releaseHDDLPlugin();


CV__DNN_INLINE_NS_END
}} // namespace

#endif // OPENCV_DNN_UTILS_INF_ENGINE_HPP
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

- **OPENCV_DNN_UTILS_INF_ENGINE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../dnn.hpp`


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

