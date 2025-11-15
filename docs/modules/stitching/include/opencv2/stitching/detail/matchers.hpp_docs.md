# Documentation for `modules/stitching/include/opencv2/stitching/detail/matchers.hpp`

## File Metadata

- **Full Path**: `modules/stitching/include/opencv2/stitching/detail/matchers.hpp`
- **File Name**: `matchers.hpp`
- **File Size**: 10,844 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/stitching/include/opencv2/stitching/detail/matchers.hpp](../../../../../../modules/stitching/include/opencv2/stitching/detail/matchers.hpp)

## Purpose and Role

This file is located in the `modules/stitching/include/opencv2/stitching/detail` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                          License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_STITCHING_MATCHERS_HPP
#define OPENCV_STITCHING_MATCHERS_HPP

#include "opencv2/core.hpp"
#include "opencv2/features2d.hpp"

#include "opencv2/opencv_modules.hpp"

namespace cv {
namespace detail {

//! @addtogroup stitching_match
//! @{

/** @brief Structure containing image keypoints and descriptors. */
struct CV_EXPORTS_W_SIMPLE ImageFeatures
{
    CV_PROP_RW int img_idx;
    CV_PROP_RW Size img_size;
    CV_PROP_RW std::vector<KeyPoint> keypoints;
    CV_PROP_RW UMat descriptors;
    CV_WRAP std::vector<KeyPoint> getKeypoints() { return keypoints; }
};
/** @brief

@param featuresFinder
@param images
@param features
@param masks
*/
CV_EXPORTS_W void computeImageFeatures(
    const Ptr<Feature2D> &featuresFinder,
    InputArrayOfArrays  images,
    CV_OUT std::vector<ImageFeatures> &features,
    InputArrayOfArrays masks = noArray());

/** @brief

@param featuresFinder
@param image
@param features
@param mask
*/
CV_EXPORTS_AS(computeImageFeatures2) void computeImageFeatures(
    const Ptr<Feature2D> &featuresFinder,
    InputArray image,
    CV_OUT ImageFeatures &features,
    InputArray mask = noArray());

/** @brief Structure containing information about matches between two images.

It's assumed that there is a transformation between those images. Transformation may be
homography or affine transformation based on selected matcher.

@sa detail::FeaturesMatcher
*/
struct CV_EXPORTS_W_SIMPLE MatchesInfo
{
    MatchesInfo();
    MatchesInfo(const MatchesInfo &other);
    MatchesInfo& operator =(const MatchesInfo &other);

    CV_PROP_RW int src_img_idx;
    CV_PROP_RW int dst_img_idx;       //!< Images indices (optional)
    CV_PROP_RW std::vector<DMatch> matches;
    CV_PROP_RW std::vector<uchar> inliers_mask;    //!< Geometrically consistent matches mask
    CV_PROP_RW int num_inliers;                    //!< Number of geometrically consistent matches
    CV_PROP_RW Mat H;                              //!< Estimated transformation
    CV_PROP_RW double confidence;                  //!< Confidence two images are from the same panorama
    CV_WRAP std::vector<DMatch> getMatches() { return matches; }
    CV_WRAP std::vector<uchar> getInliers() { return inliers_mask; }
};

/** @brief Feature matchers base class. */
class CV_EXPORTS_W FeaturesMatcher
{
public:
    CV_WRAP virtual ~FeaturesMatcher() {}

    /** @overload
    @param features1 First image features
    @param features2 Second image features
    @param matches_info Found matches
    */
    CV_WRAP_AS(apply) void operator ()(const ImageFeatures &features1, const ImageFeatures &features2,
                     CV_OUT MatchesInfo& matches_info) { match(features1, features2, matches_info); }

    /** @brief Performs images matching.

    @param features Features of the source images
    @param pairwise_matches Found pairwise matches
    @param mask Mask indicating which image pairs must be matched

    The function is parallelized with the TBB library.

    @sa detail::MatchesInfo
    */
    CV_WRAP_AS(apply2) void operator ()(const std::vector<ImageFeatures> &features, CV_OUT std::vector<MatchesInfo> &pairwise_matches,
                                        const cv::UMat &mask = cv::UMat()) { match(features, pairwise_matches, mask); }

    /** @return True, if it's possible to use the same matcher instance in parallel, false otherwise
    */
   CV_WRAP bool isThreadSafe() const { return is_thread_safe_; }

    /** @brief Frees unused memory allocated before if there is any.
    */
   CV_WRAP virtual void collectGarbage() {}

protected:
    FeaturesMatcher(bool is_thread_safe = false) : is_thread_safe_(is_thread_safe) {}

    /** @brief This method must implement matching logic in order to make the wrappers
    detail::FeaturesMatcher::operator()_ work.

    @param features1 first image features
    @param features2 second image features
    @param matches_info found matches
     */
    virtual void match(const ImageFeatures &features1, const ImageFeatures &features2,
                       MatchesInfo& matches_info) = 0;

