# Documentation for `modules/dnn/src/layers/cpu_kernels/fast_norm.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/cpu_kernels/fast_norm.cpp`
- **File Name**: `fast_norm.cpp`
- **File Size**: 8,716 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/cpu_kernels/fast_norm.cpp](../../../../../modules/dnn/src/layers/cpu_kernels/fast_norm.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers/cpu_kernels` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../../precomp.hpp"
#include "fast_norm.hpp"

namespace cv { namespace dnn {

void fastNorm(const Mat &input, Mat &output, float epsilon, size_t normalized_axis, bool normalize_variance) {
    const auto input_shape = shape(input);
    CV_CheckLT(normalized_axis, input_shape.size(), "fastNorm: axis out of range");

    size_t loops = static_cast<size_t>(total(input_shape, 0, static_cast<int>(normalized_axis))),
           norm_size = static_cast<size_t>(total(input_shape, static_cast<int>(normalized_axis)));
    float inv_norm_size = 1.0 / norm_size;

    auto fn = [&](const Range &r) {
        const auto *input_data = input.ptr<const float>();
        auto *output_data = output.ptr<float>();
        for (int i = r.start; i < r.end; i++) {
            const auto *x = input_data + norm_size * i;
            auto *y = output_data + norm_size * i;

            float mean = 0.f, mean_square = 0.f;
            for (int j = 0; j < norm_size; j++) {
                float v = x[j];
                mean += v;
                mean_square += v * v;
            }

            mean *= inv_norm_size;
            mean_square = std::sqrt(std::max(0.f, mean_square * inv_norm_size - mean * mean) + epsilon);
            float inv_stdev = normalize_variance ? 1.f / mean_square : 1.f;

            for (size_t j = 0; j < norm_size; j++) {
                y[j] = (x[j] - mean) * inv_stdev;
            }
        }
    };
    double nstripes = loops * norm_size * (1 / 1024.0);
    parallel_for_(Range(0, loops), fn, nstripes);
}

void fastNorm(const Mat &input, const Mat &scale, Mat &output, float epsilon, size_t normalized_axis) {
    const auto input_shape = shape(input);
    CV_CheckLT(normalized_axis, input_shape.size(), "fastNorm: axis out of range");

    size_t loops = static_cast<size_t>(total(input_shape, 0, static_cast<int>(normalized_axis))),
           norm_size = static_cast<size_t>(total(input_shape, static_cast<int>(normalized_axis)));
    float inv_norm_size = 1.0 / norm_size;

    auto fn = [&](const Range &r) {
        const auto *input_data = input.ptr<const float>();
        const auto *scale_data = scale.ptr<const float>();
        auto *output_data = output.ptr<float>();
        for (int i = r.start; i < r.end; i++) {
            const auto *x = input_data + norm_size * i;
            auto *y = output_data + norm_size * i;

            float mean = 0.f, mean_square = 0.f;
            for (int j = 0; j < norm_size; j++) {
                float v = x[j];
                mean += v;
                mean_square += v * v;
            }

            mean *= inv_norm_size;
            mean_square = std::sqrt(std::max(0.f, mean_square * inv_norm_size - mean * mean) + epsilon);
            float inv_stdev = 1.f / mean_square;

            for (size_t j = 0; j < norm_size; j++) {
                y[j] = scale_data[j] * (x[j] - mean) * inv_stdev;
            }
        }
    };
    double nstripes = loops * norm_size * (1 / 1024.0);
    parallel_for_(Range(0, loops), fn, nstripes);
}

void fastNorm(const Mat &input, const Mat &scale, const Mat &bias, Mat &output, float epsilon, size_t normalized_axis) {
    const auto input_shape = shape(input);
    CV_CheckLT(normalized_axis, input_shape.size(), "fastNorm: axis out of range");
    CV_CheckEQ(scale.total(), bias.total(), "fastNorm: scale and bias should have the same shape");

    size_t loops = static_cast<size_t>(total(input_shape, 0, static_cast<int>(normalized_axis))),
           norm_size = static_cast<size_t>(total(input_shape, static_cast<int>(normalized_axis)));
    float inv_norm_size = 1.0 / norm_size;

    auto fn = [&](const Range &r) {
        const auto *input_data = input.ptr<const float>();
        const auto *scale_data = scale.ptr<const float>();
        const auto *bias_data = bias.ptr<const float>();
        auto *output_data = output.ptr<float>();
        for (int i = r.start; i < r.end; i++) {
            const auto *x = input_data + norm_size * i;
            auto *y = output_data + norm_size * i;

            float mean = 0.f, mean_square = 0.f;
            for (int j = 0; j < norm_size; j++) {
                float v = x[j];
                mean += v;
                mean_square += v * v;
            }

            mean *= inv_norm_size;
            mean_square = std::sqrt(std::max(0.f, mean_square * inv_norm_size - mean * mean) + epsilon);
            float inv_stdev = 1.f / mean_square;

            for (size_t j = 0; j < norm_size; j++) {
                y[j] = scale_data[j] * (x[j] - mean) * inv_stdev + bias_data[j];
            }
        }
    };
    double nstripes = loops * norm_size * (1 / 1024.0);
    parallel_for_(Range(0, loops), fn, nstripes);
}

void fastNormChannel(const Mat &input, const Mat &scale, const Mat &bias, Mat &output, float epsilon) {
    const auto input_shape = shape(input);
    size_t N = input_shape[0], C = input_shape[1];
    CV_CheckEQ(scale.total(), bias.total(), "fastNormChannel: scale and bias should have the same shape");
    CV_CheckEQ(scale.total(), C, "fastNormChannel: scale should be a 1d tensor and match the channel of input");
    CV_CheckGE(input.dims, 3, "fastNormChannel: input dimension >= 3");

    size_t loops = N * C,
           norm_size = static_cast<size_t>(total(input_shape, 2));
    float inv_norm_size = 1.0 / norm_size;

    auto fn = [&](const Range &r) {
        const auto *input_data = input.ptr<const float>();
        const auto *scale_data = scale.ptr<const float>();
        const auto *bias_data = bias.ptr<const float>();
        auto *output_data = output.ptr<float>();
        for (int i = r.start; i < r.end; i++) {
            const auto *x = input_data + norm_size * i;
            auto *y = output_data + norm_size * i;

            float mean = 0.f, mean_square = 0.f;
            for (int j = 0; j < norm_size; j++) {
                float v = x[j];
                mean += v;
                mean_square += v * v;
            }

            mean *= inv_norm_size;
            mean_square = std::sqrt(std::max(0.f, mean_square * inv_norm_size - mean * mean) + epsilon);
            float inv_stdev = 1.f / mean_square;

            size_t c = i % C;
            float s = scale_data[c] * inv_stdev, b = bias_data[c];
            for (size_t j = 0; j < norm_size; j++) {
                y[j] = s * (x[j] - mean) + b;
            }
        }
    };
    double nstripes = loops * norm_size * (1 / 1024.0);
    parallel_for_(Range(0, loops), fn, nstripes);
}

void fastNormGroup(const Mat &input, const Mat &scale, const Mat &bias, Mat &output, float epsilon, size_t num_groups) {
    const auto input_shape = shape(input);
    size_t N = input_shape[0], C = input_shape[1];
    CV_CheckEQ(scale.total(), bias.total(), "fastNormGroup: scale and bias should have the same shape");
    CV_CheckEQ(scale.total(), C, "fastNormGroup: scale should be a 1d tensor and match the channel of input");
    CV_CheckGE(input.dims, 3, "fastNormGroup: input dimension >= 3");

    size_t channels_per_group = C / num_groups;
    size_t loops = N * num_groups;
    size_t norm_size = static_cast<size_t>(total(input_shape, 2) * channels_per_group);
    size_t step = norm_size / channels_per_group;
    float inv_norm_size = 1.0 / norm_size;

    auto fn = [&](const Range &r) {
        const auto *input_data = input.ptr<const float>();
        const auto *scale_data = scale.ptr<const float>();
        const auto *bias_data = bias.ptr<const float>();
        auto *output_data = output.ptr<float>();

        for (int i = r.start; i < r.end; i++) {
            const auto *x = input_data + norm_size * i;
            auto *y = output_data + norm_size * i;

            float mean = 0.f, mean_square = 0.f;
            for (int j = 0; j < norm_size; j++) {
                float v = x[j];
                mean += v;
                mean_square += v * v;
            }

            mean *= inv_norm_size;
            mean_square = std::sqrt(std::max(0.f, mean_square * inv_norm_size - mean * mean) + epsilon);
            float inv_stdev = 1.f / mean_square;

            size_t group_idx = i % num_groups * channels_per_group;
            for (size_t j = 0; j < norm_size; j++) {
                size_t c = group_idx + (j / step);
                float s = scale_data[c] * inv_stdev, b = bias_data[c];
                y[j] = s * (x[j] - mean) + b;
            }
        }
    };

    double nstripes = loops * norm_size * (1 / 1024.0);
    parallel_for_(Range(0, loops), fn, nstripes);
}

}} // cv::dnn
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../precomp.hpp`
- `fast_norm.hpp`


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

