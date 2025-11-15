# Documentation for `modules/core/src/check.cpp`

## File Metadata

- **Full Path**: `modules/core/src/check.cpp`
- **File Name**: `check.cpp`
- **File Size**: 7,337 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/check.cpp](../../../modules/core/src/check.cpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"

#include <sstream>

#include "opencv2/core/check.hpp"

namespace cv {

const char* depthToString(int depth)
{
    const char* s = detail::depthToString_(depth);
    return s ? s : "<invalid depth>";
}

cv::String typeToString(int type)
{
    cv::String s = detail::typeToString_(type);
    if (s.empty())
    {
        static cv::String invalidType("<invalid type>");
        return invalidType;
    }
    return s;
}


namespace detail {

static const char* getTestOpPhraseStr(unsigned testOp)
{
    static const char* _names[] = { "{custom check}", "equal to", "not equal to", "less than or equal to", "less than", "greater than or equal to", "greater than" };
    CV_DbgAssert(testOp < CV__LAST_TEST_OP);
    return testOp < CV__LAST_TEST_OP ? _names[testOp] : "???";
}
static const char* getTestOpMath(unsigned testOp)
{
    static const char* _names[] = { "???", "==", "!=", "<=", "<", ">=", ">" };
    CV_DbgAssert(testOp < CV__LAST_TEST_OP);
    return testOp < CV__LAST_TEST_OP ? _names[testOp] : "???";
}

const char* depthToString_(int depth)
{
    static const char* depthNames[] = { "CV_8U", "CV_8S", "CV_16U", "CV_16S", "CV_32S", "CV_32F", "CV_64F", "CV_16F" };
    return (depth <= CV_16F && depth >= 0) ? depthNames[depth] : NULL;
}

cv::String typeToString_(int type)
{
    int depth = CV_MAT_DEPTH(type);
    int cn = CV_MAT_CN(type);
    if (depth >= 0 && depth <= CV_16F)
        return cv::format("%sC%d", depthToString_(depth), cn);
    return cv::String();
}

template<typename T> static CV_NORETURN
void check_failed_auto_(const T& v1, const T& v2, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << " (expected: '" << ctx.p1_str << " " << getTestOpMath(ctx.testOp) << " " << ctx.p2_str << "'), where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v1 << std::endl;
    if (ctx.testOp != TEST_CUSTOM && ctx.testOp < CV__LAST_TEST_OP)
    {
        ss << "must be " << getTestOpPhraseStr(ctx.testOp) << std::endl;
    }
    ss  << "    '" << ctx.p2_str << "' is " << v2;
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatDepth(const int v1, const int v2, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << " (expected: '" << ctx.p1_str << " " << getTestOpMath(ctx.testOp) << " " << ctx.p2_str << "'), where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v1 << " (" << depthToString(v1) << ")" << std::endl;
    if (ctx.testOp != TEST_CUSTOM && ctx.testOp < CV__LAST_TEST_OP)
    {
        ss << "must be " << getTestOpPhraseStr(ctx.testOp) << std::endl;
    }
    ss  << "    '" << ctx.p2_str << "' is " << v2 << " (" << depthToString(v2) << ")";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatType(const int v1, const int v2, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << " (expected: '" << ctx.p1_str << " " << getTestOpMath(ctx.testOp) << " " << ctx.p2_str << "'), where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v1 << " (" << typeToString(v1) << ")" << std::endl;
    if (ctx.testOp != TEST_CUSTOM && ctx.testOp < CV__LAST_TEST_OP)
    {
        ss << "must be " << getTestOpPhraseStr(ctx.testOp) << std::endl;
    }
    ss  << "    '" << ctx.p2_str << "' is " << v2 << " (" << typeToString(v2) << ")";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatChannels(const int v1, const int v2, const CheckContext& ctx)
{
    check_failed_auto_<int>(v1, v2, ctx);
}
void check_failed_auto(const bool v1, const bool v2, const CheckContext& ctx)
{
    check_failed_auto_<bool>(v1, v2, ctx);
}
void check_failed_auto(const int v1, const int v2, const CheckContext& ctx)
{
    check_failed_auto_<int>(v1, v2, ctx);
}
void check_failed_auto(const size_t v1, const size_t v2, const CheckContext& ctx)
{
    check_failed_auto_<size_t>(v1, v2, ctx);
}
void check_failed_auto(const float v1, const float v2, const CheckContext& ctx)
{
    check_failed_auto_<float>(v1, v2, ctx);
}
void check_failed_auto(const double v1, const double v2, const CheckContext& ctx)
{
    check_failed_auto_<double>(v1, v2, ctx);
}
void check_failed_auto(const Size_<int> v1, const Size_<int> v2, const CheckContext& ctx)
{
    check_failed_auto_< Size_<int> >(v1, v2, ctx);
}


template<typename T> static CV_NORETURN
void check_failed_auto_(const T& v, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p2_str << "'" << std::endl
        << "where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v;
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatDepth(const int v, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p2_str << "'" << std::endl
        << "where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v << " (" << depthToString(v) << ")";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatType(const int v, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p2_str << "'" << std::endl
        << "where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v << " (" << typeToString(v) << ")";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_MatChannels(const int v, const CheckContext& ctx)
{
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p2_str << "'" << std::endl
        << "where" << std::endl
        << "    '" << ctx.p1_str << "' is " << v;
    cv::error(cv::Error::BadNumChannels, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_true(const bool v, const CheckContext& ctx)
{
    CV_UNUSED(v);
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p1_str << "' must be 'true'";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_false(const bool v, const CheckContext& ctx)
{
    CV_UNUSED(v);
    std::stringstream ss;
    ss  << ctx.message << ":" << std::endl
        << "    '" << ctx.p1_str << "' must be 'false'";
    cv::error(cv::Error::StsError, ss.str(), ctx.func, ctx.file, ctx.line);
}
void check_failed_auto(const int v, const CheckContext& ctx)
{
    check_failed_auto_<int>(v, ctx);
}
void check_failed_auto(const size_t v, const CheckContext& ctx)
{
    check_failed_auto_<size_t>(v, ctx);
}
void check_failed_auto(const float v, const CheckContext& ctx)
{
    check_failed_auto_<float>(v, ctx);
}
void check_failed_auto(const double v, const CheckContext& ctx)
{
    check_failed_auto_<double>(v, ctx);
}
void check_failed_auto(const Size_<int> v, const CheckContext& ctx)
{
    check_failed_auto_< Size_<int> >(v, ctx);
}
void check_failed_auto(const std::string& v, const CheckContext& ctx)
{
    check_failed_auto_< std::string >(v, ctx);
}


}} // namespace
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
- `precomp.hpp`
- `opencv2/core/check.hpp`
- `sstream`


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

