# Documentation for `modules/stitching/perf/perf_estimators.cpp`

## File Metadata

- **Full Path**: `modules/stitching/perf/perf_estimators.cpp`
- **File Name**: `perf_estimators.cpp`
- **File Size**: 3,660 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/stitching/perf/perf_estimators.cpp](../../../modules/stitching/perf/perf_estimators.cpp)

## Purpose and Role

This file is located in the `modules/stitching/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "perf_precomp.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/opencv_modules.hpp"

namespace opencv_test
{
using namespace perf;

typedef TestBaseWithParam<tuple<string, string> > bundleAdjuster;

#if defined(HAVE_OPENCV_XFEATURES2D) && defined(OPENCV_ENABLE_NONFREE)
#define TEST_DETECTORS testing::Values("surf", "orb")
#else
#define TEST_DETECTORS testing::Values<string>("orb")
#endif
#define WORK_MEGAPIX 0.6
#define AFFINE_FUNCTIONS testing::Values("affinePartial", "affine")

PERF_TEST_P(bundleAdjuster, affine, testing::Combine(TEST_DETECTORS, AFFINE_FUNCTIONS))
{
    Mat img1, img1_full = imread(getDataPath("stitching/s1.jpg"));
    Mat img2, img2_full = imread(getDataPath("stitching/s2.jpg"));
    float scale1 = (float)std::min(1.0, sqrt(WORK_MEGAPIX * 1e6 / img1_full.total()));
    float scale2 = (float)std::min(1.0, sqrt(WORK_MEGAPIX * 1e6 / img2_full.total()));
    resize(img1_full, img1, Size(), scale1, scale1, INTER_LINEAR_EXACT);
    resize(img2_full, img2, Size(), scale2, scale2, INTER_LINEAR_EXACT);

    string detector = get<0>(GetParam());
    string affine_fun = get<1>(GetParam());

    Ptr<Feature2D> finder = getFeatureFinder(detector);
    Ptr<detail::FeaturesMatcher> matcher;
    Ptr<detail::BundleAdjusterBase> bundle_adjuster;
    if (affine_fun == "affinePartial")
    {
        matcher = makePtr<detail::AffineBestOf2NearestMatcher>(false);
        bundle_adjuster = makePtr<detail::BundleAdjusterAffinePartial>();
    }
    else if (affine_fun == "affine")
    {
        matcher = makePtr<detail::AffineBestOf2NearestMatcher>(true);
        bundle_adjuster = makePtr<detail::BundleAdjusterAffine>();
    }
    Ptr<detail::Estimator> estimator = makePtr<detail::AffineBasedEstimator>();

    std::vector<Mat> images;
    images.push_back(img1), images.push_back(img2);
    std::vector<detail::ImageFeatures> features;
    std::vector<detail::MatchesInfo> pairwise_matches;
    std::vector<detail::CameraParams> cameras;
    std::vector<detail::CameraParams> cameras2;

    computeImageFeatures(finder, images, features);
    (*matcher)(features, pairwise_matches);
    if (!(*estimator)(features, pairwise_matches, cameras))
        FAIL() << "estimation failed. this should never happen.";
    // this is currently required
    for (size_t i = 0; i < cameras.size(); ++i)
    {
        Mat R;
        cameras[i].R.convertTo(R, CV_32F);
        cameras[i].R = R;
    }

    cameras2 = cameras;
    bool success = true;
    while(next())
    {
        cameras = cameras2; // revert cameras back to original initial guess
        startTimer();
        success = (*bundle_adjuster)(features, pairwise_matches, cameras);
        stopTimer();
    }

    EXPECT_TRUE(success);
    EXPECT_TRUE(cameras.size() == 2);

    // fist camera should be just identity
    Mat &first = cameras[0].R;
    SANITY_CHECK(first, 1e-3, ERROR_ABSOLUTE);
    // second camera should be the estimated transform between images
    // separate rotation and translation in transform matrix
    Mat T_second (cameras[1].R, Range(0, 2), Range(2, 3));
    Mat R_second (cameras[1].R, Range(0, 2), Range(0, 2));
    Mat h (cameras[1].R, Range(2, 3), Range::all());
    SANITY_CHECK(T_second, 5, ERROR_ABSOLUTE); // allow 5 pixels diff in translations
    SANITY_CHECK(R_second, .01, ERROR_ABSOLUTE); // rotations must be more precise
    // last row should be precisely (0, 0, 1) as it is just added for representation in homogeneous
    // coordinates
    EXPECT_TRUE(h.type() == CV_32F);
    EXPECT_FLOAT_EQ(h.at<float>(0), 0.f);
    EXPECT_FLOAT_EQ(h.at<float>(1), 0.f);
    EXPECT_FLOAT_EQ(h.at<float>(2), 1.f);
}

} // namespace
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

- **TestBaseWithParam()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/opencv_modules.hpp`
- `opencv2/imgcodecs.hpp`
- `perf_precomp.hpp`


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

