# Documentation for `modules/gapi/test/gapi_graph_meta_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_graph_meta_tests.cpp`
- **File Name**: `gapi_graph_meta_tests.cpp`
- **File Size**: 6,246 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_graph_meta_tests.cpp](../../../modules/gapi/test/gapi_graph_meta_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation

#include <tuple>
#include <unordered_set>

#include "test_precomp.hpp"
#include "opencv2/gapi/streaming/meta.hpp"
#include "opencv2/gapi/streaming/cap.hpp"

namespace opencv_test {

TEST(GraphMeta, Trad_AccessInput) {
    cv::GMat in;
    cv::GMat out1 = cv::gapi::blur(in, cv::Size(3,3));
    cv::GOpaque<int> out2 = cv::gapi::streaming::meta<int>(in, "foo");
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2));

    cv::Mat in_mat = cv::Mat::eye(cv::Size(64, 64), CV_8UC1);
    cv::Mat out_mat;
    int out_meta = 0;

    // manually set metadata in the input fields
    auto inputs = cv::gin(in_mat);
    inputs[0].meta["foo"] = 42;

    graph.apply(std::move(inputs), cv::gout(out_mat, out_meta));
    EXPECT_EQ(42, out_meta);
}

TEST(GraphMeta, Trad_AccessTmp) {
    cv::GMat in;
    cv::GMat tmp = cv::gapi::blur(in, cv::Size(3,3));
    cv::GMat out1 = tmp+1;
    cv::GOpaque<float> out2 = cv::gapi::streaming::meta<float>(tmp, "bar");
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2));

    cv::Mat in_mat = cv::Mat::eye(cv::Size(64, 64), CV_8UC1);
    cv::Mat out_mat;
    float out_meta = 0.f;

    // manually set metadata in the input fields
    auto inputs = cv::gin(in_mat);
    inputs[0].meta["bar"] = 1.f;

    graph.apply(std::move(inputs), cv::gout(out_mat, out_meta));
    EXPECT_EQ(1.f, out_meta);
}

TEST(GraphMeta, Trad_AccessOutput) {
    cv::GMat in;
    cv::GMat out1 = cv::gapi::blur(in, cv::Size(3,3));
    cv::GOpaque<std::string> out2 = cv::gapi::streaming::meta<std::string>(out1, "baz");
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2));

    cv::Mat in_mat = cv::Mat::eye(cv::Size(64, 64), CV_8UC1);
    cv::Mat out_mat;
    std::string out_meta;

    // manually set metadata in the input fields
    auto inputs = cv::gin(in_mat);

    // NOTE: Assigning explicitly an std::string is important,
    // otherwise a "const char*" will be stored and won't be
    // translated properly by util::any since std::string is
    // used within the graph.
    inputs[0].meta["baz"] = std::string("opencv");

    graph.apply(std::move(inputs), cv::gout(out_mat, out_meta));
    EXPECT_EQ("opencv", out_meta);
}

TEST(GraphMeta, Streaming_AccessInput) {
    cv::GMat in;
    cv::GMat out1 = cv::gapi::blur(in, cv::Size(3,3));
    cv::GOpaque<int64_t> out2 = cv::gapi::streaming::seq_id(in);
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2));

    auto ccomp = graph.compileStreaming();
    const auto path = findDataFile("cv/video/768x576.avi");
    try {
        ccomp.setSource<cv::gapi::wip::GCaptureSource>(path);
    } catch(...) {
        throw SkipTestException("Video file can not be opened");
    }
    ccomp.start();

    cv::Mat out_mat;
    int64_t out_meta = 0;
    int64_t expected_counter = 0;

    while (ccomp.pull(cv::gout(out_mat, out_meta))) {
        EXPECT_EQ(expected_counter, out_meta);
        ++expected_counter;
    }
}

TEST(GraphMeta, Streaming_AccessOutput) {
    cv::GMat in;
    cv::GMat out1 = cv::gapi::blur(in, cv::Size(3,3));
    cv::GOpaque<int64_t> out2 = cv::gapi::streaming::seq_id(out1);
    cv::GOpaque<int64_t> out3 = cv::gapi::streaming::timestamp(out1);
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2, out3));

    auto ccomp = graph.compileStreaming();
    const auto path = findDataFile("cv/video/768x576.avi");
    try {
        ccomp.setSource<cv::gapi::wip::GCaptureSource>(path);
    } catch(...) {
        throw SkipTestException("Video file can not be opened");
    }
    ccomp.start();

    cv::Mat out_mat;
    int64_t out_meta = 0;
    int64_t out_timestamp = 0;
    int64_t expected_counter = 0;
    int64_t prev_timestamp = -1;

    while (ccomp.pull(cv::gout(out_mat, out_meta, out_timestamp))) {
        EXPECT_EQ(expected_counter, out_meta);
        ++expected_counter;

        EXPECT_NE(prev_timestamp, out_timestamp);
        prev_timestamp = out_timestamp;
    }
}

TEST(GraphMeta, Streaming_AccessDesync) {
    cv::GMat in;
    cv::GOpaque<int64_t> out1 = cv::gapi::streaming::seq_id(in);
    cv::GOpaque<int64_t> out2 = cv::gapi::streaming::timestamp(in);
    cv::GMat             out3 = cv::gapi::blur(in, cv::Size(3,3));

    cv::GMat tmp = cv::gapi::streaming::desync(in);
    cv::GScalar mean = cv::gapi::mean(tmp);
    cv::GOpaque<int64_t> out4 = cv::gapi::streaming::seq_id(mean);
    cv::GOpaque<int64_t> out5 = cv::gapi::streaming::timestamp(mean);
    cv::GComputation graph(cv::GIn(in), cv::GOut(out1, out2, out3, out4, out5));

    auto ccomp = graph.compileStreaming();
    const auto path = findDataFile("cv/video/768x576.avi");
    try {
        ccomp.setSource<cv::gapi::wip::GCaptureSource>(path);
    } catch(...) {
        throw SkipTestException("Video file can not be opened");
    }
    ccomp.start();

    cv::optional<int64_t> out_sync_id;
    cv::optional<int64_t> out_sync_ts;
    cv::optional<cv::Mat> out_sync_mat;

    cv::optional<int64_t> out_desync_id;
    cv::optional<int64_t> out_desync_ts;

    std::unordered_set<int64_t> sync_ids;
    std::unordered_set<int64_t> desync_ids;

    while (ccomp.pull(cv::gout(out_sync_id, out_sync_ts, out_sync_mat,
                               out_desync_id, out_desync_ts))) {
        if (out_sync_id.has_value()) {
            CV_Assert(out_sync_ts.has_value());
            CV_Assert(out_sync_mat.has_value());
            sync_ids.insert(out_sync_id.value());
        }
        if (out_desync_id.has_value()) {
            CV_Assert(out_desync_ts.has_value());
            desync_ids.insert(out_desync_id.value());
        }
    }
    // Visually report that everything is really ok
    std::cout << sync_ids.size() << " vs " << desync_ids.size() << std::endl;

    // Desync path should generate less objects than the synchronized one
    EXPECT_GE(sync_ids.size(), desync_ids.size());

    // ..but all desynchronized IDs must be present in the synchronized set
    for (auto &&d_id : desync_ids) {
        EXPECT_TRUE(sync_ids.count(d_id) > 0);
    }
}

} // namespace opencv_test
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
- `opencv2/gapi/streaming/cap.hpp`
- `test_precomp.hpp`
- `opencv2/gapi/streaming/meta.hpp`
- `tuple`
- `unordered_set`


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

