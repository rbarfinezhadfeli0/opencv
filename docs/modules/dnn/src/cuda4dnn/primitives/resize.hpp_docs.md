# Documentation for `modules/dnn/src/cuda4dnn/primitives/resize.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/resize.hpp`
- **File Name**: `resize.hpp`
- **File Size**: 3,019 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/resize.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/resize.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_RESIZE_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_RESIZE_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"

#include "../kernels/resize.hpp"

#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn {

    enum class InterpolationType {
        NEAREST_NEIGHBOUR,
        BILINEAR
    };

    struct ResizeConfiguration {
        InterpolationType type;
        bool align_corners;
        bool half_pixel_centers;
    };

    template <class T>
    class ResizeOp final : public CUDABackendNode {
    public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        ResizeOp(csl::Stream stream_, const ResizeConfiguration& config)
            : stream(std::move(stream_))
        {
            type = config.type;
            align_corners = config.align_corners;
            half_pixel_centers = config.half_pixel_centers;
        }

        void forward(
            const std::vector<cv::Ptr<BackendWrapper>>& inputs,
            const std::vector<cv::Ptr<BackendWrapper>>& outputs,
            csl::Workspace& workspace) override
        {
            // sometimes the target shape is taken from the second input; we don't use it however
            CV_Assert((inputs.size() == 1 || inputs.size() == 2) && outputs.size() == 1);

            auto input_wrapper = inputs[0].dynamicCast<wrapper_type>();
            auto input = input_wrapper->getView();

            auto output_wrapper = outputs[0].dynamicCast<wrapper_type>();
            auto output = output_wrapper->getSpan();

            const auto compute_scale = [this](std::size_t input_size, std::size_t output_size) {
                return (align_corners && output_size > 1) ?
                            static_cast<float>(input_size - 1) / (output_size - 1) :
                            static_cast<float>(input_size) / output_size;
            };

            auto out_height = output.get_axis_size(-2), out_width = output.get_axis_size(-1);
            auto in_height = input.get_axis_size(-2), in_width = input.get_axis_size(-1);
            float scale_height = compute_scale(in_height, out_height),
                  scale_width = compute_scale(in_width, out_width);

            if (type == InterpolationType::NEAREST_NEIGHBOUR)
                kernels::resize_nn<T>(stream, output, input, scale_height, scale_width, align_corners, half_pixel_centers);
            else if (type == InterpolationType::BILINEAR)
                kernels::resize_bilinear<T>(stream, output, input, scale_height, scale_width, half_pixel_centers);
        }

    private:
        csl::Stream stream;
        InterpolationType type;
        bool align_corners, half_pixel_centers;
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_RESIZE_HPP */
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

- **ResizeOp**: A class/struct defined in this file
- **InterpolationType**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **ResizeConfiguration**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_RESIZE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../kernels/resize.hpp`
- `../csl/stream.hpp`

**Python Imports:**
- `the`


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

