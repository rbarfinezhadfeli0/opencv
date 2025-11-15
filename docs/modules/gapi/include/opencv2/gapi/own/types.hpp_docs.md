# Documentation for `modules/gapi/include/opencv2/gapi/own/types.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/own/types.hpp`
- **File Name**: `types.hpp`
- **File Size**: 3,666 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/own/types.hpp](../../../../../../modules/gapi/include/opencv2/gapi/own/types.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/own` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_TYPES_HPP
#define OPENCV_GAPI_TYPES_HPP

#include <algorithm>              // std::max, std::min
#include <ostream>

namespace cv
{
namespace gapi
{

/**
 * @brief This namespace contains G-API own data structures used in
 * its standalone mode build.
 */
namespace own
{

class Point
{
public:
    Point() = default;
    Point(int _x, int _y) : x(_x),  y(_y) {}

    int x = 0;
    int y = 0;
};

class Point2f
{
public:
    Point2f() = default;
    Point2f(float _x, float _y) : x(_x),  y(_y) {}

    float x = 0.f;
    float y = 0.f;
};

class Point3f
{
public:
    Point3f() = default;
    Point3f(float _x, float _y, float _z) : x(_x),  y(_y), z(_z) {}

    float x = 0.f;
    float y = 0.f;
    float z = 0.f;
};

class Rect
{
public:
    Rect() = default;
    Rect(int _x, int _y, int _width, int _height) : x(_x), y(_y),   width(_width),  height(_height) {}
#if !defined(GAPI_STANDALONE)
    Rect(const cv::Rect& other) : x(other.x), y(other.y), width(other.width), height(other.height) {}
    inline Rect& operator=(const cv::Rect& other)
    {
        x = other.x;
        y = other.x;
        width  = other.width;
        height = other.height;
        return *this;
    }
#endif // !defined(GAPI_STANDALONE)

    int x      = 0; //!< x coordinate of the top-left corner
    int y      = 0; //!< y coordinate of the top-left corner
    int width  = 0; //!< width of the rectangle
    int height = 0; //!< height of the rectangle
};

inline bool operator==(const Rect& lhs, const Rect& rhs)
{
    return lhs.x == rhs.x && lhs.y == rhs.y && lhs.width == rhs.width && lhs.height == rhs.height;
}

inline bool operator!=(const Rect& lhs, const Rect& rhs)
{
    return !(lhs == rhs);
}

inline Rect& operator&=(Rect& lhs, const Rect& rhs)
{
    int x1 = std::max(lhs.x, rhs.x);
    int y1 = std::max(lhs.y, rhs.y);
    lhs.width  = std::min(lhs.x + lhs.width,  rhs.x + rhs.width) -  x1;
    lhs.height = std::min(lhs.y + lhs.height, rhs.y + rhs.height) - y1;
    lhs.x = x1;
    lhs.y = y1;
    if( lhs.width <= 0 || lhs.height <= 0 )
        lhs = Rect();
    return lhs;
}

inline Rect operator&(const Rect& lhs, const Rect& rhs)
{
    Rect result = lhs;
    return result &= rhs;
}

inline std::ostream& operator<<(std::ostream& o, const Rect& rect)
{
    return o << "[" << rect.width << " x " << rect.height << " from (" << rect.x << ", " << rect.y << ")]";
}

class Size
{
public:
    Size() = default;
    Size(int _width, int _height) : width(_width),  height(_height) {}
#if !defined(GAPI_STANDALONE)
    Size(const cv::Size& other) : width(other.width), height(other.height) {}
    inline Size& operator=(const cv::Size& rhs)
    {
        width  = rhs.width;
        height = rhs.height;
        return *this;
    }
#endif // !defined(GAPI_STANDALONE)

    int width  = 0;
    int height = 0;
};

inline Size& operator+=(Size& lhs, const Size& rhs)
{
    lhs.width  += rhs.width;
    lhs.height += rhs.height;
    return lhs;
}

inline bool operator==(const Size& lhs, const Size& rhs)
{
    return lhs.width == rhs.width && lhs.height == rhs.height;
}

inline bool operator!=(const Size& lhs, const Size& rhs)
{
    return !(lhs == rhs);
}


inline std::ostream& operator<<(std::ostream& o, const Size& s)
{
    o << "[" << s.width << " x " << s.height << "]";
    return o;
}

struct VoidType {};
} // namespace own
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_TYPES_HPP
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

- **Point**: A class/struct defined in this file
- **Point3f**: A class/struct defined in this file
- **VoidType**: A class/struct defined in this file
- **Rect**: A class/struct defined in this file
- **Size**: A class/struct defined in this file
- **Point2f**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_TYPES_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `algorithm`
- `ostream`


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

