# Documentation for `modules/gapi/test/gapi_frame_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_frame_tests.cpp`
- **File Name**: `gapi_frame_tests.cpp`
- **File Size**: 8,363 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_frame_tests.cpp](../../../modules/gapi/test/gapi_frame_tests.cpp)

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

#include "test_precomp.hpp"

#include <opencv2/gapi/media.hpp>

////////////////////////////////////////////////////////////////////////////////
// cv::GFrame tests

namespace opencv_test {

G_API_OP(GBlurFrame, <GMat(GFrame)>, "test.blur_frame") {
    static GMatDesc outMeta(GFrameDesc in) {
        return cv::GMatDesc(CV_8U,3,in.size);
    }
};

GAPI_OCV_KERNEL(OCVBlurFrame, GBlurFrame) {
    static void run(const cv::MediaFrame &in, cv::Mat& out) {
        GAPI_Assert(in.desc().fmt == cv::MediaFormat::BGR);
        cv::MediaFrame::View view = in.access(cv::MediaFrame::Access::R);
        cv::blur(cv::Mat(in.desc().size, CV_8UC3, view.ptr[0], view.stride[0]),
                 out,
                 cv::Size{3,3});
    }
};

G_API_OP(GBlurFrameGray, <GMat(GFrame)>, "test.blur_frame_gray") {
    static GMatDesc outMeta(GFrameDesc in) {
        return cv::GMatDesc(CV_8U, 1, in.size);
    }
};

GAPI_OCV_KERNEL(OCVBlurFrameGray, GBlurFrameGray) {
    static void run(const cv::MediaFrame & in, cv::Mat & out) {
        GAPI_Assert(in.desc().fmt == cv::MediaFormat::GRAY);
        cv::MediaFrame::View view = in.access(cv::MediaFrame::Access::R);
        cv::blur(cv::Mat(in.desc().size, CV_8UC1, view.ptr[0], view.stride[0]),
        out,
        cv::Size{ 3,3 });
    }
};


////////////////////////////////////////////////////////////////////////////////
// cv::MediaFrame tests
namespace {
class TestMediaBGR final: public cv::MediaFrame::IAdapter {
    cv::Mat m_mat;
    using Cb = cv::MediaFrame::View::Callback;
    Cb m_cb;

public:
    explicit TestMediaBGR(cv::Mat m, Cb cb = [](){})
        : m_mat(m), m_cb(cb) {
    }
    cv::GFrameDesc meta() const override {
        return cv::GFrameDesc{cv::MediaFormat::BGR, cv::Size(m_mat.cols, m_mat.rows)};
    }
    cv::MediaFrame::View access(cv::MediaFrame::Access) override {
        cv::MediaFrame::View::Ptrs pp = { m_mat.ptr(), nullptr, nullptr, nullptr };
        cv::MediaFrame::View::Strides ss = { m_mat.step, 0u, 0u, 0u };
        return cv::MediaFrame::View(std::move(pp), std::move(ss), Cb{m_cb});
    }
};

class TestMediaNV12 final: public cv::MediaFrame::IAdapter {
    cv::Mat m_y;
    cv::Mat m_uv;
public:
    TestMediaNV12(cv::Mat y, cv::Mat uv) : m_y(y), m_uv(uv) {
    }
    cv::GFrameDesc meta() const override {
        return cv::GFrameDesc{cv::MediaFormat::NV12, cv::Size(m_y.cols, m_y.rows)};
    }
    cv::MediaFrame::View access(cv::MediaFrame::Access) override {
        cv::MediaFrame::View::Ptrs pp = {
            m_y.ptr(), m_uv.ptr(), nullptr, nullptr
        };
        cv::MediaFrame::View::Strides ss = {
            m_y.step, m_uv.step, 0u, 0u
        };
        return cv::MediaFrame::View(std::move(pp), std::move(ss));
    }
};

class TestMediaGray final : public cv::MediaFrame::IAdapter {
    cv::Mat m_mat;
    using Cb = cv::MediaFrame::View::Callback;
    Cb m_cb;

public:
    explicit TestMediaGray(cv::Mat m, Cb cb = []() {})
        : m_mat(m), m_cb(cb) {
    }
    cv::GFrameDesc meta() const override {
        return cv::GFrameDesc{ cv::MediaFormat::GRAY, cv::Size(m_mat.cols, m_mat.rows) };
    }
    cv::MediaFrame::View access(cv::MediaFrame::Access) override {
        cv::MediaFrame::View::Ptrs pp = { m_mat.ptr(), nullptr, nullptr, nullptr };
        cv::MediaFrame::View::Strides ss = { m_mat.step, 0u, 0u, 0u };
        return cv::MediaFrame::View(std::move(pp), std::move(ss), Cb{ m_cb });
    }
};

} // anonymous namespace

struct MediaFrame_Test: public ::testing::Test {
    using M = cv::Mat;
    using MF = cv::MediaFrame;
    MF frame;
};

struct MediaFrame_BGR: public MediaFrame_Test {
    M bgr;
    MediaFrame_BGR()
        : bgr(M::eye(240, 320, CV_8UC3)) {
        cv::randn(bgr, cv::Scalar::all(127.0f), cv::Scalar::all(40.f));
        frame = MF::Create<TestMediaBGR>(bgr);
    }
};

TEST_F(MediaFrame_BGR, Meta) {
    auto meta = frame.desc();
    EXPECT_EQ(cv::MediaFormat::BGR, meta.fmt);
    EXPECT_EQ(cv::Size(320,240),    meta.size);
}

TEST_F(MediaFrame_BGR, Access) {
    cv::MediaFrame::View view1 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(bgr.ptr(), view1.ptr[0]);
    EXPECT_EQ(bgr.step,  view1.stride[0]);

    cv::MediaFrame::View view2 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(bgr.ptr(), view2.ptr[0]);
    EXPECT_EQ(bgr.step,  view2.stride[0]);
}

TEST_F(MediaFrame_BGR, Input) {
    // Run the OpenCV code
    cv::Mat out_mat_ocv, out_mat_gapi;
    cv::blur(bgr, out_mat_ocv, cv::Size{3,3});

    // Run the G-API code
    cv::GFrame in;
    cv::GMat out = GBlurFrame::on(in);
    cv::GComputation(cv::GIn(in), cv::GOut(out))
        .apply(cv::gin(frame),
               cv::gout(out_mat_gapi),
               cv::compile_args(cv::gapi::kernels<OCVBlurFrame>()));

    // Compare
    EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
}

struct MediaFrame_Gray : public MediaFrame_Test {
    M gray;
    MediaFrame_Gray()
        : gray(M::eye(240, 320, CV_8UC1)) {
        cv::randn(gray, cv::Scalar::all(127.0f), cv::Scalar::all(40.f));
        frame = MF::Create<TestMediaGray>(gray);
    }
};

TEST_F(MediaFrame_Gray, Meta) {
    auto meta = frame.desc();
    EXPECT_EQ(cv::MediaFormat::GRAY, meta.fmt);
    EXPECT_EQ(cv::Size(320, 240), meta.size);
}

TEST_F(MediaFrame_Gray, Access) {
    cv::MediaFrame::View view1 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(gray.ptr(), view1.ptr[0]);
    EXPECT_EQ(gray.step, view1.stride[0]);

    cv::MediaFrame::View view2 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(gray.ptr(), view2.ptr[0]);
    EXPECT_EQ(gray.step, view2.stride[0]);
}

TEST_F(MediaFrame_Gray, Input) {
    // Run the OpenCV code
    cv::Mat out_mat_ocv, out_mat_gapi;
    cv::blur(gray, out_mat_ocv, cv::Size{ 3,3 });

    // Run the G-API code
    cv::GFrame in;
    cv::GMat out = GBlurFrameGray::on(in);
    cv::GComputation(cv::GIn(in), cv::GOut(out))
        .apply(cv::gin(frame),
            cv::gout(out_mat_gapi),
            cv::compile_args(cv::gapi::kernels<OCVBlurFrameGray>()));

    // Compare
    EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
}


struct MediaFrame_NV12: public MediaFrame_Test {
    cv::Size sz;
    cv::Mat buf, y, uv;
    MediaFrame_NV12()
        : sz {320, 240}
        , buf(M::eye(sz.height*3/2, sz.width, CV_8UC1))
        , y  (buf.rowRange(0, sz.height))
        , uv (buf.rowRange(sz.height, sz.height*3/2)) {
        frame = MF::Create<TestMediaNV12>(y, uv);
    }
};

TEST_F(MediaFrame_NV12, Meta) {
    auto meta = frame.desc();
    EXPECT_EQ(cv::MediaFormat::NV12, meta.fmt);
    EXPECT_EQ(cv::Size(320,240),     meta.size);
}

TEST_F(MediaFrame_NV12, Access) {
    cv::MediaFrame::View view1 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(y. ptr(), view1.ptr   [0]);
    EXPECT_EQ(y. step,  view1.stride[0]);
    EXPECT_EQ(uv.ptr(), view1.ptr   [1]);
    EXPECT_EQ(uv.step,  view1.stride[1]);

    cv::MediaFrame::View view2 = frame.access(cv::MediaFrame::Access::R);
    EXPECT_EQ(y. ptr(), view2.ptr   [0]);
    EXPECT_EQ(y. step,  view2.stride[0]);
    EXPECT_EQ(uv.ptr(), view2.ptr   [1]);
    EXPECT_EQ(uv.step,  view2.stride[1]);
}

TEST(MediaFrame, Callback) {
    int counter = 0;
    cv::Mat bgr = cv::Mat::eye(240, 320, CV_8UC3);
    cv::MediaFrame frame = cv::MediaFrame::Create<TestMediaBGR>(bgr, [&counter](){counter++;});

    // Test that the callback (in this case, incrementing the counter)
    // is called only on View destruction.
    EXPECT_EQ(0, counter);
    {
        cv::MediaFrame::View v1 = frame.access(cv::MediaFrame::Access::R);
        EXPECT_EQ(0, counter);
    }
    EXPECT_EQ(1, counter);
    {
        cv::MediaFrame::View v1 = frame.access(cv::MediaFrame::Access::R);
        EXPECT_EQ(1, counter);
        cv::MediaFrame::View v2 = frame.access(cv::MediaFrame::Access::W);
        EXPECT_EQ(1, counter);
    }
    EXPECT_EQ(3, counter);
}

TEST(MediaFrame, blobParams) {
    cv::Mat bgr = cv::Mat::eye(240, 320, CV_8UC3);
    cv::MediaFrame frame = cv::MediaFrame::Create<TestMediaBGR>(bgr);

    EXPECT_NO_THROW(frame.blobParams());
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

- **MediaFrame_Test**: A class/struct defined in this file
- **MediaFrame_NV12**: A class/struct defined in this file
- **TestMediaBGR**: A class/struct defined in this file
- **MediaFrame_BGR**: A class/struct defined in this file
- **TestMediaNV12**: A class/struct defined in this file
- **MediaFrame_Gray**: A class/struct defined in this file
- **TestMediaGray**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/media.hpp`
- `test_precomp.hpp`


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

