# Documentation for `modules/calib3d/test/test_reproject_image_to_3d.cpp`

## File Metadata

- **Full Path**: `modules/calib3d/test/test_reproject_image_to_3d.cpp`
- **File Name**: `test_reproject_image_to_3d.cpp`
- **File Size**: 6,784 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/calib3d/test/test_reproject_image_to_3d.cpp](../../../modules/calib3d/test/test_reproject_image_to_3d.cpp)

## Purpose and Role

This file is located in the `modules/calib3d/test` directory and serves as part of the OpenCV library infrastructure.

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

#include "test_precomp.hpp"

namespace opencv_test { namespace {

template<class T> double thres() { return 1.0; }
template<> double thres<float>() { return 1e-5; }

class CV_ReprojectImageTo3DTest : public cvtest::BaseTest
{
public:
    CV_ReprojectImageTo3DTest() {}
    ~CV_ReprojectImageTo3DTest() {}
protected:


    void run(int)
    {
        ts->set_failed_test_info(cvtest::TS::OK);
        int progress = 0;
        int caseId = 0;

        progress = update_progress( progress, 1, 14, 0 );
        runCase<float, float>(++caseId, -100.f, 100.f);
        progress = update_progress( progress, 2, 14, 0 );
        runCase<int, float>(++caseId, -100, 100);
        progress = update_progress( progress, 3, 14, 0 );
        runCase<short, float>(++caseId, -100, 100);
        progress = update_progress( progress, 4, 14, 0 );
        runCase<unsigned char, float>(++caseId, 10, 100);
        progress = update_progress( progress, 5, 14, 0 );

        runCase<float, int>(++caseId, -100.f, 100.f);
        progress = update_progress( progress, 6, 14, 0 );
        runCase<int, int>(++caseId, -100, 100);
        progress = update_progress( progress, 7, 14, 0 );
        runCase<short, int>(++caseId, -100, 100);
        progress = update_progress( progress, 8, 14, 0 );
        runCase<unsigned char, int>(++caseId, 10, 100);
        progress = update_progress( progress, 10, 14, 0 );

        runCase<float, short>(++caseId, -100.f, 100.f);
        progress = update_progress( progress, 11, 14, 0 );
        runCase<int, short>(++caseId, -100, 100);
        progress = update_progress( progress, 12, 14, 0 );
        runCase<short, short>(++caseId, -100, 100);
        progress = update_progress( progress, 13, 14, 0 );
        runCase<unsigned char, short>(++caseId, 10, 100);
        progress = update_progress( progress, 14, 14, 0 );
    }

    template<class U, class V> double error(const Vec<U, 3>& v1, const Vec<V, 3>& v2) const
    {
        double tmp, sum = 0;
        double nsum = 0;
        for(int i = 0; i < 3; ++i)
        {
            tmp = v1[i];
            nsum +=  tmp * tmp;

            tmp = tmp - v2[i];
            sum += tmp * tmp;

        }
        return sqrt(sum)/(sqrt(nsum)+1.);
    }

    template<class InT, class OutT> void runCase(int caseId, InT min, InT max)
    {
        typedef Vec<OutT, 3> out3d_t;

        bool handleMissingValues = (unsigned)theRNG() % 2 == 0;

        Mat_<InT> disp(Size(320, 240));
        randu(disp, Scalar(min), Scalar(max));

        if (handleMissingValues)
            disp(disp.rows/2, disp.cols/2) = min - 1;

        Mat_<double> Q(4, 4);
        randu(Q, Scalar(-5), Scalar(5));
        Mat_<out3d_t> _3dImg(disp.size());

        reprojectImageTo3D(disp, _3dImg, Q, handleMissingValues);

        for(int y = 0; y < disp.rows; ++y)
            for(int x = 0; x < disp.cols; ++x)
            {
                InT d = disp(y, x);

                double from[4] = {
                    static_cast<double>(x),
                    static_cast<double>(y),
                    static_cast<double>(d),
                    1.0,
                };
                Mat_<double> res = Q * Mat_<double>(4, 1, from);
                res /= res(3, 0);

                out3d_t pixel_exp = *res.ptr<Vec3d>();
                out3d_t pixel_out = _3dImg(y, x);

                const int largeZValue = 10000; /* see documentation */

                if (handleMissingValues && y == disp.rows/2 && x == disp.cols/2)
                {
                    if (pixel_out[2] == largeZValue)
                        continue;

                    ts->printf(cvtest::TS::LOG, "Missing values are handled improperly\n");
                    ts->set_failed_test_info( cvtest::TS::FAIL_BAD_ACCURACY );
                    return;
                }
                else
                {
                    double err = error(pixel_out, pixel_exp), t = thres<OutT>();
                    if ( err > t )
                    {
                        ts->printf(cvtest::TS::LOG, "case %d. too big error at (%d, %d): %g vs expected %g: res = (%g, %g, %g, w=%g) vs pixel_out = (%g, %g, %g)\n",
                            caseId, x, y, err, t, res(0,0), res(1,0), res(2,0), res(3,0),
                            (double)pixel_out[0], (double)pixel_out[1], (double)pixel_out[2]);
                        ts->set_failed_test_info( cvtest::TS::FAIL_BAD_ACCURACY );
                        return;
                    }
                }
            }
    }
};

TEST(Calib3d_ReprojectImageTo3D, accuracy) { CV_ReprojectImageTo3DTest test; test.safe_run(); }

}} // namespace
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

- **InT**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **V**: A class/struct defined in this file
- **OutT**: A class/struct defined in this file
- **CV_ReprojectImageTo3DTest**: A class/struct defined in this file
- **U**: A class/struct defined in this file

### Functions and Methods

- **Vec()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `test_precomp.hpp`

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

