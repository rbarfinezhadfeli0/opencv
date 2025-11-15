# Documentation for `modules/video/test/test_bgfg2.cpp`

## File Metadata

- **Full Path**: `modules/video/test/test_bgfg2.cpp`
- **File Name**: `test_bgfg2.cpp`
- **File Size**: 2,954 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/test/test_bgfg2.cpp](../../../modules/video/test/test_bgfg2.cpp)

## Purpose and Role

This file is located in the `modules/video/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"
#include "opencv2/video/background_segm.hpp"

namespace opencv_test { namespace {

using namespace cv;

///////////////////////// MOG2 //////////////////////////////
TEST(BackgroundSubtractorMOG2, KnownForegroundMaskShadowsTrue)
{
    Ptr<BackgroundSubtractorMOG2> mog2 = createBackgroundSubtractorMOG2(500, 16, true);

    //Black Frame
    Mat input = Mat::zeros(480,640 , CV_8UC3);

    //White Rectangle
    Mat knownFG = Mat::zeros(input.size(), CV_8U);

    rectangle(knownFG, Rect(3,3,5,5), Scalar(255,255,255), -1);

    Mat output;
    mog2->apply(input, knownFG, output);

    for(int y = 3; y < 8; y++)
    {
        for (int x = 3; x < 8; x++){
            EXPECT_EQ(255,output.at<uchar>(y,x)) << "Expected foreground at (" << x << "," << y << ")";
        }
    }
}

TEST(BackgroundSubtractorMOG2, KnownForegroundMaskShadowsFalse)
{
    Ptr<BackgroundSubtractorMOG2> mog2 = createBackgroundSubtractorMOG2(500, 16, false);

    //Black Frame
    Mat input = Mat::zeros(480,640 , CV_8UC3);

    //White Rectangle
    Mat knownFG = Mat::zeros(input.size(), CV_8U);

    rectangle(knownFG, Rect(3,3,5,5), Scalar(255,255,255), FILLED);

    Mat output;
    mog2->apply(input, knownFG, output);

    for(int y = 3; y < 8; y++)
    {
        for (int x = 3; x < 8; x++){
            EXPECT_EQ(255,output.at<uchar>(y,x)) << "Expected foreground at (" << x << "," << y << ")";
        }
    }
}

///////////////////////// KNN //////////////////////////////

TEST(BackgroundSubtractorKNN, KnownForegroundMaskShadowsTrue)
{
    Ptr<BackgroundSubtractorKNN> knn = createBackgroundSubtractorKNN(500, 400.0, true);

    //Black Frame
    Mat input = Mat::zeros(480,640 , CV_8UC3);

    //White Rectangle
    Mat knownFG = Mat::zeros(input.size(), CV_8U);

    rectangle(knownFG, Rect(3,3,5,5), Scalar(255,255,255), FILLED);

    Mat output;
    knn->apply(input, knownFG, output);

    for(int y = 3; y < 8; y++)
    {
        for (int x = 3; x < 8; x++){
            EXPECT_EQ(255,output.at<uchar>(y,x)) << "Expected foreground at (" << x << "," << y << ")";
        }
    }
}

TEST(BackgroundSubtractorKNN, KnownForegroundMaskShadowsFalse)
{
    Ptr<BackgroundSubtractorKNN> knn = createBackgroundSubtractorKNN(500, 400.0, false);

    //Black Frame
    Mat input = Mat::zeros(480,640 , CV_8UC3);

    //White Rectangle
    Mat knownFG = Mat::zeros(input.size(), CV_8U);

    rectangle(knownFG, Rect(3,3,5,5), Scalar(255,255,255), FILLED);

    Mat output;
    knn->apply(input, knownFG, output);

    for(int y = 3; y < 8; y++)
    {
        for (int x = 3; x < 8; x++){
            EXPECT_EQ(255,output.at<uchar>(y,x)) << "Expected foreground at (" << x << "," << y << ")";
        }
    }
}

}} // namespace
/* End of file. */
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
- `opencv2/video/background_segm.hpp`
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

