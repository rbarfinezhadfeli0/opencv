# Documentation for `samples/cpp/tutorial_code/ImgProc/BasicLinearTransforms.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/ImgProc/BasicLinearTransforms.cpp`
- **File Name**: `BasicLinearTransforms.cpp`
- **File Size**: 2,409 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/ImgProc/BasicLinearTransforms.cpp](../../../../samples/cpp/tutorial_code/ImgProc/BasicLinearTransforms.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/ImgProc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/**
 * @file BasicLinearTransforms.cpp
 * @brief Simple program to change contrast and brightness
 * @author OpenCV team
 */

#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"
#include <iostream>

// we're NOT "using namespace std;" here, to avoid collisions between the beta variable and std::beta in c++17
using std::cin;
using std::cout;
using std::endl;
using namespace cv;

/**
 * @function main
 * @brief Main function
 */
int main( int argc, char** argv )
{
    /// Read image given by user
    //! [basic-linear-transform-load]
    CommandLineParser parser( argc, argv, "{@input | lena.jpg | input image}" );
    Mat image = imread( samples::findFile( parser.get<String>( "@input" ) ) );
    if( image.empty() )
    {
      cout << "Could not open or find the image!\n" << endl;
      cout << "Usage: " << argv[0] << " <Input image>" << endl;
      return -1;
    }
    //! [basic-linear-transform-load]

    //! [basic-linear-transform-output]
    Mat new_image = Mat::zeros( image.size(), image.type() );
    //! [basic-linear-transform-output]

    //! [basic-linear-transform-parameters]
    double alpha = 1.0; /*< Simple contrast control */
    int beta = 0;       /*< Simple brightness control */

    /// Initialize values
    cout << " Basic Linear Transforms " << endl;
    cout << "-------------------------" << endl;
    cout << "* Enter the alpha value [1.0-3.0]: "; cin >> alpha;
    cout << "* Enter the beta value [0-100]: ";    cin >> beta;
    //! [basic-linear-transform-parameters]

    /// Do the operation new_image(i,j) = alpha*image(i,j) + beta
    /// Instead of these 'for' loops we could have used simply:
    /// image.convertTo(new_image, -1, alpha, beta);
    /// but we wanted to show you how to access the pixels :)
    //! [basic-linear-transform-operation]
    for( int y = 0; y < image.rows; y++ ) {
        for( int x = 0; x < image.cols; x++ ) {
            for( int c = 0; c < image.channels(); c++ ) {
                new_image.at<Vec3b>(y,x)[c] =
                  saturate_cast<uchar>( alpha*image.at<Vec3b>(y,x)[c] + beta );
            }
        }
    }
    //! [basic-linear-transform-operation]

    //! [basic-linear-transform-display]
    /// Show stuff
    imshow("Original Image", image);
    imshow("New Image", new_image);

    /// Wait until the user press a key
    waitKey();
    //! [basic-linear-transform-display]
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

- **main()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iostream`
- `opencv2/highgui.hpp`
- `opencv2/imgcodecs.hpp`


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

