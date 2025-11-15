# Documentation for `modules/video/src/tracking/detail/tracker_mil_state.cpp`

## File Metadata

- **Full Path**: `modules/video/src/tracking/detail/tracker_mil_state.cpp`
- **File Name**: `tracker_mil_state.cpp`
- **File Size**: 4,661 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/src/tracking/detail/tracker_mil_state.cpp](../../../../../modules/video/src/tracking/detail/tracker_mil_state.cpp)

## Purpose and Role

This file is located in the `modules/video/src/tracking/detail` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../../precomp.hpp"
#include "opencv2/video/detail/tracking.detail.hpp"
#include "tracker_mil_state.hpp"

namespace cv {
namespace detail {
inline namespace tracking {

/**
 * TrackerStateEstimatorMILBoosting::TrackerMILTargetState
 */
TrackerStateEstimatorMILBoosting::TrackerMILTargetState::TrackerMILTargetState(const Point2f& position, int width, int height, bool foreground,
        const Mat& features)
{
    setTargetPosition(position);
    setTargetWidth(width);
    setTargetHeight(height);
    setTargetFg(foreground);
    setFeatures(features);
}

void TrackerStateEstimatorMILBoosting::TrackerMILTargetState::setTargetFg(bool foreground)
{
    isTarget = foreground;
}

void TrackerStateEstimatorMILBoosting::TrackerMILTargetState::setFeatures(const Mat& features)
{
    targetFeatures = features;
}

bool TrackerStateEstimatorMILBoosting::TrackerMILTargetState::isTargetFg() const
{
    return isTarget;
}

Mat TrackerStateEstimatorMILBoosting::TrackerMILTargetState::getFeatures() const
{
    return targetFeatures;
}

TrackerStateEstimatorMILBoosting::TrackerStateEstimatorMILBoosting(int nFeatures)
{
    className = "BOOSTING";
    trained = false;
    numFeatures = nFeatures;
}

TrackerStateEstimatorMILBoosting::~TrackerStateEstimatorMILBoosting()
{
}

void TrackerStateEstimatorMILBoosting::setCurrentConfidenceMap(ConfidenceMap& confidenceMap)
{
    currentConfidenceMap.clear();
    currentConfidenceMap = confidenceMap;
}

uint TrackerStateEstimatorMILBoosting::max_idx(const std::vector<float>& v)
{
    const float* findPtr = &(*std::max_element(v.begin(), v.end()));
    const float* beginPtr = &(*v.begin());
    return (uint)(findPtr - beginPtr);
}

Ptr<TrackerTargetState> TrackerStateEstimatorMILBoosting::estimateImpl(const std::vector<ConfidenceMap>& /*confidenceMaps*/)
{
    //run ClfMilBoost classify in order to compute next location
    if (currentConfidenceMap.empty())
        return Ptr<TrackerTargetState>();

    Mat positiveStates;
    Mat negativeStates;

    prepareData(currentConfidenceMap, positiveStates, negativeStates);

    std::vector<float> prob = boostMILModel.classify(positiveStates);

    int bestind = max_idx(prob);
    //float resp = prob[bestind];

    return currentConfidenceMap.at(bestind).first;
}

void TrackerStateEstimatorMILBoosting::prepareData(const ConfidenceMap& confidenceMap, Mat& positive, Mat& negative)
{

    int posCounter = 0;
    int negCounter = 0;

    for (size_t i = 0; i < confidenceMap.size(); i++)
    {
        Ptr<TrackerMILTargetState> currentTargetState = confidenceMap.at(i).first.staticCast<TrackerMILTargetState>();
        CV_DbgAssert(currentTargetState);
        if (currentTargetState->isTargetFg())
            posCounter++;
        else
            negCounter++;
    }

    positive.create(posCounter, numFeatures, CV_32FC1);
    negative.create(negCounter, numFeatures, CV_32FC1);

    //TODO change with mat fast access
    //initialize trainData (positive and negative)

    int pc = 0;
    int nc = 0;
    for (size_t i = 0; i < confidenceMap.size(); i++)
    {
        Ptr<TrackerMILTargetState> currentTargetState = confidenceMap.at(i).first.staticCast<TrackerMILTargetState>();
        Mat stateFeatures = currentTargetState->getFeatures();

        if (currentTargetState->isTargetFg())
        {
            for (int j = 0; j < stateFeatures.rows; j++)
            {
                //fill the positive trainData with the value of the feature j for sample i
                positive.at<float>(pc, j) = stateFeatures.at<float>(j, 0);
            }
            pc++;
        }
        else
        {
            for (int j = 0; j < stateFeatures.rows; j++)
            {
                //fill the negative trainData with the value of the feature j for sample i
                negative.at<float>(nc, j) = stateFeatures.at<float>(j, 0);
            }
            nc++;
        }
    }
}

void TrackerStateEstimatorMILBoosting::updateImpl(std::vector<ConfidenceMap>& confidenceMaps)
{

    if (!trained)
    {
        //this is the first time that the classifier is built
        //init MIL
        boostMILModel.init();
        trained = true;
    }

    ConfidenceMap lastConfidenceMap = confidenceMaps.back();
    Mat positiveStates;
    Mat negativeStates;

    prepareData(lastConfidenceMap, positiveStates, negativeStates);
    //update MIL
    boostMILModel.update(positiveStates, negativeStates);
}

}}}  // namespace cv::detail::tracking
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
- `../../precomp.hpp`
- `opencv2/video/detail/tracking.detail.hpp`
- `tracker_mil_state.hpp`


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

