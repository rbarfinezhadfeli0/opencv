# Documentation for `samples/cpp/videocapture_obsensor.cpp`

## File Metadata

- **Full Path**: `samples/cpp/videocapture_obsensor.cpp`
- **File Name**: `videocapture_obsensor.cpp`
- **File Size**: 5,594 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/videocapture_obsensor.cpp](../../samples/cpp/videocapture_obsensor.cpp)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/**
 * attention: Astra2 cameras currently only support Windows and Linux kernel versions no higher than 4.15, and higher versions of Linux kernel may have exceptions.
*/

#include <opencv2/videoio.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <iostream>

using namespace cv;
int main(int argc, char** argv)
{
    cv::CommandLineParser parser(argc, argv,
                                 "{help h ? |  | help message}"
                                 "{dw |  | depth width }"
                                 "{dh |  | depth height }"
                                 "{df |  | depth fps }"
                                 "{cw |  | color width }"
                                 "{ch |  | color height }"
                                 "{cf |  | depth fps }"

    );
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }

    std::vector<int> params;
    if (parser.has("dw"))
    {
        params.push_back(CAP_PROP_OBSENSOR_DEPTH_WIDTH);
        params.push_back(parser.get<int>("dw"));
    }

    if (parser.has("dh"))
    {
        params.push_back(CAP_PROP_OBSENSOR_DEPTH_HEIGHT);
        params.push_back(parser.get<int>("dh"));
    }

    if (parser.has("df"))
    {
        params.push_back(CAP_PROP_OBSENSOR_DEPTH_FPS);
        params.push_back(parser.get<int>("df"));
    }

    if (parser.has("cw"))
    {
        params.push_back(CAP_PROP_FRAME_WIDTH);
        params.push_back(parser.get<int>("cw"));
    }

    if (parser.has("ch"))
    {
        params.push_back(CAP_PROP_FRAME_HEIGHT);
        params.push_back(parser.get<int>("ch"));
    }

    if (parser.has("cf"))
    {
        params.push_back(CAP_PROP_FPS);
        params.push_back(parser.get<int>("cf"));
    }

    VideoCapture obsensorCapture;

    if (params.empty())
        obsensorCapture.open(0, CAP_OBSENSOR);
    else
        obsensorCapture.open(0, CAP_OBSENSOR, params);
    if(!obsensorCapture.isOpened()) {
        std::cerr << "Failed to open obsensor capture! Index out of range or no response from device";
        return -1;
    }

    double fx = obsensorCapture.get(CAP_PROP_OBSENSOR_INTRINSIC_FX);
    double fy = obsensorCapture.get(CAP_PROP_OBSENSOR_INTRINSIC_FY);
    double cx = obsensorCapture.get(CAP_PROP_OBSENSOR_INTRINSIC_CX);
    double cy = obsensorCapture.get(CAP_PROP_OBSENSOR_INTRINSIC_CY);

    double k1 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K1);
    double k2 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K2);
    double k3 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K3);
    double k4 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K4);
    double k5 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K5);
    double k6 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_K6);
    double p1 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_P1);
    double p2 = obsensorCapture.get(CAP_PROP_OBSENSOR_COLOR_DISTORTION_P1);

    std::cout << "obsensor camera intrinsic params: fx=" << fx << ", fy=" << fy << ", cx=" << cx << ", cy=" << cy << std::endl;
    std::cout << "obsensor camera distortion params: k,p=" << k1 << ", " << k2 << ", " << k3 << ", "
                                                           << k4 << ", " << k5 << ", " << k6 << ", "
                                                           << p1 << ", " << p2 << std::endl;

    Mat image;
    Mat depthMap;
    Mat adjDepthMap;

    // Minimum depth value
    const double minVal = 300;
    // Maximum depth value
    const double maxVal = 5000;
    while (true)
    {
        // Grab depth map like this:
        // obsensorCapture >> depthMap;

        // Another way to grab depth map (and bgr image).
        if (obsensorCapture.grab())
        {
            if (obsensorCapture.retrieve(image, CAP_OBSENSOR_BGR_IMAGE))
            {
                imshow("RGB", image);
            }

            if (obsensorCapture.retrieve(depthMap, CAP_OBSENSOR_DEPTH_MAP))
            {
                depthMap.convertTo(adjDepthMap, CV_8U, 255.0 / (maxVal - minVal), -minVal * 255.0 / (maxVal - minVal));
                applyColorMap(adjDepthMap, adjDepthMap, COLORMAP_JET);
                imshow("DEPTH", adjDepthMap);
            }

            // depth map overlay on bgr image
            static const float alpha = 0.6f;
            if (!image.empty() && !depthMap.empty())
            {
                depthMap.convertTo(adjDepthMap, CV_8U, 255.0 / (maxVal - minVal), -minVal * 255.0 / (maxVal - minVal));
                cv::resize(adjDepthMap, adjDepthMap, cv::Size(image.cols, image.rows));
                for (int i = 0; i < image.rows; i++)
                {
                    for (int j = 0; j < image.cols; j++)
                    {
                        cv::Vec3b& outRgb = image.at<cv::Vec3b>(i, j);
                        uint8_t depthValue = 255 - adjDepthMap.at<uint8_t>(i, j);
                        if (depthValue != 0 && depthValue != 255)
                        {
                            outRgb[0] = (uint8_t)(outRgb[0] * (1.0f - alpha) + depthValue * alpha);
                            outRgb[1] = (uint8_t)(outRgb[1] * (1.0f - alpha) + depthValue *  alpha);
                            outRgb[2] = (uint8_t)(outRgb[2] * (1.0f - alpha) + depthValue *  alpha);
                        }
                    }
                }
                imshow("DepthToColor", image);
            }
            image.release();
            depthMap.release();
        }

        if (pollKey() >= 0)
            break;
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
- `iostream`
- `opencv2/highgui.hpp`
- `opencv2/imgproc.hpp`
- `opencv2/videoio.hpp`

**Python Imports:**
- `device`


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

