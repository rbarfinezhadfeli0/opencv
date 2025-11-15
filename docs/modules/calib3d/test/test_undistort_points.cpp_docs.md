# Documentation for `modules/calib3d/test/test_undistort_points.cpp`

## File Metadata

- **Full Path**: `modules/calib3d/test/test_undistort_points.cpp`
- **File Name**: `test_undistort_points.cpp`
- **File Size**: 6,995 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/calib3d/test/test_undistort_points.cpp](../../../modules/calib3d/test/test_undistort_points.cpp)

## Purpose and Role

This file is located in the `modules/calib3d/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include <opencv2/ts/cuda_test.hpp> // EXPECT_MAT_NEAR
#include "opencv2/core/types.hpp"
#include "test_precomp.hpp"

namespace opencv_test { namespace {

class UndistortPointsTest : public ::testing::Test
{
protected:
    void generate3DPointCloud(vector<Point3f>& points, Point3f pmin = Point3f(-1,
    -1, 5), Point3f pmax = Point3f(1, 1, 10));
    void generateCameraMatrix(Mat& cameraMatrix);
    void generateDistCoeffs(Mat& distCoeffs, int count);
    cv::Mat generateRotationVector();

    double thresh = 1.0e-2;
};

void UndistortPointsTest::generate3DPointCloud(vector<Point3f>& points, Point3f pmin, Point3f pmax)
{
    RNG rng_Point = cv::theRNG(); // fix the seed to use "fixed" input 3D points
    for (size_t i = 0; i < points.size(); i++)
    {
        float _x = rng_Point.uniform(pmin.x, pmax.x);
        float _y = rng_Point.uniform(pmin.y, pmax.y);
        float _z = rng_Point.uniform(pmin.z, pmax.z);
        points[i] = Point3f(_x, _y, _z);
    }
}

void UndistortPointsTest::generateCameraMatrix(Mat& cameraMatrix)
{
    const double fcMinVal = 1e-3;
    const double fcMaxVal = 100;
    cameraMatrix.create(3, 3, CV_64FC1);
    cameraMatrix.setTo(Scalar(0));
    cameraMatrix.at<double>(0,0) = theRNG().uniform(fcMinVal, fcMaxVal);
    cameraMatrix.at<double>(1,1) = theRNG().uniform(fcMinVal, fcMaxVal);
    cameraMatrix.at<double>(0,2) = theRNG().uniform(fcMinVal, fcMaxVal);
    cameraMatrix.at<double>(1,2) = theRNG().uniform(fcMinVal, fcMaxVal);
    cameraMatrix.at<double>(2,2) = 1;
}

void UndistortPointsTest::generateDistCoeffs(Mat& distCoeffs, int count)
{
    distCoeffs = Mat::zeros(count, 1, CV_64FC1);
    for (int i = 0; i < count; i++)
        distCoeffs.at<double>(i,0) = theRNG().uniform(-0.1, 0.1);
}

cv::Mat UndistortPointsTest::generateRotationVector()
{
    Mat rvec(1, 3, CV_64F);
    theRNG().fill(rvec, RNG::UNIFORM, -0.2, 0.2);

    return rvec;
}

TEST_F(UndistortPointsTest, accuracy)
{
    Mat intrinsics, distCoeffs;
    generateCameraMatrix(intrinsics);

    vector<Point3f> points(500);
    generate3DPointCloud(points);

    Mat rvec = generateRotationVector();
    Mat R;
    cv::Rodrigues(rvec, R);


    int modelMembersCount[] = {4,5,8};
    for (int idx = 0; idx < 3; idx++)
    {
        generateDistCoeffs(distCoeffs, modelMembersCount[idx]);

        /* Project points with distortion */
        vector<Point2f> projectedPoints;
        projectPoints(Mat(points), Mat::zeros(3,1,CV_64FC1),
                      Mat::zeros(3,1,CV_64FC1), intrinsics,
                      distCoeffs, projectedPoints);

        /* Project points without distortion */
        vector<Point2f> realUndistortedPoints;
        projectPoints(Mat(points), rvec,
                      Mat::zeros(3,1,CV_64FC1), intrinsics,
                      Mat::zeros(4,1,CV_64FC1), realUndistortedPoints);

        /* Undistort points */
        Mat undistortedPoints;
        undistortPoints(Mat(projectedPoints), undistortedPoints, intrinsics, distCoeffs, R, intrinsics);

        EXPECT_MAT_NEAR(realUndistortedPoints, undistortedPoints.t(), thresh);
    }
}

TEST_F(UndistortPointsTest, undistortImagePointsAccuracy)
{
    Mat intrinsics, distCoeffs;
    generateCameraMatrix(intrinsics);

    vector<Point3f> points(500);
    generate3DPointCloud(points);


    int modelMembersCount[] = {4,5,8};
    for (int idx = 0; idx < 3; idx++)
    {
        generateDistCoeffs(distCoeffs, modelMembersCount[idx]);

        /* Project points with distortion */
        vector<Point2f> projectedPoints;
        projectPoints(Mat(points), Mat::zeros(3,1,CV_64FC1),
                      Mat::zeros(3,1,CV_64FC1), intrinsics,
                      distCoeffs, projectedPoints);

        /* Project points without distortion */
        vector<Point2f> realUndistortedPoints;
        projectPoints(Mat(points), Mat::zeros(3, 1, CV_64FC1),
                      Mat::zeros(3,1,CV_64FC1), intrinsics,
                      Mat::zeros(4,1,CV_64FC1), realUndistortedPoints);

        /* Undistort points */
        Mat undistortedPoints;
        TermCriteria termCriteria(TermCriteria::MAX_ITER + TermCriteria::EPS, 5, thresh / 2);
        undistortImagePoints(Mat(projectedPoints), undistortedPoints, intrinsics, distCoeffs,
                             termCriteria);

        EXPECT_MAT_NEAR(realUndistortedPoints, undistortedPoints.t(), thresh);
    }
}


TEST_F(UndistortPointsTest, stop_criteria)
{
    Mat cameraMatrix = (Mat_<double>(3,3,CV_64F) << 857.48296979, 0, 968.06224829,
                                                        0, 876.71824265, 556.37145899,
                                                        0, 0, 1);
    Mat distCoeffs = (Mat_<double>(5,1,CV_64F) <<
                      -2.57614020e-01, 8.77086999e-02, -2.56970803e-04, -5.93390389e-04, -1.52194091e-02);

    Point2d pt_distorted(theRNG().uniform(0.0, 1920.0), theRNG().uniform(0.0, 1080.0));

    std::vector<Point2d> pt_distorted_vec;
    pt_distorted_vec.push_back(pt_distorted);

    const double maxError = 1e-6;
    TermCriteria criteria(TermCriteria::MAX_ITER + TermCriteria::EPS, 100, maxError);

    std::vector<Point2d> pt_undist_vec;
    Mat rVec = Mat(Matx31d(0.1, -0.2, 0.2));
    Mat R;
    cv::Rodrigues(rVec, R);

    undistortPoints(pt_distorted_vec, pt_undist_vec, cameraMatrix, distCoeffs, R, noArray(), criteria);

    std::vector<Point3d> pt_undist_vec_homogeneous;
    pt_undist_vec_homogeneous.emplace_back(pt_undist_vec[0].x, pt_undist_vec[0].y, 1.0 );

    std::vector<Point2d> pt_redistorted_vec;
    projectPoints(pt_undist_vec_homogeneous, -rVec,
                  Mat::zeros(3,1,CV_64F), cameraMatrix, distCoeffs, pt_redistorted_vec);

    const double obtainedError = sqrt( pow(pt_distorted.x - pt_redistorted_vec[0].x, 2) + pow(pt_distorted.y - pt_redistorted_vec[0].y, 2) );

    ASSERT_LE(obtainedError, maxError);
}

TEST_F(UndistortPointsTest, regression_14583)
{
    const int col = 720;
    // const int row = 540;
    float camera_matrix_value[] = {
        437.8995f, 0.0f, 342.9241f,
        0.0f, 438.8216f, 273.7163f,
        0.0f, 0.0f,      1.0f
    };
    cv::Mat camera_interior(3, 3, CV_32F, camera_matrix_value);

    float camera_distort_value[] = {-0.34329f, 0.11431f, 0.0f, 0.0f, -0.017375f};
    cv::Mat camera_distort(1, 5, CV_32F, camera_distort_value);

    float distort_points_value[] = {col, 0.};
    cv::Mat distort_pt(1, 1, CV_32FC2, distort_points_value);

    cv::Mat undistort_pt;
    cv::undistortPoints(distort_pt, undistort_pt, camera_interior,
                        camera_distort, cv::Mat(), camera_interior);

    EXPECT_NEAR(distort_pt.at<Vec2f>(0)[0], undistort_pt.at<Vec2f>(0)[0], col / 2)
        << "distort point: " << distort_pt << std::endl
        << "undistort point: " << undistort_pt;
}

}} // namespace
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

### Classes and Structures

- **UndistortPointsTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/cuda_test.hpp`
- `test_precomp.hpp`
- `opencv2/core/types.hpp`


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

