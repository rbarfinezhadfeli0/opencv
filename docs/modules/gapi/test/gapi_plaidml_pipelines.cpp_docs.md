# Documentation for `modules/gapi/test/gapi_plaidml_pipelines.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_plaidml_pipelines.cpp`
- **File Name**: `gapi_plaidml_pipelines.cpp`
- **File Size**: 6,249 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_plaidml_pipelines.cpp](../../../modules/gapi/test/gapi_plaidml_pipelines.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation


#include "test_precomp.hpp"

#include <stdexcept>
#include <ade/util/iota_range.hpp>
#include "logger.hpp"

#include <opencv2/gapi/plaidml/core.hpp>
#include <opencv2/gapi/plaidml/plaidml.hpp>

namespace opencv_test
{

#ifdef HAVE_PLAIDML

inline cv::gapi::plaidml::config getConfig()
{
    auto read_var_from_env = [](const char* env)
    {
        const char* raw = std::getenv(env);
        if (!raw)
        {
            cv::util::throw_error(std::runtime_error(std::string(env) + " is't set"));
        }

        return std::string(raw);
    };

    auto dev_id = read_var_from_env("PLAIDML_DEVICE");
    auto trg_id = read_var_from_env("PLAIDML_TARGET");

    return cv::gapi::plaidml::config{std::move(dev_id),
                                     std::move(trg_id)};
}

TEST(GAPI_PlaidML_Pipelines, SimpleArithmetic)
{
    cv::Size size(1920, 1080);
    int type = CV_8UC1;

    cv::Mat in_mat1(size, type);
    cv::Mat in_mat2(size, type);

    // NB: What about overflow ? PlaidML doesn't handle it
    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(127));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(127));

    cv::Mat out_mat(size, type, cv::Scalar::all(0));
    cv::Mat ref_mat(size, type, cv::Scalar::all(0));

    ////////////////////////////// G-API //////////////////////////////////////
    cv::GMat in1, in2;
    auto out = in1 + in2;

    cv::GComputation comp(cv::GIn(in1, in2), cv::GOut(out));
    comp.apply(cv::gin(in_mat1, in_mat2), cv::gout(out_mat),
               cv::compile_args(getConfig(),
                                cv::gapi::use_only{cv::gapi::core::plaidml::kernels()}));

    ////////////////////////////// OpenCV /////////////////////////////////////
    cv::add(in_mat1, in_mat2, ref_mat, cv::noArray(), type);

    EXPECT_EQ(0, cv::norm(out_mat, ref_mat));
}

// FIXME PlaidML cpu backend does't support bitwise operations
TEST(GAPI_PlaidML_Pipelines, DISABLED_ComplexArithmetic)
{
    cv::Size size(1920, 1080);
    int type = CV_8UC1;

    cv::Mat in_mat1(size, type);
    cv::Mat in_mat2(size, type);

    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));

    cv::Mat out_mat(size, type, cv::Scalar::all(0));
    cv::Mat ref_mat(size, type, cv::Scalar::all(0));

    ////////////////////////////// G-API //////////////////////////////////////
    cv::GMat in1, in2;
    auto out = in1 | (in2 ^ (in1 & (in2 + (in1 - in2))));

    cv::GComputation comp(cv::GIn(in1, in2), cv::GOut(out));
    comp.apply(cv::gin(in_mat1, in_mat2), cv::gout(out_mat),
               cv::compile_args(getConfig(),
                                cv::gapi::use_only{cv::gapi::core::plaidml::kernels()}));

    ////////////////////////////// OpenCV /////////////////////////////////////
    cv::subtract(in_mat1,  in_mat2, ref_mat, cv::noArray(), type);
    cv::add(in_mat2, ref_mat, ref_mat, cv::noArray(), type);
    cv::bitwise_and(in_mat1, ref_mat, ref_mat);
    cv::bitwise_xor(in_mat2, ref_mat, ref_mat);
    cv::bitwise_or(in_mat1, ref_mat, ref_mat);

    EXPECT_EQ(0, cv::norm(out_mat, ref_mat));
}

