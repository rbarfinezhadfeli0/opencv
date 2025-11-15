# Documentation for `modules/core/test/ref_reduce_arg.impl.hpp`

## File Metadata

- **Full Path**: `modules/core/test/ref_reduce_arg.impl.hpp`
- **File Name**: `ref_reduce_arg.impl.hpp`
- **File Size**: 2,804 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/test/ref_reduce_arg.impl.hpp](../../../modules/core/test/ref_reduce_arg.impl.hpp)

## Purpose and Role

This file is located in the `modules/core/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_TEST_REF_REDUCE_ARG_HPP
#define OPENCV_TEST_REF_REDUCE_ARG_HPP

#include "opencv2/core/detail/dispatch_helper.impl.hpp"

#include <algorithm>
#include <numeric>

namespace cvtest {

// Standard library (technically) forbids using std::min_element
// with non-strict std::less_equal, so we make our own min_element
template <typename Iter, typename Comp>
Iter custom_min_element(Iter begin, Iter end, Comp cmp_less)
{
    if (begin == end)
        return begin;
    Iter result = begin;
    while (++begin != end)
        if (cmp_less(*begin, *result))
            result = begin;
    return result;
}


template <class Cmp, typename T>
struct reduceMinMaxImpl
{
    void operator()(const cv::Mat& src, cv::Mat& dst, const int axis) const
    {
        Cmp cmp;
        std::vector<int> sizes(src.dims);
        std::copy(src.size.p, src.size.p + src.dims, sizes.begin());

        std::vector<cv::Range> idx(sizes.size(), cv::Range(0, 1));
        idx[axis] = cv::Range::all();
        const int n = std::accumulate(begin(sizes), end(sizes), 1, std::multiplies<int>());
        const std::vector<int> newShape{1, src.size[axis]};
        for (int i = 0; i < n ; ++i)
        {
            cv::Mat sub = src(idx);

            auto begin = sub.begin<T>();
            auto it = custom_min_element(begin, sub.end<T>(), cmp);
            *dst(idx).ptr<int32_t>() = static_cast<int32_t>(std::distance(begin, it));

            for (int j = static_cast<int>(idx.size()) - 1; j >= 0; --j)
            {
                if (j == axis)
                {
                    continue;
                }
                const int old_s = idx[j].start;
                const int new_s = (old_s + 1) % sizes[j];
                if (new_s > old_s)
                {
                    idx[j] = cv::Range(new_s, new_s + 1);
                    break;
                }
                idx[j] = cv::Range(0, 1);
            }
        }
    }
};

template<template<class> class Cmp>
struct MinMaxReducer{
    template <typename T>
    using Impl = reduceMinMaxImpl<Cmp<T>, T>;

    static void reduce(const Mat& src, Mat& dst, int axis)
    {
        axis = (axis + src.dims) % src.dims;
        CV_Assert(src.channels() == 1 && axis >= 0 && axis < src.dims);

        std::vector<int> sizes(src.dims);
        std::copy(src.size.p, src.size.p + src.dims, sizes.begin());
        sizes[axis] = 1;

        dst.create(sizes, CV_32SC1); // indices
        dst.setTo(cv::Scalar::all(0));

        cv::detail::depthDispatch<Impl>(src.depth(), src, dst, axis);
    }
};

}

#endif //OPENCV_TEST_REF_REDUCE_ARG_HPP
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

- **Cmp**: A class/struct defined in this file
- **MinMaxReducer**: A class/struct defined in this file
- **reduceMinMaxImpl**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_TEST_REF_REDUCE_ARG_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `algorithm`
- `numeric`
- `opencv2/core/detail/dispatch_helper.impl.hpp`


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

