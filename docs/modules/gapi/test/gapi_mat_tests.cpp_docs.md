# Documentation for `modules/gapi/test/gapi_mat_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_mat_tests.cpp`
- **File Name**: `gapi_mat_tests.cpp`
- **File Size**: 3,419 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_mat_tests.cpp](../../../modules/gapi/test/gapi_mat_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2024 Intel Corporation


#include "test_precomp.hpp"

#include <opencv2/gapi/cpu/core.hpp>
#include <opencv2/gapi/ocl/core.hpp>
#include <opencv2/gapi/fluid/core.hpp>

namespace opencv_test
{
namespace
{
enum class KernelPackage: int
{
    OCV,
    OCL,
    FLUID,
};
std::ostream& operator<< (std::ostream &os, const KernelPackage &e)
{
    switch (e)
    {
#define _C(X) case KernelPackage::X: os << #X; break
        _C(OCV);
        _C(OCL);
        _C(FLUID);
#undef _C
    default: GAPI_Error("Unknown package");
    }
    return os;
}
} // namespace

struct GMatWithValue : public TestWithParam <KernelPackage> {
    cv::GKernelPackage getKernelPackage() {
        switch (GetParam()) {
        case KernelPackage::OCV: return cv::gapi::core::cpu::kernels();
        case KernelPackage::OCL: return cv::gapi::core::ocl::kernels();
        case KernelPackage::FLUID: return cv::gapi::core::fluid::kernels();
        default: GAPI_Error("Unknown package");
        }
    }
};

TEST_P(GMatWithValue, SingleIsland)
{
    cv::Size sz(2, 2);
    cv::Mat in_mat = cv::Mat::eye(sz, CV_8U);

    cv::GComputationT<cv::GMat(cv::GMat)> addEye([&](cv::GMat in) {
        return in + cv::GMat(cv::Mat::eye(sz, CV_8U));
    });

    cv::Mat out_mat;
    addEye.apply(in_mat, out_mat, cv::compile_args(cv::gapi::use_only{getKernelPackage()}));

    cv::Mat out_mat_ref = in_mat*2;
    EXPECT_EQ(0, cvtest::norm(out_mat, out_mat_ref, NORM_INF));
}

TEST_P(GMatWithValue, GraphWithNoInput)
{
    cv::Mat cval = cv::Mat::eye(cv::Size(2, 2), CV_8U);
    cv::GMat gval = cv::GMat(cval);
    cv::GMat out = cv::gapi::bitwise_not(gval);

    cv::Mat out_mat;
    cv::GComputation f(cv::GIn(), cv::GOut(out));

    // Compiling this isn't supported for now
    EXPECT_ANY_THROW(f.compile(cv::descr_of(cval),
                               cv::compile_args(cv::gapi::use_only{getKernelPackage()})));
}

INSTANTIATE_TEST_CASE_P(GAPI_GMat, GMatWithValue,
                        Values(KernelPackage::OCV,
                               KernelPackage::OCL,
                               KernelPackage::FLUID));

TEST(GAPI_MatWithValue, MultipleIslands)
{
    // This test employs a non-trivial island fusion process
    // as there's multiple backends in the graph

    cv::Size sz(2, 2);
    cv::Mat cval2 = cv::Mat::eye(sz, CV_8U) * 2;
    cv::Mat cval1 = cv::Mat::eye(sz, CV_8U);

    cv::GMat in;
    cv::GMat tmp = in  + cv::GMat(cval2); // Will be a Fluid operation
    cv::GMat out = tmp - cv::GMat(cval1); // Will be an OCV operation

    cv::GKernelPackage fluid_kernels = cv::gapi::core::fluid::kernels();
    cv::GKernelPackage opencv_kernels = cv::gapi::core::cpu::kernels();
    fluid_kernels.remove<cv::gapi::core::GSub>();
    opencv_kernels.remove<cv::gapi::core::GAdd>();
    auto kernels = cv::gapi::combine(fluid_kernels, opencv_kernels);

    cv::Mat in_mat = cv::Mat::zeros(sz, CV_8U);
    cv::Mat out_mat;
    auto cc = cv::GComputation(in, out)
        .compile(cv::descr_of(in_mat),
                 cv::compile_args(cv::gapi::use_only{kernels}));
    cc(cv::gin(in_mat), cv::gout(out_mat));

    EXPECT_EQ(0, cvtest::norm(out_mat, cv::Mat::eye(sz, CV_8U), NORM_INF));
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

- **GMatWithValue**: A class/struct defined in this file
- **KernelPackage**: A class/struct defined in this file

### Functions and Methods

- **_C()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/fluid/core.hpp`
- `opencv2/gapi/cpu/core.hpp`
- `opencv2/gapi/ocl/core.hpp`
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

