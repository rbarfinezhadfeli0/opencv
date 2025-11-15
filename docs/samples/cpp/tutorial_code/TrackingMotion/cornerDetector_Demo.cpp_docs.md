# Documentation for `samples/cpp/tutorial_code/TrackingMotion/cornerDetector_Demo.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/TrackingMotion/cornerDetector_Demo.cpp`
- **File Name**: `cornerDetector_Demo.cpp`
- **File Size**: 4,031 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/TrackingMotion/cornerDetector_Demo.cpp](../../../../samples/cpp/tutorial_code/TrackingMotion/cornerDetector_Demo.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/TrackingMotion` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/**
 * @function cornerDetector_Demo.cpp
 * @brief Demo code for detecting corners using OpenCV built-in functions
 * @author OpenCV team
 */

#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include <iostream>

using namespace cv;
using namespace std;

/// Global variables
Mat src, src_gray;
Mat myHarris_dst, myHarris_copy, Mc;
Mat myShiTomasi_dst, myShiTomasi_copy;

int myShiTomasi_qualityLevel = 50;
int myHarris_qualityLevel = 50;
int max_qualityLevel = 100;

double myHarris_minVal, myHarris_maxVal;
double myShiTomasi_minVal, myShiTomasi_maxVal;

RNG rng(12345);

const char* myHarris_window = "My Harris corner detector";
const char* myShiTomasi_window = "My Shi Tomasi corner detector";

/// Function headers
void myShiTomasi_function( int, void* );
void myHarris_function( int, void* );

/**
 * @function main
 */
int main( int argc, char** argv )
{
    /// Load source image and convert it to gray
    CommandLineParser parser( argc, argv, "{@input | building.jpg | input image}" );
    src = imread( samples::findFile( parser.get<String>( "@input" ) ) );
    if ( src.empty() )
    {
        cout << "Could not open or find the image!\n" << endl;
        cout << "Usage: " << argv[0] << " <Input image>" << endl;
        return -1;
    }
    cvtColor( src, src_gray, COLOR_BGR2GRAY );

    /// Set some parameters
    int blockSize = 3, apertureSize = 3;

    /// My Harris matrix -- Using cornerEigenValsAndVecs
    cornerEigenValsAndVecs( src_gray, myHarris_dst, blockSize, apertureSize );

    /* calculate Mc */
    Mc = Mat( src_gray.size(), CV_32FC1 );
    for( int i = 0; i < src_gray.rows; i++ )
    {
        for( int j = 0; j < src_gray.cols; j++ )
        {
            float lambda_1 = myHarris_dst.at<Vec6f>(i, j)[0];
            float lambda_2 = myHarris_dst.at<Vec6f>(i, j)[1];
            Mc.at<float>(i, j) = lambda_1*lambda_2 - 0.04f*((lambda_1 + lambda_2) * (lambda_1 + lambda_2));
        }
    }

    minMaxLoc( Mc, &myHarris_minVal, &myHarris_maxVal );

    /* Create Window and Trackbar */
    namedWindow( myHarris_window );
    createTrackbar( "Quality Level:", myHarris_window, &myHarris_qualityLevel, max_qualityLevel, myHarris_function );
    myHarris_function( 0, 0 );

    /// My Shi-Tomasi -- Using cornerMinEigenVal
    cornerMinEigenVal( src_gray, myShiTomasi_dst, blockSize, apertureSize );

    minMaxLoc( myShiTomasi_dst, &myShiTomasi_minVal, &myShiTomasi_maxVal );

    /* Create Window and Trackbar */
    namedWindow( myShiTomasi_window );
    createTrackbar( "Quality Level:", myShiTomasi_window, &myShiTomasi_qualityLevel, max_qualityLevel, myShiTomasi_function );
    myShiTomasi_function( 0, 0 );

    waitKey();
    return 0;
}

/**
 * @function myShiTomasi_function
 */
void myShiTomasi_function( int, void* )
{
    myShiTomasi_copy = src.clone();
    myShiTomasi_qualityLevel = MAX(myShiTomasi_qualityLevel, 1);

    for( int i = 0; i < src_gray.rows; i++ )
    {
        for( int j = 0; j < src_gray.cols; j++ )
        {
            if( myShiTomasi_dst.at<float>(i,j) > myShiTomasi_minVal + ( myShiTomasi_maxVal - myShiTomasi_minVal )*myShiTomasi_qualityLevel/max_qualityLevel )
            {
                circle( myShiTomasi_copy, Point(j,i), 4, Scalar( rng.uniform(0,256), rng.uniform(0,256), rng.uniform(0,256) ), FILLED );
            }
        }
    }
    imshow( myShiTomasi_window, myShiTomasi_copy );
}

/**
 * @function myHarris_function
 */
void myHarris_function( int, void* )
{
    myHarris_copy = src.clone();
    myHarris_qualityLevel = MAX(myHarris_qualityLevel, 1);

    for( int i = 0; i < src_gray.rows; i++ )
    {
        for( int j = 0; j < src_gray.cols; j++ )
        {
            if( Mc.at<float>(i,j) > myHarris_minVal + ( myHarris_maxVal - myHarris_minVal )*myHarris_qualityLevel/max_qualityLevel )
            {
                circle( myHarris_copy, Point(j,i), 4, Scalar( rng.uniform(0,256), rng.uniform(0,256), rng.uniform(0,256) ), FILLED );
            }
        }
    }
    imshow( myHarris_window, myHarris_copy );
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

- **cornerDetector_Demo()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **myHarris_function()**: A function/method defined in this file
- **myShiTomasi_function()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iostream`
- `opencv2/highgui.hpp`
- `opencv2/imgproc.hpp`


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

