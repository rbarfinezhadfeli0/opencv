# Documentation for `modules/video/test/ocl/test_bgfg_mog2.cpp`

## File Metadata

- **Full Path**: `modules/video/test/ocl/test_bgfg_mog2.cpp`
- **File Name**: `test_bgfg_mog2.cpp`
- **File Size**: 4,000 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/test/ocl/test_bgfg_mog2.cpp](../../../../modules/video/test/ocl/test_bgfg_mog2.cpp)

## Purpose and Role

This file is located in the `modules/video/test/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "../test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

//////////////////////////Mog2_Update///////////////////////////////////

namespace
{
    IMPLEMENT_PARAM_CLASS(UseGray, bool)
    IMPLEMENT_PARAM_CLASS(DetectShadow, bool)
    IMPLEMENT_PARAM_CLASS(UseFloat, bool)
}

PARAM_TEST_CASE(Mog2_Update, UseGray, DetectShadow,UseFloat)
{
    bool useGray;
    bool detectShadow;
    bool useFloat;
    virtual void SetUp()
    {
        useGray = GET_PARAM(0);
        detectShadow = GET_PARAM(1);
        useFloat = GET_PARAM(2);
    }
};

OCL_TEST_P(Mog2_Update, Accuracy)
{
    string inputFile = string(TS::ptr()->get_data_path()) + "video/768x576.avi";
    VideoCapture cap(inputFile);
    if (!cap.isOpened())
        throw SkipTestException("Video file can not be opened");

    Ptr<BackgroundSubtractorMOG2> mog2_cpu = createBackgroundSubtractorMOG2();
    Ptr<BackgroundSubtractorMOG2> mog2_ocl = createBackgroundSubtractorMOG2();

    mog2_cpu->setDetectShadows(detectShadow);
    mog2_ocl->setDetectShadows(detectShadow);

    Mat frame, foreground;
    UMat u_foreground;

    for (int i = 0; i < 10; ++i)
    {
        cap >> frame;
        ASSERT_FALSE(frame.empty());

        if (useGray)
        {
            Mat temp;
            cvtColor(frame, temp, COLOR_BGR2GRAY);
            swap(temp, frame);
        }

        if(useFloat)
        {
            Mat temp;
            frame.convertTo(temp,CV_32F);
            swap(temp,frame);
        }

        OCL_OFF(mog2_cpu->apply(frame, foreground));
        OCL_ON (mog2_ocl->apply(frame, u_foreground));

        if (detectShadow)
            EXPECT_MAT_SIMILAR(foreground, u_foreground, 15e-3);
        else
            EXPECT_MAT_NEAR(foreground, u_foreground, 0);
    }
}

//////////////////////////Mog2_getBackgroundImage///////////////////////////////////

PARAM_TEST_CASE(Mog2_getBackgroundImage, DetectShadow, UseFloat)
{
    bool detectShadow;
    bool useFloat;
    virtual void SetUp()
    {
        detectShadow = GET_PARAM(0);
        useFloat = GET_PARAM(1);
    }
};

OCL_TEST_P(Mog2_getBackgroundImage, Accuracy)
{
    string inputFile = string(TS::ptr()->get_data_path()) + "video/768x576.avi";
    VideoCapture cap(inputFile);
    if (!cap.isOpened())
        throw SkipTestException("Video file can not be opened");

    Ptr<BackgroundSubtractorMOG2> mog2_cpu = createBackgroundSubtractorMOG2();
    Ptr<BackgroundSubtractorMOG2> mog2_ocl = createBackgroundSubtractorMOG2();

    mog2_cpu->setDetectShadows(detectShadow);
    mog2_ocl->setDetectShadows(detectShadow);

    Mat frame, foreground;
    UMat u_foreground;

    for (int i = 0; i < 10; ++i)
    {
        cap >> frame;
        ASSERT_FALSE(frame.empty());

        if(useFloat)
        {
            Mat temp;
            frame.convertTo(temp,CV_32F);
            swap(temp,frame);
        }

        OCL_OFF(mog2_cpu->apply(frame, foreground));
        OCL_ON (mog2_ocl->apply(frame, u_foreground));
    }

    Mat background;
    OCL_OFF(mog2_cpu->getBackgroundImage(background));

    UMat u_background;
    OCL_ON (mog2_ocl->getBackgroundImage(u_background));

    EXPECT_MAT_NEAR(background, u_background, 1.0);
}

///////////////////////////////////////////////////////////////////////////////////////////

OCL_INSTANTIATE_TEST_CASE_P(OCL_Video, Mog2_Update, Combine(
                                    Values(UseGray(true),UseGray(false)),
                                    Values(DetectShadow(true), DetectShadow(false)),
                                    Values(UseFloat(false),UseFloat(true)))
                           );

OCL_INSTANTIATE_TEST_CASE_P(OCL_Video, Mog2_getBackgroundImage, Combine(
                                                     Values(DetectShadow(true), DetectShadow(false)),
                                                     Values(UseFloat(false),UseFloat(true)))
                           );

}}// namespace opencv_test::ocl

#endif
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

