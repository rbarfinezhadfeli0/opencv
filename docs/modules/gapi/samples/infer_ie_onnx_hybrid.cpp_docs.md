# Documentation for `modules/gapi/samples/infer_ie_onnx_hybrid.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/infer_ie_onnx_hybrid.cpp`
- **File Name**: `infer_ie_onnx_hybrid.cpp`
- **File Size**: 7,313 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/infer_ie_onnx_hybrid.cpp](../../../modules/gapi/samples/infer_ie_onnx_hybrid.cpp)

## Purpose and Role

This file is located in the `modules/gapi/samples` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <chrono>
#include <iomanip>

#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"

#include "opencv2/gapi.hpp"
#include "opencv2/gapi/core.hpp"
#include "opencv2/gapi/imgproc.hpp"
#include "opencv2/gapi/infer.hpp"
#include "opencv2/gapi/infer/ie.hpp"
#include "opencv2/gapi/infer/onnx.hpp"
#include "opencv2/gapi/cpu/gcpukernel.hpp"
#include "opencv2/gapi/streaming/cap.hpp"

namespace {
const std::string keys =
    "{ h help |   | print this help message }"
    "{ input  |   | Path to an input video file }"
    "{ fdm    |   | IE face detection model IR }"
    "{ fdw    |   | IE face detection model weights }"
    "{ fdd    |   | IE face detection device }"
    "{ emom   |   | ONNX emotions recognition model }"
    "{ output |   | (Optional) Path to an output video file }"
    ;
} // namespace

namespace custom {
G_API_NET(Faces, <cv::GMat(cv::GMat)>, "face-detector");
G_API_NET(Emotions, <cv::GMat(cv::GMat)>, "emotions-recognition");

G_API_OP(PostProc, <cv::GArray<cv::Rect>(cv::GMat, cv::GMat)>, "custom.fd_postproc") {
    static cv::GArrayDesc outMeta(const cv::GMatDesc &, const cv::GMatDesc &) {
        return cv::empty_array_desc();
    }
};

GAPI_OCV_KERNEL(OCVPostProc, PostProc) {
    static void run(const cv::Mat &in_ssd_result,
                    const cv::Mat &in_frame,
                    std::vector<cv::Rect> &out_faces) {
        const int MAX_PROPOSALS = 200;
        const int OBJECT_SIZE   =   7;
        const cv::Size upscale = in_frame.size();
        const cv::Rect surface({0,0}, upscale);

        out_faces.clear();

        const float *data = in_ssd_result.ptr<float>();
        for (int i = 0; i < MAX_PROPOSALS; i++) {
            const float image_id   = data[i * OBJECT_SIZE + 0]; // batch id
            const float confidence = data[i * OBJECT_SIZE + 2];
            const float rc_left    = data[i * OBJECT_SIZE + 3];
            const float rc_top     = data[i * OBJECT_SIZE + 4];
            const float rc_right   = data[i * OBJECT_SIZE + 5];
            const float rc_bottom  = data[i * OBJECT_SIZE + 6];

            if (image_id < 0.f) {  // indicates end of detections
                break;
            }
            if (confidence < 0.5f) {
                continue;
            }

            cv::Rect rc;
            rc.x      = static_cast<int>(rc_left   * upscale.width);
            rc.y      = static_cast<int>(rc_top    * upscale.height);
            rc.width  = static_cast<int>(rc_right  * upscale.width)  - rc.x;
            rc.height = static_cast<int>(rc_bottom * upscale.height) - rc.y;
            out_faces.push_back(rc & surface);
        }
    }
};
//! [Postproc]

} // namespace custom

