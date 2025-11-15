# Documentation for `modules/stitching/src/autocalib.cpp`

## File Metadata

- **Full Path**: `modules/stitching/src/autocalib.cpp`
- **File Name**: `autocalib.cpp`
- **File Size**: 6,546 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/stitching/src/autocalib.cpp](../../../modules/stitching/src/autocalib.cpp)

## Purpose and Role

This file is located in the `modules/stitching/src` directory and serves as part of the OpenCV library infrastructure.

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

#include "precomp.hpp"
#include "opencv2/core/hal/hal.hpp"

using namespace cv;

namespace {

static inline bool decomposeCholesky(double* A, size_t astep, int m)
{
    if (!hal::Cholesky64f(A, astep, m, 0, 0, 0))
        return false;
    return true;
}

} // namespace


namespace cv {
namespace detail {

void focalsFromHomography(const Mat& H, double &f0, double &f1, bool &f0_ok, bool &f1_ok)
{
    CV_Assert(H.type() == CV_64F && H.size() == Size(3, 3));

    const double* h = H.ptr<double>();

    double d1, d2; // Denominators
    double v1, v2; // Focal squares value candidates

    f1_ok = true;
    d1 = h[6] * h[7];
    d2 = (h[7] - h[6]) * (h[7] + h[6]);
    v1 = -(h[0] * h[1] + h[3] * h[4]) / d1;
    v2 = (h[0] * h[0] + h[3] * h[3] - h[1] * h[1] - h[4] * h[4]) / d2;
    if (v1 < v2)
    {
        std::swap(v1, v2);
        std::swap(d1, d2);
    }
    if (v1 > 0 && v2 > 0) f1 = std::sqrt(std::abs(d1) > std::abs(d2) ? v1 : v2);
    else if (v1 > 0) f1 = std::sqrt(v1);
    else f1_ok = false;

    f0_ok = true;
    d1 = h[0] * h[3] + h[1] * h[4];
    d2 = h[0] * h[0] + h[1] * h[1] - h[3] * h[3] - h[4] * h[4];
    v1 = -h[2] * h[5] / d1;
    v2 = (h[5] * h[5] - h[2] * h[2]) / d2;
    if (v1 < v2)
    {
        std::swap(v1, v2);
        std::swap(d1, d2);
    }
    if (v1 > 0 && v2 > 0) f0 = std::sqrt(std::abs(d1) > std::abs(d2) ? v1 : v2);
    else if (v1 > 0) f0 = std::sqrt(v1);
    else f0_ok = false;
}


void estimateFocal(const std::vector<ImageFeatures> &features, const std::vector<MatchesInfo> &pairwise_matches,
                       std::vector<double> &focals)
{
    const int num_images = static_cast<int>(features.size());
    focals.resize(num_images);

    std::vector<double> all_focals;

    for (int i = 0; i < num_images; ++i)
    {
        for (int j = 0; j < num_images; ++j)
        {
            const MatchesInfo &m = pairwise_matches[i*num_images + j];
            if (m.H.empty())
                continue;
            double f0, f1;
            bool f0ok, f1ok;
            focalsFromHomography(m.H, f0, f1, f0ok, f1ok);
            if (f0ok && f1ok)
                all_focals.push_back(std::sqrt(f0 * f1));
        }
    }

    if (static_cast<int>(all_focals.size()) >= num_images - 1)
    {
        double median;

        std::sort(all_focals.begin(), all_focals.end());
        if (all_focals.size() % 2 == 1)
            median = all_focals[all_focals.size() / 2];
        else
            median = (all_focals[all_focals.size() / 2 - 1] + all_focals[all_focals.size() / 2]) * 0.5;

        for (int i = 0; i < num_images; ++i)
            focals[i] = median;
    }
    else
    {
        LOGLN("Can't estimate focal length, will use naive approach");
        double focals_sum = 0;
        for (int i = 0; i < num_images; ++i)
            focals_sum += features[i].img_size.width + features[i].img_size.height;
        for (int i = 0; i < num_images; ++i)
            focals[i] = focals_sum / num_images;
    }
}


bool calibrateRotatingCamera(const std::vector<Mat> &Hs, Mat &K)
{
    int m = static_cast<int>(Hs.size());
    CV_Assert(m >= 1);

    std::vector<Mat> Hs_(m);
    for (int i = 0; i < m; ++i)
    {
        CV_Assert(Hs[i].size() == Size(3, 3) && Hs[i].type() == CV_64F);
        Hs_[i] = Hs[i] / std::pow(determinant(Hs[i]), 1./3.);
    }

    const int idx_map[3][3] = {{0, 1, 2}, {1, 3, 4}, {2, 4, 5}};
    Mat_<double> A(6*m, 6);
    A.setTo(0);

    int eq_idx = 0;
    for (int k = 0; k < m; ++k)
    {
        Mat_<double> H(Hs_[k]);
        for (int i = 0; i < 3; ++i)
        {
            for (int j = i; j < 3; ++j, ++eq_idx)
            {
                for (int l = 0; l < 3; ++l)
                {
                    for (int s = 0; s < 3; ++s)
                    {
                        int idx = idx_map[l][s];
                        A(eq_idx, idx) += H(i,l) * H(j,s);
                    }
                }
                A(eq_idx, idx_map[i][j]) -= 1;
            }
        }
    }

    Mat_<double> wcoef;
    SVD::solveZ(A, wcoef);

    Mat_<double> W(3,3);
    for (int i = 0; i < 3; ++i)
        for (int j = i; j < 3; ++j)
            W(i,j) = W(j,i) = wcoef(idx_map[i][j], 0) / wcoef(5,0);
    if (!decomposeCholesky(W.ptr<double>(), W.step, 3))
        return false;
    W(0,1) = W(0,2) = W(1,2) = 0;
    K = W.t();
    return true;
}

} // namespace detail
} // namespace cv
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
- `precomp.hpp`
- `opencv2/core/hal/hal.hpp`

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

