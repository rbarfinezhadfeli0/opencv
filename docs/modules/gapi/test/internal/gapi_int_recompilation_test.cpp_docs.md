# Documentation for `modules/gapi/test/internal/gapi_int_recompilation_test.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/internal/gapi_int_recompilation_test.cpp`
- **File Name**: `gapi_int_recompilation_test.cpp`
- **File Size**: 7,951 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/internal/gapi_int_recompilation_test.cpp](../../../../modules/gapi/test/internal/gapi_int_recompilation_test.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/internal` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "../test_precomp.hpp"
#include "../common/gapi_tests_common.hpp"
#include "api/gcomputation_priv.hpp"

#include <opencv2/gapi/fluid/gfluidkernel.hpp>
#include <opencv2/gapi/fluid/core.hpp>
#include <opencv2/gapi/fluid/imgproc.hpp>

namespace opencv_test
{

TEST(GComputationCompile, NoRecompileWithSameMeta)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    cv::Mat in_mat1 = cv::Mat::eye  (32, 32, CV_8UC1);
    cv::Mat in_mat2 = cv::Mat::zeros(32, 32, CV_8UC1);
    cv::Mat out_mat;

    cc.apply(in_mat1, out_mat);
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are actually the same unique executable
    EXPECT_EQ(&comp1.priv(), &comp2.priv());
}

TEST(GComputationCompile, NoRecompileWithWrongMeta)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    cv::Mat in_mat1 = cv::Mat::eye  (32, 32, CV_8UC1);
    cv::Mat in_mat2 = cv::Mat::zeros(32, 32, CV_8UC1);
    cv::Mat out_mat;

    cc.apply(in_mat1, out_mat);
    auto comp1 = cc.priv().m_lastCompiled;

    EXPECT_THROW(cc.apply(cv::gin(cv::Scalar(128)), cv::gout(out_mat)), std::logic_error);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are actually the same unique executable
    EXPECT_EQ(&comp1.priv(), &comp2.priv());
}

TEST(GComputationCompile, RecompileWithDifferentMeta)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    cv::Mat in_mat1 = cv::Mat::eye  (32, 32, CV_8UC1);
    cv::Mat in_mat2 = cv::Mat::zeros(64, 64, CV_32F);
    cv::Mat out_mat;

    cc.apply(in_mat1, out_mat);
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are different
    EXPECT_NE(&comp1.priv(), &comp2.priv());
}

TEST(GComputationCompile, FluidReshapeWithDifferentDims)
{
    cv::GMat in;
    cv::GComputation cc(in, in+in);

    cv::Mat in_mat1 = cv::Mat::eye  (32, 32, CV_8UC1);
    cv::Mat in_mat2 = cv::Mat::zeros(64, 64, CV_8UC1);
    cv::Mat out_mat;

    cc.apply(in_mat1, out_mat, cv::compile_args(cv::gapi::core::fluid::kernels()));
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are actually the same unique executable
    EXPECT_EQ(&comp1.priv(), &comp2.priv());
}

TEST(GComputationCompile, FluidReshapeResizeDownScale)
{
    cv::Size szOut(4, 4);
    cv::GMat in;
    cv::GComputation cc(in, cv::gapi::resize(in, szOut));

    cv::Mat in_mat1( 8,  8, CV_8UC3);
    cv::Mat in_mat2(16, 16, CV_8UC3);
    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::Mat out_mat1, out_mat2;

    cc.apply(in_mat1, out_mat1, cv::compile_args(cv::gapi::imgproc::fluid::kernels()));
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat2);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are actually the same unique executable
    EXPECT_EQ(&comp1.priv(), &comp2.priv());

    cv::Mat cv_out_mat1, cv_out_mat2;
    cv::resize(in_mat1, cv_out_mat1, szOut);
    cv::resize(in_mat2, cv_out_mat2, szOut);
    // Fluid's and OpenCV's resizes aren't bit exact.
    // So 1 is here because it is max difference between them.
    EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat1, cv_out_mat1));
    EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat2, cv_out_mat2));
}

TEST(GComputationCompile, FluidReshapeSwitchToUpscaleFromDownscale)
{
    cv::Size szOut(4, 4);
    cv::GMat in;
    cv::GComputation cc(in, cv::gapi::resize(in, szOut));

    cv::Mat in_mat1( 8,  8, CV_8UC3);
    cv::Mat in_mat2( 2,  2, CV_8UC3);
    cv::Mat in_mat3(16, 16, CV_8UC3);
    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat3, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::Mat out_mat1, out_mat2, out_mat3;

    cc.apply(in_mat1, out_mat1, cv::compile_args(cv::gapi::imgproc::fluid::kernels()));
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat2);
    auto comp2 = cc.priv().m_lastCompiled;

    cc.apply(in_mat3, out_mat3);
    auto comp3 = cc.priv().m_lastCompiled;

    EXPECT_EQ(&comp1.priv(), &comp2.priv());
    EXPECT_EQ(&comp1.priv(), &comp3.priv());

    cv::Mat cv_out_mat1, cv_out_mat2, cv_out_mat3;
    cv::resize(in_mat1, cv_out_mat1, szOut);
    cv::resize(in_mat2, cv_out_mat2, szOut);
    cv::resize(in_mat3, cv_out_mat3, szOut);
    // Fluid's and OpenCV's Resizes aren't bit exact.
    // So 1 is here because it is max difference between them.
    EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat1, cv_out_mat1));
    EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat2, cv_out_mat2));
    EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat3, cv_out_mat3));
}

TEST(GComputationCompile, ReshapeBlur)
{
    cv::Size kernelSize{3, 3};
    cv::GMat in;
    cv::GComputation cc(in, cv::gapi::blur(in, kernelSize));

    cv::Mat in_mat1( 8,  8, CV_8UC1);
    cv::Mat in_mat2(16, 16, CV_8UC1);
    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::Mat out_mat1, out_mat2;

    cc.apply(in_mat1, out_mat1, cv::compile_args(cv::gapi::imgproc::fluid::kernels()));
    auto comp1 = cc.priv().m_lastCompiled;

    cc.apply(in_mat2, out_mat2);
    auto comp2 = cc.priv().m_lastCompiled;

    // Both compiled objects are actually the same unique executable
    EXPECT_EQ(&comp1.priv(), &comp2.priv());

    cv::Mat cv_out_mat1, cv_out_mat2;
    cv::blur(in_mat1, cv_out_mat1, kernelSize);
    cv::blur(in_mat2, cv_out_mat2, kernelSize);

    EXPECT_EQ(0, cvtest::norm(out_mat1, cv_out_mat1, NORM_INF));
    EXPECT_EQ(0, cvtest::norm(out_mat2, cv_out_mat2, NORM_INF));
}

TEST(GComputationCompile, ReshapeRois)
{
    cv::Size kernelSize{3, 3};
    cv::Size szOut(8, 8);
    cv::GMat in;
    auto blurred = cv::gapi::blur(in, kernelSize);
    cv::GComputation cc(in, cv::gapi::resize(blurred, szOut));

    cv::Mat first_in_mat(8, 8, CV_8UC3);
    cv::randn(first_in_mat, cv::Scalar::all(127), cv::Scalar::all(40.f));
    cv::Mat first_out_mat;
    auto fluidKernels = cv::gapi::combine(gapi::imgproc::fluid::kernels(),
                                          gapi::core::fluid::kernels());
    cc.apply(first_in_mat, first_out_mat, cv::compile_args(fluidKernels));
    auto first_comp = cc.priv().m_lastCompiled;

    constexpr int niter = 4;
    for (int i = 0; i < niter; i++)
    {
        int width  = 4 + 2*i;
        int height = width;
        cv::Mat in_mat(width, height, CV_8UC3);
        cv::randn(in_mat, cv::Scalar::all(127), cv::Scalar::all(40.f));
        cv::Mat out_mat = cv::Mat::zeros(szOut, CV_8UC3);

        int x = 0;
        int y = szOut.height * i / niter;
        int roiW = szOut.width;
        int roiH = szOut.height / niter;
        cv::Rect roi{x, y, roiW, roiH};

        cc.apply(in_mat, out_mat, cv::compile_args(cv::GFluidOutputRois{{roi}}));
        auto comp = cc.priv().m_lastCompiled;

        EXPECT_EQ(&first_comp.priv(), &comp.priv());

        cv::Mat blur_mat, cv_out_mat;
        cv::blur(in_mat, blur_mat, kernelSize);
        cv::resize(blur_mat, cv_out_mat, szOut);
        // Fluid's and OpenCV's resizes aren't bit exact.
        // So 1 is here because it is max difference between them.
        EXPECT_TRUE(Tolerance_FloatRel_IntAbs(1e-5, 1).to_compare_f()(out_mat(roi), cv_out_mat(roi)));
    }
}

} // opencv_test
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
- `opencv2/gapi/fluid/core.hpp`
- `../test_precomp.hpp`
- `opencv2/gapi/fluid/imgproc.hpp`
- `opencv2/gapi/fluid/gfluidkernel.hpp`
- `api/gcomputation_priv.hpp`
- `../common/gapi_tests_common.hpp`


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

