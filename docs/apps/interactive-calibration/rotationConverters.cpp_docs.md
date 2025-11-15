# Documentation for `apps/interactive-calibration/rotationConverters.cpp`

## File Metadata

- **Full Path**: `apps/interactive-calibration/rotationConverters.cpp`
- **File Name**: `rotationConverters.cpp`
- **File Size**: 3,630 bytes
- **File Type**: .cpp
- **Link to Source**: [apps/interactive-calibration/rotationConverters.cpp](../../apps/interactive-calibration/rotationConverters.cpp)

## Purpose and Role

This file is located in the `apps/interactive-calibration` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "rotationConverters.hpp"

#include <cmath>

#include <opencv2/calib3d.hpp>
#include <opencv2/core.hpp>

#define CALIB_PI 3.14159265358979323846
#define CALIB_PI_2 1.57079632679489661923

void calib::Euler(const cv::Mat& src, cv::Mat& dst, int argType)
{
    if((src.rows == 3) && (src.cols == 3))
    {
        //convert rotation matrix to 3 angles (pitch, yaw, roll)
        dst = cv::Mat(3, 1, CV_64F);
        double pitch, yaw, roll;

        if(src.at<double>(0,2) < -0.998)
        {
            pitch = -atan2(src.at<double>(1,0), src.at<double>(1,1));
            yaw = -CALIB_PI_2;
            roll = 0.;
        }
        else if(src.at<double>(0,2) > 0.998)
        {
            pitch = atan2(src.at<double>(1,0), src.at<double>(1,1));
            yaw = CALIB_PI_2;
            roll = 0.;
        }
        else
        {
            pitch = atan2(-src.at<double>(1,2), src.at<double>(2,2));
            yaw = asin(src.at<double>(0,2));
            roll = atan2(-src.at<double>(0,1), src.at<double>(0,0));
        }

        if(argType == CALIB_DEGREES)
        {
            pitch *= 180./CALIB_PI;
            yaw *= 180./CALIB_PI;
            roll *= 180./CALIB_PI;
        }
        else if(argType != CALIB_RADIANS)
            CV_Error(cv::Error::StsBadFlag, "Invalid argument type");

        dst.at<double>(0,0) = pitch;
        dst.at<double>(1,0) = yaw;
        dst.at<double>(2,0) = roll;
    }
    else if( (src.cols == 1 && src.rows == 3) ||
             (src.cols == 3 && src.rows == 1 ) )
    {
        //convert vector which contains 3 angles (pitch, yaw, roll) to rotation matrix
        double pitch, yaw, roll;
        if(src.cols == 1 && src.rows == 3)
        {
            pitch = src.at<double>(0,0);
            yaw = src.at<double>(1,0);
            roll = src.at<double>(2,0);
        }
        else{
            pitch = src.at<double>(0,0);
            yaw = src.at<double>(0,1);
            roll = src.at<double>(0,2);
        }

        if(argType == CALIB_DEGREES)
        {
            pitch *= CALIB_PI / 180.;
            yaw *= CALIB_PI / 180.;
            roll *= CALIB_PI / 180.;
        }
        else if(argType != CALIB_RADIANS)
            CV_Error(cv::Error::StsBadFlag, "Invalid argument type");

        dst = cv::Mat(3, 3, CV_64F);
        cv::Mat M(3, 3, CV_64F);
        cv::Mat i = cv::Mat::eye(3, 3, CV_64F);
        i.copyTo(dst);
        i.copyTo(M);

        double* pR = dst.ptr<double>();
        pR[4] = cos(pitch);
        pR[7] = sin(pitch);
        pR[8] = pR[4];
        pR[5] = -pR[7];

        double* pM = M.ptr<double>();
        pM[0] = cos(yaw);
        pM[2] = sin(yaw);
        pM[8] = pM[0];
        pM[6] = -pM[2];

        dst *= M;
        i.copyTo(M);
        pM[0] = cos(roll);
        pM[3] = sin(roll);
        pM[4] = pM[0];
        pM[1] = -pM[3];

        dst *= M;
    }
    else
        CV_Error(cv::Error::StsBadFlag, "Input matrix must be 1x3, 3x1 or 3x3" );
}

void calib::RodriguesToEuler(const cv::Mat& src, cv::Mat& dst, int argType)
{
    CV_Assert((src.cols == 1 && src.rows == 3) || (src.cols == 3 && src.rows == 1));
    cv::Mat R;
    cv::Rodrigues(src, R);
    Euler(R, dst, argType);
}

void calib::EulerToRodrigues(const cv::Mat& src, cv::Mat& dst, int argType)
{
    CV_Assert((src.cols == 1 && src.rows == 3) || (src.cols == 3 && src.rows == 1));
    cv::Mat R;
    Euler(src, R, argType);
    cv::Rodrigues(R, dst);
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
- `opencv2/core.hpp`
- `opencv2/calib3d.hpp`
- `rotationConverters.hpp`


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

