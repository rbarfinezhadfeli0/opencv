# Documentation for `modules/features2d/test/test_descriptors_invariance.impl.hpp`

## File Metadata

- **Full Path**: `modules/features2d/test/test_descriptors_invariance.impl.hpp`
- **File Name**: `test_descriptors_invariance.impl.hpp`
- **File Size**: 7,105 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/features2d/test/test_descriptors_invariance.impl.hpp](../../../modules/features2d/test/test_descriptors_invariance.impl.hpp)

## Purpose and Role

This file is located in the `modules/features2d/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include "test_invariance_utils.hpp"
#include <functional>

namespace opencv_test { namespace {

#define SHOW_DEBUG_LOG 1

// NOTE: using factory function (function<Ptr<Type>()>) instead of object instance (Ptr<Type>) as a
// test parameter, because parameters exist during whole test program run and consume a lot of memory
typedef std::function<cv::Ptr<cv::FeatureDetector>()> DetectorFactory;
typedef std::function<cv::Ptr<cv::DescriptorExtractor>()> ExtractorFactory;
typedef tuple<std::string, DetectorFactory, ExtractorFactory, float>
    String_FeatureDetector_DescriptorExtractor_Float_t;


static
void SetSuitableSIFTOctave(vector<KeyPoint>& keypoints,
                             int firstOctave = -1, int nOctaveLayers = 3, double sigma = 1.6)
{
    for (size_t i = 0; i < keypoints.size(); i++ )
    {
        int octv, layer;
        KeyPoint& kpt = keypoints[i];
        double octv_layer = std::log(kpt.size / sigma) / std::log(2.) - 1;
        octv = cvFloor(octv_layer);
        layer = cvRound( (octv_layer - octv) * nOctaveLayers );
        if (octv < firstOctave)
        {
            octv = firstOctave;
            layer = 0;
        }
        kpt.octave = (layer << 8) | (octv & 255);
    }
}

static
void rotateKeyPoints(const vector<KeyPoint>& src, const Mat& H, float angle, vector<KeyPoint>& dst)
{
    // suppose that H is rotation given from rotateImage() and angle has value passed to rotateImage()
    vector<Point2f> srcCenters, dstCenters;
    KeyPoint::convert(src, srcCenters);

    perspectiveTransform(srcCenters, dstCenters, H);

    dst = src;
    for(size_t i = 0; i < dst.size(); i++)
    {
        dst[i].pt = dstCenters[i];
        float dstAngle = src[i].angle + angle;
        if(dstAngle >= 360.f)
            dstAngle -= 360.f;
        dst[i].angle = dstAngle;
    }
}

class DescriptorInvariance : public TestWithParam<String_FeatureDetector_DescriptorExtractor_Float_t>
{
protected:
    virtual void SetUp() {
        // Read test data
        const std::string filename = cvtest::TS::ptr()->get_data_path() + get<0>(GetParam());
        image0 = imread(filename);
        ASSERT_FALSE(image0.empty()) << "couldn't read input image";

        featureDetector = get<1>(GetParam())();
        descriptorExtractor = get<2>(GetParam())();
        minInliersRatio = get<3>(GetParam());
    }

    Ptr<FeatureDetector> featureDetector;
    Ptr<DescriptorExtractor> descriptorExtractor;
    float minInliersRatio;
    Mat image0;
};

typedef DescriptorInvariance DescriptorScaleInvariance;
typedef DescriptorInvariance DescriptorRotationInvariance;

TEST_P(DescriptorRotationInvariance, rotation)
{
    Mat image1, mask1;
    const int borderSize = 16;
    Mat mask0(image0.size(), CV_8UC1, Scalar(0));
    mask0(Rect(borderSize, borderSize, mask0.cols - 2*borderSize, mask0.rows - 2*borderSize)).setTo(Scalar(255));

    vector<KeyPoint> keypoints0;
    Mat descriptors0;
    featureDetector->detect(image0, keypoints0, mask0);
    std::cout << "Keypoints: " << keypoints0.size() << std::endl;
    EXPECT_GE(keypoints0.size(), 15u);
    descriptorExtractor->compute(image0, keypoints0, descriptors0);

    BFMatcher bfmatcher(descriptorExtractor->defaultNorm());

    const float minIntersectRatio = 0.5f;
    const int maxAngle = 360, angleStep = 15;
    for(int angle = 0; angle < maxAngle; angle += angleStep)
    {
        Mat H = rotateImage(image0, mask0, static_cast<float>(angle), image1, mask1);

        vector<KeyPoint> keypoints1;
        rotateKeyPoints(keypoints0, H, static_cast<float>(angle), keypoints1);
        Mat descriptors1;
        descriptorExtractor->compute(image1, keypoints1, descriptors1);

        vector<DMatch> descMatches;
        bfmatcher.match(descriptors0, descriptors1, descMatches);

        int descInliersCount = 0;
        for(size_t m = 0; m < descMatches.size(); m++)
        {
            const KeyPoint& transformed_p0 = keypoints1[descMatches[m].queryIdx];
            const KeyPoint& p1 = keypoints1[descMatches[m].trainIdx];
            if(calcIntersectRatio(transformed_p0.pt, 0.5f * transformed_p0.size,
                                  p1.pt, 0.5f * p1.size) >= minIntersectRatio)
            {
                descInliersCount++;
            }
        }

        float descInliersRatio = static_cast<float>(descInliersCount) / keypoints0.size();
        EXPECT_GE(descInliersRatio, minInliersRatio);
#if SHOW_DEBUG_LOG
        std::cout
            << "angle = " << angle
            << ", inliers = " << descInliersCount
            << ", descInliersRatio = " << static_cast<float>(descInliersCount) / keypoints0.size()
            << std::endl;
#endif
    }
}


TEST_P(DescriptorScaleInvariance, scale)
{
    vector<KeyPoint> keypoints0;
    featureDetector->detect(image0, keypoints0);
    std::cout << "Keypoints: " << keypoints0.size() << std::endl;
    EXPECT_GE(keypoints0.size(), 15u);
    Mat descriptors0;
    descriptorExtractor->compute(image0, keypoints0, descriptors0);

    BFMatcher bfmatcher(descriptorExtractor->defaultNorm());
    for(int scaleIdx = 1; scaleIdx <= 3; scaleIdx++)
    {
        float scale = 1.f + scaleIdx * 0.5f;

        Mat image1;
        resize(image0, image1, Size(), 1./scale, 1./scale, INTER_LINEAR_EXACT);

        vector<KeyPoint> keypoints1;
        scaleKeyPoints(keypoints0, keypoints1, 1.0f/scale);
        if (featureDetector->getDefaultName() == "Feature2D.SIFT")
        {
            SetSuitableSIFTOctave(keypoints1);
        }
        Mat descriptors1;
        descriptorExtractor->compute(image1, keypoints1, descriptors1);

        vector<DMatch> descMatches;
        bfmatcher.match(descriptors0, descriptors1, descMatches);

        const float minIntersectRatio = 0.5f;
        int descInliersCount = 0;
        for(size_t m = 0; m < descMatches.size(); m++)
        {
            const KeyPoint& transformed_p0 = keypoints0[descMatches[m].queryIdx];
            const KeyPoint& p1 = keypoints0[descMatches[m].trainIdx];
            if(calcIntersectRatio(transformed_p0.pt, 0.5f * transformed_p0.size,
                                  p1.pt, 0.5f * p1.size) >= minIntersectRatio)
            {
                descInliersCount++;
            }
        }

        float descInliersRatio = static_cast<float>(descInliersCount) / keypoints0.size();
        EXPECT_GE(descInliersRatio, minInliersRatio);
#if SHOW_DEBUG_LOG
        std::cout
            << "scale = " << scale
            << ", inliers = " << descInliersCount
            << ", descInliersRatio = " << static_cast<float>(descInliersCount) / keypoints0.size()
            << std::endl;
#endif
    }
}

#undef SHOW_DEBUG_LOG
}} // namespace

namespace std {
using namespace opencv_test;
static inline void PrintTo(const String_FeatureDetector_DescriptorExtractor_Float_t& v, std::ostream* os)
{
    *os << "(\"" << get<0>(v)
        << "\", " << get<3>(v)
        << ")";
}
} // namespace
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

- **DescriptorInvariance**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **SHOW_DEBUG_LOG()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file
- **DescriptorInvariance()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `functional`
- `test_invariance_utils.hpp`

**Python Imports:**
- `rotateImage`


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

