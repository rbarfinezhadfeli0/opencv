# Documentation for `docs/samples/gpu/cascadeclassifier.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/gpu/cascadeclassifier.cpp_docs.md`
- **File Name**: `cascadeclassifier.cpp_docs.md`
- **File Size**: 11,591 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/gpu/cascadeclassifier.cpp_docs.md](../../../docs/samples/gpu/cascadeclassifier.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/gpu/cascadeclassifier.cpp`

## File Metadata

- **Full Path**: `samples/gpu/cascadeclassifier.cpp`
- **File Name**: `cascadeclassifier.cpp`
- **File Size**: 8,520 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/gpu/cascadeclassifier.cpp](../../samples/gpu/cascadeclassifier.cpp)

## Purpose and Role

This file is located in the `samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// WARNING: this sample is under construction! Use it on your own risk.
#if defined _MSC_VER && _MSC_VER >= 1400
#pragma warning(disable : 4100)
#endif


#include <iostream>
#include <iomanip>
#include "opencv2/objdetect.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/cudaobjdetect.hpp"
#include "opencv2/cudaimgproc.hpp"
#include "opencv2/cudawarping.hpp"

using namespace std;
using namespace cv;
using namespace cv::cuda;


static void help()
{
    cout << "Usage: ./cascadeclassifier \n\t--cascade <cascade_file>\n\t(<image>|--video <video>|--camera <camera_id>)\n"
            "Using OpenCV version " << CV_VERSION << endl << endl;
}


static void convertAndResize(const Mat& src, Mat& gray, Mat& resized, double scale)
{
    if (src.channels() == 3)
    {
        cv::cvtColor( src, gray, COLOR_BGR2GRAY );
    }
    else
    {
        gray = src;
    }

    Size sz(cvRound(gray.cols * scale), cvRound(gray.rows * scale));

    if (scale != 1)
    {
        cv::resize(gray, resized, sz);
    }
    else
    {
        resized = gray;
    }
}

static void convertAndResize(const GpuMat& src, GpuMat& gray, GpuMat& resized, double scale)
{
    if (src.channels() == 3)
    {
        cv::cuda::cvtColor( src, gray, COLOR_BGR2GRAY );
    }
    else
    {
        gray = src;
    }

    Size sz(cvRound(gray.cols * scale), cvRound(gray.rows * scale));

    if (scale != 1)
    {
        cv::cuda::resize(gray, resized, sz);
    }
    else
    {
        resized = gray;
    }
}


static void matPrint(Mat &img, int lineOffsY, Scalar fontColor, const string &ss)
{
    int fontFace = FONT_HERSHEY_DUPLEX;
    double fontScale = 0.8;
    int fontThickness = 2;
    Size fontSize = cv::getTextSize("T[]", fontFace, fontScale, fontThickness, 0);

    Point org;
    org.x = 1;
    org.y = 3 * fontSize.height * (lineOffsY + 1) / 2;
    putText(img, ss, org, fontFace, fontScale, Scalar(0,0,0), 5*fontThickness/2, 16);
    putText(img, ss, org, fontFace, fontScale, fontColor, fontThickness, 16);
}


static void displayState(Mat &canvas, bool bHelp, bool bGpu, bool bLargestFace, bool bFilter, double fps)
{
    Scalar fontColorRed = Scalar(255,0,0);
    Scalar fontColorNV  = Scalar(118,185,0);

    ostringstream ss;
    ss << "FPS = " << setprecision(1) << fixed << fps;
    matPrint(canvas, 0, fontColorRed, ss.str());
    ss.str("");
    ss << "[" << canvas.cols << "x" << canvas.rows << "], " <<
        (bGpu ? "GPU, " : "CPU, ") <<
        (bLargestFace ? "OneFace, " : "MultiFace, ") <<
        (bFilter ? "Filter:ON" : "Filter:OFF");
    matPrint(canvas, 1, fontColorRed, ss.str());

    // by Anatoly. MacOS fix. ostringstream(const string&) is a private
    // matPrint(canvas, 2, fontColorNV, ostringstream("Space - switch GPU / CPU"));
    if (bHelp)
    {
        matPrint(canvas, 2, fontColorNV, "Space - switch GPU / CPU");
        matPrint(canvas, 3, fontColorNV, "M - switch OneFace / MultiFace");
        matPrint(canvas, 4, fontColorNV, "F - toggle rectangles Filter");
        matPrint(canvas, 5, fontColorNV, "H - toggle hotkeys help");
        matPrint(canvas, 6, fontColorNV, "1/Q - increase/decrease scale");
    }
    else
    {
        matPrint(canvas, 2, fontColorNV, "H - toggle hotkeys help");
    }
}


int main(int argc, const char *argv[])
{
    if (argc == 1)
    {
        help();
        return -1;
    }

    if (getCudaEnabledDeviceCount() == 0)
    {
        return cerr << "No GPU found or the library is compiled without CUDA support" << endl, -1;
    }

    cv::cuda::printShortCudaDeviceInfo(cv::cuda::getDevice());

    string cascadeName;
    string inputName;
    bool isInputImage = false;
    bool isInputVideo = false;
    bool isInputCamera = false;

    for (int i = 1; i < argc; ++i)
    {
        if (string(argv[i]) == "--cascade")
            cascadeName = argv[++i];
        else if (string(argv[i]) == "--video")
        {
            inputName = argv[++i];
            isInputVideo = true;
        }
        else if (string(argv[i]) == "--camera")
        {
            inputName = argv[++i];
            isInputCamera = true;
        }
        else if (string(argv[i]) == "--help")
        {
            help();
            return -1;
        }
        else if (!isInputImage)
        {
            inputName = argv[i];
            isInputImage = true;
        }
        else
        {
            cout << "Unknown key: " << argv[i] << endl;
            return -1;
        }
    }

    Ptr<cuda::CascadeClassifier> cascade_gpu = cuda::CascadeClassifier::create(cascadeName);

    cv::CascadeClassifier cascade_cpu;
    if (!cascade_cpu.load(cascadeName))
    {
        return cerr << "ERROR: Could not load cascade classifier \"" << cascadeName << "\"" << endl, help(), -1;
    }

    VideoCapture capture;
    Mat image;

    if (isInputImage)
    {
        image = imread(inputName);
        CV_Assert(!image.empty());
    }
    else if (isInputVideo)
    {
        capture.open(inputName);
        CV_Assert(capture.isOpened());
    }
    else
    {
        capture.open(atoi(inputName.c_str()));
        CV_Assert(capture.isOpened());
    }

    namedWindow("result", 1);

    Mat frame, frame_cpu, gray_cpu, resized_cpu, frameDisp;
    vector<Rect> faces;

    GpuMat frame_gpu, gray_gpu, resized_gpu, facesBuf_gpu;

    /* parameters */
    bool useGPU = true;
    double scaleFactor = 1.0;
    bool findLargestObject = false;
    bool filterRects = true;
    bool helpScreen = false;

    for (;;)
    {
        if (isInputCamera || isInputVideo)
        {
            capture >> frame;
            if (frame.empty())
            {
                break;
            }
        }

        (image.empty() ? frame : image).copyTo(frame_cpu);
        frame_gpu.upload(image.empty() ? frame : image);

        convertAndResize(frame_gpu, gray_gpu, resized_gpu, scaleFactor);
        convertAndResize(frame_cpu, gray_cpu, resized_cpu, scaleFactor);

        TickMeter tm;
        tm.start();

        if (useGPU)
        {
            cascade_gpu->setFindLargestObject(findLargestObject);
            cascade_gpu->setScaleFactor(1.2);
            cascade_gpu->setMinNeighbors((filterRects || findLargestObject) ? 4 : 0);

            cascade_gpu->detectMultiScale(resized_gpu, facesBuf_gpu);
            cascade_gpu->convert(facesBuf_gpu, faces);
        }
        else
        {
            Size minSize = cascade_gpu->getClassifierSize();
            cascade_cpu.detectMultiScale(resized_cpu, faces, 1.2,
                                         (filterRects || findLargestObject) ? 4 : 0,
                                         (findLargestObject ? CASCADE_FIND_BIGGEST_OBJECT : 0)
                                            | CASCADE_SCALE_IMAGE,
                                         minSize);
        }

        for (size_t i = 0; i < faces.size(); ++i)
        {
            rectangle(resized_cpu, faces[i], Scalar(255));
        }

        tm.stop();
        double detectionTime = tm.getTimeMilli();
        double fps = 1000 / detectionTime;

        //print detections to console
        cout << setfill(' ') << setprecision(2);
        cout << setw(6) << fixed << fps << " FPS, " << faces.size() << " det";
        if ((filterRects || findLargestObject) && !faces.empty())
        {
            for (size_t i = 0; i < faces.size(); ++i)
            {
                cout << ", [" << setw(4) << faces[i].x
                     << ", " << setw(4) << faces[i].y
                     << ", " << setw(4) << faces[i].width
                     << ", " << setw(4) << faces[i].height << "]";
            }
        }
        cout << endl;

        cv::cvtColor(resized_cpu, frameDisp, COLOR_GRAY2BGR);
        displayState(frameDisp, helpScreen, useGPU, findLargestObject, filterRects, fps);
        imshow("result", frameDisp);

        char key = (char)waitKey(5);
        if (key == 27)
        {
            break;
        }

        switch (key)
        {
        case ' ':
            useGPU = !useGPU;
            break;
        case 'm':
        case 'M':
            findLargestObject = !findLargestObject;
            break;
        case 'f':
        case 'F':
            filterRects = !filterRects;
            break;
        case '1':
            scaleFactor *= 1.05;
            break;
        case 'q':
        case 'Q':
            scaleFactor /= 1.05;
            break;
        case 'h':
        case 'H':
            helpScreen = !helpScreen;
            break;
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iomanip`
- `opencv2/cudaimgproc.hpp`
- `opencv2/cudaobjdetect.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/highgui.hpp`
- `opencv2/objdetect.hpp`
- `opencv2/cudawarping.hpp`


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

