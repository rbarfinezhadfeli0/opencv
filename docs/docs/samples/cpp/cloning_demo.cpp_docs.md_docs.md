# Documentation for `docs/samples/cpp/cloning_demo.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/cloning_demo.cpp_docs.md`
- **File Name**: `cloning_demo.cpp_docs.md`
- **File Size**: 10,455 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/cloning_demo.cpp_docs.md](../../../docs/samples/cpp/cloning_demo.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/cloning_demo.cpp`

## File Metadata

- **Full Path**: `samples/cpp/cloning_demo.cpp`
- **File Name**: `cloning_demo.cpp`
- **File Size**: 7,416 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/cloning_demo.cpp](../../samples/cpp/cloning_demo.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
* cloning_demo.cpp
*
* Author:
* Siddharth Kherada <siddharthkherada27[at]gmail[dot]com>
*
* This tutorial demonstrates how to use OpenCV seamless cloning
* module without GUI.
*
* 1- Normal Cloning
* 2- Mixed Cloning
* 3- Monochrome Transfer
* 4- Color Change
* 5- Illumination change
* 6- Texture Flattening

* The program takes as input a source and a destination image (for 1-3 methods)
* and outputs the cloned image.
*
* Download test images from opencv_extra repository.
*/

#include "opencv2/photo.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/core.hpp"
#include <iostream>

using namespace std;
using namespace cv;

int main()
{
    cout << endl;
    cout << "Note: specify OPENCV_SAMPLES_DATA_PATH_HINT=<opencv_extra>/testdata/cv" << endl << endl;
    cout << "Cloning Module" << endl;
    cout << "---------------" << endl;
    cout << "Options: " << endl;
    cout << endl;
    cout << "1) Normal Cloning " << endl;
    cout << "2) Mixed Cloning " << endl;
    cout << "3) Monochrome Transfer " << endl;
    cout << "4) Local Color Change " << endl;
    cout << "5) Local Illumination Change " << endl;
    cout << "6) Texture Flattening " << endl;
    cout << endl;
    cout << "Press number 1-6 to choose from above techniques: ";
    int num = 1;
    cin >> num;
    cout << endl;

    if(num == 1)
    {
        string folder =  "cloning/Normal_Cloning/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "destination1.png");
        string original_path3 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat destination = imread(original_path2, IMREAD_COLOR);
        Mat mask = imread(original_path3, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(destination.empty())
        {
            cout << "Could not load destination image " << original_path2 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path3 << endl;
            exit(0);
        }

        Mat result;
        Point p;
        p.x = 400;
        p.y = 100;

        seamlessClone(source, destination, mask, p, result, 1);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else if(num == 2)
    {
        string folder = "cloning/Mixed_Cloning/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "destination1.png");
        string original_path3 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat destination = imread(original_path2, IMREAD_COLOR);
        Mat mask = imread(original_path3, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(destination.empty())
        {
            cout << "Could not load destination image " << original_path2 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path3 << endl;
            exit(0);
        }

        Mat result;
        Point p;
        p.x = destination.size().width/2;
        p.y = destination.size().height/2;

        seamlessClone(source, destination, mask, p, result, 2);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else if(num == 3)
    {
        string folder = "cloning/Monochrome_Transfer/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "destination1.png");
        string original_path3 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat destination = imread(original_path2, IMREAD_COLOR);
        Mat mask = imread(original_path3, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(destination.empty())
        {
            cout << "Could not load destination image " << original_path2 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path3 << endl;
            exit(0);
        }

        Mat result;
        Point p;
        p.x = destination.size().width/2;
        p.y = destination.size().height/2;

        seamlessClone(source, destination, mask, p, result, 3);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else if(num == 4)
    {
        string folder = "cloning/color_change/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat mask = imread(original_path2, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path2 << endl;
            exit(0);
        }

        Mat result;

        colorChange(source, mask, result, 1.5, .5, .5);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else if(num == 5)
    {
        string folder = "cloning/Illumination_Change/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat mask = imread(original_path2, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path2 << endl;
            exit(0);
        }

        Mat result;

        illuminationChange(source, mask, result, 0.2f, 0.4f);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else if(num == 6)
    {
        string folder = "cloning/Texture_Flattening/";
        string original_path1 = samples::findFile(folder + "source1.png");
        string original_path2 = samples::findFile(folder + "mask.png");

        Mat source = imread(original_path1, IMREAD_COLOR);
        Mat mask = imread(original_path2, IMREAD_COLOR);

        if(source.empty())
        {
            cout << "Could not load source image " << original_path1 << endl;
            exit(0);
        }
        if(mask.empty())
        {
            cout << "Could not load mask image " << original_path2 << endl;
            exit(0);
        }

        Mat result;

        textureFlattening(source, mask, result, 30, 45, 3);

        imshow("Output",result);
        imwrite("cloned.png", result);
    }
    else
    {
        cerr << "Invalid selection: " << num << endl;
        exit(1);
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
- `opencv2/imgcodecs.hpp`
- `opencv2/photo.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`

**Python Imports:**
- `above`
- `opencv_extra`


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

