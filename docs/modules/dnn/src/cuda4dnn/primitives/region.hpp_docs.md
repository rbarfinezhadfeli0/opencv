# Documentation for `modules/dnn/src/cuda4dnn/primitives/region.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/primitives/region.hpp`
- **File Name**: `region.hpp`
- **File Size**: 6,456 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/primitives/region.hpp](../../../../../modules/dnn/src/cuda4dnn/primitives/region.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/primitives` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_REGION_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_REGION_HPP

#include "../../op_cuda.hpp"

#include "../csl/stream.hpp"
#include "../csl/cudnn.hpp"
#include "../csl/tensor_ops.hpp"

#include "../kernels/region.hpp"

#include "../../nms.inl.hpp"

#include <opencv2/core.hpp>

#include <cstddef>
#include <utility>
#include <vector>

namespace cv { namespace dnn { namespace cuda4dnn {

    enum class SquashMethod {
        SOFTMAX,
        SIGMOID
    };

    template <class T>
    struct RegionConfiguration {
        /* The image is divided into (H, W) cells.
         *
         * Each cell is interested in exactly one object and predicts `boxes_per_cell` bounding boxes
         * for that object.
         *
         * Each bounding box contains:
         * - 4 box coordinates
         * - objectness confidence score
         * - `classes` number of class scores
         *
         * The object score is reduced to a probability using sigmoid and the class scores are reduced to
         * probabilities by either applying sigmoid or softmax (which is a configuration option).
         *
         * object_prob = sigmoid(object_score)
         * conditional_class_prob = sigmoid, softmax across all classes
         *
         * actual class probability = conditional_class_prob * object_prob
         */
        std::size_t classes, boxes_per_cell;
        std::size_t width_norm, height_norm;
        T scale_x_y;

        /* method for reducing class scores to probabilities */
        SquashMethod squash_method;

        /* prob cutoffs below which the prediction is nulled */
        T object_prob_cutoff;
        T class_prob_cutoff;

        T nms_iou_threshold;
        bool new_coords;
    };

    template <class T>
    class RegionOp final : public CUDABackendNode {
    public:
        using wrapper_type = GetCUDABackendWrapperType<T>;

        template <class V>
        RegionOp(csl::Stream stream_, const cv::Mat& bias, const RegionConfiguration<V>& config)
            : stream(std::move(stream_))
        {
            biasTensor = csl::makeTensorHeader<T>(bias);
            csl::copyMatToTensor<T>(bias, biasTensor, stream);

            classes = config.classes;
            boxes_per_cell = config.boxes_per_cell;

            width_norm = config.width_norm;
            height_norm = config.height_norm;

            scale_x_y = config.scale_x_y;

            squash_type = config.squash_method;
            object_prob_cutoff = config.object_prob_cutoff;
            class_prob_cutoff = config.class_prob_cutoff;

            nms_iou_threshold = config.nms_iou_threshold;
            new_coords = config.new_coords;
        }

        void forward(
            const std::vector<cv::Ptr<BackendWrapper>>& inputs,
            const std::vector<cv::Ptr<BackendWrapper>>& outputs,
            csl::Workspace& workspace) override
        {
            CV_Assert(outputs.size() == 1);

            auto input_wrapper = inputs[0].dynamicCast<wrapper_type>();
            auto input = input_wrapper->getView();

            auto output_wrapper = outputs[0].dynamicCast<wrapper_type>();
            auto output = output_wrapper->getSpan();

            auto rows = input.get_axis_size(1);
            auto cols = input.get_axis_size(2);

            auto cell_box_size = classes + 4 + 1;

            /* we squash class scores into probabilities using softmax or sigmoid */
            bool if_true_sigmoid_else_softmax = (squash_type == SquashMethod::SIGMOID);

            kernels::region<T>(stream, output, input, biasTensor,
                object_prob_cutoff, class_prob_cutoff,
                boxes_per_cell, cell_box_size,
                rows, cols, scale_x_y,
                height_norm, width_norm,
                if_true_sigmoid_else_softmax,
                new_coords
            );

            if (nms_iou_threshold > static_cast<T>(0.0f)) {
                auto output_mat = output_wrapper->getMutableHostMat();
                CV_Assert(output_mat.type() == CV_32F);
                for (int i = 0; i < input.get_axis_size(0); i++) {
                    auto sample_size = rows * cols * boxes_per_cell * cell_box_size;
                    do_nms_sort(reinterpret_cast<float*>(output_mat.data) + i * sample_size, rows * cols * boxes_per_cell, class_prob_cutoff, nms_iou_threshold);
                }
            }
        }

    private:
        void do_nms_sort(float *detections, int total, float score_thresh, float nms_thresh)
        {
            std::vector<Rect2d> boxes(total);
            std::vector<float> scores(total);

            for (int i = 0; i < total; ++i)
            {
                Rect2d &b = boxes[i];
                int box_index = i * (classes + 4 + 1);
                b.width = detections[box_index + 2];
                b.height = detections[box_index + 3];
                b.x = detections[box_index + 0] - b.width / 2;
                b.y = detections[box_index + 1] - b.height / 2;
            }

            std::vector<int> indices;
            for (int k = 0; k < classes; ++k)
            {
                for (int i = 0; i < total; ++i)
                {
                    int box_index = i * (classes + 4 + 1);
                    int class_index = box_index + 5;
                    scores[i] = detections[class_index + k];
                    detections[class_index + k] = 0;
                }
                NMSBoxes(boxes, scores, score_thresh, nms_thresh, indices);
                for (int i = 0, n = indices.size(); i < n; ++i)
                {
                    int box_index = indices[i] * (classes + 4 + 1);
                    int class_index = box_index + 5;
                    detections[class_index + k] = scores[indices[i]];
                }
            }
        }

    private:
        csl::Stream stream;

        csl::Tensor<T> biasTensor;
        std::size_t classes, boxes_per_cell;
        std::size_t width_norm, height_norm;
        T scale_x_y;

        SquashMethod squash_type;
        T object_prob_cutoff, class_prob_cutoff;

        T nms_iou_threshold;
        bool new_coords;
    };

}}} /* namespace cv::dnn::cuda4dnn */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_REGION_HPP */
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

- **T**: A class/struct defined in this file
- **RegionOp**: A class/struct defined in this file
- **V**: A class/struct defined in this file
- **scores**: A class/struct defined in this file
- **SquashMethod**: A class/struct defined in this file
- **probability**: A class/struct defined in this file
- **RegionConfiguration**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_PRIMITIVES_REGION_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `../../op_cuda.hpp`
- `../csl/cudnn.hpp`
- `../kernels/region.hpp`
- `../csl/stream.hpp`
- `vector`
- `../csl/tensor_ops.hpp`
- `cstddef`
- `../../nms.inl.hpp`
- `opencv2/core.hpp`


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

