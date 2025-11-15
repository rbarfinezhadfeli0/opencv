# Documentation for `samples/cpp/tutorial_code/objectDetection/detect_diamonds.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/objectDetection/detect_diamonds.cpp`
- **File Name**: `detect_diamonds.cpp`
- **File Size**: 7,876 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/objectDetection/detect_diamonds.cpp](../../../../samples/cpp/tutorial_code/objectDetection/detect_diamonds.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/highgui.hpp>
#include <vector>
#include <iostream>
#include <opencv2/objdetect/charuco_detector.hpp>
#include "aruco_samples_utility.hpp"

using namespace std;
using namespace cv;


namespace {
const char* about = "Detect ChArUco markers";
const char* keys  =
        "{sl       | 100   | Square side length (in meters) }"
        "{ml       | 60    | Marker side length (in meters) }"
        "{d        | 10    | dictionary: DICT_4X4_50=0, DICT_4X4_100=1, DICT_4X4_250=2,"
        "DICT_4X4_1000=3, DICT_5X5_50=4, DICT_5X5_100=5, DICT_5X5_250=6, DICT_5X5_1000=7, "
        "DICT_6X6_50=8, DICT_6X6_100=9, DICT_6X6_250=10, DICT_6X6_1000=11, DICT_7X7_50=12,"
        "DICT_7X7_100=13, DICT_7X7_250=14, DICT_7X7_1000=15, DICT_ARUCO_ORIGINAL = 16}"
        "{cd       |       | Input file with custom dictionary }"
        "{c        |       | Output file with calibrated camera parameters }"
        "{as       |       | Automatic scale. The provided number is multiplied by the last"
        "diamond id becoming an indicator of the square length. In this case, the -sl and "
        "-ml are only used to know the relative length relation between squares and markers }"
        "{v        |       | Input from video file, if ommited, input comes from camera }"
        "{ci       | 0     | Camera id if input doesnt come from video (-v) }"
        "{dp       |       | File of marker detector parameters }"
        "{refine   |       | Corner refinement: CORNER_REFINE_NONE=0, CORNER_REFINE_SUBPIX=1,"
        "CORNER_REFINE_CONTOUR=2, CORNER_REFINE_APRILTAG=3}";

const string refineMethods[4] = {
    "None",
    "Subpixel",
    "Contour",
    "AprilTag"
};

}

int main(int argc, char *argv[]) {
    CommandLineParser parser(argc, argv, keys);
    parser.about(about);

    float squareLength = parser.get<float>("sl");
    float markerLength = parser.get<float>("ml");
    bool estimatePose = parser.has("c");
    bool autoScale = parser.has("as");
    float autoScaleFactor = autoScale ? parser.get<float>("as") : 1.f;

    aruco::Dictionary dictionary = readDictionatyFromCommandLine(parser);
    Mat camMatrix, distCoeffs;
    readCameraParamsFromCommandLine(parser, camMatrix, distCoeffs);

    aruco::DetectorParameters detectorParams = readDetectorParamsFromCommandLine(parser);
    if (parser.has("refine")) {
        // override cornerRefinementMethod read from config file
        int user_method = parser.get<aruco::CornerRefineMethod>("refine");
        if (user_method < 0 || user_method >= 4)
        {
            std::cout << "Corner refinement method should be in range 0..3" << std::endl;
            return 0;
        }
        detectorParams.cornerRefinementMethod = user_method;
    }
    std::cout << "Corner refinement method: " << refineMethods[detectorParams.cornerRefinementMethod] << std::endl;

    int camId = parser.get<int>("ci");
    String video;

    if(parser.has("v")) {
        video = parser.get<String>("v");
    }

    if(!parser.check()) {
        parser.printErrors();
        return 0;
    }

    VideoCapture inputVideo;
    int waitTime;
    if(!video.empty()) {
        inputVideo.open(video);
        waitTime = 0;
    } else {
        inputVideo.open(camId);
        waitTime = 10;
    }

    double totalTime = 0;
    int totalIterations = 0;

    aruco::CharucoBoard charucoBoard(Size(3, 3), squareLength, markerLength, dictionary);
    aruco::CharucoDetector detector(charucoBoard, aruco::CharucoParameters(), detectorParams);

    while(inputVideo.grab()) {
        Mat image, imageCopy;
        inputVideo.retrieve(image);

        double tick = (double)getTickCount();

        //! [detect_diamonds]
        vector<int> markerIds;
        vector<Vec4i> diamondIds;
        vector<vector<Point2f> > markerCorners, diamondCorners;
        vector<Vec3d> rvecs, tvecs;

        detector.detectDiamonds(image, diamondCorners, diamondIds, markerCorners, markerIds);
        //! [detect_diamonds]

        //! [diamond_pose_estimation]
        // estimate diamond pose
        size_t N = diamondIds.size();
        if(estimatePose && N > 0) {
            cv::Mat objPoints(4, 1, CV_32FC3);
            rvecs.resize(N);
            tvecs.resize(N);
            if(!autoScale) {
                // set coordinate system
                objPoints.ptr<Vec3f>(0)[0] = Vec3f(-squareLength/2.f, squareLength/2.f, 0);
                objPoints.ptr<Vec3f>(0)[1] = Vec3f(squareLength/2.f, squareLength/2.f, 0);
                objPoints.ptr<Vec3f>(0)[2] = Vec3f(squareLength/2.f, -squareLength/2.f, 0);
                objPoints.ptr<Vec3f>(0)[3] = Vec3f(-squareLength/2.f, -squareLength/2.f, 0);
                // Calculate pose for each marker
                for (size_t i = 0ull; i < N; i++)
                    solvePnP(objPoints, diamondCorners.at(i), camMatrix, distCoeffs, rvecs.at(i), tvecs.at(i));
                //! [diamond_pose_estimation]
                /* //! [diamond_pose_estimation_as_charuco]
                for (size_t i = 0ull; i < N; i++) { // estimate diamond pose as Charuco board
                    Mat objPoints_b, imgPoints;
                    // The coordinate system of the diamond is placed in the board plane centered in the bottom left corner
                    vector<int> charucoIds = {0, 1, 3, 2}; // if CCW order, Z axis pointing in the plane
                    // vector<int> charucoIds = {0, 2, 3, 1}; // if CW order, Z axis pointing out the plane
                    charucoBoard.matchImagePoints(diamondCorners[i], charucoIds, objPoints_b, imgPoints);
                    solvePnP(objPoints_b, imgPoints, camMatrix, distCoeffs, rvecs[i], tvecs[i]);
                }
                //! [diamond_pose_estimation_as_charuco] */
            }
            else {
                // if autoscale, extract square size from last diamond id
                for(size_t i = 0; i < N; i++) {
                    float sqLenScale = autoScaleFactor * float(diamondIds[i].val[3]);
                    vector<vector<Point2f> > currentCorners;
                    vector<Vec3d> currentRvec, currentTvec;
                    currentCorners.push_back(diamondCorners[i]);
                    // set coordinate system
                    objPoints.ptr<Vec3f>(0)[0] = Vec3f(-sqLenScale/2.f, sqLenScale/2.f, 0);
                    objPoints.ptr<Vec3f>(0)[1] = Vec3f(sqLenScale/2.f, sqLenScale/2.f, 0);
                    objPoints.ptr<Vec3f>(0)[2] = Vec3f(sqLenScale/2.f, -sqLenScale/2.f, 0);
                    objPoints.ptr<Vec3f>(0)[3] = Vec3f(-sqLenScale/2.f, -sqLenScale/2.f, 0);
                    solvePnP(objPoints, diamondCorners.at(i), camMatrix, distCoeffs, rvecs.at(i), tvecs.at(i));
                }
            }
        }


        double currentTime = ((double)getTickCount() - tick) / getTickFrequency();
        totalTime += currentTime;
        totalIterations++;
        if(totalIterations % 30 == 0) {
            cout << "Detection Time = " << currentTime * 1000 << " ms "
                 << "(Mean = " << 1000 * totalTime / double(totalIterations) << " ms)" << endl;
        }


        // draw results
        image.copyTo(imageCopy);
        if(markerIds.size() > 0)
            aruco::drawDetectedMarkers(imageCopy, markerCorners);

        //! [draw_diamonds]
        if(diamondIds.size() > 0) {
            aruco::drawDetectedDiamonds(imageCopy, diamondCorners, diamondIds);
        //! [draw_diamonds]

            //! [draw_diamond_pose_estimation]
            if(estimatePose) {
                for(size_t i = 0u; i < diamondIds.size(); i++)
                    cv::drawFrameAxes(imageCopy, camMatrix, distCoeffs, rvecs[i], tvecs[i], squareLength*1.1f);
            }
            //! [draw_diamond_pose_estimation]
        }
        imshow("out", imageCopy);
        char key = (char)waitKey(waitTime);
        if(key == 27) break;
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
- `config`
- `video`
- `camera`
- `last`


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

