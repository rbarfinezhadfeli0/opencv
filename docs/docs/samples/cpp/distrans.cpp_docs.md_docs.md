# Documentation for `docs/samples/cpp/distrans.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/distrans.cpp_docs.md`
- **File Name**: `distrans.cpp_docs.md`
- **File Size**: 8,293 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/distrans.cpp_docs.md](../../../docs/samples/cpp/distrans.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/distrans.cpp`

## File Metadata

- **Full Path**: `samples/cpp/distrans.cpp`
- **File Name**: `distrans.cpp`
- **File Size**: 5,253 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/distrans.cpp](../../samples/cpp/distrans.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/core/utility.hpp>
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"

#include <stdio.h>

using namespace std;
using namespace cv;

int maskSize0 = DIST_MASK_5;
int voronoiType = -1;
int edgeThresh = 100;
int distType0 = DIST_L1;

// The output and temporary images
Mat gray;

// threshold trackbar callback
static void onTrackbar( int, void* )
{
    static const Scalar colors[] =
    {
        Scalar(0,0,0),
        Scalar(255,0,0),
        Scalar(255,128,0),
        Scalar(255,255,0),
        Scalar(0,255,0),
        Scalar(0,128,255),
        Scalar(0,255,255),
        Scalar(0,0,255),
        Scalar(255,0,255)
    };

    int maskSize = voronoiType >= 0 ? DIST_MASK_5 : maskSize0;
    int distType = voronoiType >= 0 ? DIST_L2 : distType0;

    Mat edge = gray >= edgeThresh, dist, labels, dist8u;

    if( voronoiType < 0 )
        distanceTransform( edge, dist, distType, maskSize );
    else
        distanceTransform( edge, dist, labels, distType, maskSize, voronoiType );

    if( voronoiType < 0 )
    {
        // begin "painting" the distance transform result
        dist *= 5000;
        pow(dist, 0.5, dist);

        Mat dist32s, dist8u1, dist8u2;

        dist.convertTo(dist32s, CV_32S, 1, 0.5);
        dist32s &= Scalar::all(255);

        dist32s.convertTo(dist8u1, CV_8U, 1, 0);
        dist32s *= -1;

        dist32s += Scalar::all(255);
        dist32s.convertTo(dist8u2, CV_8U);

        Mat planes[] = {dist8u1, dist8u2, dist8u2};
        merge(planes, 3, dist8u);
    }
    else
    {
        dist8u.create(labels.size(), CV_8UC3);
        for( int i = 0; i < labels.rows; i++ )
        {
            const int* ll = (const int*)labels.ptr(i);
            const float* dd = (const float*)dist.ptr(i);
            uchar* d = (uchar*)dist8u.ptr(i);
            for( int j = 0; j < labels.cols; j++ )
            {
                int idx = ll[j] == 0 || dd[j] == 0 ? 0 : (ll[j]-1)%8 + 1;
                float scale = 1.f/(1 + dd[j]*dd[j]*0.0004f);
                int b = cvRound(colors[idx][0]*scale);
                int g = cvRound(colors[idx][1]*scale);
                int r = cvRound(colors[idx][2]*scale);
                d[j*3] = (uchar)b;
                d[j*3+1] = (uchar)g;
                d[j*3+2] = (uchar)r;
            }
        }
    }

    imshow("Distance Map", dist8u );
}

static void help(const char** argv)
{
    printf("\nProgram to demonstrate the use of the distance transform function between edge images.\n"
            "Usage:\n"
            "%s [image_name -- default image is stuff.jpg]\n"
            "\nHot keys: \n"
            "\tESC - quit the program\n"
            "\tC - use C/Inf metric\n"
            "\tL1 - use L1 metric\n"
            "\tL2 - use L2 metric\n"
            "\t3 - use 3x3 mask\n"
            "\t5 - use 5x5 mask\n"
            "\t0 - use precise distance transform\n"
            "\tv - switch to Voronoi diagram mode\n"
            "\tp - switch to pixel-based Voronoi diagram mode\n"
            "\tSPACE - loop through all the modes\n\n", argv[0]);
}

const char* keys =
{
    "{help h||}{@image |stuff.jpg|input image file}"
};

int main( int argc, const char** argv )
{
    CommandLineParser parser(argc, argv, keys);
    help(argv);
    if (parser.has("help"))
        return 0;
    string filename = parser.get<string>(0);
    gray = imread(samples::findFile(filename), 0);
    if(gray.empty())
    {
        printf("Cannot read image file: %s\n", filename.c_str());
        help(argv);
        return -1;
    }

    namedWindow("Distance Map", 1);
    createTrackbar("Brightness Threshold", "Distance Map", &edgeThresh, 255, onTrackbar, 0);

    for(;;)
    {
        // Call to update the view
        onTrackbar(0, 0);

        char c = (char)waitKey(0);

        if( c == 27 )
            break;

        if( c == 'c' || c == 'C' || c == '1' || c == '2' ||
            c == '3' || c == '5' || c == '0' )
            voronoiType = -1;

        if( c == 'c' || c == 'C' )
            distType0 = DIST_C;
        else if( c == '1' )
            distType0 = DIST_L1;
        else if( c == '2' )
            distType0 = DIST_L2;
        else if( c == '3' )
            maskSize0 = DIST_MASK_3;
        else if( c == '5' )
            maskSize0 = DIST_MASK_5;
        else if( c == '0' )
            maskSize0 = DIST_MASK_PRECISE;
        else if( c == 'v' )
            voronoiType = 0;
        else if( c == 'p' )
            voronoiType = 1;
        else if( c == ' ' )
        {
            if( voronoiType == 0 )
                voronoiType = 1;
            else if( voronoiType == 1 )
            {
                voronoiType = -1;
                maskSize0 = DIST_MASK_3;
                distType0 = DIST_C;
            }
            else if( distType0 == DIST_C )
                distType0 = DIST_L1;
            else if( distType0 == DIST_L1 )
                distType0 = DIST_L2;
            else if( maskSize0 == DIST_MASK_3 )
                maskSize0 = DIST_MASK_5;
            else if( maskSize0 == DIST_MASK_5 )
                maskSize0 = DIST_MASK_PRECISE;
            else if( maskSize0 == DIST_MASK_PRECISE )
                voronoiType = 0;
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

### Functions and Methods

- **between()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `stdio.h`
- `opencv2/imgcodecs.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/core/utility.hpp`
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

