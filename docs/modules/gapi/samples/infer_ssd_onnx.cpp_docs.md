# Documentation for `modules/gapi/samples/infer_ssd_onnx.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/infer_ssd_onnx.cpp`
- **File Name**: `infer_ssd_onnx.cpp`
- **File Size**: 5,643 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/infer_ssd_onnx.cpp](../../../modules/gapi/samples/infer_ssd_onnx.cpp)

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
#include <opencv2/gapi/infer/onnx.hpp>
#include <opencv2/gapi/cpu/gcpukernel.hpp>
#include <opencv2/gapi/streaming/cap.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/gapi/infer/parsers.hpp>

namespace custom {

G_API_NET(ObjDetector,   <cv::GMat(cv::GMat)>, "object-detector");

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
    static void run(const std::vector<cv::Rect> &in_obj_rcs,
                          std::vector<cv::gapi::wip::draw::Prim> &out_prims) {
        out_prims.clear();
        const auto cvt = [](const cv::Rect &rc, const cv::Scalar &clr) {
            return cv::gapi::wip::draw::Rect(rc, clr, 2);
        };
        for (auto &&rc : in_obj_rcs) {
            out_prims.emplace_back(cvt(rc, CV_RGB(0,255,0)));   // green
        }

        std::cout << "Detections:";
        for (auto &&rc : in_obj_rcs) std::cout << ' ' << rc;
        std::cout << std::endl;
    }
};

} // namespace custom

namespace {
void remap_ssd_ports(const std::unordered_map<std::string, cv::Mat> &onnx,
                           std::unordered_map<std::string, cv::Mat> &gapi) {
    // Assemble ONNX-processed outputs back to a single 1x1x200x7 blob
    // to preserve compatibility with OpenVINO-based SSD pipeline
    const cv::Mat &num_detections = onnx.at("num_detections:0");
    const cv::Mat &detection_boxes = onnx.at("detection_boxes:0");
    const cv::Mat &detection_scores = onnx.at("detection_scores:0");
    const cv::Mat &detection_classes = onnx.at("detection_classes:0");

    GAPI_Assert(num_detections.depth() == CV_32F);
    GAPI_Assert(detection_boxes.depth() == CV_32F);
    GAPI_Assert(detection_scores.depth() == CV_32F);
    GAPI_Assert(detection_classes.depth() == CV_32F);

    cv::Mat &ssd_output = gapi.at("detection_output");

    const int num_objects = static_cast<int>(num_detections.ptr<float>()[0]);
    const float *in_boxes = detection_boxes.ptr<float>();
    const float *in_scores = detection_scores.ptr<float>();
    const float *in_classes = detection_classes.ptr<float>();
    float *ptr = ssd_output.ptr<float>();

    for (int i = 0; i < num_objects; i++) {
        ptr[0] = 0.f;               // "image_id"
        ptr[1] = in_classes[i];     // "label"
        ptr[2] = in_scores[i];      // "confidence"
        ptr[3] = in_boxes[4*i + 1]; // left
        ptr[4] = in_boxes[4*i + 0]; // top
        ptr[5] = in_boxes[4*i + 3]; // right
        ptr[6] = in_boxes[4*i + 2]; // bottom

        ptr      += 7;
        in_boxes += 4;
    }
    if (num_objects < ssd_output.size[2]-1) {
        // put a -1 mark at the end of output blob if there is space left
        ptr[0] = -1.f;
    }
}
} // anonymous namespace

const std::string keys =
    "{ h help | | Print this help message }"
    "{ input  | | Path to the input video file }"
    "{ output | | (Optional) path to output video file }"
    "{ detm   | | Path to an ONNX SSD object detection model (.onnx) }"
    ;

int main(int argc, char *argv[])
{
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }

    // Prepare parameters first
    const std::string input = cmd.get<std::string>("input");
    const std::string output = cmd.get<std::string>("output");
    const auto obj_model_path = cmd.get<std::string>("detm");

    auto obj_net = cv::gapi::onnx::Params<custom::ObjDetector>{obj_model_path}
        .cfgOutputLayers({"detection_output"})
        .cfgPostProc({cv::GMatDesc{CV_32F, {1,1,200,7}}}, remap_ssd_ports);
    auto kernels = cv::gapi::kernels<custom::OCVBBoxes>();
    auto networks = cv::gapi::networks(obj_net);

    // Now build the graph
    cv::GMat in;
    auto blob = cv::gapi::infer<custom::ObjDetector>(in);
    cv::GArray<cv::Rect> rcs =
        cv::gapi::parseSSD(blob, cv::gapi::streaming::size(in), 0.5f, true, true);
    auto  out = cv::gapi::wip::draw::render3ch(in, custom::BBoxes::on(rcs));
    cv::GStreamingCompiled pipeline = cv::GComputation(cv::GIn(in), cv::GOut(out))
        .compileStreaming(cv::compile_args(kernels, networks));

    auto inputs = cv::gin(cv::gapi::wip::make_src<cv::gapi::wip::GCaptureSource>(input));

    // The execution part
    pipeline.setSource(std::move(inputs));

    cv::TickMeter tm;
    cv::VideoWriter writer;
    size_t frames = 0u;
    cv::Mat outMat;

    tm.start();
    pipeline.start();
    while (pipeline.pull(cv::gout(outMat))) {
        ++frames;
        cv::imshow("Out", outMat);
        cv::waitKey(1);
        if (!output.empty()) {
            if (!writer.isOpened()) {
                const auto sz = cv::Size{outMat.cols, outMat.rows};
                writer.open(output, cv::VideoWriter::fourcc('M','J','P','G'), 25.0, sz);
                CV_Assert(writer.isOpened());
            }
            writer << outMat;
        }
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
- `opencv2/gapi/infer/onnx.hpp`
- `opencv2/gapi/infer.hpp`
- `opencv2/gapi/streaming/cap.hpp`
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

