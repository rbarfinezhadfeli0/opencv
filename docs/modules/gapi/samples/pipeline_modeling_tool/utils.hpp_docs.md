# Documentation for `modules/gapi/samples/pipeline_modeling_tool/utils.hpp`

## File Metadata

- **Full Path**: `modules/gapi/samples/pipeline_modeling_tool/utils.hpp`
- **File Name**: `utils.hpp`
- **File Size**: 4,123 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/samples/pipeline_modeling_tool/utils.hpp](../../../../modules/gapi/samples/pipeline_modeling_tool/utils.hpp)

## Purpose and Role

This file is located in the `modules/gapi/samples/pipeline_modeling_tool` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef OPENCV_GAPI_PIPELINE_MODELING_TOOL_UTILS_HPP
#define OPENCV_GAPI_PIPELINE_MODELING_TOOL_UTILS_HPP

#include <map>

#include <opencv2/core.hpp>

#if defined(_WIN32)
#include <windows.h>
#endif

// FIXME: It's better to place it somewhere in common.hpp
struct OutputDescr {
    std::vector<int> dims;
    int              precision;
};

namespace utils {

using double_ms_t = std::chrono::duration<double, std::milli>;

inline void createNDMat(cv::Mat& mat, const std::vector<int>& dims, int depth) {
    GAPI_Assert(!dims.empty());
    mat.create(dims, depth);
    if (dims.size() == 1) {
        //FIXME: Well-known 1D mat WA
        mat.dims = 1;
    }
}

inline void generateRandom(cv::Mat& out) {
    switch (out.depth()) {
        case CV_8U:
            cv::randu(out, 0, 255);
            break;
        case CV_32F:
            cv::randu(out, 0.f, 1.f);
            break;
        case CV_16F: {
            std::vector<int> dims;
            for (int i = 0; i < out.size.dims(); ++i) {
                dims.push_back(out.size[i]);
            }
            cv::Mat fp32_mat;
            createNDMat(fp32_mat, dims, CV_32F);
            cv::randu(fp32_mat, 0.f, 1.f);
            fp32_mat.convertTo(out, out.type());
            break;
        }
        default:
            throw std::logic_error("Unsupported preprocessing depth");
    }
}

inline void sleep(std::chrono::microseconds delay) {
#if defined(_WIN32)
    // FIXME: Wrap it to RAII and instance only once.
    HANDLE timer = CreateWaitableTimer(NULL, true, NULL);
    if (!timer) {
        throw std::logic_error("Failed to create timer");
    }

    LARGE_INTEGER li;
    using ns_t = std::chrono::nanoseconds;
    using ns_100_t = std::chrono::duration<ns_t::rep,
                                           std::ratio_multiply<std::ratio<100>, ns_t::period>>;
    // NB: QuadPart takes portions of 100 nanoseconds.
    li.QuadPart = -std::chrono::duration_cast<ns_100_t>(delay).count();

    if(!SetWaitableTimer(timer, &li, 0, NULL, NULL, false)){
        CloseHandle(timer);
        throw std::logic_error("Failed to set timer");
    }
    if (WaitForSingleObject(timer, INFINITE) != WAIT_OBJECT_0) {
        CloseHandle(timer);
        throw std::logic_error("Failed to wait timer");
    }
    CloseHandle(timer);
#else
    std::this_thread::sleep_for(delay);
#endif
}

template <typename duration_t>
typename duration_t::rep measure(std::function<void()> f) {
    using namespace std::chrono;
    auto start = high_resolution_clock::now();
    f();
    return duration_cast<duration_t>(
            high_resolution_clock::now() - start).count();
}

template <typename duration_t>
typename duration_t::rep timestamp() {
    using namespace std::chrono;
    auto now = high_resolution_clock::now();
    return duration_cast<duration_t>(now.time_since_epoch()).count();
}

inline void busyWait(std::chrono::microseconds delay) {
    auto start_ts     = timestamp<std::chrono::microseconds>();
    auto end_ts       = start_ts;
    auto time_to_wait = delay.count();

    while (end_ts - start_ts < time_to_wait) {
        end_ts = timestamp<std::chrono::microseconds>();
    }
}

template <typename K, typename V>
void mergeMapWith(std::map<K, V>& target, const std::map<K, V>& second) {
    for (auto&& item : second) {
        auto it = target.find(item.first);
        if (it != target.end()) {
            throw std::logic_error("Error: key: " + it->first + " is already in target map");
        }
        target.insert(item);
    }
}

template <typename T>
double avg(const std::vector<T>& vec) {
    return std::accumulate(vec.begin(), vec.end(), 0.0) / vec.size();
}

template <typename T>
T max(const std::vector<T>& vec) {
    return *std::max_element(vec.begin(), vec.end());
}

template <typename T>
T min(const std::vector<T>& vec) {
    return *std::min_element(vec.begin(), vec.end());
}

template <typename T>
int64_t ms_to_mcs(T ms) {
    using namespace std::chrono;
    return duration_cast<microseconds>(duration<T, std::milli>(ms)).count();
}

} // namespace utils

#endif // OPENCV_GAPI_PIPELINE_MODELING_TOOL_UTILS_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **OutputDescr**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_PIPELINE_MODELING_TOOL_UTILS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `windows.h`
- `map`


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

