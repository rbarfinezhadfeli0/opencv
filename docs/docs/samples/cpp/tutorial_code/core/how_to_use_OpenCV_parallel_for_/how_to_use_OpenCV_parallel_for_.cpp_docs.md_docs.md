# Documentation for `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp_docs.md`
- **File Name**: `how_to_use_OpenCV_parallel_for_.cpp_docs.md`
- **File Size**: 7,634 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp_docs.md](../../../../../../docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp`
- **File Name**: `how_to_use_OpenCV_parallel_for_.cpp`
- **File Size**: 4,167 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp](../../../../../samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/imgcodecs.hpp>

#define PARALLEL_FOR_LAMBDA

using namespace std;
using namespace cv;

namespace
{
//! [mandelbrot-escape-time-algorithm]
int mandelbrot(const complex<float> &z0, const int max)
{
    complex<float> z = z0;
    for (int t = 0; t < max; t++)
    {
        if (z.real()*z.real() + z.imag()*z.imag() > 4.0f) return t;
        z = z*z + z0;
    }

    return max;
}
//! [mandelbrot-escape-time-algorithm]

//! [mandelbrot-grayscale-value]
int mandelbrotFormula(const complex<float> &z0, const int maxIter=500) {
    int value = mandelbrot(z0, maxIter);
    if(maxIter - value == 0)
    {
        return 0;
    }

    return cvRound(sqrt(value / (float) maxIter) * 255);
}
//! [mandelbrot-grayscale-value]

#ifndef PARALLEL_FOR_LAMBDA

//! [mandelbrot-parallel]
class ParallelMandelbrot : public ParallelLoopBody
{
public:
    ParallelMandelbrot (Mat &img, const float x1, const float y1, const float scaleX, const float scaleY)
        : m_img(img), m_x1(x1), m_y1(y1), m_scaleX(scaleX), m_scaleY(scaleY)
    {
    }

    virtual void operator ()(const Range& range) const CV_OVERRIDE
    {
        for (int r = range.start; r < range.end; r++)
        {
            int i = r / m_img.cols;
            int j = r % m_img.cols;

            float x0 = j / m_scaleX + m_x1;
            float y0 = i / m_scaleY + m_y1;

            complex<float> z0(x0, y0);
            uchar value = (uchar) mandelbrotFormula(z0);
            m_img.ptr<uchar>(i)[j] = value;
        }
    }

    ParallelMandelbrot& operator=(const ParallelMandelbrot &) {
        return *this;
    };

private:
    Mat &m_img;
    float m_x1;
    float m_y1;
    float m_scaleX;
    float m_scaleY;
};
//! [mandelbrot-parallel]

#endif // !PARALLEL_FOR_LAMBDA

//! [mandelbrot-sequential]
void sequentialMandelbrot(Mat &img, const float x1, const float y1, const float scaleX, const float scaleY)
{
    for (int i = 0; i < img.rows; i++)
    {
        for (int j = 0; j < img.cols; j++)
        {
            float x0 = j / scaleX + x1;
            float y0 = i / scaleY + y1;

            complex<float> z0(x0, y0);
            uchar value = (uchar) mandelbrotFormula(z0);
            img.ptr<uchar>(i)[j] = value;
        }
    }
}
//! [mandelbrot-sequential]
}

int main()
{
    //! [mandelbrot-transformation]
    Mat mandelbrotImg(4800, 5400, CV_8U);
    float x1 = -2.1f, x2 = 0.6f;
    float y1 = -1.2f, y2 = 1.2f;
    float scaleX = mandelbrotImg.cols / (x2 - x1);
    float scaleY = mandelbrotImg.rows / (y2 - y1);
    //! [mandelbrot-transformation]

    double t1 = (double) getTickCount();

#ifdef PARALLEL_FOR_LAMBDA

    //! [mandelbrot-parallel-call-cxx11]
    parallel_for_(Range(0, mandelbrotImg.rows*mandelbrotImg.cols), [&](const Range& range){
        for (int r = range.start; r < range.end; r++)
        {
            int i = r / mandelbrotImg.cols;
            int j = r % mandelbrotImg.cols;

            float x0 = j / scaleX + x1;
            float y0 = i / scaleY + y1;

            complex<float> z0(x0, y0);
            uchar value = (uchar) mandelbrotFormula(z0);
            mandelbrotImg.ptr<uchar>(i)[j] = value;
        }
    });
    //! [mandelbrot-parallel-call-cxx11]

#else // PARALLEL_FOR_LAMBDA

    //! [mandelbrot-parallel-call]
    ParallelMandelbrot parallelMandelbrot(mandelbrotImg, x1, y1, scaleX, scaleY);
    parallel_for_(Range(0, mandelbrotImg.rows*mandelbrotImg.cols), parallelMandelbrot);
    //! [mandelbrot-parallel-call]

#endif // PARALLEL_FOR_LAMBDA

    t1 = ((double) getTickCount() - t1) / getTickFrequency();
    cout << "Parallel Mandelbrot: " << t1 << " s" << endl;

    Mat mandelbrotImgSequential(4800, 5400, CV_8U);
    double t2 = (double) getTickCount();
    sequentialMandelbrot(mandelbrotImgSequential, x1, y1, scaleX, scaleY);
    t2 = ((double) getTickCount() - t2) / getTickFrequency();
    cout << "Sequential Mandelbrot: " << t2 << " s" << endl;
    cout << "Speed-up: " << t2/t1 << " X" << endl;

    imwrite("Mandelbrot_parallel.png", mandelbrotImg);
    imwrite("Mandelbrot_sequential.png", mandelbrotImgSequential);

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

### Classes and Structures

- **ParallelMandelbrot**: A class/struct defined in this file

### Functions and Methods

- **PARALLEL_FOR_LAMBDA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `iostream`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

