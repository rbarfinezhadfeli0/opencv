# Documentation for `modules/photo/src/contrast_preserve.cpp`

## File Metadata

- **Full Path**: `modules/photo/src/contrast_preserve.cpp`
- **File Name**: `contrast_preserve.cpp`
- **File Size**: 5,664 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/photo/src/contrast_preserve.cpp](../../../modules/photo/src/contrast_preserve.cpp)

## Purpose and Role

This file is located in the `modules/photo/src` directory and serves as part of the OpenCV library infrastructure.

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
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
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
#include "opencv2/photo.hpp"
#include <cmath>
#include <vector>
#include <limits>
#include "contrast_preserve.hpp"

using namespace std;
using namespace cv;

void cv::decolor(InputArray _src, OutputArray _dst, OutputArray _color_boost)
{
    CV_INSTRUMENT_REGION();

    Mat I = _src.getMat();
    _dst.create(I.size(), CV_8UC1);
    Mat dst = _dst.getMat();

    _color_boost.create(I.size(), CV_8UC3);
    Mat color_boost = _color_boost.getMat();

    CV_Assert(!I.empty() && (I.channels()==3));

    // Parameter Setting
    const int maxIter = 15;
    const double tol = .0001;
    int iterCount = 0;
    double E = 0;
    double pre_E = std::numeric_limits<double>::infinity();

    Mat img;
    I.convertTo(img, CV_32FC3, 1.0/255.0);

    // Initialization
    Decolor obj;

    vector <double> Cg;
    vector < vector <double> > polyGrad;
    vector <Vec3i> comb;
    vector <double> alf;

    obj.grad_system(img,polyGrad,Cg,comb);
    obj.weak_order(img,alf);

    // Solver
    Mat Mt = Mat(int(polyGrad.size()),int(polyGrad[0].size()), CV_32FC1);
    obj.wei_update_matrix(polyGrad,Cg,Mt);

    vector <double> wei;
    obj.wei_inti(comb,wei);

    //////////////////////////////// main loop starting ////////////////////////////////////////

    vector <double> G_pos(alf.size());
    vector <double> G_neg(alf.size());
    vector <double> EXPsum(G_pos.size());
    vector <double> EXPterm(G_pos.size());
    vector <double> temp(polyGrad[0].size());
    vector <double> temp1(polyGrad[0].size());
    vector <double> temp2(EXPsum.size());
    vector <double> wei1(polyGrad.size());

    while(sqrt(pow(E-pre_E,2)) > tol)
    {
        iterCount +=1;
        pre_E = E;

        for(size_t i=0; i<polyGrad[0].size(); i++)
        {
            double val = 0.0;
            for(size_t j=0; j<polyGrad.size(); j++)
                val = val + (polyGrad[j][i] * wei[j]);
            temp[i] = val - Cg[i];
            temp1[i] = val + Cg[i];
        }

        for(size_t i=0; i<alf.size(); i++)
        {
            const double sqSigma = obj.sigma * obj.sigma;
            const double pos = ((1 + alf[i])/2) * exp(-1.0 * 0.5 * (temp[i] * temp[i]) / sqSigma);
            const double neg = ((1 - alf[i])/2) * exp(-1.0 * 0.5 * (temp1[i] * temp1[i]) / sqSigma);
            G_pos[i] = pos;
            G_neg[i] = neg;
        }

        for(size_t i=0; i<G_pos.size(); i++)
            EXPsum[i] = G_pos[i]+G_neg[i];

        for(size_t i=0; i<EXPsum.size(); i++)
            temp2[i] = (EXPsum[i] == 0) ? 1.0 : 0.0;

        for(size_t i=0; i<G_pos.size(); i++)
            EXPterm[i] = (G_pos[i] - G_neg[i])/(EXPsum[i] + temp2[i]);

        for(int i=0; i<int(polyGrad.size()); i++)
        {
            double val1 = 0.0;
            for(int j=0; j<int(polyGrad[0].size()); j++)
            {
                val1 = val1 + (Mt.at<float>(i,j) * EXPterm[j]);
            }
            wei1[i] = val1;
        }

        for(size_t i=0; i<wei.size(); i++)
            wei[i] = wei1[i];

        E = obj.energyCalcu(Cg, polyGrad, wei);

        if(iterCount > maxIter)
            break;
    }

    Mat Gray = Mat::zeros(img.size(),CV_32FC1);
    obj.grayImContruct(wei, img, Gray);

    Gray.convertTo(dst,CV_8UC1,255);

    ///////////////////////////////////       Contrast Boosting   /////////////////////////////////

    Mat lab;
    cvtColor(I,lab,COLOR_BGR2Lab);

    vector <Mat> lab_channel;
    split(lab,lab_channel);

    dst.copyTo(lab_channel[0]);

    merge(lab_channel,lab);

    cvtColor(lab,color_boost,COLOR_Lab2BGR);
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
- `cmath`
- `contrast_preserve.hpp`
- `vector`
- `opencv2/photo.hpp`
- `limits`
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

