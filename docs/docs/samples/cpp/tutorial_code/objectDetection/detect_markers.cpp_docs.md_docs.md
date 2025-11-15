# Documentation for `docs/samples/cpp/tutorial_code/objectDetection/detect_markers.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/objectDetection/detect_markers.cpp_docs.md`
- **File Name**: `detect_markers.cpp_docs.md`
- **File Size**: 8,978 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/objectDetection/detect_markers.cpp_docs.md](../../../../../docs/samples/cpp/tutorial_code/objectDetection/detect_markers.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/objectDetection/detect_markers.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/objectDetection/detect_markers.cpp`
- **File Name**: `detect_markers.cpp`
- **File Size**: 5,759 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/objectDetection/detect_markers.cpp](../../../../samples/cpp/tutorial_code/objectDetection/detect_markers.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/highgui.hpp>
#include <opencv2/objdetect/aruco_detector.hpp>
#include <iostream>
#include "aruco_samples_utility.hpp"

using namespace std;
using namespace cv;

namespace {
const char* about = "Basic marker detection";

//! [aruco_detect_markers_keys]
const char* keys  =
        "{d        | 0     | dictionary: DICT_4X4_50=0, DICT_4X4_100=1, DICT_4X4_250=2,"
        "DICT_4X4_1000=3, DICT_5X5_50=4, DICT_5X5_100=5, DICT_5X5_250=6, DICT_5X5_1000=7, "
        "DICT_6X6_50=8, DICT_6X6_100=9, DICT_6X6_250=10, DICT_6X6_1000=11, DICT_7X7_50=12,"
        "DICT_7X7_100=13, DICT_7X7_250=14, DICT_7X7_1000=15, DICT_ARUCO_ORIGINAL = 16,"
        "DICT_APRILTAG_16h5=17, DICT_APRILTAG_25h9=18, DICT_APRILTAG_36h10=19, DICT_APRILTAG_36h11=20}"
        "{cd       |       | Input file with custom dictionary }"
        "{v        |       | Input from video or image file, if ommited, input comes from camera }"
        "{ci       | 0     | Camera id if input doesnt come from video (-v) }"
        "{c        |       | Camera intrinsic parameters. Needed for camera pose }"
        "{l        | 0.1   | Marker side length (in meters). Needed for correct scale in camera pose }"
        "{dp       |       | File of marker detector parameters }"
        "{r        |       | show rejected candidates too }"
        "{refine   |       | Corner refinement: CORNER_REFINE_NONE=0, CORNER_REFINE_SUBPIX=1,"
        "CORNER_REFINE_CONTOUR=2, CORNER_REFINE_APRILTAG=3}";

//! [aruco_detect_markers_keys]

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

    bool showRejected = parser.has("r");
    bool estimatePose = parser.has("c");
    float markerLength = parser.get<float>("l");

    aruco::DetectorParameters detectorParams = readDetectorParamsFromCommandLine(parser);
    aruco::Dictionary dictionary = readDictionatyFromCommandLine(parser);

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

    //! [aruco_pose_estimation1]
    Mat camMatrix, distCoeffs;
    if(estimatePose) {
        // You can read camera parameters from tutorial_camera_params.yml
        readCameraParamsFromCommandLine(parser, camMatrix, distCoeffs);
    }
    //! [aruco_pose_estimation1]
    //! [aruco_detect_markers]
    cv::aruco::ArucoDetector detector(dictionary, detectorParams);
    cv::VideoCapture inputVideo;
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

    //! [aruco_pose_estimation2]
    // set coordinate system
    cv::Mat objPoints(4, 1, CV_32FC3);
    objPoints.ptr<Vec3f>(0)[0] = Vec3f(-markerLength/2.f, markerLength/2.f, 0);
    objPoints.ptr<Vec3f>(0)[1] = Vec3f(markerLength/2.f, markerLength/2.f, 0);
    objPoints.ptr<Vec3f>(0)[2] = Vec3f(markerLength/2.f, -markerLength/2.f, 0);
    objPoints.ptr<Vec3f>(0)[3] = Vec3f(-markerLength/2.f, -markerLength/2.f, 0);
    //! [aruco_pose_estimation2]

    while(inputVideo.grab()) {
        cv::Mat image, imageCopy;
        inputVideo.retrieve(image);

        double tick = (double)getTickCount();

        //! [aruco_pose_estimation3]
        vector<int> ids;
        vector<vector<Point2f> > corners, rejected;

        // detect markers and estimate pose
        detector.detectMarkers(image, corners, ids, rejected);

        size_t nMarkers = corners.size();
        vector<Vec3d> rvecs(nMarkers), tvecs(nMarkers);

        if(estimatePose && !ids.empty()) {
            // Calculate pose for each marker
            for (size_t i = 0; i < nMarkers; i++) {
                solvePnP(objPoints, corners.at(i), camMatrix, distCoeffs, rvecs.at(i), tvecs.at(i));
            }
        }
        //! [aruco_pose_estimation3]
        double currentTime = ((double)getTickCount() - tick) / getTickFrequency();
        totalTime += currentTime;
        totalIterations++;
        if(totalIterations % 30 == 0) {
            cout << "Detection Time = " << currentTime * 1000 << " ms "
                 << "(Mean = " << 1000 * totalTime / double(totalIterations) << " ms)" << endl;
        }
        //! [aruco_draw_pose_estimation]
        // draw results
        image.copyTo(imageCopy);
        if(!ids.empty()) {
            cv::aruco::drawDetectedMarkers(imageCopy, corners, ids);

            if(estimatePose) {
                for(unsigned int i = 0; i < ids.size(); i++)
                    cv::drawFrameAxes(imageCopy, camMatrix, distCoeffs, rvecs[i], tvecs[i], markerLength * 1.5f, 2);
            }
        }
        //! [aruco_draw_pose_estimation]

        if(showRejected && !rejected.empty())
            cv::aruco::drawDetectedMarkers(imageCopy, rejected, noArray(), Scalar(100, 0, 255));

        imshow("out", imageCopy);
        char key = (char)waitKey(waitTime);
        if(key == 27) break;
    }
    //! [aruco_detect_markers]
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
- `opencv2/highgui.hpp`
- `aruco_samples_utility.hpp`
- `opencv2/objdetect/aruco_detector.hpp`

**Python Imports:**
- `config`
- `video`
- `camera`
- `tutorial_camera_params.yml`


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

