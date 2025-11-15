# Documentation for `modules/video/src/tracking/tracker_mil.cpp`

## File Metadata

- **Full Path**: `modules/video/src/tracking/tracker_mil.cpp`
- **File Name**: `tracker_mil.cpp`
- **File Size**: 7,664 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/src/tracking/tracker_mil.cpp](../../../../modules/video/src/tracking/tracker_mil.cpp)

## Purpose and Role

This file is located in the `modules/video/src/tracking` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "detail/tracker_mil_model.hpp"

#include "detail/tracker_feature_haar.impl.hpp"

namespace cv {
inline namespace tracking {
namespace impl {

using cv::detail::tracking::internal::TrackerFeatureHAAR;


class TrackerMILImpl CV_FINAL : public TrackerMIL
{
public:
    TrackerMILImpl(const TrackerMIL::Params& parameters);

    virtual void init(InputArray image, const Rect& boundingBox) CV_OVERRIDE;
    virtual bool update(InputArray image, Rect& boundingBox) CV_OVERRIDE;

    void compute_integral(const Mat& img, Mat& ii_img);

    TrackerMIL::Params params;

    Ptr<TrackerMILModel> model;
    Ptr<TrackerSampler> sampler;
    Ptr<TrackerFeatureSet> featureSet;
};

TrackerMILImpl::TrackerMILImpl(const TrackerMIL::Params& parameters)
    : params(parameters)
{
    // nothing
}

void TrackerMILImpl::compute_integral(const Mat& img, Mat& ii_img)
{
    Mat ii;
    std::vector<Mat> ii_imgs;
    integral(img, ii, CV_32F);  // FIXIT split first
    split(ii, ii_imgs);
    ii_img = ii_imgs[0];
}

void TrackerMILImpl::init(InputArray image, const Rect& boundingBox)
{
    sampler = makePtr<TrackerSampler>();
    featureSet = makePtr<TrackerFeatureSet>();

    Mat intImage;
    compute_integral(image.getMat(), intImage);
    TrackerSamplerCSC::Params CSCparameters;
    CSCparameters.initInRad = params.samplerInitInRadius;
    CSCparameters.searchWinSize = params.samplerSearchWinSize;
    CSCparameters.initMaxNegNum = params.samplerInitMaxNegNum;
    CSCparameters.trackInPosRad = params.samplerTrackInRadius;
    CSCparameters.trackMaxPosNum = params.samplerTrackMaxPosNum;
    CSCparameters.trackMaxNegNum = params.samplerTrackMaxNegNum;

    Ptr<TrackerSamplerAlgorithm> CSCSampler = makePtr<TrackerSamplerCSC>(CSCparameters);
    CV_Assert(sampler->addTrackerSamplerAlgorithm(CSCSampler));

    //or add CSC sampler with default parameters
    //sampler->addTrackerSamplerAlgorithm( "CSC" );

    //Positive sampling
    CSCSampler.staticCast<TrackerSamplerCSC>()->setMode(TrackerSamplerCSC::MODE_INIT_POS);
    sampler->sampling(intImage, boundingBox);
    std::vector<Mat> posSamples = sampler->getSamples();

    //Negative sampling
    CSCSampler.staticCast<TrackerSamplerCSC>()->setMode(TrackerSamplerCSC::MODE_INIT_NEG);
    sampler->sampling(intImage, boundingBox);
    std::vector<Mat> negSamples = sampler->getSamples();

    CV_Assert(!posSamples.empty());
    CV_Assert(!negSamples.empty());

    //compute HAAR features
    TrackerFeatureHAAR::Params HAARparameters;
    HAARparameters.numFeatures = params.featureSetNumFeatures;
    HAARparameters.rectSize = Size((int)boundingBox.width, (int)boundingBox.height);
    HAARparameters.isIntegral = true;
    Ptr<TrackerFeature> trackerFeature = makePtr<TrackerFeatureHAAR>(HAARparameters);
    featureSet->addTrackerFeature(trackerFeature);

    featureSet->extraction(posSamples);
    const std::vector<Mat> posResponse = featureSet->getResponses();

    featureSet->extraction(negSamples);
    const std::vector<Mat> negResponse = featureSet->getResponses();

    model = makePtr<TrackerMILModel>(boundingBox);
    Ptr<TrackerStateEstimatorMILBoosting> stateEstimator = makePtr<TrackerStateEstimatorMILBoosting>(params.featureSetNumFeatures);
    model->setTrackerStateEstimator(stateEstimator);

    //Run model estimation and update
    model.staticCast<TrackerMILModel>()->setMode(TrackerMILModel::MODE_POSITIVE, posSamples);
    model->modelEstimation(posResponse);
    model.staticCast<TrackerMILModel>()->setMode(TrackerMILModel::MODE_NEGATIVE, negSamples);
    model->modelEstimation(negResponse);
    model->modelUpdate();
}

bool TrackerMILImpl::update(InputArray image, Rect& boundingBox)
{
    Mat intImage;
    compute_integral(image.getMat(), intImage);

    //get the last location [AAM] X(k-1)
    Ptr<TrackerTargetState> lastLocation = model->getLastTargetState();
    Rect lastBoundingBox((int)lastLocation->getTargetPosition().x, (int)lastLocation->getTargetPosition().y, lastLocation->getTargetWidth(),
            lastLocation->getTargetHeight());

    //sampling new frame based on last location
    auto& samplers = sampler->getSamplers();
    CV_Assert(!samplers.empty());
    CV_Assert(samplers[0]);
    samplers[0].staticCast<TrackerSamplerCSC>()->setMode(TrackerSamplerCSC::MODE_DETECT);
    sampler->sampling(intImage, lastBoundingBox);
    std::vector<Mat> detectSamples = sampler->getSamples();
    if (detectSamples.empty())
        return false;

    /*//TODO debug samples
   Mat f;
   image.copyTo(f);

   for( size_t i = 0; i < detectSamples.size(); i=i+10 )
   {
   Size sz;
   Point off;
   detectSamples.at(i).locateROI(sz, off);
   rectangle(f, Rect(off.x,off.y,detectSamples.at(i).cols,detectSamples.at(i).rows), Scalar(255,0,0), 1);
   }*/

    //extract features from new samples
    featureSet->extraction(detectSamples);
    std::vector<Mat> response = featureSet->getResponses();

    //predict new location
    ConfidenceMap cmap;
    model.staticCast<TrackerMILModel>()->setMode(TrackerMILModel::MODE_ESTIMATON, detectSamples);
    model.staticCast<TrackerMILModel>()->responseToConfidenceMap(response, cmap);
    model->getTrackerStateEstimator().staticCast<TrackerStateEstimatorMILBoosting>()->setCurrentConfidenceMap(cmap);

    if (!model->runStateEstimator())
    {
        return false;
    }

    Ptr<TrackerTargetState> currentState = model->getLastTargetState();
    boundingBox = Rect((int)currentState->getTargetPosition().x, (int)currentState->getTargetPosition().y, currentState->getTargetWidth(),
            currentState->getTargetHeight());

    /*//TODO debug
   rectangle(f, lastBoundingBox, Scalar(0,255,0), 1);
   rectangle(f, boundingBox, Scalar(0,0,255), 1);
   imshow("f", f);
   //waitKey( 0 );*/

    //sampling new frame based on new location
    //Positive sampling
    samplers[0].staticCast<TrackerSamplerCSC>()->setMode(TrackerSamplerCSC::MODE_INIT_POS);
    sampler->sampling(intImage, boundingBox);
    std::vector<Mat> posSamples = sampler->getSamples();

    //Negative sampling
    samplers[0].staticCast<TrackerSamplerCSC>()->setMode(TrackerSamplerCSC::MODE_INIT_NEG);
    sampler->sampling(intImage, boundingBox);
    std::vector<Mat> negSamples = sampler->getSamples();

    if (posSamples.empty() || negSamples.empty())
        return false;

    //extract features
    featureSet->extraction(posSamples);
    std::vector<Mat> posResponse = featureSet->getResponses();

    featureSet->extraction(negSamples);
    std::vector<Mat> negResponse = featureSet->getResponses();

    //model estimate
    model.staticCast<TrackerMILModel>()->setMode(TrackerMILModel::MODE_POSITIVE, posSamples);
    model->modelEstimation(posResponse);
    model.staticCast<TrackerMILModel>()->setMode(TrackerMILModel::MODE_NEGATIVE, negSamples);
    model->modelEstimation(negResponse);

    //model update
    model->modelUpdate();

    return true;
}

}}  // namespace tracking::impl

TrackerMIL::Params::Params()
{
    samplerInitInRadius = 3;
    samplerSearchWinSize = 25;
    samplerInitMaxNegNum = 65;
    samplerTrackInRadius = 4;
    samplerTrackMaxPosNum = 100000;
    samplerTrackMaxNegNum = 65;
    featureSetNumFeatures = 250;
}

TrackerMIL::TrackerMIL()
{
    // nothing
}

TrackerMIL::~TrackerMIL()
{
    // nothing
}

Ptr<TrackerMIL> TrackerMIL::create(const TrackerMIL::Params& parameters)
{
    return makePtr<tracking::impl::TrackerMILImpl>(parameters);
}

}  // namespace cv
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

- **TrackerMILImpl**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `detail/tracker_feature_haar.impl.hpp`
- `detail/tracker_mil_model.hpp`
- `../precomp.hpp`

**Python Imports:**
- `new`


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

