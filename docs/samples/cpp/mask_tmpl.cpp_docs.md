# Documentation for `samples/cpp/mask_tmpl.cpp`

## File Metadata

- **Full Path**: `samples/cpp/mask_tmpl.cpp`
- **File Name**: `mask_tmpl.cpp`
- **File Size**: 2,193 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/mask_tmpl.cpp](../../samples/cpp/mask_tmpl.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
#include <iostream>

using namespace std;
using namespace cv;

int main( int argc, const char** argv )
{
    CommandLineParser parser(argc, argv,
        "{ i | lena_tmpl.jpg |image name }"
        "{ t | tmpl.png |template name }"
        "{ m | mask.png |mask name }"
        "{ cm| 3 |comparison method }");

    cout << "This program demonstrates the use of template matching with mask." << endl
         << endl
         << "Available methods: https://docs.opencv.org/4.x/df/dfb/group__imgproc__object.html#ga3a7850640f1fe1f58fe91a2d7583695d" << endl
         << "    TM_SQDIFF = " << (int)TM_SQDIFF << endl
         << "    TM_SQDIFF_NORMED = " << (int)TM_SQDIFF_NORMED << endl
         << "    TM_CCORR = " << (int)TM_CCORR << endl
         << "    TM_CCORR_NORMED = " << (int)TM_CCORR_NORMED << endl
         << "    TM_CCOEFF = " << (int)TM_CCOEFF << endl
         << "    TM_CCOEFF_NORMED = " << (int)TM_CCOEFF_NORMED << endl
         << endl;

    parser.printMessage();

    string filename = samples::findFile(parser.get<string>("i"));
    string tmplname = samples::findFile(parser.get<string>("t"));
    string maskname = samples::findFile(parser.get<string>("m"));
    Mat img = imread(filename);
    Mat tmpl = imread(tmplname);
    Mat mask = imread(maskname);
    Mat res;

    if(img.empty())
    {
        cout << "can not open " << filename << endl;
        return -1;
    }

    if(tmpl.empty())
    {
        cout << "can not open " << tmplname << endl;
        return -1;
    }

    if(mask.empty())
    {
        cout << "can not open " << maskname << endl;
        return -1;
    }

    int method = parser.get<int>("cm"); // default 3 (cv::TM_CCORR_NORMED)
    matchTemplate(img, tmpl, res, method, mask);

    double minVal, maxVal;
    Point minLoc, maxLoc;
    Rect rect;
    minMaxLoc(res, &minVal, &maxVal, &minLoc, &maxLoc);

    if(method == TM_SQDIFF || method == TM_SQDIFF_NORMED)
        rect = Rect(minLoc, tmpl.size());
    else
        rect = Rect(maxLoc, tmpl.size());

    rectangle(img, rect, Scalar(0, 255, 0), 2);

    imshow("detected template", img);
    waitKey();

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
- `iostream`
- `opencv2/imgproc.hpp`
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

