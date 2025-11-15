# Documentation for `docs/samples/cpp/tutorial_code/objectDetection/detect_board.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/objectDetection/detect_board.cpp_docs.md`
- **File Name**: `detect_board.cpp_docs.md`
- **File Size**: 8,754 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/objectDetection/detect_board.cpp_docs.md](../../../../../docs/samples/cpp/tutorial_code/objectDetection/detect_board.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/objectDetection/detect_board.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/objectDetection/detect_board.cpp`
- **File Name**: `detect_board.cpp`
- **File Size**: 5,497 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/objectDetection/detect_board.cpp](../../../../samples/cpp/tutorial_code/objectDetection/detect_board.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <vector>
#include <opencv2/highgui.hpp>
#include <opencv2/objdetect/aruco_detector.hpp>
#include "aruco_samples_utility.hpp"

using namespace std;
using namespace cv;

namespace {
const char* about = "Pose estimation using a ArUco Planar Grid board";

//! [aruco_detect_board_keys]
const char* keys  =
        "{w        |       | Number of squares in X direction }"
        "{h        |       | Number of squares in Y direction }"
        "{l        |       | Marker side length (in pixels) }"
        "{s        |       | Separation between two consecutive markers in the grid (in pixels)}"
        "{d        |       | dictionary: DICT_4X4_50=0, DICT_4X4_100=1, DICT_4X4_250=2,"
        "DICT_4X4_1000=3, DICT_5X5_50=4, DICT_5X5_100=5, DICT_5X5_250=6, DICT_5X5_1000=7, "
        "DICT_6X6_50=8, DICT_6X6_100=9, DICT_6X6_250=10, DICT_6X6_1000=11, DICT_7X7_50=12,"
        "DICT_7X7_100=13, DICT_7X7_250=14, DICT_7X7_1000=15, DICT_ARUCO_ORIGINAL = 16}"
        "{cd       |       | Input file with custom dictionary }"
        "{c        |       | Output file with calibrated camera parameters }"
        "{v        |       | Input from video or image file, if omitted, input comes from camera }"
        "{ci       | 0     | Camera id if input doesnt come from video (-v) }"
        "{dp       |       | File of marker detector parameters }"
        "{rs       |       | Apply refind strategy }"
        "{r        |       | show rejected candidates too }";
}
//! [aruco_detect_board_keys]

int main(int argc, char *argv[]) {
    CommandLineParser parser(argc, argv, keys);
    parser.about(about);

    if(argc < 7) {
        parser.printMessage();
        return 0;
    }

    //! [aruco_detect_board_full_sample]
    int markersX = parser.get<int>("w");
    int markersY = parser.get<int>("h");
    float markerLength = parser.get<float>("l");
    float markerSeparation = parser.get<float>("s");
    bool showRejected = parser.has("r");
    bool refindStrategy = parser.has("rs");
    int camId = parser.get<int>("ci");


    Mat camMatrix, distCoeffs;
    readCameraParamsFromCommandLine(parser, camMatrix, distCoeffs);
    aruco::Dictionary dictionary = readDictionatyFromCommandLine(parser);
    aruco::DetectorParameters detectorParams = readDetectorParamsFromCommandLine(parser);

    String video;
    if(parser.has("v")) {
        video = parser.get<String>("v");
    }

    if(!parser.check()) {
        parser.printErrors();
        return 0;
    }

    aruco::ArucoDetector detector(dictionary, detectorParams);
    VideoCapture inputVideo;
    int waitTime;
    if(!video.empty()) {
        inputVideo.open(video);
        waitTime = 0;
    } else {
        inputVideo.open(camId);
        waitTime = 10;
    }

    float axisLength = 0.5f * ((float)min(markersX, markersY) * (markerLength + markerSeparation) +
                               markerSeparation);

    // Create GridBoard object
    //! [aruco_create_board]
    aruco::GridBoard board(Size(markersX, markersY), markerLength, markerSeparation, dictionary);
    //! [aruco_create_board]

    // Also you could create Board object
    //vector<vector<Point3f> > objPoints; // array of object points of all the marker corners in the board
    //vector<int> ids; // vector of the identifiers of the markers in the board
    //aruco::Board board(objPoints, dictionary, ids);

    double totalTime = 0;
    int totalIterations = 0;

    while(inputVideo.grab()) {
        Mat image, imageCopy;
        inputVideo.retrieve(image);

        double tick = (double)getTickCount();

        vector<int> ids;
        vector<vector<Point2f>> corners, rejected;
        Vec3d rvec, tvec;

        //! [aruco_detect_and_refine]

        // Detect markers
        detector.detectMarkers(image, corners, ids, rejected);

        // Refind strategy to detect more markers
        if(refindStrategy)
            detector.refineDetectedMarkers(image, board, corners, ids, rejected, camMatrix,
                                           distCoeffs);

        //! [aruco_detect_and_refine]

        // Estimate board pose
        int markersOfBoardDetected = 0;
        if(!ids.empty()) {
            // Get object and image points for the solvePnP function
            cv::Mat objPoints, imgPoints;
            board.matchImagePoints(corners, ids, objPoints, imgPoints);

            // Find pose
            cv::solvePnP(objPoints, imgPoints, camMatrix, distCoeffs, rvec, tvec);

            markersOfBoardDetected = (int)objPoints.total() / 4;
        }

        double currentTime = ((double)getTickCount() - tick) / getTickFrequency();
        totalTime += currentTime;
        totalIterations++;
        if(totalIterations % 30 == 0) {
            cout << "Detection Time = " << currentTime * 1000 << " ms "
                 << "(Mean = " << 1000 * totalTime / double(totalIterations) << " ms)" << endl;
        }

        // Draw results
        image.copyTo(imageCopy);
        if(!ids.empty())
            aruco::drawDetectedMarkers(imageCopy, corners, ids);

        if(showRejected && !rejected.empty())
            aruco::drawDetectedMarkers(imageCopy, rejected, noArray(), Scalar(100, 0, 255));

        if(markersOfBoardDetected > 0)
            cv::drawFrameAxes(imageCopy, camMatrix, distCoeffs, rvec, tvec, axisLength);

        imshow("out", imageCopy);
        char key = (char)waitKey(waitTime);
        if(key == 27) break;
    //! [aruco_detect_board_full_sample]
    }

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

### Functions and Methods

- **cv()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `vector`
- `iostream`
- `opencv2/objdetect/aruco_detector.hpp`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