TEST(GAPI_PlaidML_Pipelines, TwoInputOperations)
{
    cv::Size size(1920, 1080);
    int type = CV_8UC1;

    constexpr int kNumInputs = 4;
    std::vector<cv::Mat> in_mat(kNumInputs, cv::Mat(size, type));
    for (int i = 0; i < kNumInputs; ++i)
    {
        cv::randu(in_mat[i], cv::Scalar::all(0), cv::Scalar::all(60));
    }

    cv::Mat out_mat(size, type, cv::Scalar::all(0));
    cv::Mat ref_mat(size, type, cv::Scalar::all(0));

    ////////////////////////////// G-API //////////////////////////////////////
    cv::GMat in[4];
    auto out = (in[3] - in[0]) + (in[2] - in[1]);

    cv::GComputation comp(cv::GIn(in[0], in[1], in[2], in[3]), cv::GOut(out));

    // FIXME Doesn't work just apply(in_mat, out_mat, ...)
    comp.apply(cv::gin(in_mat[0], in_mat[1], in_mat[2], in_mat[3]), cv::gout(out_mat),
               cv::compile_args(getConfig(),
                                cv::gapi::use_only{cv::gapi::core::plaidml::kernels()}));

    ////////////////////////////// OpenCV /////////////////////////////////////
    cv::subtract(in_mat[3], in_mat[0],  ref_mat, cv::noArray(), type);
    cv::add(ref_mat, in_mat[2], ref_mat, cv::noArray(), type);
    cv::subtract(ref_mat, in_mat[1], ref_mat, cv::noArray(), type);

    EXPECT_EQ(0, cv::norm(out_mat, ref_mat));
}

TEST(GAPI_PlaidML_Pipelines, TwoOutputOperations)
{
    cv::Size size(1920, 1080);
    int type = CV_8UC1;

    constexpr int kNumInputs = 4;
    std::vector<cv::Mat> in_mat(kNumInputs, cv::Mat(size, type));
    for (int i = 0; i < kNumInputs; ++i)
    {
        cv::randu(in_mat[i], cv::Scalar::all(0), cv::Scalar::all(60));
    }

    std::vector<cv::Mat> out_mat(kNumInputs, cv::Mat(size, type, cv::Scalar::all(0)));
    std::vector<cv::Mat> ref_mat(kNumInputs, cv::Mat(size, type, cv::Scalar::all(0)));

    ////////////////////////////// G-API //////////////////////////////////////
    cv::GMat in[4], out[2];
    out[0] = in[0] + in[3];
    out[1] = in[1] + in[2];

    cv::GComputation comp(cv::GIn(in[0], in[1], in[2], in[3]), cv::GOut(out[0], out[1]));

    // FIXME Doesn't work just apply(in_mat, out_mat, ...)
    comp.apply(cv::gin(in_mat[0], in_mat[1], in_mat[2], in_mat[3]),
               cv::gout(out_mat[0], out_mat[1]),
               cv::compile_args(getConfig(),
                                cv::gapi::use_only{cv::gapi::core::plaidml::kernels()}));

    ////////////////////////////// OpenCV /////////////////////////////////////
    cv::add(in_mat[0], in_mat[3], ref_mat[0], cv::noArray(), type);
    cv::add(in_mat[1], in_mat[2], ref_mat[1], cv::noArray(), type);

    EXPECT_EQ(0, cv::norm(out_mat[0], ref_mat[0]));
    EXPECT_EQ(0, cv::norm(out_mat[1], ref_mat[1]));
}

#else // HAVE_PLAIDML

TEST(GAPI_PlaidML_Pipelines, ThrowIfPlaidMLNotFound)
{
    ASSERT_ANY_THROW(cv::gapi::core::plaidml::kernels());
}

#endif // HAVE_PLAIDML

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

### Functions and Methods

- **HAVE_PLAIDML()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/plaidml/core.hpp`
- `ade/util/iota_range.hpp`
- `test_precomp.hpp`
- `stdexcept`
- `opencv2/gapi/plaidml/plaidml.hpp`
- `logger.hpp`


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

