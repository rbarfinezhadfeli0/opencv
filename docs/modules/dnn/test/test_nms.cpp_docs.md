# Documentation for `modules/dnn/test/test_nms.cpp`

## File Metadata

- **Full Path**: `modules/dnn/test/test_nms.cpp`
- **File Name**: `test_nms.cpp`
- **File Size**: 3,423 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/test/test_nms.cpp](../../../modules/dnn/test/test_nms.cpp)

## Purpose and Role

This file is located in the `modules/dnn/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "test_precomp.hpp"

namespace opencv_test { namespace {

TEST(NMS, Accuracy)
{
    //reference results obtained using tf.image.non_max_suppression with iou_threshold=0.5
    std::string dataPath = findDataFile("dnn/nms_reference.yml");
    FileStorage fs(dataPath, FileStorage::READ);

    std::vector<Rect> bboxes;
    std::vector<float> scores;
    std::vector<int> ref_indices;

    fs["boxes"] >> bboxes;
    fs["probs"] >> scores;
    fs["output"] >> ref_indices;

    const float nms_thresh = .5f;
    const float score_thresh = .01f;
    std::vector<int> indices;
    cv::dnn::NMSBoxes(bboxes, scores, score_thresh, nms_thresh, indices);

    ASSERT_EQ(ref_indices.size(), indices.size());

    std::sort(indices.begin(), indices.end());
    std::sort(ref_indices.begin(), ref_indices.end());

    for(size_t i = 0; i < indices.size(); i++)
        ASSERT_EQ(indices[i], ref_indices[i]);
}

TEST(BatchedNMS, Accuracy)
{
    //reference results obtained using tf.image.non_max_suppression with iou_threshold=0.5
    std::string dataPath = findDataFile("dnn/batched_nms_reference.yml");
    FileStorage fs(dataPath, FileStorage::READ);

    std::vector<Rect> bboxes;
    std::vector<float> scores;
    std::vector<int> idxs;
    std::vector<int> ref_indices;

    fs["boxes"] >> bboxes;
    fs["probs"] >> scores;
    fs["idxs"] >> idxs;
    fs["output"] >> ref_indices;

    const float nms_thresh = .5f;
    const float score_thresh = .05f;
    std::vector<int> indices;
    cv::dnn::NMSBoxesBatched(bboxes, scores, idxs, score_thresh, nms_thresh, indices);

    ASSERT_EQ(ref_indices.size(), indices.size());

    std::sort(indices.begin(), indices.end());
    std::sort(ref_indices.begin(), ref_indices.end());

    for(size_t i = 0; i < indices.size(); i++)
        ASSERT_EQ(indices[i], ref_indices[i]);
}

TEST(SoftNMS, Accuracy)
{
    //reference results are obtained using TF v2.7 tf.image.non_max_suppression_with_scores
    std::string dataPath = findDataFile("dnn/soft_nms_reference.yml");
    FileStorage fs(dataPath, FileStorage::READ);

    std::vector<Rect> bboxes;
    std::vector<float> scores;
    std::vector<int> ref_indices;
    std::vector<float> ref_updated_scores;

    fs["boxes"] >> bboxes;
    fs["probs"] >> scores;
    fs["indices"] >> ref_indices;
    fs["updated_scores"] >> ref_updated_scores;

    std::vector<float> updated_scores;
    const float score_thresh = .01f;
    const float nms_thresh = .5f;
    std::vector<int> indices;
    const size_t top_k = 0;
    const float sigma = 1.; // sigma in TF is being multiplied by 2, so 0.5 should be passed there
    cv::dnn::softNMSBoxes(bboxes, scores, updated_scores, score_thresh, nms_thresh, indices, top_k, sigma);

    ASSERT_EQ(ref_indices.size(), indices.size());
    for(size_t i = 0; i < indices.size(); i++)
    {
        ASSERT_EQ(indices[i], ref_indices[i]);
    }

    ASSERT_EQ(ref_updated_scores.size(), updated_scores.size());
    for(size_t i = 0; i < updated_scores.size(); i++)
    {
        EXPECT_NEAR(updated_scores[i], ref_updated_scores[i], 1e-7);
    }
}

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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `test_precomp.hpp`


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

