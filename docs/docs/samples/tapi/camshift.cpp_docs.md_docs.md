# Documentation for `docs/samples/tapi/camshift.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/tapi/camshift.cpp_docs.md`
- **File Name**: `camshift.cpp_docs.md`
- **File Size**: 10,565 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/tapi/camshift.cpp_docs.md](../../../docs/samples/tapi/camshift.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/tapi/camshift.cpp`

## File Metadata

- **Full Path**: `samples/tapi/camshift.cpp`
- **File Name**: `camshift.cpp`
- **File Size**: 7,488 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/tapi/camshift.cpp](../../samples/tapi/camshift.cpp)

## Purpose and Role

This file is located in the `samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "opencv2/core/utility.hpp"
#include "opencv2/core/ocl.hpp"
#include "opencv2/video/tracking.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"

#include <iostream>
#include <cctype>

static cv::UMat image;
static bool backprojMode = false;
static bool selectObject = false;
static int trackObject = 0;
static bool showHist = true;
static cv::Rect selection;
static int vmin = 10, vmax = 256, smin = 30;

static void onMouse(int event, int x, int y, int, void*)
{
    static cv::Point origin;

    if (selectObject)
    {
        selection.x = std::min(x, origin.x);
        selection.y = std::min(y, origin.y);
        selection.width = std::abs(x - origin.x);
        selection.height = std::abs(y - origin.y);

        selection &= cv::Rect(0, 0, image.cols, image.rows);
    }

    switch (event)
    {
    case cv::EVENT_LBUTTONDOWN:
        origin = cv::Point(x, y);
        selection = cv::Rect(x, y, 0, 0);
        selectObject = true;
        break;
    case cv::EVENT_LBUTTONUP:
        selectObject = false;
        if (selection.width > 0 && selection.height > 0)
            trackObject = -1;
        break;
    default:
        break;
    }
}

static void help()
{
    std::cout << "\nThis is a demo that shows mean-shift based tracking using Transparent API\n"
            "You select a color objects such as your face and it tracks it.\n"
            "This reads from video camera (0 by default, or the camera number the user enters\n"
            "Usage: \n"
            "   ./camshiftdemo [camera number]\n";

    std::cout << "\n\nHot keys: \n"
            "\tESC - quit the program\n"
            "\ts - stop the tracking\n"
            "\tb - switch to/from backprojection view\n"
            "\th - show/hide object histogram\n"
            "\tp - pause video\n"
            "\tc - use OpenCL or not\n"
            "To initialize tracking, select the object with mouse\n";
}

int main(int argc, const char ** argv)
{
    help();

    cv::VideoCapture cap;
    cv::Rect trackWindow;
    int hsize = 16;
    float hranges[2] = { 0, 180 };

    const char * const keys = { "{@camera_number| 0 | camera number}" };
    cv::CommandLineParser parser(argc, argv, keys);
    int camNum = parser.get<int>(0);

    cap.open(camNum);

    if (!cap.isOpened())
    {
        help();

        std::cout << "***Could not initialize capturing...***\n";
        std::cout << "Current parameter's value: \n";
        parser.printMessage();

        return EXIT_FAILURE;
    }

    cv::namedWindow("Histogram", cv::WINDOW_NORMAL);
    cv::namedWindow("CamShift Demo", cv::WINDOW_NORMAL);
    cv::setMouseCallback("CamShift Demo", onMouse);
    cv::createTrackbar("Vmin", "CamShift Demo", &vmin, 256);
    cv::createTrackbar("Vmax", "CamShift Demo", &vmax, 256);
    cv::createTrackbar("Smin", "CamShift Demo", &smin, 256);

    cv::Mat frame, histimg(200, 320, CV_8UC3, cv::Scalar::all(0));
    cv::UMat hsv, hist, hue, mask, backproj;
    bool paused = false;

    for ( ; ; )
    {
        if (!paused)
        {
            cap >> frame;
            if (frame.empty())
                break;
        }

        frame.copyTo(image);

        if (!paused)
        {
            cv::cvtColor(image, hsv, cv::COLOR_BGR2HSV);

            if (trackObject)
            {
                int _vmin = vmin, _vmax = vmax;

                cv::inRange(hsv, cv::Scalar(0, smin, std::min(_vmin, _vmax)),
                        cv::Scalar(180, 256, std::max(_vmin, _vmax)), mask);

                int fromTo[2] = { 0,0 };
                hue.create(hsv.size(), hsv.depth());
                cv::mixChannels(std::vector<cv::UMat>(1, hsv), std::vector<cv::UMat>(1, hue), fromTo, 1);

                if (trackObject < 0)
                {
                    cv::UMat roi(hue, selection), maskroi(mask, selection);
                    cv::calcHist(std::vector<cv::Mat>(1, roi.getMat(cv::ACCESS_READ)), std::vector<int>(1, 0),
                                 maskroi, hist, std::vector<int>(1, hsize), std::vector<float>(hranges, hranges + 2));
                    cv::normalize(hist, hist, 0, 255, cv::NORM_MINMAX);

                    trackWindow = selection;
                    trackObject = 1;

                    histimg = cv::Scalar::all(0);
                    int binW = histimg.cols / hsize;
                    cv::Mat buf (1, hsize, CV_8UC3);
                    for (int i = 0; i < hsize; i++)
                        buf.at<cv::Vec3b>(i) = cv::Vec3b(cv::saturate_cast<uchar>(i*180./hsize), 255, 255);
                    cv::cvtColor(buf, buf, cv::COLOR_HSV2BGR);

                    {
                        cv::Mat _hist = hist.getMat(cv::ACCESS_READ);
                        for (int i = 0; i < hsize; i++)
                        {
                            int val = cv::saturate_cast<int>(_hist.at<float>(i)*histimg.rows/255);
                            cv::rectangle(histimg, cv::Point(i*binW, histimg.rows),
                                       cv::Point((i+1)*binW, histimg.rows - val),
                                       cv::Scalar(buf.at<cv::Vec3b>(i)), -1, 8);
                        }
                    }
                }

                cv::calcBackProject(std::vector<cv::UMat>(1, hue), std::vector<int>(1, 0), hist, backproj,
                                    std::vector<float>(hranges, hranges + 2), 1.0);
                cv::bitwise_and(backproj, mask, backproj);

                cv::RotatedRect trackBox = cv::CamShift(backproj, trackWindow,
                                    cv::TermCriteria(cv::TermCriteria::EPS | cv::TermCriteria::COUNT, 10, 1));
                if (trackWindow.area() <= 1)
                {
                    int cols = backproj.cols, rows = backproj.rows, r = (std::min(cols, rows) + 5)/6;
                    trackWindow = cv::Rect(trackWindow.x - r, trackWindow.y - r,
                                       trackWindow.x + r, trackWindow.y + r) &
                                  cv::Rect(0, 0, cols, rows);
                }

                if (backprojMode)
                    cv::cvtColor(backproj, image, cv::COLOR_GRAY2BGR);

                {
                    cv::Mat _image = image.getMat(cv::ACCESS_RW);
                    cv::ellipse(_image, trackBox, cv::Scalar(0, 0, 255), 3, cv::LINE_AA);
                }
            }
        }
        else if (trackObject < 0)
            paused = false;

        if (selectObject && selection.width > 0 && selection.height > 0)
        {
            cv::UMat roi(image, selection);
            cv::bitwise_not(roi, roi);
        }

        cv::imshow("CamShift Demo", image);
        if (showHist)
            cv::imshow("Histogram", histimg);

        char c = (char)cv::waitKey(10);
        if (c == 27)
            break;

        switch(c)
        {
        case 'b':
            backprojMode = !backprojMode;
            break;
        case 't':
            trackObject = 0;
            histimg = cv::Scalar::all(0);
            break;
        case 'h':
            showHist = !showHist;
            if (!showHist)
                cv::destroyWindow("Histogram");
            else
                cv::namedWindow("Histogram", cv::WINDOW_AUTOSIZE);
            break;
        case 'p':
            paused = !paused;
            break;
        case 'c':
            cv::ocl::setUseOpenCL(!cv::ocl::useOpenCL());
        default:
            break;
        }
    }

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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cctype`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/video/tracking.hpp`
- `opencv2/core/utility.hpp`
- `opencv2/videoio.hpp`
- `opencv2/highgui.hpp`
- `opencv2/core/ocl.hpp`

**Python Imports:**
- `video`
- `backprojection`


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

