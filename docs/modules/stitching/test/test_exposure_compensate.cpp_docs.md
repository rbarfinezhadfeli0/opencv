# Documentation for `modules/stitching/test/test_exposure_compensate.cpp`

## File Metadata

- **Full Path**: `modules/stitching/test/test_exposure_compensate.cpp`
- **File Name**: `test_exposure_compensate.cpp`
- **File Size**: 2,292 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/stitching/test/test_exposure_compensate.cpp](../../../modules/stitching/test/test_exposure_compensate.cpp)

## Purpose and Role

This file is located in the `modules/stitching/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"

namespace opencv_test {
namespace {

double minPSNR(UMat src1, UMat src2)
{
    std::vector<UMat> src1_channels, src2_channels;
    split(src1, src1_channels);
    split(src2, src2_channels);

    double psnr = cvtest::PSNR(src1_channels[0], src2_channels[0]);
    psnr = std::min(psnr, cvtest::PSNR(src1_channels[1], src2_channels[1]));
    return std::min(psnr, cvtest::PSNR(src1_channels[2], src2_channels[2]));
}

TEST(ExposureCompensate, SimilarityThreshold)
{
    UMat source;
    imread(cvtest::TS::ptr()->get_data_path() + "stitching/s1.jpg").copyTo(source);

    UMat image1 = source.clone();
    UMat image2 = source.clone();

    // Add a big artifact
    image2(Rect(150, 150, 100, 100)).setTo(Scalar(0, 0, 255));

    UMat mask(image1.size(), CV_8U);
    mask.setTo(255);

    detail::BlocksChannelsCompensator compensator;
    compensator.setNrGainsFilteringIterations(0); // makes it more clear

    // Feed the compensator, image 1 and 2 are perfectly
    // identical, except for the red artifact in image 2
    // Apart from that artifact, there is no exposure to compensate
    compensator.setSimilarityThreshold(1);
    uchar xff = 255;
    compensator.feed(
        {{}, {}},
        {image1, image2},
        {{mask, xff}, {mask, xff}}
    );
    // Verify that the artifact in image 2 did create
    // an artifact in image1 during the exposure compensation
    UMat image1_result = image1.clone();
    compensator.apply(0, {}, image1_result, mask);
    double psnr_no_similarity_mask = minPSNR(image1, image1_result);
    EXPECT_LT(psnr_no_similarity_mask, 45);

    // Add a similarity threshold and verify that
    // the artifact in image1 is gone
    compensator.setSimilarityThreshold(0.1);
    compensator.feed(
        {{}, {}},
        {image1, image2},
        {{mask, xff}, {mask, xff}}
    );
    image1_result = image1.clone();
    compensator.apply(0, {}, image1_result, mask);
    double psnr_similarity_mask = minPSNR(image1, image1_result);
    EXPECT_GT(psnr_similarity_mask, 300);
}

} // namespace
} // namespace opencv_test
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

**Python Imports:**
- `that`


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

