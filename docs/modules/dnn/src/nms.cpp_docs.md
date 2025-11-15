# Documentation for `modules/dnn/src/nms.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/nms.cpp`
- **File Name**: `nms.cpp`
- **File Size**: 7,236 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/nms.cpp](../../../modules/dnn/src/nms.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2017, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "precomp.hpp"
#include "nms.inl.hpp"

#include <opencv2/imgproc.hpp>

namespace cv { namespace dnn {
CV__DNN_INLINE_NS_BEGIN

template <typename T>
static inline float rectOverlap(const T& a, const T& b)
{
    return 1.f - static_cast<float>(jaccardDistance(a, b));
}

void NMSBoxes(const std::vector<Rect>& bboxes, const std::vector<float>& scores,
                          const float score_threshold, const float nms_threshold,
                          std::vector<int>& indices, const float eta, const int top_k)
{
    CV_Assert_N(bboxes.size() == scores.size(), score_threshold >= 0,
        nms_threshold >= 0, eta > 0);
    NMSFast_(bboxes, scores, score_threshold, nms_threshold, eta, top_k, indices, rectOverlap);
}

void NMSBoxes(const std::vector<Rect2d>& bboxes, const std::vector<float>& scores,
                          const float score_threshold, const float nms_threshold,
                          std::vector<int>& indices, const float eta, const int top_k)
{
    CV_Assert_N(bboxes.size() == scores.size(), score_threshold >= 0,
        nms_threshold >= 0, eta > 0);
    NMSFast_(bboxes, scores, score_threshold, nms_threshold, eta, top_k, indices, rectOverlap);
}

static inline float rotatedRectIOU(const RotatedRect& a, const RotatedRect& b)
{
    std::vector<Point2f> inter;
    int res = rotatedRectangleIntersection(a, b, inter);
    if (inter.empty() || res == INTERSECT_NONE)
        return 0.0f;
    if (res == INTERSECT_FULL)
        return 1.0f;
    float interArea = contourArea(inter);
    return interArea / (a.size.area() + b.size.area() - interArea);
}

void NMSBoxes(const std::vector<RotatedRect>& bboxes, const std::vector<float>& scores,
              const float score_threshold, const float nms_threshold,
              std::vector<int>& indices, const float eta, const int top_k)
{
    CV_Assert_N(bboxes.size() == scores.size(), score_threshold >= 0,
        nms_threshold >= 0, eta > 0);
    NMSFast_(bboxes, scores, score_threshold, nms_threshold, eta, top_k, indices, rotatedRectIOU);
}

template<class Rect_t>
static inline void NMSBoxesBatchedImpl(const std::vector<Rect_t>& bboxes,
                                       const std::vector<float>& scores, const std::vector<int>& class_ids,
                                       const float score_threshold, const float nms_threshold,
                                       std::vector<int>& indices, const float eta, const int top_k)
{
    double x1, y1, x2, y2, max_coord = 0;
    for (int i = 0; i < bboxes.size(); i++)
    {
        x1 = bboxes[i].x;
        y1 = bboxes[i].y;
        x2 = x1 + bboxes[i].width;
        y2 = y1 + bboxes[i].height;

        max_coord = std::max(x1, max_coord);
        max_coord = std::max(y1, max_coord);
        max_coord = std::max(x2, max_coord);
        max_coord = std::max(y2, max_coord);
    }

    // calculate offset and add offset to each bbox
    std::vector<Rect_t> bboxes_offset;
    double offset;
    for (int i = 0; i < bboxes.size(); i++)
    {
        offset = class_ids[i] * (max_coord + 1);
        bboxes_offset.push_back(
            Rect_t(bboxes[i].x + offset, bboxes[i].y + offset,
                   bboxes[i].width, bboxes[i].height)
        );
    }

    NMSFast_(bboxes_offset, scores, score_threshold, nms_threshold, eta, top_k, indices, rectOverlap);
}

void NMSBoxesBatched(const std::vector<Rect>& bboxes,
                     const std::vector<float>& scores, const std::vector<int>& class_ids,
                     const float score_threshold, const float nms_threshold,
                     std::vector<int>& indices, const float eta, const int top_k)
{
    CV_Assert_N(bboxes.size() == scores.size(), scores.size() == class_ids.size(), nms_threshold >= 0, eta > 0);

    NMSBoxesBatchedImpl(bboxes, scores, class_ids, score_threshold, nms_threshold, indices, eta, top_k);
}

void NMSBoxesBatched(const std::vector<Rect2d>& bboxes,
                     const std::vector<float>& scores, const std::vector<int>& class_ids,
                     const float score_threshold, const float nms_threshold,
                     std::vector<int>& indices, const float eta, const int top_k)
{
    CV_Assert_N(bboxes.size() == scores.size(), scores.size() == class_ids.size(), nms_threshold >= 0, eta > 0);

    NMSBoxesBatchedImpl(bboxes, scores, class_ids, score_threshold, nms_threshold, indices, eta, top_k);
}

void softNMSBoxes(const std::vector<Rect>& bboxes,
                  const std::vector<float>& scores,
                  std::vector<float>& updated_scores,
                  const float score_threshold,
                  const float nms_threshold,
                  std::vector<int>& indices,
                  size_t top_k,
                  const float sigma,
                  SoftNMSMethod method)
{
    CV_Assert_N(bboxes.size() == scores.size(), score_threshold >= 0,
                nms_threshold >= 0, sigma >= 0);

    indices.clear();
    updated_scores.clear();

    std::vector<std::pair<float, size_t> > score_index_vec(scores.size());
    for (size_t i = 0; i < scores.size(); i++)
    {
        score_index_vec[i].first = scores[i];
        score_index_vec[i].second = i;
    }

    const auto score_cmp = [](const std::pair<float, size_t>& a, const std::pair<float, size_t>& b)
    {
        return a.first == b.first ? a.second > b.second : a.first < b.first;
    };

    top_k = top_k == 0 ? scores.size() : std::min(top_k, scores.size());
    ptrdiff_t start = 0;
    while (indices.size() < top_k)
    {
        auto it = std::max_element(score_index_vec.begin() + start, score_index_vec.end(), score_cmp);

        float bscore = it->first;
        size_t bidx = it->second;

        if (bscore < score_threshold)
        {
            break;
        }

        indices.push_back(static_cast<int>(bidx));
        updated_scores.push_back(bscore);
        std::swap(score_index_vec[start], *it); // first start elements are chosen

        for (size_t i = start + 1; i < scores.size(); ++i)
        {
            float& bscore_i = score_index_vec[i].first;
            const size_t bidx_i = score_index_vec[i].second;

            if (bscore_i < score_threshold)
            {
                continue;
            }

            float overlap = rectOverlap(bboxes[bidx], bboxes[bidx_i]);

            switch (method)
            {
                case SoftNMSMethod::SOFTNMS_LINEAR:
                    if (overlap > nms_threshold)
                    {
                        bscore_i *= 1.f - overlap;
                    }
                    break;
                case SoftNMSMethod::SOFTNMS_GAUSSIAN:
                    bscore_i *= exp(-(overlap * overlap) / sigma);
                    break;
                default:
                    CV_Error(Error::StsBadArg, "Not supported SoftNMS method.");
            }
        }
        ++start;
    }
}

CV__DNN_INLINE_NS_END
}// dnn
}// cv
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

### Classes and Structures

- **Rect_t**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `precomp.hpp`
- `nms.inl.hpp`
- `opencv2/imgproc.hpp`


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

