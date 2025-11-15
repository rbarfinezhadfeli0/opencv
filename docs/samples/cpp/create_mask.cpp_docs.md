# Documentation for `samples/cpp/create_mask.cpp`

## File Metadata

- **Full Path**: `samples/cpp/create_mask.cpp`
- **File Name**: `create_mask.cpp`
- **File Size**: 2,716 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/create_mask.cpp](../../samples/cpp/create_mask.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
* create_mask.cpp
*
* Author:
* Siddharth Kherada <siddharthkherada27[at]gmail[dot]com>
*
* This tutorial demonstrates how to make mask image (black and white).
* The program takes as input a source image and outputs its corresponding
* mask image.
*/

#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include <iostream>

using namespace std;
using namespace cv;

Mat src, img1, mask, final;

Point point;
vector<Point> pts;
int drag = 0;
int var = 0;
int flag = 0;

void mouseHandler(int, int, int, int, void*);

void mouseHandler(int event, int x, int y, int, void*)
{

    if (event == EVENT_LBUTTONDOWN && !drag)
    {
        if (flag == 0)
        {
            if (var == 0)
                img1 = src.clone();
            point = Point(x, y);
            circle(img1, point, 2, Scalar(0, 0, 255), -1, 8, 0);
            pts.push_back(point);
            var++;
            drag  = 1;

            if (var > 1)
                line(img1,pts[var-2], point, Scalar(0, 0, 255), 2, 8, 0);

            imshow("Source", img1);
        }
    }

    if (event == EVENT_LBUTTONUP && drag)
    {
        imshow("Source", img1);
        drag = 0;
    }

    if (event == EVENT_RBUTTONDOWN)
    {
        flag = 1;
        img1 = src.clone();

        if (var != 0)
        {
            polylines( img1, pts, 1, Scalar(0,0,0), 2, 8, 0);
        }

        imshow("Source", img1);
    }

    if (event == EVENT_RBUTTONUP)
    {
        flag = var;
        final = Mat::zeros(src.size(), CV_8UC3);
        mask = Mat::zeros(src.size(), CV_8UC1);

        fillPoly(mask, pts, Scalar(255, 255, 255), 8, 0);
        bitwise_and(src, src, final, mask);
        imshow("Mask", mask);
        imshow("Result", final);
        imshow("Source", img1);
    }

    if (event == EVENT_MBUTTONDOWN)
    {
        pts.clear();
        var = 0;
        drag = 0;
        flag = 0;
        imshow("Source", src);
    }
}

int main(int argc, char **argv)
{
    CommandLineParser parser(argc, argv, "{@input | lena.jpg | input image}");
    parser.about("This program demonstrates using mouse events\n");
    parser.printMessage();
    cout << "\n\tleft mouse button - set a point to create mask shape\n"
        "\tright mouse button - create mask from points\n"
        "\tmiddle mouse button - reset\n";
    String input_image = parser.get<String>("@input");

    src = imread(samples::findFile(input_image));

    if (src.empty())
    {
        printf("Error opening image: %s\n", input_image.c_str());
        return 0;
    }

    namedWindow("Source", WINDOW_AUTOSIZE);
    setMouseCallback("Source", mouseHandler, NULL);
    imshow("Source", src);
    waitKey(0);

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
- `opencv2/imgcodecs.hpp`
- `opencv2/highgui.hpp`

**Python Imports:**
- `points`


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

