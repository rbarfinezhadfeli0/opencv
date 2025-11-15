# Documentation for `docs/samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp_docs.md`
- **File Name**: `pose_from_homography.cpp_docs.md`
- **File Size**: 8,543 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp_docs.md](../../../../../../docs/samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp`
- **File Name**: `pose_from_homography.cpp`
- **File Size**: 5,298 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp](../../../../../samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/calib3d.hpp>
#include <opencv2/highgui.hpp>

using namespace std;
using namespace cv;

namespace
{
enum Pattern { CHESSBOARD, CIRCLES_GRID, ASYMMETRIC_CIRCLES_GRID };

void calcChessboardCorners(Size boardSize, float squareSize, vector<Point3f>& corners, Pattern patternType = CHESSBOARD)
{
    corners.resize(0);

    switch (patternType)
    {
    case CHESSBOARD:
    case CIRCLES_GRID:
        //! [compute-chessboard-object-points]
        for( int i = 0; i < boardSize.height; i++ )
            for( int j = 0; j < boardSize.width; j++ )
                corners.push_back(Point3f(float(j*squareSize),
                                          float(i*squareSize), 0));
        //! [compute-chessboard-object-points]
        break;

    case ASYMMETRIC_CIRCLES_GRID:
        for( int i = 0; i < boardSize.height; i++ )
            for( int j = 0; j < boardSize.width; j++ )
                corners.push_back(Point3f(float((2*j + i % 2)*squareSize),
                                          float(i*squareSize), 0));
        break;

    default:
        CV_Error(Error::StsBadArg, "Unknown pattern type\n");
    }
}

void poseEstimationFromCoplanarPoints(const string &imgPath, const string &intrinsicsPath, const Size &patternSize,
                                             const float squareSize)
{
    Mat img = imread( samples::findFile( imgPath) );
    Mat img_corners = img.clone(), img_pose = img.clone();

    //! [find-chessboard-corners]
    vector<Point2f> corners;
    bool found = findChessboardCorners(img, patternSize, corners);
    //! [find-chessboard-corners]

    if (!found)
    {
        cout << "Cannot find chessboard corners." << endl;
        return;
    }
    drawChessboardCorners(img_corners, patternSize, corners, found);
    imshow("Chessboard corners detection", img_corners);

    //! [compute-object-points]
    vector<Point3f> objectPoints;
    calcChessboardCorners(patternSize, squareSize, objectPoints);
    vector<Point2f> objectPointsPlanar;
    for (size_t i = 0; i < objectPoints.size(); i++)
    {
        objectPointsPlanar.push_back(Point2f(objectPoints[i].x, objectPoints[i].y));
    }
    //! [compute-object-points]

    //! [load-intrinsics]
    FileStorage fs( samples::findFile( intrinsicsPath ), FileStorage::READ);
    Mat cameraMatrix, distCoeffs;
    fs["camera_matrix"] >> cameraMatrix;
    fs["distortion_coefficients"] >> distCoeffs;
    //! [load-intrinsics]

    //! [compute-image-points]
    vector<Point2f> imagePoints;
    undistortPoints(corners, imagePoints, cameraMatrix, distCoeffs);
    //! [compute-image-points]

    //! [estimate-homography]
    Mat H = findHomography(objectPointsPlanar, imagePoints);
    cout << "H:\n" << H << endl;
    //! [estimate-homography]

    //! [pose-from-homography]
    // Normalization to ensure that ||c1|| = 1
    double norm = sqrt(H.at<double>(0,0)*H.at<double>(0,0) +
                       H.at<double>(1,0)*H.at<double>(1,0) +
                       H.at<double>(2,0)*H.at<double>(2,0));

    H /= norm;
    Mat c1  = H.col(0);
    Mat c2  = H.col(1);
    Mat c3 = c1.cross(c2);

    Mat tvec = H.col(2);
    Mat R(3, 3, CV_64F);

    for (int i = 0; i < 3; i++)
    {
        R.at<double>(i,0) = c1.at<double>(i,0);
        R.at<double>(i,1) = c2.at<double>(i,0);
        R.at<double>(i,2) = c3.at<double>(i,0);
    }
    //! [pose-from-homography]

    //! [polar-decomposition-of-the-rotation-matrix]
    cout << "R (before polar decomposition):\n" << R << "\ndet(R): " << determinant(R) << endl;
    Mat_<double> W, U, Vt;
    SVDecomp(R, W, U, Vt);
    R = U*Vt;
    double det = determinant(R);
    if (det < 0)
    {
        Vt.at<double>(2,0) *= -1;
        Vt.at<double>(2,1) *= -1;
        Vt.at<double>(2,2) *= -1;

        R = U*Vt;
    }
    cout << "R (after polar decomposition):\n" << R << "\ndet(R): " << determinant(R) << endl;
    //! [polar-decomposition-of-the-rotation-matrix]

    //! [display-pose]
    Mat rvec;
    Rodrigues(R, rvec);
    drawFrameAxes(img_pose, cameraMatrix, distCoeffs, rvec, tvec, 2*squareSize);
    imshow("Pose from coplanar points", img_pose);
    waitKey();
    //! [display-pose]
}

const char* params
    = "{ help h         |       | print usage }"
      "{ image          | left04.jpg | path to a chessboard image }"
      "{ intrinsics     | left_intrinsics.yml | path to camera intrinsics }"
      "{ width bw       | 9     | chessboard width }"
      "{ height bh      | 6     | chessboard height }"
      "{ square_size    | 0.025 | chessboard square size }";
}

int main(int argc, char *argv[])
{
    CommandLineParser parser(argc, argv, params);

    if (parser.has("help"))
    {
        parser.about("Code for homography tutorial.\n"
            "Example 1: pose from homography with coplanar points.\n");
        parser.printMessage();
        return 0;
    }

    Size patternSize(parser.get<int>("width"), parser.get<int>("height"));
    float squareSize = (float) parser.get<double>("square_size");
    poseEstimationFromCoplanarPoints(parser.get<String>("image"),
                                     parser.get<String>("intrinsics"),
                                     patternSize, squareSize);

    return 0;
}
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
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/calib3d.hpp`

**Python Imports:**
- `coplanar`
- `homography`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

