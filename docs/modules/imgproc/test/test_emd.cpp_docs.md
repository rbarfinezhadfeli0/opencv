# Documentation for `modules/imgproc/test/test_emd.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/test/test_emd.cpp`
- **File Name**: `test_emd.cpp`
- **File Size**: 8,050 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/test/test_emd.cpp](../../../modules/imgproc/test/test_emd.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include "opencv2/imgproc.hpp"
#include "test_precomp.hpp"

using namespace cv;
using namespace std;

namespace opencv_test { namespace {

//==============================================================================
// Utility

template <typename T>
inline T sqr(T val)
{
    return val * val;
}

inline static float calcEMD(Mat w1, Mat w2, Mat& flow, int dist, int dims)
{
    float mass1 = 0.f, mass2 = 0.f, work = 0.f;
    for (int i = 0; i < flow.rows; ++i)
    {
        mass1 += w1.at<float>(i, 0);
        for (int j = 0; j < flow.cols; ++j)
        {
            if (i == 0)
                mass2 += w2.at<float>(j, 0);
            float dist_ = 0.f;
            switch (dist)
            {
                case DIST_L1:
                {
                    for (int k = 1; k <= dims; ++k)
                    {
                        dist_ += abs(w1.at<float>(i, k) - w2.at<float>(j, k));
                    }
                    break;
                }
                case DIST_L2:
                {
                    for (int k = 1; k <= dims; ++k)
                    {
                        dist_ += sqr(w1.at<float>(i, k) - w2.at<float>(j, k));
                    }
                    dist_ = sqrt(dist_);
                    break;
                }
                case DIST_C:
                {
                    for (int k = 1; k <= dims; ++k)
                    {
                        const float val = abs(w1.at<float>(i, k) - w2.at<float>(j, k));
                        if (val > dist_)
                            dist_ = val;
                    }
                    break;
                }
            }
            const float weight = flow.at<float>(i, j);
            work += dist_ * weight;
        }
    }
    return work / max(mass1, mass2);
}

//==============================================================================

TEST(Imgproc_EMD, regression)
{
    // input data
    const float M = 10000;
    Matx<float, 4, 1> w1 {50, 60, 50, 50};
    Matx<float, 5, 1> w2 {30, 20, 70, 30, 60};
    Matx<float, 4, 5> cost {16, 16, 13, 22, 17, 14, 14, 13, 19, 15,
                            19, 19, 20, 23, M,  M,  0,  M,  0,  0};

    // expected results
    const double emd0 = 2460. / 210;
    Matx<float, 4, 5> flow0 {0, 0, 50, 0, 0, 0, 0, 20, 0, 40, 30, 20, 0, 0, 0, 0, 0, 0, 30, 20};

    // basic call with cost
    {
        float emd = 0.f;
        ASSERT_NO_THROW(emd = EMD(w1, w2, DIST_USER, cost));
        EXPECT_NEAR(emd, emd0, 1e-6 * emd0);
    }

    // basic call with cost and flow output
    {
        Mat flow;
        float emd = 0.f;
        ASSERT_NO_THROW(emd = EMD(w1, w2, DIST_USER, cost, nullptr, flow));
        EXPECT_NEAR(emd, emd0, 1e-6 * emd0);
        EXPECT_MAT_NEAR(Mat(flow0), flow, 1e-6);
    }
    // no cost and DIST_USER - error
    {
        Mat flow;
        EXPECT_THROW(EMD(w1, w2, DIST_USER, noArray(), nullptr, flow), cv::Exception);
        EXPECT_THROW(EMD(w1, w2, DIST_USER), cv::Exception);
    }
}

TEST(Imgproc_EMD, distance_types)
{
    // 1D (sum = 210)
    Matx<float, 4, 2> w1 {50, 1, 60, 2, 50, 3, 50, 4};
    Matx<float, 5, 2> w2 {30, 1, 20, 2, 70, 3, 30, 4, 60, 5};

    // 2D (sum = 210)
    Matx<float, 4, 3> w3 {50, 0, 0, 60, 0, 1, 50, 1, 0, 50, 1, 1};
    Matx<float, 5, 3> w4 {20, 0, 1, 70, 1, 0, 30, 1, 1, 60, 2, 2, 30, 3, 3};

    // basic call with all distance types
    {
        const vector<DistanceTypes> good_types {DIST_L1, DIST_L2, DIST_C};
        for (const auto& dt : good_types)
        {
            SCOPED_TRACE(cv::format("dt=%d", dt));
            float emd = 0.f;
            Mat flow;
            // 1D
            {
                ASSERT_NO_THROW(emd = EMD(w1, w2, dt, noArray(), nullptr, flow));
                const float emd0 = calcEMD(Mat(w1), Mat(w2), flow, dt, 1);
                EXPECT_NEAR(emd0, emd, 1e-6);
            }
            // 2D
            {
                ASSERT_NO_THROW(emd = EMD(w3, w4, dt, noArray(), nullptr, flow));
                const float emd0 = calcEMD(Mat(w3), Mat(w4), flow, dt, 2);
                EXPECT_NEAR(emd0, emd, 1e-6);
            }
        }
    }
}

typedef testing::TestWithParam<int> Imgproc_EMD_dist;

TEST_P(Imgproc_EMD_dist, random_flow_verify)
{
    const int dist = GetParam();
    for (size_t iter = 0; iter < 100; ++iter)
    {
        SCOPED_TRACE(cv::format("iter=%zu", iter));
        RNG& rng = TS::ptr()->get_rng();
        const int dims = rng.uniform(1, 10);
        Mat w1(rng.uniform(1, 10), dims + 1, CV_32FC1);
        Mat w2(rng.uniform(1, 10), dims + 1, CV_32FC1);

        // weights > 0
        {
            Mat w1_weights = w1.col(0);
            Mat w2_weights = w2.col(0);
            cvtest::randUni(rng, w1_weights, 0, 100);
            cvtest::randUni(rng, w2_weights, 0, 100);
        }

        // coord
        {
            Mat w1_coord = w1.colRange(1, dims + 1);
            Mat w2_coord = w2.colRange(1, dims + 1);
            cvtest::randUni(rng, w1_coord, -10, +10);
            cvtest::randUni(rng, w2_coord, -10, +10);
        }

        float emd1 = 0.f, emd2 = 0.f;
        const float eps = 1e-5f;
        Mat flow;
        {
            ASSERT_NO_THROW(emd1 = EMD(w1, w2, dist, noArray(), nullptr, flow));
            const float emd0 = calcEMD(w1, w2, flow, dist, dims);
            EXPECT_NEAR(emd0, emd1, eps);
        }
        {
            ASSERT_NO_THROW(emd2 = EMD(w2, w1, dist, noArray(), nullptr, flow));
            const float emd0 = calcEMD(w2, w1, flow, dist, dims);
            EXPECT_NEAR(emd0, emd2, eps);
        }
        EXPECT_NEAR(emd1, emd2, eps);
    }
}

INSTANTIATE_TEST_CASE_P(, Imgproc_EMD_dist, testing::Values(DIST_L1, DIST_L2, DIST_C));


TEST(Imgproc_EMD, invalid)
{
    Matx<float, 4, 2> w1 {50, 1, 60, 2, 50, 3, 50, 4};
    Matx<float, 5, 2> w2 {30, 1, 20, 2, 70, 3, 30, 4, 60, 5};

    // empty signature
    {
        Mat empty;
        EXPECT_THROW(EMD(empty, w2, DIST_USER), cv::Exception);
        EXPECT_THROW(EMD(w1, empty, DIST_USER), cv::Exception);
    }

    // zero total weight, negative weight
    {
        Matx<float, 3, 1> wz {0, 0, 0};
        Matx<float, 3, 2> wz1 {0, 1, 0, 2, 0, 3};
        Matx<float, 3, 1> wn {0, 3, -2};
        Matx<float, 3, 2> wn1 {0, 1, 3, 2, -2, 3};
        EXPECT_THROW(EMD(wz, w2, DIST_USER), cv::Exception);
        EXPECT_THROW(EMD(wz1, w2, DIST_USER), cv::Exception);
        EXPECT_THROW(EMD(wn, w2, DIST_USER), cv::Exception);
        EXPECT_THROW(EMD(wn1, w2, DIST_USER), cv::Exception);
    }

    // user distance type, but no cost matrix provided or is wrong
    {
        Mat cost(3, 3, CV_32FC1, Scalar::all(0)), cost8u(4, 5, CV_8UC1, Scalar::all(0)), empty;
        EXPECT_THROW(EMD(w1, w2, DIST_USER, noArray()), cv::Exception);
        EXPECT_THROW(EMD(w1, w2, DIST_USER, empty), cv::Exception);
        EXPECT_THROW(EMD(w1, w2, DIST_USER, cost8u), cv::Exception);
        EXPECT_THROW(EMD(w1, w2, DIST_USER, cost), cv::Exception);
    }

    // lower_bound is set together with cost
    {
        Mat cost(4, 5, CV_32FC1, Scalar::all(0));
        float bound = 0.f;
        EXPECT_THROW(EMD(w1, w2, DIST_USER, cost, &bound), cv::Exception);
    }

    // zero dimensions with non-user distance type
    const vector<DistanceTypes> good_types {DIST_L1, DIST_L2, DIST_C};
    for (const auto& dt : good_types)
    {
        SCOPED_TRACE(cv::format("dt=%d", dt));
        Matx<float, 4, 1> w01 {20, 30, 40, 50};
        Matx<float, 5, 1> w02 {20, 30, 40, 50, 10};
        EXPECT_THROW(EMD(w01, w02, dt), cv::Exception);
    }

    // wrong distance type
    const vector<DistanceTypes> bad_types {DIST_L12, DIST_FAIR, DIST_WELSCH, DIST_HUBER};
    for (const auto& dt : bad_types)
    {
        SCOPED_TRACE(cv::format("dt=%d", dt));
        EXPECT_THROW(EMD(w1, w2, dt), cv::Exception);
    }
}

}}  // namespace opencv_test
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

- **testing()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/imgproc.hpp`
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

