# Documentation for `samples/cpp/npr_demo.cpp`

## File Metadata

- **Full Path**: `samples/cpp/npr_demo.cpp`
- **File Name**: `npr_demo.cpp`
- **File Size**: 2,313 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/npr_demo.cpp](../../samples/cpp/npr_demo.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
* npr_demo.cpp
*
* Author:
* Siddharth Kherada <siddharthkherada27[at]gmail[dot]com>
*
* This tutorial demonstrates how to use OpenCV Non-Photorealistic Rendering Module.
* 1) Edge Preserve Smoothing
*    -> Using Normalized convolution Filter
*    -> Using Recursive Filter
* 2) Detail Enhancement
* 3) Pencil sketch/Color Pencil Drawing
* 4) Stylization
*
*/

#include <signal.h>
#include "opencv2/photo.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/core.hpp"
#include <iostream>
#include <stdlib.h>

using namespace std;
using namespace cv;

int main(int argc, char* argv[])
{
    cv::CommandLineParser parser(argc, argv, "{help h||show help message}{@image|lena.jpg|input image}");
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }
    string filename = samples::findFile(parser.get<string>("@image"));

    Mat I = imread(filename);

    int num,type;

    if(I.empty())
    {
        cout <<  "Image not found" << endl;
        return 1;
    }

    cout << endl;
    cout << " Edge Preserve Filter" << endl;
    cout << "----------------------" << endl;

    cout << "Options: " << endl;
    cout << endl;

    cout << "1) Edge Preserve Smoothing" << endl;
    cout << "   -> Using Normalized convolution Filter" << endl;
    cout << "   -> Using Recursive Filter" << endl;
    cout << "2) Detail Enhancement" << endl;
    cout << "3) Pencil sketch/Color Pencil Drawing" << endl;
    cout << "4) Stylization" << endl;
    cout << endl;

    cout << "Press number 1-4 to choose from above techniques: ";

    cin >> num;

    Mat img;

    if(num == 1)
    {
        cout << endl;
        cout << "Press 1 for Normalized Convolution Filter and 2 for Recursive Filter: ";

        cin >> type;

        edgePreservingFilter(I,img,type);
        imshow("Edge Preserve Smoothing",img);

    }
    else if(num == 2)
    {
        detailEnhance(I,img);
        imshow("Detail Enhanced",img);
    }
    else if(num == 3)
    {
        Mat img1;
        pencilSketch(I,img1, img, 10 , 0.1f, 0.03f);
        imshow("Pencil Sketch",img1);
        imshow("Color Pencil Sketch",img);
    }
    else if(num == 4)
    {
        stylization(I,img);
        imshow("Stylization",img);
    }
    waitKey(0);
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
- `stdlib.h`
- `opencv2/imgcodecs.hpp`
- `opencv2/photo.hpp`
- `iostream`
- `signal.h`
- `opencv2/imgproc.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`

**Python Imports:**
- `above`


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

