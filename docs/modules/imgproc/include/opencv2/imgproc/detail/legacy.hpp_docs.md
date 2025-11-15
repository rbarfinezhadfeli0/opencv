# Documentation for `modules/imgproc/include/opencv2/imgproc/detail/legacy.hpp`

## File Metadata

- **Full Path**: `modules/imgproc/include/opencv2/imgproc/detail/legacy.hpp`
- **File Name**: `legacy.hpp`
- **File Size**: 1,402 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/imgproc/include/opencv2/imgproc/detail/legacy.hpp](../../../../../../modules/imgproc/include/opencv2/imgproc/detail/legacy.hpp)

## Purpose and Role

This file is located in the `modules/imgproc/include/opencv2/imgproc/detail` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#ifndef OPENCV_IMGPROC_DETAIL_LEGACY_HPP
#define OPENCV_IMGPROC_DETAIL_LEGACY_HPP

#include "opencv2/imgproc.hpp"

namespace cv {

#ifdef __OPENCV_BUILD

CV_EXPORTS void findContours_legacy(InputArray _image,
                                    OutputArrayOfArrays _contours,
                                    OutputArray _hierarchy,
                                    int mode,
                                    int method,
                                    Point offset = Point());
CV_EXPORTS void findContours_legacy(InputArray image,
                                    OutputArrayOfArrays contours,
                                    int mode,
                                    int method,
                                    Point offset = Point());

CV_EXPORTS float EMD_legacy( InputArray _signature1, InputArray _signature2,
               int distType, InputArray _cost,
               float* lowerBound, OutputArray _flow );

CV_EXPORTS float wrapperEMD_legacy(InputArray _signature1, InputArray _signature2,
               int distType, InputArray _cost,
               Ptr<float> lowerBound, OutputArray _flow);

#endif

}  // namespace cv

#endif  // OPENCV_IMGPROC_DETAIL_LEGACY_HPP
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

### Functions and Methods

- **OPENCV_IMGPROC_DETAIL_LEGACY_HPP()**: A function/method defined in this file
- **__OPENCV_BUILD()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
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

