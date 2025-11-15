# Documentation for `samples/gpu/super_resolution.cpp`

## File Metadata

- **Full Path**: `samples/gpu/super_resolution.cpp`
- **File Name**: `super_resolution.cpp`
- **File Size**: 4,875 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/gpu/super_resolution.cpp](../../samples/gpu/super_resolution.cpp)

## Purpose and Role

This file is located in the `samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <iomanip>
#include <string>
#include <ctype.h>

#include "opencv2/core.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/superres.hpp"
#include "opencv2/superres/optical_flow.hpp"
#include "opencv2/opencv_modules.hpp"

using namespace std;
using namespace cv;
using namespace cv::superres;

#define MEASURE_TIME(op) \
    { \
        TickMeter tm; \
        tm.start(); \
        op; \
        tm.stop(); \
        cout << tm.getTimeSec() << " sec" << endl; \
    }

static Ptr<cv::superres::DenseOpticalFlowExt> createOptFlow(const string& name, bool useGpu)
{
    if (name == "farneback")
    {
        if (useGpu)
            return cv::superres::createOptFlow_Farneback_CUDA();
        else
            return cv::superres::createOptFlow_Farneback();
    }
    /*else if (name == "simple")
        return createOptFlow_Simple();*/
    else if (name == "tvl1")
    {
        if (useGpu)
            return cv::superres::createOptFlow_DualTVL1_CUDA();
        else
            return cv::superres::createOptFlow_DualTVL1();
    }
    else if (name == "brox")
        return cv::superres::createOptFlow_Brox_CUDA();
    else if (name == "pyrlk")
        return cv::superres::createOptFlow_PyrLK_CUDA();
    else
        cerr << "Incorrect Optical Flow algorithm - " << name << endl;

    return Ptr<cv::superres::DenseOpticalFlowExt>();
}

int main(int argc, const char* argv[])
{
    CommandLineParser cmd(argc, argv,
        "{ v video      |           | Input video (mandatory)}"
        "{ o output     |           | Output video }"
        "{ s scale      | 4         | Scale factor }"
        "{ i iterations | 180       | Iteration count }"
        "{ t temporal   | 4         | Radius of the temporal search area }"
        "{ f flow       | farneback | Optical flow algorithm (farneback, tvl1, brox, pyrlk) }"
        "{ g gpu        | false     | CPU as default device, cuda for CUDA }"
        "{ h help       | false     | Print help message }"
    );

    const string inputVideoName = cmd.get<string>("video");
    if (cmd.get<bool>("help") || inputVideoName.empty())
    {
        cout << "This sample demonstrates Super Resolution algorithms for video sequence" << endl;
        cmd.printMessage();
        return EXIT_SUCCESS;
    }

    const string outputVideoName = cmd.get<string>("output");
    const int scale = cmd.get<int>("scale");
    const int iterations = cmd.get<int>("iterations");
    const int temporalAreaRadius = cmd.get<int>("temporal");
    const string optFlow = cmd.get<string>("flow");
    string gpuOption = cmd.get<string>("gpu");

    std::transform(gpuOption.begin(), gpuOption.end(), gpuOption.begin(), ::tolower);

    bool useCuda = gpuOption.compare("cuda") == 0;
    Ptr<SuperResolution> superRes;

    if (useCuda)
        superRes = createSuperResolution_BTVL1_CUDA();
    else
        superRes = createSuperResolution_BTVL1();

    Ptr<cv::superres::DenseOpticalFlowExt> of = createOptFlow(optFlow, useCuda);

    if (of.empty())
        return EXIT_FAILURE;
    superRes->setOpticalFlow(of);

    superRes->setScale(scale);
    superRes->setIterations(iterations);
    superRes->setTemporalAreaRadius(temporalAreaRadius);

    Ptr<FrameSource> frameSource;
    if (useCuda)
    {
        // Try to use gpu Video Decoding
        try
        {
            frameSource = createFrameSource_Video_CUDA(inputVideoName);
            Mat frame;
            frameSource->nextFrame(frame);
        }
        catch (const cv::Exception&)
        {
            frameSource.release();
        }
    }
    if (!frameSource)
        frameSource = createFrameSource_Video(inputVideoName);

    // skip first frame, it is usually corrupted
    {
        Mat frame;
        frameSource->nextFrame(frame);
        cout << "Input           : " << inputVideoName << " " << frame.size() << endl;
        cout << "Scale factor    : " << scale << endl;
        cout << "Iterations      : " << iterations << endl;
        cout << "Temporal radius : " << temporalAreaRadius << endl;
        cout << "Optical Flow    : " << optFlow << endl;
        cout << "Mode            : " << (useCuda ? "CUDA" : "CPU") << endl;
    }

    superRes->setInput(frameSource);

    VideoWriter writer;

    for (int i = 0;; ++i)
    {
        cout << '[' << setw(3) << i << "] : " << flush;
        Mat result;

        MEASURE_TIME(superRes->nextFrame(result));

        if (result.empty())
            break;

        imshow("Super Resolution", result);

        if (waitKey(1000) > 0)
            break;

        if (!outputVideoName.empty())
        {
            if (!writer.isOpened())
                writer.open(outputVideoName, VideoWriter::fourcc('X', 'V', 'I', 'D'), 25.0, result.size());
            writer << result;
        }
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
- `ctype.h`
- `iomanip`
- `opencv2/superres/optical_flow.hpp`
- `opencv2/opencv_modules.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `string`
- `opencv2/superres.hpp`
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

