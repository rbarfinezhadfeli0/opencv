# Documentation for `docs/samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp_docs.md`
- **File Name**: `openni_orbbec_astra.cpp_docs.md`
- **File Size**: 9,247 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp_docs.md](../../../../../../docs/samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/videoio/openni_orbbec_astra` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp`
- **File Name**: `openni_orbbec_astra.cpp`
- **File Size**: 5,848 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp](../../../../../samples/cpp/tutorial_code/videoio/openni_orbbec_astra/openni_orbbec_astra.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/videoio/openni_orbbec_astra` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/videoio/videoio.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>

#include <list>
#include <iostream>


#if !defined(HAVE_THREADS)
int main()
{
    std::cout << "This sample is built without threading support. Sample code is disabled." << std::endl;
    return 0;
}
#else


#include <thread>
#include <mutex>
#include <condition_variable>
#include <atomic>

using namespace cv;
using std::cout;
using std::cerr;
using std::endl;


// Stores frames along with their timestamps
struct Frame
{
    int64 timestamp;
    Mat frame;
};

int main()
{
    //! [Open streams]
    // Open depth stream
    VideoCapture depthStream(CAP_OPENNI2_ASTRA);
    // Open color stream
    VideoCapture colorStream(0, CAP_V4L2);
    //! [Open streams]

    // Check that stream has opened
    if (!colorStream.isOpened())
    {
        cerr << "ERROR: Unable to open color stream" << endl;
        return 1;
    }

    // Check that stream has opened
    if (!depthStream.isOpened())
    {
        cerr << "ERROR: Unable to open depth stream" << endl;
        return 1;
    }

    //! [Setup streams]
    // Set color and depth stream parameters
    colorStream.set(CAP_PROP_FRAME_WIDTH,  640);
    colorStream.set(CAP_PROP_FRAME_HEIGHT, 480);
    depthStream.set(CAP_PROP_FRAME_WIDTH,  640);
    depthStream.set(CAP_PROP_FRAME_HEIGHT, 480);
    depthStream.set(CAP_PROP_OPENNI2_MIRROR, 0);
    //! [Setup streams]

    // Print color stream parameters
    cout << "Color stream: "
         << colorStream.get(CAP_PROP_FRAME_WIDTH) << "x" << colorStream.get(CAP_PROP_FRAME_HEIGHT)
         << " @" << colorStream.get(CAP_PROP_FPS) << " fps" << endl;

    //! [Get properties]
    // Print depth stream parameters
    cout << "Depth stream: "
         << depthStream.get(CAP_PROP_FRAME_WIDTH) << "x" << depthStream.get(CAP_PROP_FRAME_HEIGHT)
         << " @" << depthStream.get(CAP_PROP_FPS) << " fps" << endl;
    //! [Get properties]

    //! [Read streams]
    // Create two lists to store frames
    std::list<Frame> depthFrames, colorFrames;
    const std::size_t maxFrames = 64;

    // Synchronization objects
    std::mutex mtx;
    std::condition_variable dataReady;
    std::atomic<bool> isFinish;

    isFinish = false;

    // Start depth reading thread
    std::thread depthReader([&]
    {
        while (!isFinish)
        {
            // Grab and decode new frame
            if (depthStream.grab())
            {
                Frame f;
                f.timestamp = cv::getTickCount();
                depthStream.retrieve(f.frame, CAP_OPENNI_DEPTH_MAP);
                if (f.frame.empty())
                {
                    cerr << "ERROR: Failed to decode frame from depth stream" << endl;
                    break;
                }

                {
                    std::lock_guard<std::mutex> lk(mtx);
                    if (depthFrames.size() >= maxFrames)
                        depthFrames.pop_front();
                    depthFrames.push_back(f);
                }
                dataReady.notify_one();
            }
        }
    });

    // Start color reading thread
    std::thread colorReader([&]
    {
        while (!isFinish)
        {
            // Grab and decode new frame
            if (colorStream.grab())
            {
                Frame f;
                f.timestamp = cv::getTickCount();
                colorStream.retrieve(f.frame);
                if (f.frame.empty())
                {
                    cerr << "ERROR: Failed to decode frame from color stream" << endl;
                    break;
                }

                {
                    std::lock_guard<std::mutex> lk(mtx);
                    if (colorFrames.size() >= maxFrames)
                        colorFrames.pop_front();
                    colorFrames.push_back(f);
                }
                dataReady.notify_one();
            }
        }
    });
    //! [Read streams]

    //! [Pair frames]
    // Pair depth and color frames
    while (!isFinish)
    {
        std::unique_lock<std::mutex> lk(mtx);
        while (!isFinish && (depthFrames.empty() || colorFrames.empty()))
            dataReady.wait(lk);

        while (!depthFrames.empty() && !colorFrames.empty())
        {
            if (!lk.owns_lock())
                lk.lock();

            // Get a frame from the list
            Frame depthFrame = depthFrames.front();
            int64 depthT = depthFrame.timestamp;

            // Get a frame from the list
            Frame colorFrame = colorFrames.front();
            int64 colorT = colorFrame.timestamp;

            // Half of frame period is a maximum time diff between frames
            const int64 maxTdiff = int64(1000000000 / (2 * colorStream.get(CAP_PROP_FPS)));
            if (depthT + maxTdiff < colorT)
            {
                depthFrames.pop_front();
                continue;
            }
            else if (colorT + maxTdiff < depthT)
            {
                colorFrames.pop_front();
                continue;
            }
            depthFrames.pop_front();
            colorFrames.pop_front();
            lk.unlock();

            //! [Show frames]
            // Show depth frame
            Mat d8, dColor;
            depthFrame.frame.convertTo(d8, CV_8U, 255.0 / 2500);
            applyColorMap(d8, dColor, COLORMAP_OCEAN);
            imshow("Depth (colored)", dColor);

            // Show color frame
            imshow("Color", colorFrame.frame);
            //! [Show frames]

            // Exit on Esc key press
            int key = waitKey(1);
            if (key == 27) // ESC
            {
                isFinish = true;
                break;
            }
        }
    }
    //! [Pair frames]

    dataReady.notify_one();
    depthReader.join();
    colorReader.join();

    return 0;
}

#endif
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

- **Frame**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `list`
- `atomic`
- `iostream`
- `opencv2/imgproc.hpp`
- `thread`
- `opencv2/highgui.hpp`
- `condition_variable`
- `opencv2/videoio/videoio.hpp`
- `mutex`

**Python Imports:**
- `the`
- `depth`
- `color`


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

