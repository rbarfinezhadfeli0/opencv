# Documentation for `modules/gapi/samples/oak_small_hetero_pipeline.cpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/oak_small_hetero_pipeline.cpp`
- **File Name**: `oak_small_hetero_pipeline.cpp`
- **File Size**: 1,940 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/samples/oak_small_hetero_pipeline.cpp](../../../modules/gapi/samples/oak_small_hetero_pipeline.cpp)

## Purpose and Role

This file is located in the `modules/gapi/samples` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/gapi.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/cpu/core.hpp>
#include <opencv2/gapi/gframe.hpp>
#include <opencv2/gapi/media.hpp>

#include <opencv2/gapi/oak/oak.hpp>
#include <opencv2/gapi/streaming/format.hpp> // BGR accessor

#include <opencv2/highgui.hpp> // CommandLineParser

const std::string keys =
    "{ h help  |              | Print this help message }"
    "{ output  | output.png   | Path to the output file }";

int main(int argc, char *argv[]) {
    cv::CommandLineParser cmd(argc, argv, keys);
    if (cmd.has("help")) {
        cmd.printMessage();
        return 0;
    }

    const std::string output_name = cmd.get<std::string>("output");

    std::vector<int> h = {1, 0, -1,
                          2, 0, -2,
                          1, 0, -1};
    std::vector<int> v = { 1,  2,  1,
                           0,  0,  0,
                          -1, -2, -1};
    cv::Mat hk(3, 3, CV_32SC1, h.data());
    cv::Mat vk(3, 3, CV_32SC1, v.data());

    // Heterogeneous pipeline:
    // OAK camera -> Sobel -> streaming accessor (CPU)
    cv::GFrame in;
    cv::GFrame sobel = cv::gapi::oak::sobelXY(in, hk, vk);
    // Default camera and then sobel work only with nv12 format
    cv::GMat out = cv::gapi::streaming::Y(sobel);

    auto args = cv::compile_args(cv::gapi::oak::ColorCameraParams{},
                                 cv::gapi::oak::kernels());

    auto pipeline = cv::GComputation(cv::GIn(in), cv::GOut(out)).compileStreaming(std::move(args));

    // Graph execution /////////////////////////////////////////////////////////
    cv::Mat out_mat(1920, 1080, CV_8UC1);

    pipeline.setSource(cv::gapi::wip::make_src<cv::gapi::oak::ColorCamera>());
    pipeline.start();

    // pull 1 frame
    pipeline.pull(cv::gout(out_mat));

    cv::imwrite(output_name, out_mat);

    std::cout << "Pipeline finished: " << output_name << " file has been written." << std::endl;
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
- `opencv2/gapi/gframe.hpp`
- `opencv2/gapi/oak/oak.hpp`
- `opencv2/gapi.hpp`
- `opencv2/gapi/media.hpp`
- `opencv2/highgui.hpp`
- `opencv2/gapi/cpu/core.hpp`
- `opencv2/gapi/streaming/format.hpp`
- `opencv2/gapi/core.hpp`


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

