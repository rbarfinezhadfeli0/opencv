# Documentation for `modules/video/perf/opencl/perf_bgfg_knn.cpp`

## File Metadata

- **Full Path**: `modules/video/perf/opencl/perf_bgfg_knn.cpp`
- **File Name**: `perf_bgfg_knn.cpp`
- **File Size**: 2,760 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/video/perf/opencl/perf_bgfg_knn.cpp](../../../../modules/video/perf/opencl/perf_bgfg_knn.cpp)

## Purpose and Role

This file is located in the `modules/video/perf/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../perf_precomp.hpp"
#include "opencv2/ts/ocl_perf.hpp"

#ifdef HAVE_OPENCL
#include "../perf_bgfg_utils.hpp"

namespace cvtest {
namespace ocl {

//////////////////////////// KNN//////////////////////////

typedef tuple<string, int> VideoKNNParamType;
typedef TestBaseWithParam<VideoKNNParamType> KNN_Apply;
typedef TestBaseWithParam<VideoKNNParamType> KNN_GetBackgroundImage;

using namespace opencv_test;

OCL_PERF_TEST_P(KNN_Apply, KNN, Combine(Values("cv/video/768x576.avi", "cv/video/1920x1080.avi"), Values(1,3)))
{
    VideoKNNParamType params = GetParam();

    const string inputFile = getDataPath(get<0>(params));

    const int cn = get<1>(params);
    int nFrame = 5;

    vector<Mat> frame_buffer(nFrame);

    cv::VideoCapture cap(inputFile);
    if (!cap.isOpened())
        throw SkipTestException("Video file can not be opened");
    prepareData(cap, cn, frame_buffer);

    UMat u_foreground;

    OCL_TEST_CYCLE()
    {
        Ptr<cv::BackgroundSubtractorKNN> knn = createBackgroundSubtractorKNN();
        knn->setDetectShadows(false);
        u_foreground.release();
        for (int i = 0; i < nFrame; i++)
        {
            knn->apply(frame_buffer[i], u_foreground);
        }
    }
    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST_P(KNN_GetBackgroundImage, KNN, Values(
        std::make_pair<string, int>("cv/video/768x576.avi", 5),
        std::make_pair<string, int>("cv/video/1920x1080.avi", 5)))
{
    VideoKNNParamType params = GetParam();

    const string inputFile = getDataPath(get<0>(params));

    const int cn = 3;
    const int skipFrames = get<1>(params);
    int nFrame = 10;

    vector<Mat> frame_buffer(nFrame);

    cv::VideoCapture cap(inputFile);
    if (!cap.isOpened())
        throw SkipTestException("Video file can not be opened");
    prepareData(cap, cn, frame_buffer, skipFrames);

    UMat u_foreground, u_background;

    OCL_TEST_CYCLE()
    {
        Ptr<cv::BackgroundSubtractorKNN> knn = createBackgroundSubtractorKNN();
        knn->setDetectShadows(false);
        u_foreground.release();
        u_background.release();
        for (int i = 0; i < nFrame; i++)
        {
            knn->apply(frame_buffer[i], u_foreground);
        }
        knn->getBackgroundImage(u_background);
    }
#ifdef DEBUG_BGFG
    imwrite(format("fg_%d_%d_knn_ocl.png", frame_buffer[0].rows, cn), u_foreground.getMat(ACCESS_READ));
    imwrite(format("bg_%d_%d_knn_ocl.png", frame_buffer[0].rows, cn), u_background.getMat(ACCESS_READ));
#endif
    SANITY_CHECK_NOTHING();
}

}}// namespace cvtest::ocl

#endif
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

### Functions and Methods

- **HAVE_OPENCL()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file
- **TestBaseWithParam()**: A function/method defined in this file
- **DEBUG_BGFG()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_perf.hpp`
- `../perf_bgfg_utils.hpp`
- `../perf_precomp.hpp`


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

