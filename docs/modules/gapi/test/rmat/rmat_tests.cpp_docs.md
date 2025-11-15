# Documentation for `modules/gapi/test/rmat/rmat_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/rmat/rmat_tests.cpp`
- **File Name**: `rmat_tests.cpp`
- **File Size**: 3,968 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/rmat/rmat_tests.cpp](../../../../modules/gapi/test/rmat/rmat_tests.cpp)

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

namespace opencv_test {
namespace {
void randomizeMat(cv::Mat& m) {
    auto ref = m.clone();
    while (cv::norm(m, ref, cv::NORM_INF) == 0) {
        cv::randu(m, cv::Scalar::all(127), cv::Scalar::all(40));
    }
}

template <typename RMatAdapterT>
struct RMatTest {
    using AdapterT = RMatAdapterT;
    RMatTest()
        : m_deviceMat(cv::Mat::zeros(8,8,CV_8UC1))
        , m_rmat(make_rmat<RMatAdapterT>(m_deviceMat, m_callbackCalled)) {
        randomizeMat(m_deviceMat);
        expectNoCallbackCalled();
    }

    RMat& rmat() { return m_rmat; }
    cv::Mat cloneDeviceMat() { return m_deviceMat.clone(); }
    void expectCallbackCalled() { EXPECT_TRUE(m_callbackCalled); }
    void expectNoCallbackCalled() { EXPECT_FALSE(m_callbackCalled); }

    void expectDeviceDataEqual(const cv::Mat& mat) {
        EXPECT_EQ(0, cv::norm(mat, m_deviceMat, NORM_INF));
    }
    void expectDeviceDataNotEqual(const cv::Mat& mat) {
        EXPECT_NE(0, cv::norm(mat, m_deviceMat, NORM_INF));
    }

private:
    cv::Mat m_deviceMat;
    bool m_callbackCalled = false;
    cv::RMat m_rmat;
};
} // anonymous namespace

template<typename T>
struct RMatTypedTest : public ::testing::Test, public T { using Type = T; };

using RMatTestTypes = ::testing::Types< RMatTest<RMatAdapterRef>
                                      , RMatTest<RMatAdapterCopy>
                                      >;

TYPED_TEST_CASE(RMatTypedTest, RMatTestTypes);

TYPED_TEST(RMatTypedTest, Smoke) {
    auto view = this->rmat().access(RMat::Access::R);
    auto matFromDevice = cv::Mat(view.size(), view.type(), view.ptr());
    EXPECT_TRUE(cv::descr_of(this->cloneDeviceMat()) == this->rmat().desc());
    this->expectDeviceDataEqual(matFromDevice);
}

static Mat asMat(RMat::View& view) {
    return Mat(view.size(), view.type(), view.ptr(), view.step());
}

TYPED_TEST(RMatTypedTest, BasicWorkflow) {
    {
        auto view = this->rmat().access(RMat::Access::R);
        this->expectDeviceDataEqual(asMat(view));
    }
    this->expectNoCallbackCalled();

    cv::Mat dataToWrite = this->cloneDeviceMat();
    randomizeMat(dataToWrite);
    this->expectDeviceDataNotEqual(dataToWrite);
    {
        auto view = this->rmat().access(RMat::Access::W);
        dataToWrite.copyTo(asMat(view));
    }
    this->expectCallbackCalled();
    this->expectDeviceDataEqual(dataToWrite);
}

TEST(RMat, TestEmptyAdapter) {
    RMat rmat;
    EXPECT_ANY_THROW(rmat.get<RMatAdapterCopy>());
}

TYPED_TEST(RMatTypedTest, CorrectAdapterCast) {
    using T = typename TestFixture::Type::AdapterT;
    EXPECT_NE(nullptr, this->rmat().template get<T>());
}

class DummyAdapter : public RMat::IAdapter {
    virtual RMat::View access(RMat::Access) override { return {}; }
    virtual cv::GMatDesc desc() const override { return {}; }
};

TYPED_TEST(RMatTypedTest, IncorrectAdapterCast) {
    EXPECT_EQ(nullptr, this->rmat().template get<DummyAdapter>());
}

class RMatAdapterForBackend : public RMat::IAdapter {
    int m_i;
public:
    RMatAdapterForBackend(int i) : m_i(i) {}
    virtual RMat::View access(RMat::Access) override { return {}; }
    virtual GMatDesc desc() const override { return {}; }
    int deviceSpecificData() const { return m_i; }
};

// RMat's usage scenario in the backend:
// we have some specific data hidden under RMat,
// test that we can obtain it via RMat.as<T>() method
TEST(RMat, UsageInBackend) {
    int i = 123456;
    auto rmat = cv::make_rmat<RMatAdapterForBackend>(i);

    auto adapter = rmat.get<RMatAdapterForBackend>();
    ASSERT_NE(nullptr, adapter);
    EXPECT_EQ(i, adapter->deviceSpecificData());
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

- **RMatTypedTest**: A class/struct defined in this file
- **DummyAdapter**: A class/struct defined in this file
- **RMatTest**: A class/struct defined in this file
- **RMatAdapterForBackend**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/rmat.hpp`
- `rmat_test_common.hpp`
- `../test_precomp.hpp`


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

