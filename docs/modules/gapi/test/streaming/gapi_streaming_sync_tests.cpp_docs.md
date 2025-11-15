# Documentation for `modules/gapi/test/streaming/gapi_streaming_sync_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/streaming/gapi_streaming_sync_tests.cpp`
- **File Name**: `gapi_streaming_sync_tests.cpp`
- **File Size**: 6,908 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/streaming/gapi_streaming_sync_tests.cpp](../../../../modules/gapi/test/streaming/gapi_streaming_sync_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include "../test_precomp.hpp"

#include <opencv2/gapi/streaming/cap.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/fluid/imgproc.hpp>
#include <opencv2/gapi/streaming/cap.hpp>
#include <opencv2/gapi/streaming/sync.hpp>

namespace opencv_test {
namespace {

using ts_t = int64_t;
using ts_vec = std::vector<ts_t>;
using cv::gapi::streaming::sync_policy;

ts_t calcLeastCommonMultiple(const ts_vec& values) {
    ts_t res = *std::max_element(values.begin(), values.end());
    auto isDivisor = [&](ts_t v) { return res % v == 0; };
    while(!std::all_of(values.begin(), values.end(), isDivisor)) {
        res++;
    }
    return res;
}

struct TimestampGenerationParams {
    const ts_vec frame_times;
    sync_policy policy;
    ts_t end_time;
    TimestampGenerationParams(const ts_vec& ft, sync_policy sp, ts_t et = 25)
        : frame_times(ft), policy(sp), end_time(et) {
    }
};

class MultiFrameSource {
    class SingleSource : public cv::gapi::wip::IStreamSource {
        MultiFrameSource& m_source;
        std::size_t m_idx;
    public:
        SingleSource(MultiFrameSource& s, std::size_t idx)
            : m_source(s)
            , m_idx(idx)
        {}
        virtual bool pull(cv::gapi::wip::Data& data) {
            return m_source.pull(data, m_idx);
        }
        virtual GMetaArg descr_of() const { return GMetaArg{m_source.desc()}; }
    };

    TimestampGenerationParams p;
    ts_vec m_current_times;
    cv::Mat m_mat;

public:
    MultiFrameSource(const TimestampGenerationParams& params)
        : p(params)
        , m_current_times(p.frame_times.size(), 0u)
        , m_mat(8, 8, CV_8UC1) {
    }

    bool pull(cv::gapi::wip::Data& data, std::size_t idx) {
        cv::randn(m_mat, 127, 32);
        GAPI_Assert(idx < p.frame_times.size());
        m_current_times[idx] += p.frame_times[idx];
        if (m_current_times[idx] >= p.end_time) {
            return false;
        }
        data = m_mat.clone();
        data.meta[cv::gapi::streaming::meta_tag::timestamp] = m_current_times[idx];
        return true;
    }

    cv::gapi::wip::IStreamSource::Ptr getSource(std::size_t idx) {
        return cv::gapi::wip::IStreamSource::Ptr{new SingleSource(*this, idx)};
    }

    GMatDesc desc() const { return cv::descr_of(m_mat); }
};

class TimestampChecker {
    TimestampGenerationParams p;
    ts_t m_synced_time = 0u;
    ts_t m_synced_frame_time = 0u;
public:
    TimestampChecker(const TimestampGenerationParams& params)
        : p(params)
        , m_synced_frame_time(calcLeastCommonMultiple(p.frame_times)) {
    }

    void checkNext(const ts_vec& timestamps) {
        if (p.policy == sync_policy::dont_sync) {
            // don't check timestamps if the policy is dont_sync
            return;
        }
        m_synced_time += m_synced_frame_time;
        for (const auto& ts : timestamps) {
            EXPECT_EQ(m_synced_time, ts);
        }
    }

    std::size_t nFrames() const {
        auto frame_time = p.policy == sync_policy::dont_sync
                          ? *std::max_element(p.frame_times.begin(), p.frame_times.end())
                          : m_synced_frame_time;
        auto n_frames = p.end_time / frame_time;
        GAPI_Assert(n_frames > 0u);
        return (std::size_t)n_frames;
    }
};

struct TimestampSyncTest : public ::testing::TestWithParam<sync_policy> {
    void run(cv::GProtoInputArgs&& ins, cv::GProtoOutputArgs&& outs,
             const ts_vec& frame_times) {
        auto video_in_n = frame_times.size();
        GAPI_Assert(video_in_n <= ins.m_args.size());
        // Assume that all remaining inputs are const
        auto const_in_n = ins.m_args.size() - video_in_n;
        auto out_n = outs.m_args.size();
        auto policy = GetParam();
        TimestampGenerationParams ts_params(frame_times, policy);
        MultiFrameSource source(ts_params);

        GRunArgs gins;
        for (std::size_t i = 0; i < video_in_n; i++) {
            gins += cv::gin(source.getSource(i));
        }
        auto desc = source.desc();
        cv::Mat const_mat = cv::Mat::eye(desc.size.height,
                                         desc.size.width,
                                         CV_MAKE_TYPE(desc.depth, desc.chan));
        for (std::size_t i = 0; i < const_in_n; i++) {
            gins += cv::gin(const_mat);
        }
        ts_vec out_timestamps(out_n);
        cv::GRunArgsP gouts{};
        for (auto& t : out_timestamps) {
            gouts += cv::gout(t);
        }

        auto pipe = cv::GComputation(std::move(ins), std::move(outs))
                    .compileStreaming(cv::compile_args(policy));

        pipe.setSource(std::move(gins));
        pipe.start();

        std::size_t frames = 0u;
        TimestampChecker checker(ts_params);
        while(pipe.pull(std::move(gouts))) {
            checker.checkNext(out_timestamps);
            frames++;
        }

        EXPECT_EQ(checker.nFrames(), frames);
    }
};

} // anonymous namespace

TEST_P(TimestampSyncTest, Basic)
{
    cv::GMat in1, in2;
    auto out = cv::gapi::add(in1, in2);
    auto ts = cv::gapi::streaming::timestamp(out);

    run(cv::GIn(in1, in2), cv::GOut(ts), {1,2});
}

TEST_P(TimestampSyncTest, ThreeInputs)
{
    cv::GMat in1, in2, in3;
    auto tmp = cv::gapi::add(in1, in2);
    auto out = cv::gapi::add(tmp, in3);
    auto ts = cv::gapi::streaming::timestamp(out);

    run(cv::GIn(in1, in2, in3), cv::GOut(ts), {2,4,3});
}

TEST_P(TimestampSyncTest, TwoOutputs)
{
    cv::GMat in1, in2, in3;
    auto out1 = cv::gapi::add(in1, in3);
    auto out2 = cv::gapi::add(in2, in3);
    auto ts1 = cv::gapi::streaming::timestamp(out1);
    auto ts2 = cv::gapi::streaming::timestamp(out2);

    run(cv::GIn(in1, in2, in3), cv::GOut(ts1, ts2), {1,4,2});
}

TEST_P(TimestampSyncTest, ConstInput)
{
    cv::GMat in1, in2, in3;
    auto out1 = cv::gapi::add(in1, in3);
    auto out2 = cv::gapi::add(in2, in3);
    auto ts1 = cv::gapi::streaming::timestamp(out1);
    auto ts2 = cv::gapi::streaming::timestamp(out2);

    run(cv::GIn(in1, in2, in3), cv::GOut(ts1, ts2), {1,2});
}

TEST_P(TimestampSyncTest, ChangeSource)
{
    cv::GMat in1, in2, in3;
    auto out1 = cv::gapi::add(in1, in3);
    auto out2 = cv::gapi::add(in2, in3);
    auto ts1 = cv::gapi::streaming::timestamp(out1);
    auto ts2 = cv::gapi::streaming::timestamp(out2);

    run(cv::GIn(in1, in2, in3), cv::GOut(ts1, ts2), {1,2});
    run(cv::GIn(in1, in2, in3), cv::GOut(ts1, ts2), {1,2});
}

INSTANTIATE_TEST_CASE_P(InputSynchronization, TimestampSyncTest,
                        Values(sync_policy::dont_sync,
                               sync_policy::drop));
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

### Classes and Structures

- **TimestampSyncTest**: A class/struct defined in this file
- **TimestampGenerationParams**: A class/struct defined in this file
- **MultiFrameSource**: A class/struct defined in this file
- **SingleSource**: A class/struct defined in this file
- **TimestampChecker**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/streaming/sync.hpp`
- `opencv2/gapi/streaming/cap.hpp`
- `../test_precomp.hpp`
- `opencv2/gapi/fluid/imgproc.hpp`
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

