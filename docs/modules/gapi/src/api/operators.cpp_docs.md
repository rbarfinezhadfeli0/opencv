# Documentation for `modules/gapi/src/api/operators.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/operators.cpp`
- **File Name**: `operators.cpp`
- **File Size**: 4,832 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/operators.cpp](../../../../modules/gapi/src/api/operators.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "precomp.hpp"

#include <opencv2/gapi/imgproc.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/gscalar.hpp>
#include <opencv2/gapi/operators.hpp>

namespace cv
{
cv::GMat operator+(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::add(lhs, rhs);
}

cv::GMat operator+(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::addC(lhs, rhs);
}

cv::GMat operator+(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::addC(rhs, lhs);
}

cv::GMat operator-(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::sub(lhs, rhs);
}

cv::GMat operator-(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::subC(lhs, rhs);
}

cv::GMat operator-(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::subRC(lhs, rhs);
}

cv::GMat operator*(const cv::GMat& lhs, float rhs)
{
    return cv::gapi::mulC(lhs, static_cast<double>(rhs));
}

cv::GMat operator*(float lhs, const cv::GMat& rhs)
{
    return cv::gapi::mulC(rhs, static_cast<double>(lhs));
}

cv::GMat operator*(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::mulC(lhs, rhs);
}

cv::GMat operator*(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::mulC(rhs, lhs);
}

cv::GMat operator/(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::divC(lhs, rhs, 1.0);
}

cv::GMat operator/(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::div(lhs, rhs, 1.0);
}

cv::GMat operator/(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::divRC(lhs, rhs, 1.0);
}

cv::GMat operator&(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_and(lhs, rhs);
}

cv::GMat operator&(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::bitwise_and(lhs, rhs);
}

cv::GMat operator&(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_and(rhs, lhs);
}

cv::GMat operator|(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_or(lhs, rhs);
}

cv::GMat operator|(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::bitwise_or(lhs, rhs);
}

cv::GMat operator|(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_or(rhs, lhs);
}

cv::GMat operator^(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_xor(lhs, rhs);
}

cv::GMat operator^(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::bitwise_xor(lhs, rhs);
}

cv::GMat operator^(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::bitwise_xor(rhs, lhs);
}

cv::GMat operator~(const cv::GMat& lhs)
{
    return cv::gapi::bitwise_not(lhs);
}

cv::GMat operator>(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpGT(lhs, rhs);
}

cv::GMat operator>=(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpGE(lhs, rhs);
}

cv::GMat operator<(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpLT(lhs, rhs);
}

cv::GMat operator<=(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpLE(lhs, rhs);
}

cv::GMat operator==(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpEQ(lhs, rhs);
}

cv::GMat operator!=(const cv::GMat& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpNE(lhs, rhs);
}

cv::GMat operator>(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpGT(lhs, rhs);
}

cv::GMat operator>=(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpGE(lhs, rhs);
}

cv::GMat operator<(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpLT(lhs, rhs);
}

cv::GMat operator<=(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpLE(lhs, rhs);
}

cv::GMat operator==(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpEQ(lhs, rhs);
}

cv::GMat operator!=(const cv::GMat& lhs, const cv::GScalar& rhs)
{
    return cv::gapi::cmpNE(lhs, rhs);
}

cv::GMat operator>(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpLT(rhs, lhs);
}
cv::GMat operator>=(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpLE(rhs, lhs);
}
cv::GMat operator<(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpGT(rhs, lhs);
}
cv::GMat operator<=(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpGE(rhs, lhs);
}
cv::GMat operator==(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpEQ(rhs, lhs);
}
cv::GMat operator!=(const cv::GScalar& lhs, const cv::GMat& rhs)
{
    return cv::gapi::cmpNE(rhs, lhs);
}
} // cv
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
- `opencv2/gapi/imgproc.hpp`
- `opencv2/gapi/operators.hpp`
- `opencv2/gapi/gscalar.hpp`
- `precomp.hpp`
- `opencv2/gapi/core.hpp`


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

