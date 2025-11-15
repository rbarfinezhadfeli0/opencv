# Documentation for `modules/features2d/src/affine_feature.cpp`

## File Metadata

- **Full Path**: `modules/features2d/src/affine_feature.cpp`
- **File Name**: `affine_feature.cpp`
- **File Size**: 12,680 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/features2d/src/affine_feature.cpp](../../../modules/features2d/src/affine_feature.cpp)

## Purpose and Role

This file is located in the `modules/features2d/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// This file is based on code issued with the following license.
/*********************************************************************
* Software License Agreement (BSD License)
*
*  Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
*  Copyright (C) 2008-2013, Willow Garage Inc., all rights reserved.
*  Copyright (C) 2013, Evgeny Toropov, all rights reserved.
*  Third party copyrights are property of their respective owners.
*
*  Redistribution and use in source and binary forms, with or without
*  modification, are permitted provided that the following conditions
*  are met:
*
*   * Redistributions of source code must retain the above copyright
*     notice, this list of conditions and the following disclaimer.
*   * Redistributions in binary form must reproduce the above
*     copyright notice, this list of conditions and the following
*     disclaimer in the documentation and/or other materials provided
*     with the distribution.
*   * The name of the copyright holders may not be used to endorse
*     or promote products derived from this software without specific
*     prior written permission.
*
*  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
*  "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
*  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
*  FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
*  COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
*  INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
*  BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
*  LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
*  CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
*  LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
*  ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
*  POSSIBILITY OF SUCH DAMAGE.
*********************************************************************/

/*
 Guoshen Yu, Jean-Michel Morel, ASIFT: An Algorithm for Fully Affine
 Invariant Comparison,  Image Processing On Line, 1 (2011), pp. 11-38.
 https://doi.org/10.5201/ipol.2011.my-asift
 */

#include "precomp.hpp"
#include <iostream>
namespace cv {

class AffineFeature_Impl CV_FINAL : public AffineFeature
{
public:
    explicit AffineFeature_Impl(const Ptr<Feature2D>& backend,
            int maxTilt, int minTilt, float tiltStep, float rotateStepBase);

    int descriptorSize() const CV_OVERRIDE
    {
        return backend_->descriptorSize();
    }

    int descriptorType() const CV_OVERRIDE
    {
        return backend_->descriptorType();
    }

    int defaultNorm() const CV_OVERRIDE
    {
        return backend_->defaultNorm();
    }

    void detectAndCompute(InputArray image, InputArray mask, std::vector<KeyPoint>& keypoints,
            OutputArray descriptors, bool useProvidedKeypoints=false) CV_OVERRIDE;

    void setViewParams(const std::vector<float>& tilts, const std::vector<float>& rolls) CV_OVERRIDE;
    void getViewParams(std::vector<float>& tilts, std::vector<float>& rolls) const CV_OVERRIDE;

protected:
    void splitKeypointsByView(const std::vector<KeyPoint>& keypoints_,
            std::vector< std::vector<KeyPoint> >& keypointsByView) const;

    const Ptr<Feature2D> backend_;
    int maxTilt_;
    int minTilt_;
    float tiltStep_;
    float rotateStepBase_;

    // Tilt factors.
    std::vector<float> tilts_;
    // Roll factors.
    std::vector<float> rolls_;

private:
    AffineFeature_Impl(const AffineFeature_Impl &); // copy disabled
    AffineFeature_Impl& operator=(const AffineFeature_Impl &); // assign disabled
};

AffineFeature_Impl::AffineFeature_Impl(const Ptr<FeatureDetector>& backend,
        int maxTilt, int minTilt, float tiltStep, float rotateStepBase)
    : backend_(backend), maxTilt_(maxTilt), minTilt_(minTilt), tiltStep_(tiltStep), rotateStepBase_(rotateStepBase)
{
    int i = minTilt_;
    if( i == 0 )
    {
        tilts_.push_back(1);
        rolls_.push_back(0);
        i++;
    }
    float tilt = 1;
    for( ; i <= maxTilt_; i++ )
    {
        tilt *= tiltStep_;
        float rotateStep = rotateStepBase_ / tilt;
        int rollN = cvFloor(180.0f / rotateStep);
        if( rollN * rotateStep == 180.0f )
            rollN--;
        for( int j = 0; j <= rollN; j++ )
        {
            tilts_.push_back(tilt);
            rolls_.push_back(rotateStep * j);
        }
    }
}

void AffineFeature_Impl::setViewParams(const std::vector<float>& tilts,
        const std::vector<float>& rolls)
{
    CV_Assert(tilts.size() == rolls.size());
    tilts_ = tilts;
    rolls_ = rolls;
}

void AffineFeature_Impl::getViewParams(std::vector<float>& tilts,
        std::vector<float>& rolls) const
{
    tilts = tilts_;
    rolls = rolls_;
}

void AffineFeature_Impl::splitKeypointsByView(const std::vector<KeyPoint>& keypoints_,
        std::vector< std::vector<KeyPoint> >& keypointsByView) const
{
    for( size_t i = 0; i < keypoints_.size(); i++ )
    {
        const KeyPoint& kp = keypoints_[i];
        CV_Assert( kp.class_id >= 0 && kp.class_id < (int)tilts_.size() );
        keypointsByView[kp.class_id].push_back(kp);
    }
}

class skewedDetectAndCompute : public ParallelLoopBody
{
public:
    skewedDetectAndCompute(
        const std::vector<float>& _tilts,
        const std::vector<float>& _rolls,
        std::vector< std::vector<KeyPoint> >& _keypointsCollection,
        std::vector<Mat>& _descriptorCollection,
        const Mat& _image,
        const Mat& _mask,
        const bool _do_keypoints,
        const bool _do_descriptors,
        const Ptr<Feature2D>& _backend)
        : tilts(_tilts),
          rolls(_rolls),
          keypointsCollection(_keypointsCollection),
          descriptorCollection(_descriptorCollection),
          image(_image),
          mask(_mask),
          do_keypoints(_do_keypoints),
          do_descriptors(_do_descriptors),
          backend(_backend) {}

    void operator()( const cv::Range& range ) const CV_OVERRIDE
    {
        CV_TRACE_FUNCTION();

        const int begin = range.start;
        const int end = range.end;

        for( int a = begin; a < end; a++ )
        {
            Mat warpedImage, warpedMask;
            Matx23f pose, invPose;
            affineSkew(tilts[a], rolls[a], warpedImage, warpedMask, pose);
            invertAffineTransform(pose, invPose);

            std::vector<KeyPoint> wKeypoints;
            Mat wDescriptors;
            if( !do_keypoints )
            {
                const std::vector<KeyPoint>& keypointsInView = keypointsCollection[a];
                if( keypointsInView.size() == 0 ) // when there are no keypoints in this affine view
                    continue;

                std::vector<Point2f> pts_, pts;
                KeyPoint::convert(keypointsInView, pts_);
                transform(pts_, pts, pose);
                wKeypoints.resize(keypointsInView.size());
                for( size_t wi = 0; wi < wKeypoints.size(); wi++ )
                {
                    wKeypoints[wi] = keypointsInView[wi];
                    wKeypoints[wi].pt = pts[wi];
                }
            }
            backend->detectAndCompute(warpedImage, warpedMask, wKeypoints, wDescriptors, !do_keypoints);
            if( do_keypoints )
            {
                // KeyPointsFilter::runByPixelsMask( wKeypoints, warpedMask );
                if( wKeypoints.size() == 0 )
                {
                    keypointsCollection[a].clear();
                    continue;
                }
                std::vector<Point2f> pts_, pts;
                KeyPoint::convert(wKeypoints, pts_);
                transform(pts_, pts, invPose);

                keypointsCollection[a].resize(wKeypoints.size());
                for( size_t wi = 0; wi < wKeypoints.size(); wi++ )
                {
                    keypointsCollection[a][wi] = wKeypoints[wi];
                    keypointsCollection[a][wi].pt = pts[wi];
                    keypointsCollection[a][wi].class_id = a;
                }
            }
            if( do_descriptors )
                wDescriptors.copyTo(descriptorCollection[a]);
        }
    }
private:
    void affineSkew(float tilt, float phi,
            Mat& warpedImage, Mat& warpedMask, Matx23f& pose) const
    {
        int h = image.size().height;
        int w = image.size().width;
        Mat rotImage;

        Mat mask0;
        if( mask.empty() )
            mask0 = Mat(h, w, CV_8UC1, 255);
        else
            mask0 = mask;
        pose = Matx23f(1,0,0,
                       0,1,0);

        if( phi == 0 )
            image.copyTo(rotImage);
        else
        {
            phi = phi * (float)CV_PI / 180;
            float s = std::sin(phi);
            float c = std::cos(phi);
            Matx22f A(c, -s, s, c);
            Matx<float, 4, 2> corners(0, 0, (float)w, 0, (float)w,(float)h, 0, (float)h);
            Mat tf(corners * A.t());
            Mat tcorners;
            tf.convertTo(tcorners, CV_32S);
            Rect rect = boundingRect(tcorners);
            h = rect.height; w = rect.width;
            pose = Matx23f(c, -s, -(float)rect.x,
                        s,  c, -(float)rect.y);
            warpAffine(image, rotImage, pose, Size(w, h), INTER_LINEAR, BORDER_REPLICATE);
        }
        if( tilt == 1 )
            warpedImage = rotImage;
        else
        {
            float s = 0.8f * sqrt(tilt * tilt - 1);
            GaussianBlur(rotImage, rotImage, Size(0, 0), s, 0.01);
            resize(rotImage, warpedImage, Size(0, 0), 1.0/tilt, 1.0, INTER_NEAREST);
            pose(0, 0) /= tilt;
            pose(0, 1) /= tilt;
            pose(0, 2) /= tilt;
        }
        if( phi != 0 || tilt != 1 )
            warpAffine(mask0, warpedMask, pose, warpedImage.size(), INTER_NEAREST);
        else
            warpedMask = mask0;
    }


    const std::vector<float>& tilts;
    const std::vector<float>& rolls;
    std::vector< std::vector<KeyPoint> >& keypointsCollection;
    std::vector<Mat>& descriptorCollection;
    const Mat& image;
    const Mat& mask;
    const bool do_keypoints;
    const bool do_descriptors;
    const Ptr<Feature2D>& backend;
};

void AffineFeature_Impl::detectAndCompute(InputArray _image, InputArray _mask,
        std::vector<KeyPoint>& keypoints,
        OutputArray _descriptors,
        bool useProvidedKeypoints)
{
    CV_TRACE_FUNCTION();

    bool do_keypoints = !useProvidedKeypoints;
    bool do_descriptors = _descriptors.needed();
    Mat image = _image.getMat(), mask = _mask.getMat();
    Mat descriptors;

    if( (!do_keypoints && !do_descriptors) || _image.empty() )
        return;

    std::vector< std::vector<KeyPoint> > keypointsCollection(tilts_.size());
    std::vector< Mat > descriptorCollection(tilts_.size());

    if( do_keypoints )
        keypoints.clear();
    else
        splitKeypointsByView(keypoints, keypointsCollection);

    parallel_for_(Range(0, (int)tilts_.size()), skewedDetectAndCompute(tilts_, rolls_, keypointsCollection, descriptorCollection,
        image, mask, do_keypoints, do_descriptors, backend_));

    if( do_keypoints )
        for( size_t i = 0; i < keypointsCollection.size(); i++ )
        {
            const std::vector<KeyPoint>& keys = keypointsCollection[i];
            keypoints.insert(keypoints.end(), keys.begin(), keys.end());
        }

    if( do_descriptors )
    {
        _descriptors.create((int)keypoints.size(), backend_->descriptorSize(), backend_->descriptorType());
        descriptors = _descriptors.getMat();
        int iter = 0;
        for( size_t i = 0; i < descriptorCollection.size(); i++ )
        {
            const Mat& descs = descriptorCollection[i];
            if( descs.empty() )
                continue;
            Mat roi(descriptors, Rect(0, iter, descriptors.cols, descs.rows));
            descs.copyTo(roi);
            iter += descs.rows;
        }
    }
}


Ptr<AffineFeature> AffineFeature::create(const Ptr<Feature2D>& backend,
                                         int maxTilt, int minTilt, float tiltStep, float rotateStepBase)
{
    CV_Assert(minTilt < maxTilt);
    CV_Assert(tiltStep > 0);
    CV_Assert(rotateStepBase > 0);
    return makePtr<AffineFeature_Impl>(backend, maxTilt, minTilt, tiltStep, rotateStepBase);
}

String AffineFeature::getDefaultName() const
{
    return (Feature2D::getDefaultName() + ".AffineFeature");
}

} // namespace
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

- **skewedDetectAndCompute**: A class/struct defined in this file
- **AffineFeature_Impl**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iostream`
- `precomp.hpp`

**Python Imports:**
- `this`


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

