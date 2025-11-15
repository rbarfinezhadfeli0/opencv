# Documentation for `modules/photo/test/ocl/test_denoising.cpp`

## File Metadata

- **Full Path**: `modules/photo/test/ocl/test_denoising.cpp`
- **File Name**: `test_denoising.cpp`
- **File Size**: 4,451 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/test/ocl/test_denoising.cpp](../../../../modules/photo/test/ocl/test_denoising.cpp)

## Purpose and Role

This file is located in the `modules/photo/test/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2014, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.

#include "../test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

PARAM_TEST_CASE(FastNlMeansDenoisingTestBase, Channels, int, bool, bool)
{
    int cn, normType, templateWindowSize, searchWindowSize;
    std::vector<float> h;
    bool use_roi, use_image;

    TEST_DECLARE_INPUT_PARAMETER(src);
    TEST_DECLARE_OUTPUT_PARAMETER(dst);

    virtual void SetUp()
    {
        cn = GET_PARAM(0);
        normType = GET_PARAM(1);
        use_roi = GET_PARAM(2);
        use_image = GET_PARAM(3);

        templateWindowSize = 7;
        searchWindowSize = 21;

        h.resize(cn);
        for (int i=0; i<cn; i++)
            h[i] = 3.0f + 0.5f*i;
    }

    void generateTestData()
    {
        const int type = CV_8UC(cn);
        Mat image;

        if (use_image) {
            image = readImage("denoising/lena_noised_gaussian_sigma=10.png",
                                  cn == 1 ? IMREAD_GRAYSCALE : IMREAD_COLOR);
            ASSERT_FALSE(image.empty());
        }

        Size roiSize = use_image ? image.size() : randomSize(1, MAX_VALUE);
        Border srcBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(src, src_roi, roiSize, srcBorder, type, 0, 255);
        if (use_image) {
            ASSERT_TRUE(cn > 0 && cn <= 4);
            if (cn == 2) {
                int from_to[] = { 0,0, 1,1 };
                src_roi.create(roiSize, type);
                mixChannels(&image, 1, &src_roi, 1, from_to, 2);
            }
            else if (cn == 4) {
                int from_to[] = { 0,0, 1,1, 2,2, 1,3};
                src_roi.create(roiSize, type);
                mixChannels(&image, 1, &src_roi, 1, from_to, 4);
            }
            else image.copyTo(src_roi);
        }

        Border dstBorder = randomBorder(0, use_roi ? MAX_VALUE : 0);
        randomSubMat(dst, dst_roi, roiSize, dstBorder, type, 0, 255);

        UMAT_UPLOAD_INPUT_PARAMETER(src);
        UMAT_UPLOAD_OUTPUT_PARAMETER(dst);
    }
};

typedef FastNlMeansDenoisingTestBase FastNlMeansDenoising;

OCL_TEST_P(FastNlMeansDenoising, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        generateTestData();

        OCL_OFF(cv::fastNlMeansDenoising(src_roi, dst_roi, std::vector<float>(1, h[0]), templateWindowSize, searchWindowSize, normType));
        OCL_ON(cv::fastNlMeansDenoising(usrc_roi, udst_roi, std::vector<float>(1, h[0]), templateWindowSize, searchWindowSize, normType));

        OCL_EXPECT_MATS_NEAR(dst, 1);
    }
}

typedef FastNlMeansDenoisingTestBase FastNlMeansDenoising_hsep;

OCL_TEST_P(FastNlMeansDenoising_hsep, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        generateTestData();

        OCL_OFF(cv::fastNlMeansDenoising(src_roi, dst_roi, h, templateWindowSize, searchWindowSize, normType));
        OCL_ON(cv::fastNlMeansDenoising(usrc_roi, udst_roi, h, templateWindowSize, searchWindowSize, normType));

        OCL_EXPECT_MATS_NEAR(dst, 1);
    }
}

typedef FastNlMeansDenoisingTestBase FastNlMeansDenoisingColored;

OCL_TEST_P(FastNlMeansDenoisingColored, Mat)
{
    for (int j = 0; j < test_loop_times; j++)
    {
        generateTestData();

        OCL_OFF(cv::fastNlMeansDenoisingColored(src_roi, dst_roi, h[0], h[0], templateWindowSize, searchWindowSize));
        OCL_ON(cv::fastNlMeansDenoisingColored(usrc_roi, udst_roi, h[0], h[0], templateWindowSize, searchWindowSize));

        OCL_EXPECT_MATS_NEAR(dst, 1);
    }
}

OCL_INSTANTIATE_TEST_CASE_P(Photo, FastNlMeansDenoising,
                            Combine(Values(1, 2, 3, 4), Values((int)NORM_L2, (int)NORM_L1),
                                    Bool(), Values(true)));
OCL_INSTANTIATE_TEST_CASE_P(Photo, FastNlMeansDenoising_hsep,
                            Combine(Values(1, 2, 3, 4), Values((int)NORM_L2, (int)NORM_L1),
                                    Bool(), Values(true)));
OCL_INSTANTIATE_TEST_CASE_P(Photo, FastNlMeansDenoisingColored,
                            Combine(Values(3, 4), Values((int)NORM_L2), Bool(), Values(false)));

} } // namespace opencv_test::ocl

#endif // HAVE_OPENCL
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

### Functions and Methods

- **HAVE_OPENCL()**: A function/method defined in this file
- **FastNlMeansDenoisingTestBase()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../test_precomp.hpp`
- `opencv2/ts/ocl_test.hpp`


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

