# Documentation for `modules/gapi/include/opencv2/gapi/gcall.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcall.hpp`
- **File Name**: `gcall.hpp`
- **File Size**: 2,169 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcall.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcall.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_GCALL_HPP
#define OPENCV_GAPI_GCALL_HPP

#include <opencv2/gapi/garg.hpp>      // GArg
#include <opencv2/gapi/gmat.hpp>      // GMat
#include <opencv2/gapi/gscalar.hpp>   // GScalar
#include <opencv2/gapi/gframe.hpp>    // GFrame
#include <opencv2/gapi/garray.hpp>    // GArray<T>
#include <opencv2/gapi/gopaque.hpp>   // GOpaque<T>

namespace cv {

struct GKernel;

// The whole idea of this class is to represent an operation
// which is applied to arguments. This is part of public API,
// since it is what users should use to define kernel interfaces.

class GAPI_EXPORTS GCall final
{
public:
    class Priv;

    explicit GCall(const GKernel &k);
    ~GCall();

    template<typename... Ts>
    GCall& pass(Ts&&... args)
    {
        setArgs({cv::GArg(std::move(args))...});
        return *this;
    }

    // A generic yield method - obtain a link to operator's particular GMat output
    GMat    yield      (int output = 0);
    GMatP   yieldP     (int output = 0);
    GScalar yieldScalar(int output = 0);
    GFrame  yieldFrame (int output = 0);

    template<class T> GArray<T> yieldArray(int output = 0)
    {
        return GArray<T>(yieldArray(output));
    }

    template<class T> GOpaque<T> yieldOpaque(int output = 0)
    {
        return GOpaque<T>(yieldOpaque(output));
    }

    // Internal use only
    Priv& priv();
    const Priv& priv() const;

    // GKernel and params can be modified, it's needed for infer<Generic>,
    // because information about output shapes doesn't exist in compile time
    GKernel& kernel();
    cv::util::any& params();

    void setArgs(std::vector<GArg> &&args);

protected:
    std::shared_ptr<Priv> m_priv;

    // Public versions return a typed array or opaque, those are implementation details
    detail::GArrayU yieldArray(int output = 0);
    detail::GOpaqueU yieldOpaque(int output = 0);
};

} // namespace cv

#endif // OPENCV_GAPI_GCALL_HPP
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

- **GAPI_EXPORTS**: A class/struct defined in this file
- **is**: A class/struct defined in this file
- **GKernel**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **Priv**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GCALL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gframe.hpp`
- `opencv2/gapi/gmat.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi/gscalar.hpp`
- `opencv2/gapi/garray.hpp`
- `opencv2/gapi/gopaque.hpp`


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

