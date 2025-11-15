# Documentation for `modules/gapi/samples/oak_basic_infer.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/oak_basic_infer.cpp`
- **File Name**: `oak_basic_infer.cpp`
- **File Size**: 4,173 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/oak_basic_infer.cpp](../../../modules/gapi/samples/oak_basic_infer.cpp)

## Purpose and Role

This file is located in the `modules/gapi/samples` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <algorithm>
#include <iostream>
#include <sstream>

#include <opencv2/imgproc.hpp>
#include <opencv2/imgcodecs.hpp>
#include <opencv2/gapi.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/imgproc.hpp>
#include <opencv2/gapi/infer.hpp>
#include <opencv2/gapi/infer/parsers.hpp>
#include <opencv2/gapi/render.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>
#include <opencv2/highgui.hpp>

#include <opencv2/gapi/oak/oak.hpp>
#include <opencv2/gapi/oak/infer.hpp>

const std::string keys =
    "{ h help              |             | Print this help message }"
    "{ detector            |             | Path to compiled .blob face detector model }"
    "{ duration            | 100         | Number of frames to pull from camera and run inference on }";

namespace custom {

G_API_NET(FaceDetector, <cv::GMat(cv::GFrame)>, "sample.custom.face-detector");

using GDetections = cv::GArray<cv::Rect>;
using GSize       = cv::GOpaque<cv::Size>;
using GPrims      = cv::GArray<cv::gapi::wip::draw::Prim>;

G_API_OP(BBoxes, <GPrims(GDetections)>, "sample.custom.b-boxes") {
    static cv::GArrayDesc outMeta(const cv::GArrayDesc &) {
        return cv::empty_array_desc();
    }
};

GAPI_OCV_KERNEL(OCVBBoxes, BBoxes) {
    // This kernel converts the rectangles into G-API's
    // rendering primitives
    static void run(const std::vector<cv::Rect> &in_face_rcs,
                          std::vector<cv::gapi::wip::draw::Prim> &out_prims) {
        out_prims.clear();
        const auto cvt = [](const cv::Rect &rc, const cv::Scalar &clr) {
            return cv::gapi::wip::draw::Rect(rc, clr, 2);
        };
        for (auto &&rc : in_face_rcs) {
            out_prims.emplace_back(cvt(rc, CV_RGB(0,255,0))); // green
        }
    }
};

} // namespace custom

int main(int argc, char *argv[]) {
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }

    const auto det_name = cmd.get<std::string>("detector");
    const auto duration = cmd.get<int>("duration");

    if (det_name.empty()) {
        std::cerr << "FATAL: path to detection model is not provided for the sample."
                  << "Please specify it with --detector options."
                  << std::endl;
        return 1;
    }

    // Prepare G-API kernels and networks packages:
    auto detector = cv::gapi::oak::Params<custom::FaceDetector>(det_name);
    auto networks = cv::gapi::networks(detector);

    auto kernels = cv::gapi::combine(
        cv::gapi::kernels<custom::OCVBBoxes>(),
        cv::gapi::oak::kernels());

    auto args = cv::compile_args(kernels, networks);

    // Initialize graph structure
    cv::GFrame in;
    cv::GFrame copy = cv::gapi::oak::copy(in); // NV12 transfered to host + passthrough copy for infer
    cv::GOpaque<cv::Size> sz = cv::gapi::streaming::size(copy);

    // infer is not affected by the actual copy here
    cv::GMat blob = cv::gapi::infer<custom::FaceDetector>(copy);
    // FIXME: OAK infer detects faces slightly out of frame bounds
    cv::GArray<cv::Rect> rcs = cv::gapi::parseSSD(blob, sz, 0.5f, true, false);
    auto rendered = cv::gapi::wip::draw::renderFrame(copy, custom::BBoxes::on(rcs));
    // on-the-fly conversion NV12->BGR
    cv::GMat out = cv::gapi::streaming::BGR(rendered);

    auto pipeline  = cv::GComputation(cv::GIn(in), cv::GOut(out, rcs))
        .compileStreaming(std::move(args));

    // Graph execution
    pipeline.setSource(cv::gapi::wip::make_src<cv::gapi::oak::ColorCamera>());
    pipeline.start();

    cv::Mat out_mat;
    std::vector<cv::Rect> out_dets;
    int frames = 0;
    while (pipeline.pull(cv::gout(out_mat, out_dets))) {
        std::string name = "oak_infer_frame_" + std::to_string(frames) + ".png";

        cv::imwrite(name, out_mat);

        if (!out_dets.empty()) {
            std::cout << "Got " << out_dets.size() << " detections on frame #" << frames << std::endl;
        }

        ++frames;
        if (frames == duration) {
            pipeline.stop();
            break;
        }
    }
    std::cout << "Pipeline finished. Processed " << frames << " frames" << std::endl;
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
- `opencv2/gapi/imgproc.hpp`
- `opencv2/gapi/infer.hpp`
- `opencv2/gapi/oak/oak.hpp`
- `opencv2/imgcodecs.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `opencv2/highgui.hpp`
- `opencv2/gapi/oak/infer.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/infer/parsers.hpp`
- `sstream`
- `opencv2/gapi/core.hpp`
- `opencv2/gapi/render.hpp`
- `algorithm`
- `opencv2/gapi.hpp`

**Python Imports:**
- `camera`


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

