# Documentation for `modules/gapi/samples/onevpl_source_to_bgr_conv.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/onevpl_source_to_bgr_conv.cpp`
- **File Name**: `onevpl_source_to_bgr_conv.cpp`
- **File Size**: 3,955 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/onevpl_source_to_bgr_conv.cpp](../../../modules/gapi/samples/onevpl_source_to_bgr_conv.cpp)

## Purpose and Role

This file is located in the `modules/gapi/samples` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <algorithm>
#include <fstream>
#include <iostream>
#include <cctype>
#include <tuple>
#include <memory>

#include <opencv2/imgproc.hpp>
#include <opencv2/gapi.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/gpu/ggpukernel.hpp>
#include <opencv2/gapi/streaming/onevpl/source.hpp>
#include <opencv2/gapi/streaming/onevpl/data_provider_interface.hpp>
#include <opencv2/gapi/streaming/onevpl/default.hpp>
#include <opencv2/highgui.hpp> // CommandLineParser
#include <opencv2/gapi/ocl/core.hpp>

const std::string about =
    "This is an example presents decoding on GPU using VPL Source and passing it to OpenCL backend";
const std::string keys =
    "{ h help       |                                                                | Print this help message }"
    "{ input        |                                                                | Path to the input video file. Use .avi extension }"
    "{ accel_mode   | mfxImplDescription.AccelerationMode:MFX_ACCEL_MODE_VIA_D3D11   | Acceleration mode for VPL }";

namespace {
namespace cfg {
// FIXME: Move OneVPL arguments parser to a single place
typename cv::gapi::wip::onevpl::CfgParam create_from_string(const std::string &line);
} // namespace cfg
} // anonymous namespace

int main(int argc, char *argv[]) {
    cv::CommandLineParser cmd(argc, argv, keys);
    cmd.about(about);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }

    // Get file name
    const auto input = cmd.get<std::string>("input");
    const auto accel_mode = cmd.get<std::string>("accel_mode");

    // Create VPL config
    std::vector<cv::gapi::wip::onevpl::CfgParam> source_cfgs;
    source_cfgs.push_back(cfg::create_from_string(accel_mode));

    // Create VPL-based source
    std::shared_ptr<cv::gapi::wip::onevpl::IDeviceSelector> default_device_selector =
                                                cv::gapi::wip::onevpl::getDefaultDeviceSelector(source_cfgs);

    cv::gapi::wip::IStreamSource::Ptr source = cv::gapi::wip::make_onevpl_src(input, source_cfgs,
                                                                              default_device_selector);

    // Build the graph
    cv::GFrame in; // input frame from VPL source
    auto bgr_gmat = cv::gapi::streaming::BGR(in); // conversion from VPL source frame to BGR UMat
    auto out = cv::gapi::blur(bgr_gmat, cv::Size(4,4)); // ocl kernel of blur operation

    cv::GStreamingCompiled pipeline = cv::GComputation(cv::GIn(in), cv::GOut(out))
        .compileStreaming(cv::compile_args(cv::gapi::core::ocl::kernels()));
    pipeline.setSource(std::move(source));

    // The execution part
    size_t frames = 0u;
    cv::TickMeter tm;
    cv::Mat outMat;

    pipeline.start();
    tm.start();

    while (pipeline.pull(cv::gout(outMat))) {
        cv::imshow("OutVideo", outMat);
        cv::waitKey(1);
        ++frames;
    }
    tm.stop();
    std::cout << "Processed " << frames << " frames" << " (" << frames / tm.getTimeSec() << " FPS)" << std::endl;

    return 0;
}

namespace {
namespace cfg {
typename cv::gapi::wip::onevpl::CfgParam create_from_string(const std::string &line) {
    using namespace cv::gapi::wip;

    if (line.empty()) {
        throw std::runtime_error("Cannot parse CfgParam from emply line");
    }

    std::string::size_type name_endline_pos = line.find(':');
    if (name_endline_pos == std::string::npos) {
        throw std::runtime_error("Cannot parse CfgParam from: " + line +
                                 "\nExpected separator \":\"");
    }

    std::string name = line.substr(0, name_endline_pos);
    std::string value = line.substr(name_endline_pos + 1);

    return cv::gapi::wip::onevpl::CfgParam::create(name, value,
                                                   /* vpp params strongly optional */
                                                   name.find("vpp.") == std::string::npos);
}
} // namespace cfg
} // anonymous namespace
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
- `fstream`
- `opencv2/gapi/ocl/core.hpp`
- `cctype`
- `opencv2/gapi/streaming/onevpl/data_provider_interface.hpp`
- `opencv2/gapi/streaming/onevpl/default.hpp`
- `opencv2/highgui.hpp`
- `opencv2/gapi/gpu/ggpukernel.hpp`
- `iostream`
- `opencv2/imgproc.hpp`
- `memory`
- `tuple`
- `opencv2/gapi/core.hpp`
- `opencv2/gapi/streaming/onevpl/source.hpp`
- `algorithm`
- `opencv2/gapi.hpp`

**Python Imports:**
- `emply`
- `VPL`


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

