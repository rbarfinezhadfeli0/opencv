# Documentation for `modules/gapi/samples/infer_single_roi.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/infer_single_roi.cpp`
- **File Name**: `infer_single_roi.cpp`
- **File Size**: 7,478 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/infer_single_roi.cpp](../../../modules/gapi/samples/infer_single_roi.cpp)

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
#include <opencv2/gapi/render.hpp>
#include <opencv2/gapi/infer/ie.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>
#include <opencv2/gapi/streaming/cap.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/gapi/infer/parsers.hpp>

const std::string keys =
    "{ h help |                              | Print this help message }"
    "{ input  |                              | Path to the input video file }"
    "{ facem  | face-detection-adas-0001.xml | Path to OpenVINO IE face detection model (.xml) }"
    "{ faced  | CPU                          | Target device for face detection model (e.g. CPU, GPU, VPU, ...) }"
    "{ r roi  | -1,-1,-1,-1                  | Region of interest (ROI) to use for inference. Identified automatically when not set }";

namespace {

std::string weights_path(const std::string &model_path) {
    const auto EXT_LEN = 4u;
    const auto sz = model_path.size();
    CV_Assert(sz > EXT_LEN);

    auto ext = model_path.substr(sz - EXT_LEN);
    std::transform(ext.begin(), ext.end(), ext.begin(), [](unsigned char c){
            return static_cast<unsigned char>(std::tolower(c));
        });
    CV_Assert(ext == ".xml");
    return model_path.substr(0u, sz - EXT_LEN) + ".bin";
}

cv::util::optional<cv::Rect> parse_roi(const std::string &rc) {
    cv::Rect rv;
    char delim[3];

    std::stringstream is(rc);
    is >> rv.x >> delim[0] >> rv.y >> delim[1] >> rv.width >> delim[2] >> rv.height;
    if (is.bad()) {
        return cv::util::optional<cv::Rect>(); // empty value
    }
    const auto is_delim = [](char c) {
        return c == ',';
    };
    if (!std::all_of(std::begin(delim), std::end(delim), is_delim)) {
        return cv::util::optional<cv::Rect>(); // empty value

    }
    if (rv.x < 0 || rv.y < 0 || rv.width <= 0 || rv.height <= 0) {
        return cv::util::optional<cv::Rect>(); // empty value
    }
    return cv::util::make_optional(std::move(rv));
}

} // namespace

namespace custom {

G_API_NET(FaceDetector,   <cv::GMat(cv::GMat)>, "face-detector");

using GDetections = cv::GArray<cv::Rect>;
using GRect       = cv::GOpaque<cv::Rect>;
using GSize       = cv::GOpaque<cv::Size>;
using GPrims      = cv::GArray<cv::gapi::wip::draw::Prim>;

G_API_OP(LocateROI, <GRect(cv::GMat)>, "sample.custom.locate-roi") {
    static cv::GOpaqueDesc outMeta(const cv::GMatDesc &) {
        return cv::empty_gopaque_desc();
    }
};

G_API_OP(BBoxes, <GPrims(GDetections, GRect)>, "sample.custom.b-boxes") {
    static cv::GArrayDesc outMeta(const cv::GArrayDesc &, const cv::GOpaqueDesc &) {
        return cv::empty_array_desc();
    }
};

GAPI_OCV_KERNEL(OCVLocateROI, LocateROI) {
    // This is the place where we can run extra analytics
    // on the input image frame and select the ROI (region
    // of interest) where we want to detect our objects (or
    // run any other inference).
    //
    // Currently it doesn't do anything intelligent,
    // but only crops the input image to square (this is
    // the most convenient aspect ratio for detectors to use)

    static void run(const cv::Mat &in_mat, cv::Rect &out_rect) {

        // Identify the central point & square size (- some padding)
        const auto center = cv::Point{in_mat.cols/2, in_mat.rows/2};
        auto sqside = std::min(in_mat.cols, in_mat.rows);

        // Now build the central square ROI
        out_rect = cv::Rect{ center.x - sqside/2
                           , center.y - sqside/2
                           , sqside
                           , sqside
                           };
    }
};

GAPI_OCV_KERNEL(OCVBBoxes, BBoxes) {
    // This kernel converts the rectangles into G-API's
    // rendering primitives
    static void run(const std::vector<cv::Rect> &in_face_rcs,
                    const             cv::Rect  &in_roi,
                          std::vector<cv::gapi::wip::draw::Prim> &out_prims) {
        out_prims.clear();
        const auto cvt = [](const cv::Rect &rc, const cv::Scalar &clr) {
            return cv::gapi::wip::draw::Rect(rc, clr, 2);
        };
        out_prims.emplace_back(cvt(in_roi, CV_RGB(0,255,255))); // cyan
        for (auto &&rc : in_face_rcs) {
            out_prims.emplace_back(cvt(rc, CV_RGB(0,255,0)));   // green
        }
    }
};

} // namespace custom

