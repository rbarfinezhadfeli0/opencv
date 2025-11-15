# Documentation for `modules/video/test/test_trackers.cpp`

## File Metadata

- **Full Path**: `modules/video/test/test_trackers.cpp`
- **File Name**: `test_trackers.cpp`
- **File Size**: 6,218 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/test/test_trackers.cpp](../../../modules/video/test/test_trackers.cpp)

## Purpose and Role

This file is located in the `modules/video/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"

//#define DEBUG_TEST
#ifdef DEBUG_TEST
#include <opencv2/highgui.hpp>
#endif

namespace opencv_test { namespace {
//using namespace cv::tracking;

#define TESTSET_NAMES testing::Values("david", "dudek", "faceocc2")

const string TRACKING_DIR = "tracking";
const string FOLDER_IMG = "data";
const string FOLDER_OMIT_INIT = "initOmit";

#include "test_trackers.impl.hpp"

//[TESTDATA]
PARAM_TEST_CASE(DistanceAndOverlap, string, int)
{
    string dataset;
    int numFramesLimit;
    virtual void SetUp()
    {
        dataset = GET_PARAM(0);
        numFramesLimit = GET_PARAM(1);
    }
};

TEST_P(DistanceAndOverlap, MIL)
{
    TrackerTest<Tracker, Rect> test(TrackerMIL::create(), dataset, 30, .65f, NoTransform);
    test.run(numFramesLimit);
}

TEST_P(DistanceAndOverlap, Shifted_Data_MIL)
{
    TrackerTest<Tracker, Rect> test(TrackerMIL::create(), dataset, 30, .6f, CenterShiftLeft);
    test.run(numFramesLimit);
}

/***************************************************************************************/
//Tests with scaled initial window

TEST_P(DistanceAndOverlap, Scaled_Data_MIL)
{
    TrackerTest<Tracker, Rect> test(TrackerMIL::create(), dataset, 30, .7f, Scale_1_1);
    test.run(numFramesLimit);
}

TEST_P(DistanceAndOverlap, GOTURN)
{
    std::string model = cvtest::findDataFile("dnn/gsoc2016-goturn/goturn.prototxt");
    std::string weights = cvtest::findDataFile("dnn/gsoc2016-goturn/goturn.caffemodel", false);
    cv::TrackerGOTURN::Params params;
    params.modelTxt = model;
    params.modelBin = weights;
    TrackerTest<Tracker, Rect> test(TrackerGOTURN::create(params), dataset, 35, .35f, NoTransform);
    test.run(numFramesLimit);
}

INSTANTIATE_TEST_CASE_P(Tracking, DistanceAndOverlap,
    testing::Combine(
        TESTSET_NAMES,
        testing::Values(0)
    )
);

INSTANTIATE_TEST_CASE_P(Tracking5Frames, DistanceAndOverlap,
    testing::Combine(
        TESTSET_NAMES,
        testing::Values(5)
    )
);


static bool checkIOU(const Rect& r0, const Rect& r1, double threshold)
{
    int interArea = (r0 & r1).area();
    double iouVal = (interArea * 1.0 )/ (r0.area() + r1.area() - interArea);;

    if (iouVal > threshold)
        return true;
    else
    {
        std::cout <<"Unmatched IOU:  expect IOU val ("<<iouVal <<") > the IOU threadhold ("<<threshold<<")! Box 0 is "
                                << r0 <<", and Box 1 is "<<r1<< std::endl;
        return false;
    }
}

static void checkTrackingAccuracy(cv::Ptr<Tracker>& tracker, double iouThreshold = 0.7)
{
    // Template image
    Mat img0 = imread(findDataFile("tracking/bag/00000001.jpg"), 1);

    // Tracking image sequence.
    std::vector<Mat> imgs;
    imgs.push_back(imread(findDataFile("tracking/bag/00000002.jpg"), 1));
    imgs.push_back(imread(findDataFile("tracking/bag/00000003.jpg"), 1));
    imgs.push_back(imread(findDataFile("tracking/bag/00000004.jpg"), 1));
    imgs.push_back(imread(findDataFile("tracking/bag/00000005.jpg"), 1));
    imgs.push_back(imread(findDataFile("tracking/bag/00000006.jpg"), 1));

    cv::Rect roi(325, 164, 100, 100);
    std::vector<Rect> targetRois;
    targetRois.push_back(cv::Rect(278, 133, 99, 104));
    targetRois.push_back(cv::Rect(293, 88, 93, 110));
    targetRois.push_back(cv::Rect(287, 76, 89, 116));
    targetRois.push_back(cv::Rect(297, 74, 82, 122));
    targetRois.push_back(cv::Rect(311, 83, 78, 125));

    tracker->init(img0, roi);
    CV_Assert(targetRois.size() == imgs.size());

    for (int i = 0; i < (int)imgs.size(); i++)
    {
        bool res = tracker->update(imgs[i], roi);
        ASSERT_TRUE(res);
        ASSERT_TRUE(checkIOU(roi, targetRois[i], iouThreshold)) << cv::format("Fail at img %d.",i);
    }
}

TEST(GOTURN, accuracy)
{
    std::string model = cvtest::findDataFile("dnn/gsoc2016-goturn/goturn.prototxt");
    std::string weights = cvtest::findDataFile("dnn/gsoc2016-goturn/goturn.caffemodel", false);
    cv::TrackerGOTURN::Params params;
    params.modelTxt = model;
    params.modelBin = weights;
    cv::Ptr<Tracker> tracker = TrackerGOTURN::create(params);
    // TODO! GOTURN have low accuracy. Try to remove this api at 5.x.
    checkTrackingAccuracy(tracker, 0.08);
}

TEST(DaSiamRPN, accuracy)
{
    std::string model = cvtest::findDataFile("dnn/onnx/models/dasiamrpn_model.onnx", false);
    std::string kernel_r1 = cvtest::findDataFile("dnn/onnx/models/dasiamrpn_kernel_r1.onnx", false);
    std::string kernel_cls1 = cvtest::findDataFile("dnn/onnx/models/dasiamrpn_kernel_cls1.onnx", false);
    cv::TrackerDaSiamRPN::Params params;
    params.model = model;
    params.kernel_r1 = kernel_r1;
    params.kernel_cls1 = kernel_cls1;
    cv::Ptr<Tracker> tracker = TrackerDaSiamRPN::create(params);
    checkTrackingAccuracy(tracker, 0.7);
}

TEST(NanoTrack, accuracy_NanoTrack_V1)
{
    std::string backbonePath = cvtest::findDataFile("dnn/onnx/models/nanotrack_backbone_sim.onnx", false);
    std::string neckheadPath = cvtest::findDataFile("dnn/onnx/models/nanotrack_head_sim.onnx", false);

    cv::TrackerNano::Params params;
    params.backbone = backbonePath;
    params.neckhead = neckheadPath;
    cv::Ptr<Tracker> tracker = TrackerNano::create(params);
    checkTrackingAccuracy(tracker);
}

TEST(NanoTrack, accuracy_NanoTrack_V2)
{
    std::string backbonePath = cvtest::findDataFile("dnn/onnx/models/nanotrack_backbone_sim_v2.onnx", false);
    std::string neckheadPath = cvtest::findDataFile("dnn/onnx/models/nanotrack_head_sim_v2.onnx", false);

    cv::TrackerNano::Params params;
    params.backbone = backbonePath;
    params.neckhead = neckheadPath;
    cv::Ptr<Tracker> tracker = TrackerNano::create(params);
    checkTrackingAccuracy(tracker, 0.69);
}

TEST(vittrack, accuracy_vittrack)
{
    std::string model = cvtest::findDataFile("dnn/onnx/models/vitTracker.onnx");
    cv::TrackerVit::Params params;
    params.net = model;
    cv::Ptr<Tracker> tracker = TrackerVit::create(params);
    checkTrackingAccuracy(tracker, 0.64);
}

}}  // namespace opencv_test::
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

- **DEBUG_TEST()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/highgui.hpp`
- `test_trackers.impl.hpp`
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

