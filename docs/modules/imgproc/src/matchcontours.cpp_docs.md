# Documentation for `modules/imgproc/src/matchcontours.cpp`

## File Metadata

- **Full Path**: `modules/imgproc/src/matchcontours.cpp`
- **File Name**: `matchcontours.cpp`
- **File Size**: 5,677 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgproc/src/matchcontours.cpp](../../../modules/imgproc/src/matchcontours.cpp)

## Purpose and Role

This file is located in the `modules/imgproc/src` directory and serves as part of the OpenCV library infrastructure.

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
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
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
//   * The name of Intel Corporation may not be used to endorse or promote products
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


double cv::matchShapes(InputArray contour1, InputArray contour2, int method, double)
{
    CV_INSTRUMENT_REGION();

    double ma[7], mb[7];
    int i, sma, smb;
    double eps = 1.e-5;
    double mmm;
    double result = 0;
    bool anyA = false, anyB = false;

    HuMoments( moments(contour1), ma );
    HuMoments( moments(contour2), mb );

    switch (method)
    {
    case 1:
        for( i = 0; i < 7; i++ )
        {
            double ama = fabs( ma[i] );
            double amb = fabs( mb[i] );

            if (ama > 0)
                anyA = true;
            if (amb > 0)
                anyB = true;

            if( ma[i] > 0 )
                sma = 1;
            else if( ma[i] < 0 )
                sma = -1;
            else
                sma = 0;
            if( mb[i] > 0 )
                smb = 1;
            else if( mb[i] < 0 )
                smb = -1;
            else
                smb = 0;

            if( ama > eps && amb > eps )
            {
                ama = 1. / (sma * log10( ama ));
                amb = 1. / (smb * log10( amb ));
                result += fabs( -ama + amb );
            }
        }
        break;

    case 2:
        for( i = 0; i < 7; i++ )
        {
            double ama = fabs( ma[i] );
            double amb = fabs( mb[i] );

            if (ama > 0)
                anyA = true;
            if (amb > 0)
                anyB = true;

            if( ma[i] > 0 )
                sma = 1;
            else if( ma[i] < 0 )
                sma = -1;
            else
                sma = 0;
            if( mb[i] > 0 )
                smb = 1;
            else if( mb[i] < 0 )
                smb = -1;
            else
                smb = 0;

            if( ama > eps && amb > eps )
            {
                ama = sma * log10( ama );
                amb = smb * log10( amb );
                result += fabs( -ama + amb );
            }
        }
        break;

    case 3:
        for( i = 0; i < 7; i++ )
        {
            double ama = fabs( ma[i] );
            double amb = fabs( mb[i] );

            if (ama > 0)
                anyA = true;
            if (amb > 0)
                anyB = true;

            if( ma[i] > 0 )
                sma = 1;
            else if( ma[i] < 0 )
                sma = -1;
            else
                sma = 0;
            if( mb[i] > 0 )
                smb = 1;
            else if( mb[i] < 0 )
                smb = -1;
            else
                smb = 0;

            if( ama > eps && amb > eps )
            {
                ama = sma * log10( ama );
                amb = smb * log10( amb );
                mmm = fabs( (ama - amb) / ama );
                if( result < mmm )
                    result = mmm;
            }
        }
        break;
    default:
        CV_Error( cv::Error::StsBadArg, "Unknown comparison method" );
    }

    //If anyA and anyB are both true, the result is correct.
    //If anyA and anyB are both false, the distance is 0, perfect match.
    //If only one is true, then it's a false 0 and return large error.
    if (anyA != anyB)
        result = DBL_MAX;

    return result;
}


CV_IMPL  double
cvMatchShapes( const void* _contour1, const void* _contour2,
               int method, double parameter )
{
    cv::AutoBuffer<double> abuf1, abuf2;
    cv::Mat contour1 = cv::cvarrToMat(_contour1, false, false, 0, &abuf1);
    cv::Mat contour2 = cv::cvarrToMat(_contour2, false, false, 0, &abuf2);

    return cv::matchShapes(contour1, contour2, method, parameter);
}

/* End of file. */
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