namespace labels {
// Labels as defined in
// https://github.com/onnx/models/tree/master/vision/body_analysis/emotion_ferplus
//
const std::string emotions[] = {
    "neutral", "happiness", "surprise", "sadness", "anger", "disgust", "fear", "contempt"
};
namespace {
template<typename Iter>
std::vector<float> softmax(Iter begin, Iter end) {
    std::vector<float> prob(end - begin, 0.f);
    std::transform(begin, end, prob.begin(), [](float x) { return std::exp(x); });
    float sum = std::accumulate(prob.begin(), prob.end(), 0.0f);
    for (int i = 0; i < static_cast<int>(prob.size()); i++)
        prob[i] /= sum;
    return prob;
}

void DrawResults(cv::Mat &frame,
                 const std::vector<cv::Rect> &faces,
                 const std::vector<cv::Mat>  &out_emotions) {
    CV_Assert(faces.size() == out_emotions.size());

    for (auto it = faces.begin(); it != faces.end(); ++it) {
        const auto idx = std::distance(faces.begin(), it);
        const auto &rc = *it;

        const float *emotions_data = out_emotions[idx].ptr<float>();
        auto sm = softmax(emotions_data, emotions_data + 8);
        const auto emo_id = std::max_element(sm.begin(), sm.end()) - sm.begin();

        const int ATTRIB_OFFSET = 15;
        cv::rectangle(frame, rc, {0, 255, 0},  4);
        cv::putText(frame, emotions[emo_id],
                    cv::Point(rc.x, rc.y - ATTRIB_OFFSET),
                    cv::FONT_HERSHEY_COMPLEX_SMALL,
                    1,
                    cv::Scalar(0, 0, 255));

        std::cout << emotions[emo_id] << " at " << rc << std::endl;
    }
}
} // anonymous namespace
} // namespace labels

int main(int argc, char *argv[])
{
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }
    const std::string input = cmd.get<std::string>("input");
    const std::string output = cmd.get<std::string>("output");

    // OpenVINO FD parameters here
    auto det_net = cv::gapi::ie::Params<custom::Faces> {
        cmd.get<std::string>("fdm"),   // read cmd args: path to topology IR
        cmd.get<std::string>("fdw"),   // read cmd args: path to weights
        cmd.get<std::string>("fdd"),   // read cmd args: device specifier
    };

    // ONNX Emotions parameters here
    auto emo_net = cv::gapi::onnx::Params<custom::Emotions> {
        cmd.get<std::string>("emom"),   // read cmd args: path to the ONNX model
    }.cfgNormalize({false}); // model accepts 0..255 range in FP32

    auto kernels = cv::gapi::kernels<custom::OCVPostProc>();
    auto networks = cv::gapi::networks(det_net, emo_net);

    cv::GMat in;
    cv::GMat bgr = cv::gapi::copy(in);
    cv::GMat frame = cv::gapi::streaming::desync(bgr);
    cv::GMat detections = cv::gapi::infer<custom::Faces>(frame);
    cv::GArray<cv::Rect> faces = custom::PostProc::on(detections, frame);
    cv::GArray<cv::GMat> emotions = cv::gapi::infer<custom::Emotions>(faces, frame);
    auto pipeline = cv::GComputation(cv::GIn(in), cv::GOut(bgr, faces, emotions))
        .compileStreaming(cv::compile_args(kernels, networks));

    auto in_src = cv::gapi::wip::make_src<cv::gapi::wip::GCaptureSource>(input);
    pipeline.setSource(cv::gin(in_src));

    cv::util::optional<cv::Mat>               out_frame;
    cv::util::optional<std::vector<cv::Rect>> out_faces;
    cv::util::optional<std::vector<cv::Mat>>  out_emotions;

    cv::Mat               last_mat;
    std::vector<cv::Rect> last_faces;
    std::vector<cv::Mat>  last_emotions;

    cv::VideoWriter writer;
    cv::TickMeter tm;
    std::size_t frames = 0u;

    tm.start();
    pipeline.start();
    while (pipeline.pull(cv::gout(out_frame, out_faces, out_emotions))) {
        ++frames;
        if (out_faces && out_emotions) {
            last_faces = *out_faces;
            last_emotions = *out_emotions;
        }
        if (out_frame) {
            last_mat = *out_frame;
            labels::DrawResults(last_mat, last_faces, last_emotions);

            if (!output.empty()) {
                if (!writer.isOpened()) {
                    const auto sz = cv::Size{last_mat.cols, last_mat.rows};
                    writer.open(output, cv::VideoWriter::fourcc('M','J','P','G'), 25.0, sz);
                    CV_Assert(writer.isOpened());
                }
                writer << last_mat;
            }
        }
        if (!last_mat.empty()) {
            cv::imshow("Out", last_mat);
            cv::waitKey(1);
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
- `opencv2/gapi/infer/ie.hpp`
- `iomanip`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `chrono`
- `opencv2/imgproc.hpp`
- `opencv2/gapi/core.hpp`
- `opencv2/highgui.hpp`
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

