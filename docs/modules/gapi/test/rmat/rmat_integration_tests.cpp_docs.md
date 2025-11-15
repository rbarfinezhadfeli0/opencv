# Documentation for `modules/gapi/test/rmat/rmat_integration_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/rmat/rmat_integration_tests.cpp`
- **File Name**: `rmat_integration_tests.cpp`
- **File Size**: 6,049 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/rmat/rmat_integration_tests.cpp](../../../../modules/gapi/test/rmat/rmat_integration_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/rmat` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020 Intel Corporation

#include "../test_precomp.hpp"
#include <opencv2/gapi/rmat.hpp>
#include "rmat_test_common.hpp"

#include <opencv2/gapi/fluid/imgproc.hpp>
#include <opencv2/gapi/cpu/imgproc.hpp>

namespace opencv_test
{

// This test set takes RMat type as a template parameter and launces simple
// blur(isl1) -> blur(isl2) computation passing RMat as input, as output
// and both input and output
template<typename RMatAdapterT>
struct RMatIntTestBase {
    cv::Mat in_mat;
    cv::Mat out_mat;
    cv::Mat out_mat_ref;
    cv::GComputation comp;
    bool inCallbackCalled;
    bool outCallbackCalled;

    static constexpr int w = 8;
    static constexpr int h = 8;

    RMatIntTestBase()
        : in_mat(h, w, CV_8UC1)
        , out_mat(h, w, CV_8UC1)
        , out_mat_ref(h, w, CV_8UC1)
        , comp([](){
              cv::GMat in;
              auto tmp = cv::gapi::blur(in, {3,3});
              auto out = cv::gapi::blur(tmp, {3,3});
              cv::gapi::island("test", cv::GIn(in), cv::GOut(tmp));
              return cv::GComputation(in, out);
          })
        , inCallbackCalled(false)
        , outCallbackCalled(false) {
        cv::randu(in_mat, cv::Scalar::all(127), cv::Scalar::all(40));
    }

    void check() {
        comp.apply(in_mat, out_mat_ref);
        EXPECT_EQ(0, cvtest::norm(out_mat_ref, out_mat, NORM_INF));
    }

    RMat createRMat(cv::Mat& mat, bool& callbackCalled) {
        return {cv::make_rmat<RMatAdapterT>(mat, callbackCalled)};
    }
};

template<typename RMatAdapterT>
struct RMatIntTest : public RMatIntTestBase<RMatAdapterT>
{
    template<typename In, typename Out>
    void run(const In& in, Out& out, cv::GCompileArgs&& compile_args) {
        for (int i = 0; i < 2; i++) {
            EXPECT_FALSE(this->inCallbackCalled);
            EXPECT_FALSE(this->outCallbackCalled);
            auto compile_args_copy = compile_args;
            this->comp.apply(cv::gin(in), cv::gout(out), std::move(compile_args_copy));
            EXPECT_FALSE(this->inCallbackCalled);
            if (std::is_same<RMat,Out>::value) {
                EXPECT_TRUE(this->outCallbackCalled);
            } else {
                EXPECT_FALSE(this->outCallbackCalled);
            }
            this->outCallbackCalled = false;
        }
        this->check();
    }
};

template<typename RMatAdapterT>
struct RMatIntTestStreaming : public RMatIntTestBase<RMatAdapterT>
{
    template <typename M>
    cv::GMatDesc getDesc(const M& m) { return cv::descr_of(m); }

    void checkOutput(const cv::Mat&) { this->check(); }

    void checkOutput(const RMat& rm) {
        auto view = rm.access(RMat::Access::R);
        this->out_mat = cv::Mat(view.size(), view.type(), view.ptr());
        this->check();
    }

    template<typename In, typename Out>
    void run(const In& in, Out& out, cv::GCompileArgs&& compile_args) {
        auto sc = this->comp.compileStreaming(getDesc(in), std::move(compile_args));

        sc.setSource(cv::gin(in));
        sc.start();

        std::size_t frame = 0u;
        constexpr std::size_t num_frames = 10u;
        EXPECT_FALSE(this->inCallbackCalled);
        EXPECT_FALSE(this->outCallbackCalled);
        while (sc.pull(cv::gout(out)) && frame < num_frames) {
            frame++;
            this->checkOutput(out);
            EXPECT_FALSE(this->inCallbackCalled);
            EXPECT_FALSE(this->outCallbackCalled);
        }
        EXPECT_EQ(num_frames, frame);
    }
};

struct OcvKernels {
    cv::GKernelPackage kernels() { return cv::gapi::imgproc::cpu::kernels(); }
};
struct FluidKernels {
    cv::GKernelPackage kernels() { return cv::gapi::imgproc::fluid::kernels(); }
};

struct RMatIntTestCpuRef : public
    RMatIntTest<RMatAdapterRef>, OcvKernels {};
struct RMatIntTestCpuCopy : public
    RMatIntTest<RMatAdapterCopy>, OcvKernels {};
struct RMatIntTestCpuRefStreaming : public
    RMatIntTestStreaming<RMatAdapterRef>, OcvKernels  {};
struct RMatIntTestCpuCopyStreaming : public
    RMatIntTestStreaming<RMatAdapterCopy>, OcvKernels {};
struct RMatIntTestCpuRefFluid : public
    RMatIntTest<RMatAdapterRef>, FluidKernels {};
struct RMatIntTestCpuCopyFluid : public
    RMatIntTest<RMatAdapterCopy>, FluidKernels {};
struct RMatIntTestCpuRefStreamingFluid : public
    RMatIntTestStreaming<RMatAdapterRef>, FluidKernels {};
struct RMatIntTestCpuCopyStreamingFluid : public
    RMatIntTestStreaming<RMatAdapterCopy>, FluidKernels {};

template<typename T>
struct RMatIntTypedTest : public ::testing::Test, public T {};

using RMatIntTestTypes = ::testing::Types< RMatIntTestCpuRef
                                         , RMatIntTestCpuCopy
                                         , RMatIntTestCpuRefStreaming
                                         , RMatIntTestCpuCopyStreaming
                                         , RMatIntTestCpuRefFluid
                                         , RMatIntTestCpuCopyFluid
                                         , RMatIntTestCpuRefStreamingFluid
                                         , RMatIntTestCpuCopyStreamingFluid
                                         >;

TYPED_TEST_CASE(RMatIntTypedTest, RMatIntTestTypes);

TYPED_TEST(RMatIntTypedTest, In) {
    auto in_rmat = this->createRMat(this->in_mat, this->inCallbackCalled);
    this->run(in_rmat, this->out_mat, cv::compile_args(this->kernels()));
}

TYPED_TEST(RMatIntTypedTest, Out) {
    auto out_rmat = this->createRMat(this->out_mat, this->outCallbackCalled);
    this->run(this->in_mat, out_rmat, cv::compile_args(this->kernels()));
}

TYPED_TEST(RMatIntTypedTest, InOut) {
    auto  in_rmat = this->createRMat(this->in_mat, this->inCallbackCalled);
    auto out_rmat = this->createRMat(this->out_mat, this->outCallbackCalled);
    this->run(in_rmat, out_rmat, cv::compile_args(this->kernels()));
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

### Classes and Structures

- **RMatIntTestBase**: A class/struct defined in this file
- **RMatIntTestCpuCopyFluid**: A class/struct defined in this file
- **RMatIntTestCpuRefFluid**: A class/struct defined in this file
- **RMatIntTypedTest**: A class/struct defined in this file
- **RMatIntTest**: A class/struct defined in this file
- **RMatIntTestStreaming**: A class/struct defined in this file
- **RMatIntTestCpuCopy**: A class/struct defined in this file
- **OcvKernels**: A class/struct defined in this file
- **RMatIntTestCpuRefStreaming**: A class/struct defined in this file
- **RMatIntTestCpuCopyStreamingFluid**: A class/struct defined in this file
- **FluidKernels**: A class/struct defined in this file
- **RMatIntTestCpuRef**: A class/struct defined in this file
- **RMatIntTestCpuRefStreamingFluid**: A class/struct defined in this file
- **RMatIntTestCpuCopyStreaming**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/rmat.hpp`
- `../test_precomp.hpp`
- `opencv2/gapi/cpu/imgproc.hpp`
- `rmat_test_common.hpp`
- `opencv2/gapi/fluid/imgproc.hpp`


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

