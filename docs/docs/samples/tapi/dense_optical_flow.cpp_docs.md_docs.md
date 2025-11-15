# Documentation for `docs/samples/tapi/dense_optical_flow.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/tapi/dense_optical_flow.cpp_docs.md`
- **File Name**: `dense_optical_flow.cpp_docs.md`
- **File Size**: 7,917 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/tapi/dense_optical_flow.cpp_docs.md](../../../docs/samples/tapi/dense_optical_flow.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/tapi/dense_optical_flow.cpp`

## File Metadata

- **Full Path**: `samples/tapi/dense_optical_flow.cpp`
- **File Name**: `dense_optical_flow.cpp`
- **File Size**: 4,797 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/tapi/dense_optical_flow.cpp](../../samples/tapi/dense_optical_flow.cpp)

## Purpose and Role

This file is located in the `samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include <iostream>
#include <iomanip>
#include <vector>

#include "opencv2/core/ocl.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"
#include "opencv2/video.hpp"

using namespace std;
using namespace cv;

static Mat getVisibleFlow(InputArray flow)
{
    vector<UMat> flow_vec;
    split(flow, flow_vec);
    UMat magnitude, angle;
    cartToPolar(flow_vec[0], flow_vec[1], magnitude, angle, true);
    magnitude.convertTo(magnitude, CV_32F, 0.2);
    vector<UMat> hsv_vec;
    hsv_vec.push_back(angle);
    hsv_vec.push_back(UMat::ones(angle.size(), angle.type()));
    hsv_vec.push_back(magnitude);
    UMat hsv;
    merge(hsv_vec, hsv);
    Mat img;
    cvtColor(hsv, img, COLOR_HSV2BGR);
    return img;
}

static Size fitSize(const Size & sz,  const Size & bounds)
{
    CV_Assert(!sz.empty());
    if (sz.width > bounds.width || sz.height > bounds.height)
    {
        double scale = std::min((double)bounds.width / sz.width, (double)bounds.height / sz.height);
        return Size(cvRound(sz.width * scale), cvRound(sz.height * scale));
    }
    return sz;
}

int main(int argc, const char* argv[])
{
    const char* keys =
            "{ h help     |     | print help message }"
            "{ c camera   | 0   | capture video from camera (device index starting from 0) }"
            "{ a algorithm | fb | algorithm (supported: 'fb', 'dis')}"
            "{ m cpu      |     | run without OpenCL }"
            "{ v video    |     | use video as input }"
            "{ o original |     | use original frame size (do not resize to 640x480)}"
            ;
    CommandLineParser parser(argc, argv, keys);
    parser.about("This sample demonstrates using of dense optical flow algorithms.");
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }
    int camera = parser.get<int>("camera");
    string algorithm = parser.get<string>("algorithm");
    bool useCPU = parser.has("cpu");
    string filename = parser.get<string>("video");
    bool useOriginalSize = parser.has("original");
    if (!parser.check())
    {
        parser.printErrors();
        return 1;
    }

    VideoCapture cap;
    if(filename.empty())
        cap.open(camera);
    else
        cap.open(filename);
    if (!cap.isOpened())
    {
        cout << "Can not open video stream: '" << (filename.empty() ? "<camera>" : filename) << "'" << endl;
        return 2;
    }

    Ptr<DenseOpticalFlow> alg;
    if (algorithm == "fb")
        alg = FarnebackOpticalFlow::create();
    else if (algorithm == "dis")
        alg = DISOpticalFlow::create(DISOpticalFlow::PRESET_FAST);
    else
    {
        cout << "Invalid algorithm: " << algorithm << endl;
        return 3;
    }

    ocl::setUseOpenCL(!useCPU);

    cout << "Press 'm' to toggle CPU/GPU processing mode" << endl;
    cout << "Press ESC or 'q' to exit" << endl;

    UMat prevFrame, frame, input_frame, flow;
    for(;;)
    {
        if (!cap.read(input_frame) || input_frame.empty())
        {
            cout << "Finished reading: empty frame" << endl;
            break;
        }
        Size small_size = fitSize(input_frame.size(), Size(640, 480));
        if (!useOriginalSize && small_size != input_frame.size())
            resize(input_frame, frame, small_size);
        else
            frame = input_frame;
        cvtColor(frame, frame, COLOR_BGR2GRAY);
        imshow("frame", frame);
        if (!prevFrame.empty())
        {
            int64 t = getTickCount();
            alg->calc(prevFrame, frame, flow);
            t = getTickCount() - t;
            {
                Mat img = getVisibleFlow(flow);
                ostringstream buf;
                buf << "Algo: " << algorithm << " | "
                    << "Mode: " << (useCPU ? "CPU" : "GPU") << " | "
                    << "FPS: " << fixed << setprecision(1) << (getTickFrequency() / (double)t);
                putText(img, buf.str(), Point(10, 30), FONT_HERSHEY_PLAIN, 2.0, Scalar(0, 0, 255), 2, LINE_AA);
                imshow("Dense optical flow field", img);
            }
        }
        frame.copyTo(prevFrame);

        // interact with user
        const char key = (char)waitKey(30);
        if (key == 27 || key == 'q') // ESC
        {
            cout << "Exit requested" << endl;
            break;
        }
        else if (key == 'm')
        {
            useCPU = !useCPU;
            ocl::setUseOpenCL(!useCPU);
            cout << "Set processing mode to: " << (useCPU ? "CPU" : "GPU") << endl;
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
- `opencv2/video.hpp`
- `iomanip`
- `vector`
- `opencv2/imgcodecs.hpp`
- `iostream`
- `opencv2/core/utility.hpp`
- `opencv2/videoio.hpp`
- `opencv2/highgui.hpp`
- `opencv2/core/ocl.hpp`

**Python Imports:**
- `camera`
- `0`


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

