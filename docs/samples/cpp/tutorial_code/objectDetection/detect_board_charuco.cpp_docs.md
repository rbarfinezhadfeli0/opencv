# Documentation for `samples/cpp/tutorial_code/objectDetection/detect_board_charuco.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/objectDetection/detect_board_charuco.cpp`
- **File Name**: `detect_board_charuco.cpp`
- **File Size**: 5,290 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/objectDetection/detect_board_charuco.cpp](../../../../samples/cpp/tutorial_code/objectDetection/detect_board_charuco.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/highgui.hpp>
//! [charucohdr]
#include <opencv2/objdetect/charuco_detector.hpp>
//! [charucohdr]
#include <vector>
#include <iostream>
#include "aruco_samples_utility.hpp"

using namespace std;
using namespace cv;

namespace {
const char* about = "Pose estimation using a ChArUco board";
const char* keys  =
        "{w        |       | Number of squares in X direction }"
        "{h        |       | Number of squares in Y direction }"
        "{sl       |       | Square side length (in meters) }"
        "{ml       |       | Marker side length (in meters) }"
        "{d        |       | dictionary: DICT_4X4_50=0, DICT_4X4_100=1, DICT_4X4_250=2,"
        "DICT_4X4_1000=3, DICT_5X5_50=4, DICT_5X5_100=5, DICT_5X5_250=6, DICT_5X5_1000=7, "
        "DICT_6X6_50=8, DICT_6X6_100=9, DICT_6X6_250=10, DICT_6X6_1000=11, DICT_7X7_50=12,"
        "DICT_7X7_100=13, DICT_7X7_250=14, DICT_7X7_1000=15, DICT_ARUCO_ORIGINAL = 16}"
        "{cd       |       | Input file with custom dictionary }"
        "{c        |       | Output file with calibrated camera parameters }"
        "{v        |       | Input from video or image file, if ommited, input comes from camera }"
        "{ci       | 0     | Camera id if input doesnt come from video (-v) }"
        "{dp       |       | File of marker detector parameters }"
        "{rs       |       | Apply refind strategy }";
}


int main(int argc, char *argv[]) {
    CommandLineParser parser(argc, argv, keys);
    parser.about(about);

    if(argc < 6) {
        parser.printMessage();
        return 0;
    }

    //! [charuco_detect_board_full_sample]
    int squaresX = parser.get<int>("w");
    int squaresY = parser.get<int>("h");
    float squareLength = parser.get<float>("sl");
    float markerLength = parser.get<float>("ml");
    bool refine = parser.has("rs");
    int camId = parser.get<int>("ci");

    string video;
    if(parser.has("v")) {
        video = parser.get<string>("v");
    }

    Mat camMatrix, distCoeffs;
    readCameraParamsFromCommandLine(parser, camMatrix, distCoeffs);
    aruco::DetectorParameters detectorParams = readDetectorParamsFromCommandLine(parser);
    aruco::Dictionary dictionary = readDictionatyFromCommandLine(parser);

    if(!parser.check()) {
        parser.printErrors();
        return 0;
    }

    VideoCapture inputVideo;
    int waitTime = 0;
    if(!video.empty()) {
        inputVideo.open(video);
    } else {
        inputVideo.open(camId);
        waitTime = 10;
    }

    float axisLength = 0.5f * ((float)min(squaresX, squaresY) * (squareLength));

    // create charuco board object
    aruco::CharucoBoard charucoBoard(Size(squaresX, squaresY), squareLength, markerLength, dictionary);

    // create charuco detector
    aruco::CharucoParameters charucoParams;
    charucoParams.tryRefineMarkers = refine; // if tryRefineMarkers, refineDetectedMarkers() will be used in detectBoard()
    charucoParams.cameraMatrix = camMatrix; // cameraMatrix can be used in detectBoard()
    charucoParams.distCoeffs = distCoeffs; // distCoeffs can be used in detectBoard()
    aruco::CharucoDetector charucoDetector(charucoBoard, charucoParams, detectorParams);

    double totalTime = 0;
    int totalIterations = 0;

    while(inputVideo.grab()) {
        //! [inputImg]
        Mat image, imageCopy;
        inputVideo.retrieve(image);
        //! [inputImg]

        double tick = (double)getTickCount();

        vector<int> markerIds, charucoIds;
        vector<vector<Point2f> > markerCorners;
        vector<Point2f> charucoCorners;
        Vec3d rvec, tvec;

        //! [interpolateCornersCharuco]
        // detect markers and charuco corners
        charucoDetector.detectBoard(image, charucoCorners, charucoIds, markerCorners, markerIds);
        //! [interpolateCornersCharuco]

        //! [poseCharuco]
        // estimate charuco board pose
        bool validPose = false;
        if(camMatrix.total() != 0 && distCoeffs.total() != 0 && charucoIds.size() >= 4) {
            Mat objPoints, imgPoints;
            charucoBoard.matchImagePoints(charucoCorners, charucoIds, objPoints, imgPoints);
            validPose = solvePnP(objPoints, imgPoints, camMatrix, distCoeffs, rvec, tvec);
        }
        //! [poseCharuco]

        double currentTime = ((double)getTickCount() - tick) / getTickFrequency();
        totalTime += currentTime;
        totalIterations++;
        if(totalIterations % 30 == 0) {
            cout << "Detection Time = " << currentTime * 1000 << " ms "
                 << "(Mean = " << 1000 * totalTime / double(totalIterations) << " ms)" << endl;
        }

        // draw results
        image.copyTo(imageCopy);
        if(markerIds.size() > 0) {
            aruco::drawDetectedMarkers(imageCopy, markerCorners);
        }

        if(charucoIds.size() > 0) {
            //! [drawDetectedCornersCharuco]
            aruco::drawDetectedCornersCharuco(imageCopy, charucoCorners, charucoIds, cv::Scalar(255, 0, 0));
            //! [drawDetectedCornersCharuco]
        }

        if(validPose)
            cv::drawFrameAxes(imageCopy, camMatrix, distCoeffs, rvec, tvec, axisLength);

        imshow("out", imageCopy);
        if(waitKey(waitTime) == 27) break;
    }
    //! [charuco_detect_board_full_sample]
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
- `opencv2/objdetect/charuco_detector.hpp`
- `vector`
- `iostream`
- `opencv2/highgui.hpp`
- `aruco_samples_utility.hpp`

**Python Imports:**
- `video`
- `camera`


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