int main(int argc, char *argv[])
{
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }

    // Prepare parameters first
    const std::string input = cmd.get<std::string>("input");
    const auto opt_roi = parse_roi(cmd.get<std::string>("roi"));

    const auto face_model_path = cmd.get<std::string>("facem");
    auto face_net = cv::gapi::ie::Params<custom::FaceDetector> {
        face_model_path,                 // path to topology IR
        weights_path(face_model_path),   // path to weights
        cmd.get<std::string>("faced"),   // device specifier
    };
    auto kernels = cv::gapi::kernels
        <custom::OCVLocateROI
        , custom::OCVBBoxes>();
    auto networks = cv::gapi::networks(face_net);

    // Now build the graph. The graph structure may vary
    // passed on the input parameters
    cv::GStreamingCompiled pipeline;
    auto inputs = cv::gin(cv::gapi::wip::make_src<cv::gapi::wip::GCaptureSource>(input));

    cv::GMat in;
    cv::GOpaque<cv::Size> sz = cv::gapi::streaming::size(in);
    if (opt_roi.has_value()) {
        // Use the value provided by user
        std::cout << "Will run inference for static region "
                  << opt_roi.value()
                  << " only"
                  << std::endl;
        cv::GOpaque<cv::Rect> in_roi;
        auto blob = cv::gapi::infer<custom::FaceDetector>(in_roi, in);
        cv::GArray<cv::Rect> rcs = cv::gapi::parseSSD(blob, sz, 0.5f, true, true);
        auto  out = cv::gapi::wip::draw::render3ch(in, custom::BBoxes::on(rcs, in_roi));
        pipeline  = cv::GComputation(cv::GIn(in, in_roi), cv::GOut(out))
            .compileStreaming(cv::compile_args(kernels, networks));

        // Since the ROI to detect is manual, make it part of the input vector
        inputs.push_back(cv::gin(opt_roi.value())[0]);
    } else {
        // Automatically detect ROI to infer. Make it output parameter
        std::cout << "ROI is not set or invalid. Locating it automatically"
                  << std::endl;
        cv::GOpaque<cv::Rect> roi = custom::LocateROI::on(in);
        auto blob = cv::gapi::infer<custom::FaceDetector>(roi, in);
        cv::GArray<cv::Rect> rcs = cv::gapi::parseSSD(blob, sz, 0.5f, true, true);
        auto  out = cv::gapi::wip::draw::render3ch(in, custom::BBoxes::on(rcs, roi));
        pipeline  = cv::GComputation(cv::GIn(in), cv::GOut(out))
            .compileStreaming(cv::compile_args(kernels, networks));
    }

    // The execution part
    pipeline.setSource(std::move(inputs));
    pipeline.start();

    cv::Mat out;
    size_t frames = 0u;
    cv::TickMeter tm;
    tm.start();
    while (pipeline.pull(cv::gout(out))) {
        cv::imshow("Out", out);
        cv::waitKey(1);
        ++frames;
    }
    tm.stop();
    std::cout << "Processed " << frames << " frames" << " (" << frames / tm.getTimeSec() << " FPS)" << std::endl;
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
- `opencv2/gapi/streaming/cap.hpp`
- `opencv2/gapi/infer/ie.hpp`
- `opencv2/imgcodecs.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `opencv2/highgui.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/render.hpp`
- `sstream`
- `opencv2/gapi/core.hpp`
- `opencv2/gapi/infer/parsers.hpp`
- `algorithm`
- `opencv2/gapi.hpp`


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

