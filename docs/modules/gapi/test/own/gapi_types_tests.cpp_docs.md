# Documentation for `modules/gapi/test/own/gapi_types_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/own/gapi_types_tests.cpp`
- **File Name**: `gapi_types_tests.cpp`
- **File Size**: 2,960 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/own/gapi_types_tests.cpp](../../../../modules/gapi/test/own/gapi_types_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/own` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "../test_precomp.hpp"
#include <opencv2/gapi/own/types.hpp>

namespace opencv_test
{

TEST(Point, CreateEmpty)
{
    cv::gapi::own::Point p;

    EXPECT_EQ(0, p.x);
    EXPECT_EQ(0, p.y);
}

TEST(Point, CreateWithParams)
{
    cv::gapi::own::Point p = {1, 2};

    EXPECT_EQ(1, p.x);
    EXPECT_EQ(2, p.y);
}

TEST(Point2f, CreateEmpty)
{
    cv::gapi::own::Point2f p;

    EXPECT_EQ(0.f, p.x);
    EXPECT_EQ(0.f, p.y);
}

TEST(Point2f, CreateWithParams)
{
    cv::gapi::own::Point2f p = {3.14f, 2.71f};

    EXPECT_EQ(3.14f, p.x);
    EXPECT_EQ(2.71f, p.y);
}

TEST(Rect, CreateEmpty)
{
    cv::gapi::own::Rect r;

    EXPECT_EQ(0, r.x);
    EXPECT_EQ(0, r.y);
    EXPECT_EQ(0, r.width);
    EXPECT_EQ(0, r.height);
}

TEST(Rect, CreateWithParams)
{
    cv::gapi::own::Rect r(1, 2, 3, 4);

    EXPECT_EQ(1, r.x);
    EXPECT_EQ(2, r.y);
    EXPECT_EQ(3, r.width);
    EXPECT_EQ(4, r.height);
}

TEST(Rect, CompareEqual)
{
    cv::gapi::own::Rect r1(1, 2, 3, 4);

    cv::gapi::own::Rect r2(1, 2, 3, 4);

    EXPECT_TRUE(r1 == r2);
}

TEST(Rect, CompareDefaultEqual)
{
    cv::gapi::own::Rect r1;

    cv::gapi::own::Rect r2;

    EXPECT_TRUE(r1 == r2);
}

TEST(Rect, CompareNotEqual)
{
    cv::gapi::own::Rect r1(1, 2, 3, 4);

    cv::gapi::own::Rect r2;

    EXPECT_TRUE(r1 != r2);
}

TEST(Rect, Intersection)
{
    cv::gapi::own::Rect r1(2, 2, 3, 3);
    cv::gapi::own::Rect r2(3, 1, 3, 3);

    cv::gapi::own::Rect intersect = r1 & r2;

    EXPECT_EQ(3, intersect.x);
    EXPECT_EQ(2, intersect.y);
    EXPECT_EQ(2, intersect.width);
    EXPECT_EQ(2, intersect.height);
}

TEST(Rect, AssignIntersection)
{
    cv::gapi::own::Rect r1(2, 2, 3, 3);
    cv::gapi::own::Rect r2(3, 1, 3, 3);

    r1 &= r2;

    EXPECT_EQ(3, r1.x);
    EXPECT_EQ(2, r1.y);
    EXPECT_EQ(2, r1.width);
    EXPECT_EQ(2, r1.height);
}

TEST(Size, CreateEmpty)
{
    cv::gapi::own::Size s;

    EXPECT_EQ(0, s.width);
    EXPECT_EQ(0, s.height);
}

TEST(Size, CreateWithParams)
{
    cv::gapi::own::Size s(640, 480);

    EXPECT_EQ(640, s.width);
    EXPECT_EQ(480, s.height);
}

TEST(Size, AdditionAssignment)
{
    cv::gapi::own::Size s1(1, 2);
    cv::gapi::own::Size s2(2, 3);

    s1 += s2;

    EXPECT_EQ(3, s1.width);
    EXPECT_EQ(5, s1.height);
}

TEST(Size, CompareEqual)
{
    cv::gapi::own::Size s1(1, 2);

    cv::gapi::own::Size s2(1, 2);

    EXPECT_TRUE(s1 == s2);
    EXPECT_FALSE(s1 != s2);
}

TEST(Size, CompareDefaultEqual)
{
    cv::gapi::own::Size s1;
    cv::gapi::own::Size s2;

    EXPECT_TRUE(s1 == s2);
    EXPECT_FALSE(s1 != s2);
}

TEST(Size, CompareNotEqual)
{
    cv::gapi::own::Size s1(1, 2);

    cv::gapi::own::Size s2(3, 4);

    EXPECT_FALSE(s1 == s2);
    EXPECT_TRUE(s1 != s2);
}

} // opencv_test
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
- `../test_precomp.hpp`
- `opencv2/gapi/own/types.hpp`


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

