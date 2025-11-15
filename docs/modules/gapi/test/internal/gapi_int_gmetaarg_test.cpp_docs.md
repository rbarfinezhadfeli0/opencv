# Documentation for `modules/gapi/test/internal/gapi_int_gmetaarg_test.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/internal/gapi_int_gmetaarg_test.cpp`
- **File Name**: `gapi_int_gmetaarg_test.cpp`
- **File Size**: 5,908 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/internal/gapi_int_gmetaarg_test.cpp](../../../../modules/gapi/test/internal/gapi_int_gmetaarg_test.cpp)

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

#include "api/gcomputation_priv.hpp"

namespace opencv_test
{

TEST(GMetaArg, Traits_Is_Positive)
{
    using namespace cv::detail;

    static_assert(is_meta_descr<cv::GScalarDesc>::value,
                  "GScalarDesc is a meta description type");

    static_assert(is_meta_descr<cv::GMatDesc>::value,
                  "GMatDesc is a meta description type");
}

TEST(GMetaArg, Traits_Is_Negative)
{
    using namespace cv::detail;

    static_assert(!is_meta_descr<cv::GCompileArgs>::value,
                  "GCompileArgs is NOT a meta description type");

    static_assert(!is_meta_descr<int>::value,
                  "int is NOT a meta description type");

    static_assert(!is_meta_descr<std::string>::value,
                  "str::string is NOT a meta description type");
}

TEST(GMetaArg, Traits_Are_EntireList_Positive)
{
    using namespace cv::detail;

    static_assert(are_meta_descrs<cv::GScalarDesc>::value,
                  "GScalarDesc is a meta description type");

    static_assert(are_meta_descrs<cv::GMatDesc>::value,
                  "GMatDesc is a meta description type");

    static_assert(are_meta_descrs<cv::GMatDesc, cv::GScalarDesc>::value,
                  "Both GMatDesc and GScalarDesc are meta types");
}

TEST(GMetaArg, Traits_Are_EntireList_Negative)
{
    using namespace cv::detail;

    static_assert(!are_meta_descrs<cv::GCompileArgs>::value,
                  "GCompileArgs is NOT among meta types");

    static_assert(!are_meta_descrs<int, std::string>::value,
                  "Both int and std::string is NOT among meta types");

    static_assert(!are_meta_descrs<cv::GMatDesc, cv::GScalarDesc, int>::value,
                  "List of type is not valid for meta as there\'s int");

    static_assert(!are_meta_descrs<cv::GMatDesc, cv::GScalarDesc, cv::GCompileArgs>::value,
                  "List of type is not valid for meta as there\'s GCompileArgs");
}

TEST(GMetaArg, Traits_Are_ButLast_Positive)
{
    using namespace cv::detail;

    static_assert(are_meta_descrs_but_last<cv::GScalarDesc, int>::value,
                  "List is valid (int is omitted)");

    static_assert(are_meta_descrs_but_last<cv::GMatDesc, cv::GScalarDesc, cv::GCompileArgs>::value,
                  "List is valid (GCompileArgs are omitted)");
}

TEST(GMetaArg, Traits_Are_ButLast_Negative)
{
    using namespace cv::detail;

    static_assert(!are_meta_descrs_but_last<int, std::string>::value,
                  "Both int is NOT among meta types (std::string is omitted)");

    static_assert(!are_meta_descrs_but_last<cv::GMatDesc, cv::GScalarDesc, int, int>::value,
                  "List of type is not valid for meta as there\'s two ints");

    static_assert(!are_meta_descrs_but_last<cv::GMatDesc, cv::GScalarDesc, cv::GCompileArgs, float>::value,
                  "List of type is not valid for meta as there\'s GCompileArgs");
}

TEST(GMetaArg, Can_Get_Metas_From_Input_Run_Args)
{
    cv::Mat m(3, 3, CV_8UC3);
    cv::Scalar s;
    std::vector<int> v;

    GMatDesc m_desc;
    GMetaArgs meta_args = descr_of(cv::gin(m, s, v));

    EXPECT_EQ(3u, meta_args.size());
    EXPECT_NO_THROW(m_desc = util::get<cv::GMatDesc>(meta_args[0]));
    EXPECT_NO_THROW(util::get<cv::GScalarDesc>(meta_args[1]));
    EXPECT_NO_THROW(util::get<cv::GArrayDesc>(meta_args[2]));

    EXPECT_EQ(CV_8U, m_desc.depth);
    EXPECT_EQ(3, m_desc.chan);
    EXPECT_EQ(cv::gapi::own::Size(3, 3), m_desc.size);
}

TEST(GMetaArg, Can_Get_Metas_From_Output_Run_Args)
{
    cv::Mat m(3, 3, CV_8UC3);
    cv::Scalar s;
    std::vector<int> v;

    GMatDesc m_desc;
    GRunArgsP out_run_args = cv::gout(m, s, v);
    GMetaArg m_meta = descr_of(out_run_args[0]);
    GMetaArg s_meta = descr_of(out_run_args[1]);
    GMetaArg v_meta = descr_of(out_run_args[2]);

    EXPECT_NO_THROW(m_desc = util::get<cv::GMatDesc>(m_meta));
    EXPECT_NO_THROW(util::get<cv::GScalarDesc>(s_meta));
    EXPECT_NO_THROW(util::get<cv::GArrayDesc>(v_meta));

    EXPECT_EQ(CV_8U, m_desc.depth);
    EXPECT_EQ(3, m_desc.chan);
    EXPECT_EQ(cv::Size(3, 3), m_desc.size);
}

TEST(GMetaArg, Can_Describe_RunArg)
{
    cv::Mat m(3, 3, CV_8UC3);
    cv::UMat um(3, 3, CV_8UC3);
    cv::Scalar s;
    cv::Scalar os;
    std::vector<int> v;

    GMetaArgs metas = {GMetaArg(descr_of(m)),
                       GMetaArg(descr_of(um)),
                       GMetaArg(descr_of(s)),
                       GMetaArg(descr_of(os)),
                       GMetaArg(descr_of(v))};

    auto in_run_args = cv::gin(m, um, s, os, v);

    for (size_t i = 0; i < metas.size(); i++) {
        EXPECT_TRUE(can_describe(metas[i], in_run_args[i]));
    }
}

TEST(GMetaArg, Can_Describe_RunArgs)
{
    cv::Mat m(3, 3, CV_8UC3);
    cv::Scalar s;
    std::vector<int> v;

    GMetaArgs metas0 = {GMetaArg(descr_of(m)), GMetaArg(descr_of(s)), GMetaArg(descr_of(v))};
    auto in_run_args0  = cv::gin(m, s, v);

    EXPECT_TRUE(can_describe(metas0, in_run_args0));

    auto in_run_args01  = cv::gin(m, s);
    EXPECT_FALSE(can_describe(metas0, in_run_args01));
}

TEST(GMetaArg, Can_Describe_RunArgP)
{
    cv::Mat m(3, 3, CV_8UC3);
    cv::UMat um(3, 3, CV_8UC3);
    cv::Scalar s;
    cv::Scalar os;
    std::vector<int> v;

    GMetaArgs metas = {GMetaArg(descr_of(m)),
                       GMetaArg(descr_of(um)),
                       GMetaArg(descr_of(s)),
                       GMetaArg(descr_of(os)),
                       GMetaArg(descr_of(v))};

    auto out_run_args = cv::gout(m, um, s, os, v);

    for (size_t i = 0; i < metas.size(); i++) {
        EXPECT_TRUE(can_describe(metas[i], out_run_args[i]));
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
- `api/gcomputation_priv.hpp`
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

