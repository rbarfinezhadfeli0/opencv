# Documentation for `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp_docs.md`
- **File Name**: `how_to_use_OpenCV_parallel_for_new.cpp_docs.md`
- **File Size**: 13,133 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp_docs.md](../../../../../../docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp`
- **File Name**: `how_to_use_OpenCV_parallel_for_new.cpp`
- **File Size**: 9,479 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp](../../../../../samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_/how_to_use_OpenCV_parallel_for_new.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/core/how_to_use_OpenCV_parallel_for_` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/imgcodecs.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>

#define PARALLEL_FOR_LAMBDA

using namespace std;
using namespace cv;

namespace
{
//! [convolution-sequential]
void conv_seq(Mat src, Mat &dst, Mat kernel)
{
    //![convolution-make-borders]
    int rows = src.rows, cols = src.cols;
    dst = Mat(rows, cols, src.type());

    // Taking care of edge values
    // Make border = kernel.rows / 2;

    int sz = kernel.rows / 2;
    copyMakeBorder(src, src, sz, sz, sz, sz, BORDER_REPLICATE);
    //![convolution-make-borders]

    //! [convolution-kernel-loop]
    for (int i = 0; i < rows; i++)
    {
        uchar *dptr = dst.ptr(i);
        for (int j = 0; j < cols; j++)
        {
            double value = 0;

            for (int k = -sz; k <= sz; k++)
            {
                // slightly faster results when we create a ptr due to more efficient memory access.
                uchar *sptr = src.ptr(i + sz + k);
                for (int l = -sz; l <= sz; l++)
                {
                    value += kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                }
            }
            dptr[j] = saturate_cast<uchar>(value);
        }
    }
    //! [convolution-kernel-loop]
}
//! [convolution-sequential]

#ifdef PARALLEL_FOR_LAMBDA

void conv_parallel(Mat src, Mat &dst, Mat kernel)
{
    int rows = src.rows, cols = src.cols;

    dst = Mat(rows, cols, CV_8UC1, Scalar(0));

    // Taking care of edge values
    // Make border = kernel.rows / 2;

    int sz = kernel.rows / 2;
    copyMakeBorder(src, src, sz, sz, sz, sz, BORDER_REPLICATE);

    //! [convolution-parallel-cxx11]
    parallel_for_(Range(0, rows * cols), [&](const Range &range)
                    {
                        for (int r = range.start; r < range.end; r++)
                        {
                            int i = r / cols, j = r % cols;

                            double value = 0;
                            for (int k = -sz; k <= sz; k++)
                            {
                                uchar *sptr = src.ptr(i + sz + k);
                                for (int l = -sz; l <= sz; l++)
                                {
                                    value += kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                                }
                            }
                            dst.ptr(i)[j] = saturate_cast<uchar>(value);
                        }
                    });
    //! [convolution-parallel-cxx11]
}

void conv_parallel_row_split(Mat src, Mat &dst, Mat kernel)
{
    int rows = src.rows, cols = src.cols;

    dst = Mat(rows, cols, CV_8UC1, Scalar(0));

    // Taking care of edge values
    // Make border = kernel.rows / 2;

    int sz = kernel.rows / 2;
    copyMakeBorder(src, src, sz, sz, sz, sz, BORDER_REPLICATE);

    //! [convolution-parallel-cxx11-row-split]
    parallel_for_(Range(0, rows), [&](const Range &range)
                    {
                        for (int i = range.start; i < range.end; i++)
                        {

                            uchar *dptr = dst.ptr(i);
                            for (int j = 0; j < cols; j++)
                            {
                                double value = 0;
                                for (int k = -sz; k <= sz; k++)
                                {
                                    uchar *sptr = src.ptr(i + sz + k);
                                    for (int l = -sz; l <= sz; l++)
                                    {
                                        value += kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                                    }
                                }
                                dptr[j] = saturate_cast<uchar>(value);
                            }
                        }
                    });
    //! [convolution-parallel-cxx11-row-split]
}

#else // PARALLEL_FOR_LAMBDA

//! [convolution-parallel]
class parallelConvolution : public ParallelLoopBody
{
private:
    Mat m_src, &m_dst;
    Mat m_kernel;
    int sz;

public:
    parallelConvolution(Mat src, Mat &dst, Mat kernel)
        : m_src(src), m_dst(dst), m_kernel(kernel)
    {
        sz = kernel.rows / 2;
    }

    //! [overload-full]
    virtual void operator()(const Range &range) const CV_OVERRIDE
    {
        for (int r = range.start; r < range.end; r++)
        {
            int i = r / m_src.cols, j = r % m_src.cols;

            double value = 0;
            for (int k = -sz; k <= sz; k++)
            {
                uchar *sptr = m_src.ptr(i + sz + k);
                for (int l = -sz; l <= sz; l++)
                {
                    value += m_kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                }
            }
            m_dst.ptr(i)[j] = saturate_cast<uchar>(value);
        }
    }
    //! [overload-full]
};
//! [convolution-parallel]

void conv_parallel(Mat src, Mat &dst, Mat kernel)
{
    int rows = src.rows, cols = src.cols;

    dst = Mat(rows, cols, CV_8UC1, Scalar(0));

    // Taking care of edge values
    // Make border = kernel.rows / 2;

    int sz = kernel.rows / 2;
    copyMakeBorder(src, src, sz, sz, sz, sz, BORDER_REPLICATE);

    //! [convolution-parallel-function]
    parallelConvolution obj(src, dst, kernel);
    parallel_for_(Range(0, rows * cols), obj);
    //! [convolution-parallel-function]
}

//! [conv-parallel-row-split]
class parallelConvolutionRowSplit : public ParallelLoopBody
{
private:
    Mat m_src, &m_dst;
    Mat m_kernel;
    int sz;

public:
    parallelConvolutionRowSplit(Mat src, Mat &dst, Mat kernel)
        : m_src(src), m_dst(dst), m_kernel(kernel)
    {
        sz = kernel.rows / 2;
    }

    //! [overload-row-split]
    virtual void operator()(const Range &range) const CV_OVERRIDE
    {
        for (int i = range.start; i < range.end; i++)
        {

            uchar *dptr = dst.ptr(i);
            for (int j = 0; j < cols; j++)
            {
                double value = 0;
                for (int k = -sz; k <= sz; k++)
                {
                    uchar *sptr = src.ptr(i + sz + k);
                    for (int l = -sz; l <= sz; l++)
                    {
                        value += kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                    }
                }
                dptr[j] = saturate_cast<uchar>(value);
            }
        }
    }
    //! [overload-row-split]
};
//! [conv-parallel-row-split]

void conv_parallel_row_split(Mat src, Mat &dst, Mat kernel)
{
    int rows = src.rows, cols = src.cols;

    dst = Mat(rows, cols, CV_8UC1, Scalar(0));

    // Taking care of edge values
    // Make border = kernel.rows / 2;

    int sz = kernel.rows / 2;
    copyMakeBorder(src, src, sz, sz, sz, sz, BORDER_REPLICATE);

    //! [convolution-parallel-function-row]
    parallelConvolutionRowSplit obj(src, dst, kernel);
    parallel_for_(Range(0, rows), obj);
    //! [convolution-parallel-function-row]
}

#endif // PARALLEL_FOR_LAMBDA

static void help(char *progName)
{
    cout << endl
            << " This program shows how to use the OpenCV parallel_for_ function and \n"
            << " compares the performance of the sequential and parallel implementations for a \n"
            << " convolution operation\n"
            << " Usage:\n "
            << progName << " [image_path -- default lena.jpg] " << endl
            << endl;
}
}

int main(int argc, char *argv[])
{

    help(argv[0]);
    const char *filepath = argc >= 2 ? argv[1] : "../../../../data/lena.jpg";

    Mat src, dst, kernel;
    src = imread(filepath, IMREAD_GRAYSCALE);

    if (src.empty())
    {
        cerr << "Can't open [" << filepath << "]" << endl;
        return EXIT_FAILURE;
    }
    namedWindow("Input", 1);
    namedWindow("Output1", 1);
    namedWindow("Output2", 1);
    namedWindow("Output3", 1);
    imshow("Input", src);

    kernel = (Mat_<double>(3, 3) << 1, 0, -1,
              1, 0, -1,
              1, 0, -1);

    /*
    Uncomment the kernels you want to use or write your own kernels to test out
    performance.
    */

    /*
    kernel = (Mat_<double>(5, 5) <<   1, 1, 1, 1, 1,
                                      1, 1, 1, 1, 1,
                                      1, 1, 1, 1, 1,
                                      1, 1, 1, 1, 1,
                                      1, 1, 1, 1, 1);
    kernel /= 100;
    */

    /*
    kernel = (Mat_<double>(3, 3) <<  1,  1,  1,
                                     0,  0,  0,
                                    -1, -1, -1);

    */

    double t = (double)getTickCount();

    conv_seq(src, dst, kernel);

    t = ((double)getTickCount() - t) / getTickFrequency();
    cout << " Sequential implementation: " << t << "s" << endl;

    imshow("Output1", dst);
    waitKey(0);

    t = (double)getTickCount();

    conv_parallel(src, dst, kernel);

    t = ((double)getTickCount() - t) / getTickFrequency();
    cout << " Parallel Implementation: " << t << "s" << endl;

    imshow("Output2", dst);
    waitKey(0);

    t = (double)getTickCount();

    conv_parallel_row_split(src, dst, kernel);

    t = ((double)getTickCount() - t) / getTickFrequency();
    cout << " Parallel Implementation(Row Split): " << t << "s" << endl
         << endl;

    imshow("Output3", dst);
    waitKey(0);

    // imwrite("src.png", src);
    // imwrite("dst.png", dst);

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

### Classes and Structures

- **parallelConvolutionRowSplit**: A class/struct defined in this file
- **parallelConvolution**: A class/struct defined in this file

### Functions and Methods

- **and()**: A function/method defined in this file
- **PARALLEL_FOR_LAMBDA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/imgcodecs.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/core.hpp`
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

