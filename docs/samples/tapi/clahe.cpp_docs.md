# Documentation for `samples/tapi/clahe.cpp`

## File Metadata

- **Full Path**: `samples/tapi/clahe.cpp`
- **File Name**: `clahe.cpp`
- **File Size**: 3,014 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/tapi/clahe.cpp](../../samples/tapi/clahe.cpp)

## Purpose and Role

This file is located in the `samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include "opencv2/core.hpp"
#include "opencv2/core/ocl.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"

using namespace cv;
using namespace std;

Ptr<CLAHE> pFilter;
int tilesize;
int cliplimit;

static void TSize_Callback(int pos, void* /*data*/)
{
    if(pos==0)
        pFilter->setTilesGridSize(Size(1,1));
    else
        pFilter->setTilesGridSize(Size(tilesize,tilesize));
}

static void Clip_Callback(int, void* /*data*/)
{
    pFilter->setClipLimit(cliplimit);
}

int main(int argc, char** argv)
{
    const char* keys =
        "{ i input    |                    | specify input image }"
        "{ c camera   |  0                 | specify camera id   }"
        "{ o output   | clahe_output.jpg   | specify output save path}"
        "{ h help     |                    | print help message }";

    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help"))
    {
        cout << "Usage : clahe [options]" << endl;
        cout << "Available options:" << endl;
        cmd.printMessage();
        return EXIT_SUCCESS;
    }

    string infile = cmd.get<string>("i"), outfile = cmd.get<string>("o");
    int camid = cmd.get<int>("c");
    VideoCapture capture;

    namedWindow("CLAHE");
    createTrackbar("Tile Size", "CLAHE", &tilesize, 32, (TrackbarCallback)TSize_Callback);
    createTrackbar("Clip Limit", "CLAHE", &cliplimit, 20, (TrackbarCallback)Clip_Callback);

    UMat frame, outframe;

    int cur_clip;
    Size cur_tilesize;
    pFilter = createCLAHE();

    cur_clip = (int)pFilter->getClipLimit();
    cur_tilesize = pFilter->getTilesGridSize();
    setTrackbarPos("Tile Size", "CLAHE", cur_tilesize.width);
    setTrackbarPos("Clip Limit", "CLAHE", cur_clip);

    if(!infile.empty())
    {
        infile = samples::findFile(infile);
        imread(infile).copyTo(frame);
        if(frame.empty())
        {
            cout << "error read image: " << infile << endl;
            return EXIT_FAILURE;
        }
    }
    else
        capture.open(camid);

    cout << "\nControls:\n"
         << "\to - save output image\n"
         << "\tm - switch OpenCL <-> CPU mode"
         << "\tESC - exit\n";

    for (;;)
    {
        if(capture.isOpened())
            capture.read(frame);
        else
            imread(infile).copyTo(frame);
        if(frame.empty())
        {
            waitKey();
            break;
        }

        cvtColor(frame, frame, COLOR_BGR2GRAY);
        pFilter->apply(frame, outframe);

        imshow("CLAHE", outframe);

        char key = (char)waitKey(3);
        if(key == 'o')
            imwrite(outfile, outframe);
        else if(key == 27)
            break;
        else if(key == 'm')
        {
            ocl::setUseOpenCL(!cv::ocl::useOpenCL());
            cout << "Switched to " << (ocl::useOpenCL() ? "OpenCL enabled" : "CPU") << " mode\n";
        }
    }
    return EXIT_SUCCESS;
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
- `opencv2/imgcodecs.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/core/utility.hpp`
- `opencv2/videoio.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/core/ocl.hpp`


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

