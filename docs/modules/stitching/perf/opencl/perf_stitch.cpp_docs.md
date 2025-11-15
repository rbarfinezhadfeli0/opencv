# Documentation for `modules/stitching/perf/opencl/perf_stitch.cpp`

## File Metadata

- **Full Path**: `modules/stitching/perf/opencl/perf_stitch.cpp`
- **File Name**: `perf_stitch.cpp`
- **File Size**: 4,756 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/stitching/perf/opencl/perf_stitch.cpp](../../../../modules/stitching/perf/opencl/perf_stitch.cpp)

## Purpose and Role

This file is located in the `modules/stitching/perf/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2014, Itseez, Inc, all rights reserved.

#include "../perf_precomp.hpp"
#include "opencv2/ts/ocl_perf.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
using namespace perf;
namespace ocl {

#define SURF_MATCH_CONFIDENCE 0.65f
#define ORB_MATCH_CONFIDENCE  0.3f
#define WORK_MEGAPIX 0.6

typedef TestBaseWithParam<string> stitch;

#if defined(HAVE_OPENCV_XFEATURES2D) && defined(OPENCV_ENABLE_NONFREE)
#define TEST_DETECTORS testing::Values("surf", "orb", "akaze")
#else
#define TEST_DETECTORS testing::Values("orb", "akaze")
#endif

OCL_PERF_TEST_P(stitch, a123, TEST_DETECTORS)
{
    UMat pano;

    vector<Mat> _imgs;
    _imgs.push_back( imread( getDataPath("stitching/a1.png") ) );
    _imgs.push_back( imread( getDataPath("stitching/a2.png") ) );
    _imgs.push_back( imread( getDataPath("stitching/a3.png") ) );
    vector<UMat> imgs = ToUMat(_imgs);

    Ptr<Feature2D> featuresFinder = getFeatureFinder(GetParam());
    Ptr<detail::FeaturesMatcher> featuresMatcher = GetParam() == "orb"
            ? makePtr<detail::BestOf2NearestMatcher>(false, ORB_MATCH_CONFIDENCE)
            : makePtr<detail::BestOf2NearestMatcher>(false, SURF_MATCH_CONFIDENCE);

    declare.iterations(20);

    while(next())
    {
        Ptr<Stitcher> stitcher = Stitcher::create();
        stitcher->setFeaturesFinder(featuresFinder);
        stitcher->setFeaturesMatcher(featuresMatcher);
        stitcher->setWarper(makePtr<SphericalWarper>());
        stitcher->setRegistrationResol(WORK_MEGAPIX);

        startTimer();
        stitcher->stitch(imgs, pano);
        stopTimer();
    }

    EXPECT_NEAR(pano.size().width, 1182, 50);
    EXPECT_NEAR(pano.size().height, 682, 30);

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST_P(stitch, b12, TEST_DETECTORS)
{
    UMat pano;

    vector<Mat> imgs;
    imgs.push_back( imread( getDataPath("stitching/b1.png") ) );
    imgs.push_back( imread( getDataPath("stitching/b2.png") ) );

    Ptr<Feature2D> featuresFinder = getFeatureFinder(GetParam());
    Ptr<detail::FeaturesMatcher> featuresMatcher = GetParam() == "orb"
            ? makePtr<detail::BestOf2NearestMatcher>(false, ORB_MATCH_CONFIDENCE)
            : makePtr<detail::BestOf2NearestMatcher>(false, SURF_MATCH_CONFIDENCE);

    declare.iterations(20);

    while(next())
    {
        Ptr<Stitcher> stitcher = Stitcher::create();
        stitcher->setFeaturesFinder(featuresFinder);
        stitcher->setFeaturesMatcher(featuresMatcher);
        stitcher->setWarper(makePtr<SphericalWarper>());
        stitcher->setRegistrationResol(WORK_MEGAPIX);

        startTimer();
        stitcher->stitch(imgs, pano);
        stopTimer();
    }

    EXPECT_NEAR(pano.size().width, 1124, GetParam() == "surf" ? 100 : 50);
    EXPECT_NEAR(pano.size().height, 644, GetParam() == "surf" ? 60 : 30);

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST_P(stitch, boat, TEST_DETECTORS)
{
    Size expected_dst_size(10789, 2663);
    checkDeviceMaxMemoryAllocSize(expected_dst_size, CV_16SC3, 4);

#if defined(_WIN32) && !defined(_WIN64)
    if (cv::ocl::useOpenCL())
        throw ::perf::TestBase::PerfSkipTestException();
#endif

    UMat pano;

    vector<Mat> _imgs;
    _imgs.push_back( imread( getDataPath("stitching/boat1.jpg") ) );
    _imgs.push_back( imread( getDataPath("stitching/boat2.jpg") ) );
    _imgs.push_back( imread( getDataPath("stitching/boat3.jpg") ) );
    _imgs.push_back( imread( getDataPath("stitching/boat4.jpg") ) );
    _imgs.push_back( imread( getDataPath("stitching/boat5.jpg") ) );
    _imgs.push_back( imread( getDataPath("stitching/boat6.jpg") ) );
    vector<UMat> imgs = ToUMat(_imgs);

    Ptr<Feature2D> featuresFinder = getFeatureFinder(GetParam());
    Ptr<detail::FeaturesMatcher> featuresMatcher = GetParam() == "orb"
            ? makePtr<detail::BestOf2NearestMatcher>(false, ORB_MATCH_CONFIDENCE)
            : makePtr<detail::BestOf2NearestMatcher>(false, SURF_MATCH_CONFIDENCE);

    declare.iterations(20);

    while(next())
    {
        Ptr<Stitcher> stitcher = Stitcher::create();
        stitcher->setFeaturesFinder(featuresFinder);
        stitcher->setFeaturesMatcher(featuresMatcher);
        stitcher->setWarper(makePtr<SphericalWarper>());
        stitcher->setRegistrationResol(WORK_MEGAPIX);

        startTimer();
        stitcher->stitch(imgs, pano);
        stopTimer();
    }

    EXPECT_NEAR(pano.size().width, expected_dst_size.width, 200);
    EXPECT_NEAR(pano.size().height, expected_dst_size.height, 100);

    SANITY_CHECK_NOTHING();
}

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
- **TestBaseWithParam()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_perf.hpp`
- `../perf_precomp.hpp`


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

