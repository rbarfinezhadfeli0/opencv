# Documentation for `samples/tapi/bgfg_segm.cpp`

## File Metadata

- **Full Path**: `samples/tapi/bgfg_segm.cpp`
- **File Name**: `bgfg_segm.cpp`
- **File Size**: 3,022 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/tapi/bgfg_segm.cpp](../../samples/tapi/bgfg_segm.cpp)

## Purpose and Role

This file is located in the `samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <string>

#include "opencv2/core.hpp"
#include "opencv2/core/ocl.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/video.hpp"

using namespace std;
using namespace cv;

#define M_MOG2 2
#define M_KNN  3

int main(int argc, const char** argv)
{
    CommandLineParser cmd(argc, argv,
        "{ c camera   |                    | use camera }"
        "{ f file     | ../data/vtest.avi  | input video file }"
        "{ t type     | mog2               | method's type (knn, mog2) }"
        "{ h help     |                    | print help message }"
        "{ m cpu_mode | false              | press 'm' to switch OpenCL<->CPU}");

    if (cmd.has("help"))
    {
        cout << "Usage : bgfg_segm [options]" << endl;
        cout << "Available options:" << endl;
        cmd.printMessage();
        return EXIT_SUCCESS;
    }

    bool useCamera = cmd.has("camera");
    string file = cmd.get<string>("file");
    string method = cmd.get<string>("type");

    if (method != "mog" && method != "mog2")
    {
        cerr << "Incorrect method" << endl;
        return EXIT_FAILURE;
    }

    int m = method == "mog2" ? M_MOG2 : M_KNN;

    VideoCapture cap;
    if (useCamera)
        cap.open(0);
    else
        cap.open(file);

    if (!cap.isOpened())
    {
        cout << "can not open camera or video file" << endl;
        return EXIT_FAILURE;
    }

    UMat frame, fgmask, fgimg;
    cap >> frame;
    fgimg.create(frame.size(), frame.type());

    Ptr<BackgroundSubtractorKNN> knn = createBackgroundSubtractorKNN();
    Ptr<BackgroundSubtractorMOG2> mog2 = createBackgroundSubtractorMOG2();

    switch (m)
    {
    case M_KNN:
        knn->apply(frame, fgmask);
        break;

    case M_MOG2:
        mog2->apply(frame, fgmask);
        break;
    }
    bool running=true;
    for (;;)
    {
        if(!running)
            break;
        cap >> frame;
        if (frame.empty())
            break;

        int64 start = getTickCount();

        //update the model
        switch (m)
        {
        case M_KNN:
            knn->apply(frame, fgmask);
            break;

        case M_MOG2:
            mog2->apply(frame, fgmask);
            break;
        }

        double fps = getTickFrequency() / (getTickCount() - start);
        std::cout << "FPS : " << fps << std::endl;
        std::cout << fgimg.size() << std::endl;
        fgimg.setTo(Scalar::all(0));
        frame.copyTo(fgimg, fgmask);

        imshow("image", frame);
        imshow("foreground mask", fgmask);
        imshow("foreground image", fgimg);

        char key = (char)waitKey(30);

        switch (key)
        {
        case 27:
            running = false;
            break;
        case 'm':
        case 'M':
            ocl::setUseOpenCL(!ocl::useOpenCL());
            cout << "Switched to " << (ocl::useOpenCL() ? "OpenCL enabled" : "CPU") << " mode\n";
            break;
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
- `opencv2/video.hpp`
- `iostream`
- `string`
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

