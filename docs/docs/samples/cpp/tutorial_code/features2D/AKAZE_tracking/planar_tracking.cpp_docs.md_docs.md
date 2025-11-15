# Documentation for `docs/samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp_docs.md`
- **File Name**: `planar_tracking.cpp_docs.md`
- **File Size**: 10,469 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp_docs.md](../../../../../../docs/samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/features2D/AKAZE_tracking` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp`
- **File Name**: `planar_tracking.cpp`
- **File Size**: 7,121 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp](../../../../../samples/cpp/tutorial_code/features2D/AKAZE_tracking/planar_tracking.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/features2D/AKAZE_tracking` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/features2d.hpp>
#include <opencv2/videoio.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/calib3d.hpp>
#include <opencv2/highgui.hpp>      //for imshow
#include <vector>
#include <iostream>
#include <iomanip>

#include "stats.h" // Stats structure definition
#include "utils.h" // Drawing and printing functions

using namespace std;
using namespace cv;

const double akaze_thresh = 3e-4; // AKAZE detection threshold set to locate about 1000 keypoints
const double ransac_thresh = 2.5f; // RANSAC inlier threshold
const double nn_match_ratio = 0.8f; // Nearest-neighbour matching ratio
const int bb_min_inliers = 100; // Minimal number of inliers to draw bounding box
const int stats_update_period = 10; // On-screen statistics are updated every 10 frames

namespace example {
class Tracker
{
public:
    Tracker(Ptr<Feature2D> _detector, Ptr<DescriptorMatcher> _matcher) :
        detector(_detector),
        matcher(_matcher)
    {}

    void setFirstFrame(const Mat frame, vector<Point2f> bb, string title, Stats& stats);
    Mat process(const Mat frame, Stats& stats);
    Ptr<Feature2D> getDetector() {
        return detector;
    }
protected:
    Ptr<Feature2D> detector;
    Ptr<DescriptorMatcher> matcher;
    Mat first_frame, first_desc;
    vector<KeyPoint> first_kp;
    vector<Point2f> object_bb;
};

void Tracker::setFirstFrame(const Mat frame, vector<Point2f> bb, string title, Stats& stats)
{
    cv::Point *ptMask = new cv::Point[bb.size()];
    const Point* ptContain = { &ptMask[0] };
    int iSize = static_cast<int>(bb.size());
    for (size_t i=0; i<bb.size(); i++) {
        ptMask[i].x = static_cast<int>(bb[i].x);
        ptMask[i].y = static_cast<int>(bb[i].y);
    }
    first_frame = frame.clone();
    cv::Mat matMask = cv::Mat::zeros(frame.size(), CV_8UC1);
    cv::fillPoly(matMask, &ptContain, &iSize, 1, cv::Scalar::all(255));
    detector->detectAndCompute(first_frame, matMask, first_kp, first_desc);
    stats.keypoints = (int)first_kp.size();
    drawBoundingBox(first_frame, bb);
    putText(first_frame, title, Point(0, 60), FONT_HERSHEY_PLAIN, 5, Scalar::all(0), 4);
    object_bb = bb;
    delete[] ptMask;
}

Mat Tracker::process(const Mat frame, Stats& stats)
{
    TickMeter tm;
    vector<KeyPoint> kp;
    Mat desc;

    tm.start();
    detector->detectAndCompute(frame, noArray(), kp, desc);
    stats.keypoints = (int)kp.size();

    vector< vector<DMatch> > matches;
    vector<KeyPoint> matched1, matched2;
    matcher->knnMatch(first_desc, desc, matches, 2);
    for(unsigned i = 0; i < matches.size(); i++) {
        if(matches[i][0].distance < nn_match_ratio * matches[i][1].distance) {
            matched1.push_back(first_kp[matches[i][0].queryIdx]);
            matched2.push_back(      kp[matches[i][0].trainIdx]);
        }
    }
    stats.matches = (int)matched1.size();

    Mat inlier_mask, homography;
    vector<KeyPoint> inliers1, inliers2;
    vector<DMatch> inlier_matches;
    if(matched1.size() >= 4) {
        homography = findHomography(Points(matched1), Points(matched2),
                                    RANSAC, ransac_thresh, inlier_mask);
    }
    tm.stop();
    stats.fps = 1. / tm.getTimeSec();

    if(matched1.size() < 4 || homography.empty()) {
        Mat res;
        hconcat(first_frame, frame, res);
        stats.inliers = 0;
        stats.ratio = 0;
        return res;
    }
    for(unsigned i = 0; i < matched1.size(); i++) {
        if(inlier_mask.at<uchar>(i)) {
            int new_i = static_cast<int>(inliers1.size());
            inliers1.push_back(matched1[i]);
            inliers2.push_back(matched2[i]);
            inlier_matches.push_back(DMatch(new_i, new_i, 0));
        }
    }
    stats.inliers = (int)inliers1.size();
    stats.ratio = stats.inliers * 1.0 / stats.matches;

    vector<Point2f> new_bb;
    perspectiveTransform(object_bb, new_bb, homography);
    Mat frame_with_bb = frame.clone();
    if(stats.inliers >= bb_min_inliers) {
        drawBoundingBox(frame_with_bb, new_bb);
    }
    Mat res;
    drawMatches(first_frame, inliers1, frame_with_bb, inliers2,
                inlier_matches, res,
                Scalar(255, 0, 0), Scalar(255, 0, 0));
    return res;
}
}

int main(int argc, char **argv)
{
    CommandLineParser parser(argc, argv, "{@input_path |0|input path can be a camera id, like 0,1,2 or a video filename}");
    parser.printMessage();
    string input_path = parser.get<string>(0);
    string video_name = input_path;

    VideoCapture video_in;

    if ( ( isdigit(input_path[0]) && input_path.size() == 1 ) )
    {
    int camera_no = input_path[0] - '0';
        video_in.open( camera_no );
    }
    else {
        video_in.open(video_name);
    }

    if(!video_in.isOpened()) {
        cerr << "Couldn't open " << video_name << endl;
        return 1;
    }

    Stats stats, akaze_stats, orb_stats;
    Ptr<AKAZE> akaze = AKAZE::create();
    akaze->setThreshold(akaze_thresh);
    Ptr<ORB> orb = ORB::create();
    Ptr<DescriptorMatcher> matcher = DescriptorMatcher::create("BruteForce-Hamming");
    example::Tracker akaze_tracker(akaze, matcher);
    example::Tracker orb_tracker(orb, matcher);

    Mat frame;
    namedWindow(video_name, WINDOW_NORMAL);
    cout << "\nPress any key to stop the video and select a bounding box" << endl;

    while ( waitKey(1) < 1 )
    {
        video_in >> frame;
        cv::resizeWindow(video_name, frame.size());
        imshow(video_name, frame);
    }

    vector<Point2f> bb;
    cv::Rect uBox = cv::selectROI(video_name, frame);
    bb.push_back(cv::Point2f(static_cast<float>(uBox.x), static_cast<float>(uBox.y)));
    bb.push_back(cv::Point2f(static_cast<float>(uBox.x+uBox.width), static_cast<float>(uBox.y)));
    bb.push_back(cv::Point2f(static_cast<float>(uBox.x+uBox.width), static_cast<float>(uBox.y+uBox.height)));
    bb.push_back(cv::Point2f(static_cast<float>(uBox.x), static_cast<float>(uBox.y+uBox.height)));

    akaze_tracker.setFirstFrame(frame, bb, "AKAZE", stats);
    orb_tracker.setFirstFrame(frame, bb, "ORB", stats);

    Stats akaze_draw_stats, orb_draw_stats;
    Mat akaze_res, orb_res, res_frame;
    int i = 0;
    for(;;) {
        i++;
        bool update_stats = (i % stats_update_period == 0);
        video_in >> frame;
        // stop the program if no more images
        if(frame.empty()) break;

        akaze_res = akaze_tracker.process(frame, stats);
        akaze_stats += stats;
        if(update_stats) {
            akaze_draw_stats = stats;
        }

        orb->setMaxFeatures(stats.keypoints);
        orb_res = orb_tracker.process(frame, stats);
        orb_stats += stats;
        if(update_stats) {
            orb_draw_stats = stats;
        }

        drawStatistics(akaze_res, akaze_draw_stats);
        drawStatistics(orb_res, orb_draw_stats);
        vconcat(akaze_res, orb_res, res_frame);
        cv::imshow(video_name, res_frame);
        if(waitKey(1)==27) break; //quit on ESC button
    }
    akaze_stats /= i - 1;
    orb_stats /= i - 1;
    printStatistics("AKAZE", akaze_stats);
    printStatistics("ORB", orb_stats);
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

### Classes and Structures

- **Tracker**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utils.h`
- `iomanip`
- `vector`
- `opencv2/features2d.hpp`
- `opencv2/imgproc.hpp`
- `iostream`
- `stats.h`
- `opencv2/videoio.hpp`
- `opencv2/highgui.hpp`
- `opencv2/calib3d.hpp`


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

