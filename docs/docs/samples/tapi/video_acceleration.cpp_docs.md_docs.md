# Documentation for `docs/samples/tapi/video_acceleration.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/samples/tapi/video_acceleration.cpp_docs.md`
- **File Name**: `video_acceleration.cpp_docs.md`
- **File Size**: 10,616 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/tapi/video_acceleration.cpp_docs.md](../../../docs/samples/tapi/video_acceleration.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/tapi/video_acceleration.cpp`

## File Metadata

- **Full Path**: `samples/tapi/video_acceleration.cpp`
- **File Name**: `video_acceleration.cpp`
- **File Size**: 7,466 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/tapi/video_acceleration.cpp](../../samples/tapi/video_acceleration.cpp)

## Purpose and Role

This file is located in the `samples/tapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <iostream>
#include <chrono>
#include "opencv2/core.hpp"
#include "opencv2/core/ocl.hpp"
#include "opencv2/core/utility.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/videoio.hpp"
#include "opencv2/highgui.hpp"

using namespace cv;
using namespace std;

const char* keys =
"{ i input    |        | input video file }"
"{ o output   |        | output video file, or specify 'null' to measure decoding without rendering to screen}"
"{ backend    | any    | VideoCapture and VideoWriter backend, valid values: 'any', 'ffmpeg', 'msmf', 'gstreamer' }"
"{ accel      | any    | GPU Video Acceleration, valid values: 'none', 'any', 'd3d11', 'vaapi', 'mfx' }"
"{ device     | -1     | Video Acceleration device (GPU) index (-1 means default device) }"
"{ out_w      |        | output width (resize by calling cv::resize) }"
"{ out_h      |        | output height (resize by calling cv::resize) }"
"{ bitwise_not| false  | apply simple image processing - bitwise_not pixels by calling cv::bitwise_not }"
"{ opencl     | true   | use OpenCL (inside VideoCapture/VideoWriter and for image processing) }"
"{ codec      | H264   | codec id (four characters string) of output file encoder }"
"{ h help     |        | print help message }";

struct {
    cv::VideoCaptureAPIs backend;
    const char* str;
} backend_strings[] = {
    { cv::CAP_ANY, "any" },
    { cv::CAP_FFMPEG, "ffmpeg" },
    { cv::CAP_MSMF, "msmf" },
    { cv::CAP_GSTREAMER, "gstreamer" },
};

struct {
    VideoAccelerationType acceleration;
    const char* str;
} acceleration_strings[] = {
    { VIDEO_ACCELERATION_NONE, "none" },
    { VIDEO_ACCELERATION_ANY, "any" },
    { VIDEO_ACCELERATION_D3D11, "d3d11" },
    { VIDEO_ACCELERATION_VAAPI, "vaapi" },
    { VIDEO_ACCELERATION_MFX, "mfx" },
    { VIDEO_ACCELERATION_DRM, "drm" },
};

class FPSCounter {
public:
    FPSCounter(double _interval) : interval(_interval) {
    }

    ~FPSCounter() {
        NewFrame(true);
    }

    void NewFrame(bool last_frame = false) {
        num_frames++;
        auto now = std::chrono::high_resolution_clock::now();
        if (!last_time.time_since_epoch().count()) {
            last_time = now;
        }

        double sec = std::chrono::duration_cast<std::chrono::duration<double>>(now - last_time).count();
        if (sec >= interval || last_frame) {
            printf("FPS(last %.2f sec) = %.2f\n", sec, num_frames / sec);
            fflush(stdout);
            num_frames = 0;
            last_time = now;
        }
    }

private:
    double interval = 1;
    std::chrono::time_point<std::chrono::high_resolution_clock> last_time;
    int num_frames = 0;
};

int main(int argc, char** argv)
{
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help"))
    {
        cout << "Usage : video_acceleration [options]" << endl;
        cout << "Available options:" << endl;
        cmd.printMessage();
        return EXIT_SUCCESS;
    }

    string infile = cmd.get<string>("i");
    string outfile = cmd.get<string>("o");
    string codec = cmd.get<string>("codec");
    int device = cmd.get<int>("device");
    int out_w = cmd.get<int>("out_w");
    int out_h = cmd.get<int>("out_h");
    bool use_opencl = cmd.get<bool>("opencl");
    bool bitwise_not = cmd.get<bool>("bitwise_not");

    cv::VideoCaptureAPIs backend = cv::CAP_ANY;
    string backend_str = cmd.get<string>("backend");
    for (size_t i = 0; i < sizeof(backend_strings)/sizeof(backend_strings[0]); i++) {
        if (backend_str == backend_strings[i].str) {
            backend = backend_strings[i].backend;
            break;
        }
    }

    VideoAccelerationType accel = VIDEO_ACCELERATION_ANY;
    string accel_str = cmd.get<string>("accel");
    for (size_t i = 0; i < sizeof(acceleration_strings) / sizeof(acceleration_strings[0]); i++) {
        if (accel_str == acceleration_strings[i].str) {
            accel = acceleration_strings[i].acceleration;
            break;
        }
    }

    ocl::setUseOpenCL(use_opencl);

    VideoCapture capture(infile, backend, {
            CAP_PROP_HW_ACCELERATION, (int)accel,
            CAP_PROP_HW_DEVICE, device
    });
    if (!capture.isOpened()) {
        cerr << "Failed to open VideoCapture" << endl;
        return 1;
    }
    cout << "VideoCapture backend = " << capture.getBackendName() << endl;
    VideoAccelerationType actual_accel = static_cast<VideoAccelerationType>(static_cast<int>(capture.get(CAP_PROP_HW_ACCELERATION)));
    for (size_t i = 0; i < sizeof(acceleration_strings) / sizeof(acceleration_strings[0]); i++) {
        if (actual_accel == acceleration_strings[i].acceleration) {
            cout << "VideoCapture acceleration = " << acceleration_strings[i].str << endl;
            cout << "VideoCapture acceleration device = " << (int)capture.get(CAP_PROP_HW_DEVICE) << endl;
            break;
        }
    }

    VideoWriter writer;
    if (!outfile.empty() && outfile != "null") {
        const char* codec_str = codec.c_str();
        int fourcc = VideoWriter::fourcc(codec_str[0], codec_str[1], codec_str[2], codec_str[3]);
        double fps = capture.get(CAP_PROP_FPS);
        Size frameSize = { out_w, out_h };
        if (!out_w || !out_h) {
            frameSize = { (int)capture.get(CAP_PROP_FRAME_WIDTH), (int)capture.get(CAP_PROP_FRAME_HEIGHT) };
        }
        writer = VideoWriter(outfile, backend, fourcc, fps, frameSize, {
                VIDEOWRITER_PROP_HW_ACCELERATION, (int)accel,
                VIDEOWRITER_PROP_HW_DEVICE, device
        });
        if (!writer.isOpened()) {
            cerr << "Failed to open VideoWriter" << endl;
            return 1;
        }
        cout << "VideoWriter backend = " << writer.getBackendName() << endl;
        actual_accel = static_cast<VideoAccelerationType>(static_cast<int>(writer.get(VIDEOWRITER_PROP_HW_ACCELERATION)));
        for (size_t i = 0; i < sizeof(acceleration_strings) / sizeof(acceleration_strings[0]); i++) {
            if (actual_accel == acceleration_strings[i].acceleration) {
                cout << "VideoWriter acceleration = " << acceleration_strings[i].str << endl;
                cout << "VideoWriter acceleration device = " << (int)writer.get(VIDEOWRITER_PROP_HW_DEVICE) << endl;
                break;
            }
        }
    }

    cout << "\nStarting frame loop. Press ESC to exit\n";

    FPSCounter fps_counter(0.5); // print FPS every 0.5 seconds

    UMat frame, frame2, frame3;

    for (;;)
    {
        capture.read(frame);
        if (frame.empty()) {
            cout << "End of stream" << endl;
            break;
        }

        if (out_w && out_h) {
            cv::resize(frame, frame2, cv::Size(out_w, out_h));
            //cv::cvtColor(frame, outframe, COLOR_BGRA2RGBA);
        }
        else {
            frame2 = frame;
        }

        if (bitwise_not) {
            cv::bitwise_not(frame2, frame3);
        }
        else {
            frame3 = frame2;
        }

        if (writer.isOpened()) {
            writer.write(frame3);
        }

        if (outfile.empty()) {
            imshow("output", frame3);
            char key = (char) waitKey(1);
            if (key == 27)
                break;
            else if (key == 'm') {
                ocl::setUseOpenCL(!cv::ocl::useOpenCL());
                cout << "Switched to " << (ocl::useOpenCL() ? "OpenCL enabled" : "CPU") << " mode\n";
            }
        }
        fps_counter.NewFrame();
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

### Classes and Structures

- **FPSCounter**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `chrono`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/core/utility.hpp`
- `opencv2/videoio.hpp`
- `opencv2/core.hpp`
- `opencv2/highgui.hpp`
- `opencv2/core/ocl.hpp`


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

