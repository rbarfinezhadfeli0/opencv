# Documentation for `docs/samples/gpu/farneback_optical_flow.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/gpu/farneback_optical_flow.cpp_docs.md`
- **File Name**: `farneback_optical_flow.cpp_docs.md`
- **File Size**: 7,349 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/gpu/farneback_optical_flow.cpp_docs.md](../../../docs/samples/gpu/farneback_optical_flow.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/gpu/farneback_optical_flow.cpp`

## File Metadata

- **Full Path**: `samples/gpu/farneback_optical_flow.cpp`
- **File Name**: `farneback_optical_flow.cpp`
- **File Size**: 4,241 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/gpu/farneback_optical_flow.cpp](../../samples/gpu/farneback_optical_flow.cpp)

## Purpose and Role

This file is located in the `samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <vector>
#include <sstream>
#include <cmath>

#include "opencv2/core.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/video.hpp"
#include "opencv2/cudaoptflow.hpp"
#include "opencv2/cudaarithm.hpp"

using namespace std;
using namespace cv;
using namespace cv::cuda;

template <typename T>
inline T mapVal(T x, T a, T b, T c, T d)
{
    x = ::max(::min(x, b), a);
    return c + (d-c) * (x-a) / (b-a);
}

static void colorizeFlow(const Mat &u, const Mat &v, Mat &dst)
{
    double uMin, uMax;
    cv::minMaxLoc(u, &uMin, &uMax, 0, 0);
    double vMin, vMax;
    cv::minMaxLoc(v, &vMin, &vMax, 0, 0);
    uMin = ::abs(uMin); uMax = ::abs(uMax);
    vMin = ::abs(vMin); vMax = ::abs(vMax);
    float dMax = static_cast<float>(::max(::max(uMin, uMax), ::max(vMin, vMax)));

    dst.create(u.size(), CV_8UC3);
    for (int y = 0; y < u.rows; ++y)
    {
        for (int x = 0; x < u.cols; ++x)
        {
            dst.at<uchar>(y,3*x) = 0;
            dst.at<uchar>(y,3*x+1) = (uchar)mapVal(-v.at<float>(y,x), -dMax, dMax, 0.f, 255.f);
            dst.at<uchar>(y,3*x+2) = (uchar)mapVal(u.at<float>(y,x), -dMax, dMax, 0.f, 255.f);
        }
    }
}

int main(int argc, char **argv)
{
    CommandLineParser cmd(argc, argv,
            "{ l left  | ../data/basketball1.png | specify left image }"
            "{ r right | ../data/basketball2.png | specify right image }"
            "{ h help  | | print help message }");

    cmd.about("Farneback's optical flow sample.");
    if (cmd.has("help") || !cmd.check())
    {
        cmd.printMessage();
        cmd.printErrors();
        return 0;
    }


    string pathL = cmd.get<string>("left");
    string pathR = cmd.get<string>("right");
    if (pathL.empty()) cout << "Specify left image path\n";
    if (pathR.empty()) cout << "Specify right image path\n";
    if (pathL.empty() || pathR.empty()) return -1;

    Mat frameL = imread(pathL, IMREAD_GRAYSCALE);
    Mat frameR = imread(pathR, IMREAD_GRAYSCALE);
    if (frameL.empty()) cout << "Can't open '" << pathL << "'\n";
    if (frameR.empty()) cout << "Can't open '" << pathR << "'\n";
    if (frameL.empty() || frameR.empty()) return -1;

    GpuMat d_frameL(frameL), d_frameR(frameR);
    GpuMat d_flow;
    Ptr<cuda::FarnebackOpticalFlow> d_calc = cuda::FarnebackOpticalFlow::create();
    Mat flowxy, flowx, flowy, image;

    bool running = true, gpuMode = true;
    int64 t, t0=0, t1=1, tc0, tc1;

    cout << "Use 'm' for CPU/GPU toggling\n";

    while (running)
    {
        t = getTickCount();

        if (gpuMode)
        {
            tc0 = getTickCount();
            d_calc->calc(d_frameL, d_frameR, d_flow);
            tc1 = getTickCount();

            GpuMat planes[2];
            cuda::split(d_flow, planes);

            planes[0].download(flowx);
            planes[1].download(flowy);
        }
        else
        {
            tc0 = getTickCount();
            calcOpticalFlowFarneback(
                        frameL, frameR, flowxy, d_calc->getPyrScale(), d_calc->getNumLevels(), d_calc->getWinSize(),
                        d_calc->getNumIters(), d_calc->getPolyN(), d_calc->getPolySigma(), d_calc->getFlags());
            tc1 = getTickCount();

            Mat planes[] = {flowx, flowy};
            split(flowxy, planes);
            flowx = planes[0]; flowy = planes[1];
        }

        colorizeFlow(flowx, flowy, image);

        stringstream s;
        s << "mode: " << (gpuMode?"GPU":"CPU");
        putText(image, s.str(), Point(5, 25), FONT_HERSHEY_SIMPLEX, 1., Scalar(255,0,255), 2);

        s.str("");
        s << "opt. flow FPS: " << cvRound((getTickFrequency()/(tc1-tc0)));
        putText(image, s.str(), Point(5, 65), FONT_HERSHEY_SIMPLEX, 1., Scalar(255,0,255), 2);

        s.str("");
        s << "total FPS: " << cvRound((getTickFrequency()/(t1-t0)));
        putText(image, s.str(), Point(5, 105), FONT_HERSHEY_SIMPLEX, 1., Scalar(255,0,255), 2);

        imshow("flow", image);

        char ch = (char)waitKey(3);
        if (ch == 27)
            running = false;
        else if (ch == 'm' || ch == 'M')
            gpuMode = !gpuMode;

        t0 = t;
        t1 = getTickCount();
    }

    return 0;
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
- `opencv2/video.hpp`
- `vector`
- `opencv2/cudaoptflow.hpp`
- `iostream`
- `opencv2/cudaarithm.hpp`
- `sstream`
- `opencv2/core/utility.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

