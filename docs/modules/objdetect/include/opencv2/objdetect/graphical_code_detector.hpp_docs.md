# Documentation for `modules/objdetect/include/opencv2/objdetect/graphical_code_detector.hpp`

## File Metadata

- **Full Path**: `modules/objdetect/include/opencv2/objdetect/graphical_code_detector.hpp`
- **File Name**: `graphical_code_detector.hpp`
- **File Size**: 5,327 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/objdetect/include/opencv2/objdetect/graphical_code_detector.hpp](../../../../../modules/objdetect/include/opencv2/objdetect/graphical_code_detector.hpp)

## Purpose and Role

This file is located in the `modules/objdetect/include/opencv2/objdetect` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html
#ifndef OPENCV_OBJDETECT_GRAPHICAL_CODE_DETECTOR_HPP
#define OPENCV_OBJDETECT_GRAPHICAL_CODE_DETECTOR_HPP

#include <opencv2/core.hpp>

namespace cv {

//! @addtogroup objdetect_common
//! @{

class CV_EXPORTS_W_SIMPLE GraphicalCodeDetector {
public:
    CV_DEPRECATED_EXTERNAL  // avoid using in C++ code, will be moved to "protected" (need to fix bindings first)
    GraphicalCodeDetector();

    GraphicalCodeDetector(const GraphicalCodeDetector&) = default;
    GraphicalCodeDetector(GraphicalCodeDetector&&) = default;
    GraphicalCodeDetector& operator=(const GraphicalCodeDetector&) = default;
    GraphicalCodeDetector& operator=(GraphicalCodeDetector&&) = default;

    /** @brief Detects graphical code in image and returns the quadrangle containing the code.
     @param img grayscale or color (BGR) image containing (or not) graphical code.
     @param points Output vector of vertices of the minimum-area quadrangle containing the code.
     */
    CV_WRAP bool detect(InputArray img, OutputArray points) const;

    /** @brief Decodes graphical code in image once it's found by the detect() method.

     Returns UTF8-encoded output string or empty string if the code cannot be decoded.
     @param img grayscale or color (BGR) image containing graphical code.
     @param points Quadrangle vertices found by detect() method (or some other algorithm).
     @param straight_code The optional output image containing binarized code, will be empty if not found.
     */
    CV_WRAP std::string decode(InputArray img, InputArray points, OutputArray straight_code = noArray()) const;

    /** @brief Both detects and decodes graphical code

     @param img grayscale or color (BGR) image containing graphical code.
     @param points optional output array of vertices of the found graphical code quadrangle, will be empty if not found.
     @param straight_code The optional output image containing binarized code
     */
    CV_WRAP std::string detectAndDecode(InputArray img, OutputArray points = noArray(),
                                        OutputArray straight_code = noArray()) const;


    /** @brief Detects graphical codes in image and returns the vector of the quadrangles containing the codes.
     @param img grayscale or color (BGR) image containing (or not) graphical codes.
     @param points Output vector of vector of vertices of the minimum-area quadrangle containing the codes.
     */
    CV_WRAP bool detectMulti(InputArray img, OutputArray points) const;

    /** @brief Decodes graphical codes in image once it's found by the detect() method.
     @param img grayscale or color (BGR) image containing graphical codes.
     @param decoded_info UTF8-encoded output vector of string or empty vector of string if the codes cannot be decoded.
     @param points vector of Quadrangle vertices found by detect() method (or some other algorithm).
     @param straight_code The optional output vector of images containing binarized codes
     */
    CV_WRAP bool decodeMulti(InputArray img, InputArray points, CV_OUT std::vector<std::string>& decoded_info,
                             OutputArrayOfArrays straight_code = noArray()) const;

    /** @brief Both detects and decodes graphical codes
    @param img grayscale or color (BGR) image containing graphical codes.
    @param decoded_info UTF8-encoded output vector of string or empty vector of string if the codes cannot be decoded.
    @param points optional output vector of vertices of the found graphical code quadrangles. Will be empty if not found.
    @param straight_code The optional vector of images containing binarized codes

    - If there are QR codes encoded with a Structured Append mode on the image and all of them detected and decoded correctly,
    method writes a full message to position corresponds to 0-th code in a sequence. The rest of QR codes from the same sequence
    have empty string.
    */
    CV_WRAP bool detectAndDecodeMulti(InputArray img, CV_OUT std::vector<std::string>& decoded_info, OutputArray points = noArray(),
                                      OutputArrayOfArrays straight_code = noArray()) const;

#ifdef OPENCV_BINDINGS_PARSER
    CV_WRAP_AS(detectAndDecodeBytes) NativeByteArray detectAndDecode(InputArray img, OutputArray points = noArray(),
                                                                     OutputArray straight_code = noArray()) const;
    CV_WRAP_AS(decodeBytes) NativeByteArray decode(InputArray img, InputArray points, OutputArray straight_code = noArray()) const;
    CV_WRAP_AS(decodeBytesMulti) bool decodeMulti(InputArray img, InputArray points, CV_OUT std::vector<NativeByteArray>& decoded_info,
                                                  OutputArrayOfArrays straight_code = noArray()) const;
    CV_WRAP_AS(detectAndDecodeBytesMulti) bool detectAndDecodeMulti(InputArray img, CV_OUT std::vector<NativeByteArray>& decoded_info, OutputArray points = noArray(),
                                                                    OutputArrayOfArrays straight_code = noArray()) const;
#endif

    struct Impl;
protected:
    Ptr<Impl> p;
};

//! @}

}

#endif
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

- **Impl**: A class/struct defined in this file
- **CV_EXPORTS_W_SIMPLE**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_BINDINGS_PARSER()**: A function/method defined in this file
- **OPENCV_OBJDETECT_GRAPHICAL_CODE_DETECTOR_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`

**Python Imports:**
- `the`


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