    /** @brief This method implements logic to match features between arbitrary number of features.
    By default this checks every pair of inputs in the input, but the behaviour can be changed by subclasses.

    @param features vector of image features
    @param pairwise_matches found matches
    @param mask (optional) mask indicating which image pairs should be matched
     */
    virtual void match(const std::vector<ImageFeatures> &features, std::vector<MatchesInfo> &pairwise_matches,
                       const cv::UMat &mask = cv::UMat());

    bool is_thread_safe_;
};

/** @brief Features matcher which finds two best matches for each feature and leaves the best one only if the
ratio between descriptor distances is greater than the threshold match_conf

@sa detail::FeaturesMatcher
 */
class CV_EXPORTS_W BestOf2NearestMatcher : public FeaturesMatcher
{
public:
    /** @brief Constructs a "best of 2 nearest" matcher.

    @param try_use_gpu Should try to use GPU or not
    @param match_conf Match distances ration threshold
    @param num_matches_thresh1 Minimum number of matches required for the 2D projective transform
    estimation used in the inliers classification step
    @param num_matches_thresh2 Minimum number of matches required for the 2D projective transform
    re-estimation on inliers
    @param matches_confindece_thresh Matching confidence threshold to take the match into account.
    The threshold was determined experimentally and set to 3 by default.
     */
    CV_WRAP BestOf2NearestMatcher(bool try_use_gpu = false, float match_conf = 0.3f, int num_matches_thresh1 = 6,
                          int num_matches_thresh2 = 6, double matches_confindece_thresh = 3.);

    CV_WRAP void collectGarbage() CV_OVERRIDE;
    CV_WRAP static Ptr<BestOf2NearestMatcher> create(bool try_use_gpu = false, float match_conf = 0.3f, int num_matches_thresh1 = 6,
        int num_matches_thresh2 = 6, double matches_confindece_thresh = 3.);

protected:

    void match(const ImageFeatures &features1, const ImageFeatures &features2, MatchesInfo &matches_info) CV_OVERRIDE;
    int num_matches_thresh1_;
    int num_matches_thresh2_;
    double matches_confindece_thresh_;
    Ptr<FeaturesMatcher> impl_;
};

class CV_EXPORTS_W BestOf2NearestRangeMatcher : public BestOf2NearestMatcher
{
public:
    CV_WRAP BestOf2NearestRangeMatcher(int range_width = 5, bool try_use_gpu = false, float match_conf = 0.3f,
                            int num_matches_thresh1 = 6, int num_matches_thresh2 = 6);

protected:
    // indicate that we do not want to hide the base class match method with a different signature
    using BestOf2NearestMatcher::match;
    void match(const std::vector<ImageFeatures> &features, std::vector<MatchesInfo> &pairwise_matches,
               const cv::UMat &mask = cv::UMat()) CV_OVERRIDE;

    int range_width_;
};

/** @brief Features matcher similar to cv::detail::BestOf2NearestMatcher which
finds two best matches for each feature and leaves the best one only if the
ratio between descriptor distances is greater than the threshold match_conf.

Unlike cv::detail::BestOf2NearestMatcher this matcher uses affine
transformation (affine transformation estimate will be placed in matches_info).

@sa cv::detail::FeaturesMatcher cv::detail::BestOf2NearestMatcher
 */
class CV_EXPORTS_W AffineBestOf2NearestMatcher : public BestOf2NearestMatcher
{
public:
    /** @brief Constructs a "best of 2 nearest" matcher that expects affine transformation
    between images

    @param full_affine whether to use full affine transformation with 6 degress of freedom or reduced
    transformation with 4 degrees of freedom using only rotation, translation and uniform scaling
    @param try_use_gpu Should try to use GPU or not
    @param match_conf Match distances ration threshold
    @param num_matches_thresh1 Minimum number of matches required for the 2D affine transform
    estimation used in the inliers classification step

    @sa cv::estimateAffine2D cv::estimateAffinePartial2D
     */
    CV_WRAP AffineBestOf2NearestMatcher(bool full_affine = false, bool try_use_gpu = false,
                                float match_conf = 0.3f, int num_matches_thresh1 = 6) :
        BestOf2NearestMatcher(try_use_gpu, match_conf, num_matches_thresh1, num_matches_thresh1),
        full_affine_(full_affine) {}

protected:
    void match(const ImageFeatures &features1, const ImageFeatures &features2, MatchesInfo &matches_info) CV_OVERRIDE;

    bool full_affine_;
};

//! @} stitching_match

} // namespace detail
} // namespace cv

#endif // OPENCV_STITCHING_MATCHERS_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **CV_EXPORTS_W**: A class/struct defined in this file
- **match**: A class/struct defined in this file
- **CV_EXPORTS_W_SIMPLE**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_STITCHING_MATCHERS_HPP()**: A function/method defined in this file
- **is()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/features2d.hpp`
- `opencv2/opencv_modules.hpp`

**Python Imports:**
- `the`
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

