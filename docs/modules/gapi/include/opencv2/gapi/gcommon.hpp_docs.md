# Documentation for `modules/gapi/include/opencv2/gapi/gcommon.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcommon.hpp`
- **File Name**: `gcommon.hpp`
- **File Size**: 10,907 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcommon.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcommon.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_GCOMMON_HPP
#define OPENCV_GAPI_GCOMMON_HPP

#include <functional>   // std::hash
#include <vector>       // std::vector
#include <type_traits>  // decay

#include <opencv2/gapi/opencv_includes.hpp>

#include <opencv2/gapi/util/any.hpp>
#include <opencv2/gapi/util/optional.hpp>
#include <opencv2/gapi/own/exports.hpp>
#include <opencv2/gapi/own/assert.hpp>
#include <opencv2/gapi/render/render_types.hpp>
#include <opencv2/gapi/s11n/base.hpp>

namespace cv {

class GMat; // FIXME: forward declaration for GOpaqueTraits

namespace detail
{
    // This is a trait-like structure to mark backend-specific compile arguments
    // with tags
    template<typename T> struct CompileArgTag;

    // These structures are tags which separate kernels and transformations
    struct KernelTag
    {};
    struct TransformTag
    {};

    // This enum is utilized mostly by GArray and GOpaque to store and recognize their internal data
    // types (aka Host type). Also it is widely used during serialization routine.
    enum class OpaqueKind: int
    {
        CV_UNKNOWN,    // Unknown, generic, opaque-to-GAPI data type unsupported in graph seriallization
        CV_BOOL,       // bool user G-API data
        CV_INT,        // int user G-API data
        CV_INT64,      // int64_t user G-API data
        CV_DOUBLE,     // double user G-API data
        CV_FLOAT,      // float user G-API data
        CV_UINT64,     // uint64_t user G-API data
        CV_STRING,     // std::string user G-API data
        CV_POINT,      // cv::Point user G-API data
        CV_POINT2F,    // cv::Point2f user G-API data
        CV_POINT3F,    // cv::Point3f user G-API data
        CV_SIZE,       // cv::Size user G-API data
        CV_RECT,       // cv::Rect user G-API data
        CV_SCALAR,     // cv::Scalar user G-API data
        CV_MAT,        // cv::Mat user G-API data
        CV_DRAW_PRIM,  // cv::gapi::wip::draw::Prim user G-API data
    };

    // Type traits helper which simplifies the extraction of kind from type
    template<typename T> struct GOpaqueTraits;
    template<typename T> struct GOpaqueTraits    { static constexpr const OpaqueKind kind = OpaqueKind::CV_UNKNOWN; };
    template<> struct GOpaqueTraits<int>         { static constexpr const OpaqueKind kind = OpaqueKind::CV_INT; };
    template<> struct GOpaqueTraits<int64_t>     { static constexpr const OpaqueKind kind = OpaqueKind::CV_INT64; };
    template<> struct GOpaqueTraits<double>      { static constexpr const OpaqueKind kind = OpaqueKind::CV_DOUBLE; };
    template<> struct GOpaqueTraits<float>       { static constexpr const OpaqueKind kind = OpaqueKind::CV_FLOAT; };
    template<> struct GOpaqueTraits<uint64_t>    { static constexpr const OpaqueKind kind = OpaqueKind::CV_UINT64; };
    template<> struct GOpaqueTraits<bool>        { static constexpr const OpaqueKind kind = OpaqueKind::CV_BOOL; };
    template<> struct GOpaqueTraits<std::string> { static constexpr const OpaqueKind kind = OpaqueKind::CV_STRING; };
    template<> struct GOpaqueTraits<cv::Size>    { static constexpr const OpaqueKind kind = OpaqueKind::CV_SIZE; };
    template<> struct GOpaqueTraits<cv::Scalar>  { static constexpr const OpaqueKind kind = OpaqueKind::CV_SCALAR; };
    template<> struct GOpaqueTraits<cv::Point>   { static constexpr const OpaqueKind kind = OpaqueKind::CV_POINT; };
    template<> struct GOpaqueTraits<cv::Point2f> { static constexpr const OpaqueKind kind = OpaqueKind::CV_POINT2F; };
    template<> struct GOpaqueTraits<cv::Point3f> { static constexpr const OpaqueKind kind = OpaqueKind::CV_POINT3F; };
    template<> struct GOpaqueTraits<cv::Mat>     { static constexpr const OpaqueKind kind = OpaqueKind::CV_MAT; };
    template<> struct GOpaqueTraits<cv::Rect>    { static constexpr const OpaqueKind kind = OpaqueKind::CV_RECT; };
    template<> struct GOpaqueTraits<cv::GMat>    { static constexpr const OpaqueKind kind = OpaqueKind::CV_MAT; };
    template<> struct GOpaqueTraits<cv::gapi::wip::draw::Prim>
                                                 { static constexpr const OpaqueKind kind = OpaqueKind::CV_DRAW_PRIM; };
    using GOpaqueTraitsArrayTypes = std::tuple<int, double, float, uint64_t, bool, std::string, cv::Size, cv::Scalar, cv::Point, cv::Point2f,
                                               cv::Point3f, cv::Mat, cv::Rect, cv::gapi::wip::draw::Prim>;
    // GOpaque is not supporting cv::Mat and cv::Scalar since there are GScalar and GMat types
    using GOpaqueTraitsOpaqueTypes = std::tuple<int, double, float, uint64_t, bool, std::string, cv::Size, cv::Point, cv::Point2f, cv::Point3f,
                                                cv::Rect, cv::gapi::wip::draw::Prim>;
} // namespace detail

// This definition is here because it is reused by both public(?) and internal
// modules. Keeping it here wouldn't expose public details (e.g., API-level)
// to components which are internal and operate on a lower-level entities
// (e.g., compiler, backends).
// FIXME: merge with ArgKind?
// FIXME: replace with variant[format desc]?
enum class GShape: int
{
    GMAT,
    GSCALAR,
    GARRAY,
    GOPAQUE,
    GFRAME,
};

namespace gapi {
namespace s11n {
namespace detail {
template<typename T> struct wrap_serialize;
} // namespace detail
} // namespace s11n
} // namespace gapi


struct GCompileArg;

namespace detail {
    template<typename T>
    using is_compile_arg = std::is_same<GCompileArg, typename std::decay<T>::type>;
} // namespace detail

// CompileArg is an unified interface over backend-specific compilation
// information
// FIXME: Move to a separate file?
/** \addtogroup gapi_compile_args
 * @{
 *
 * @brief Compilation arguments: data structures controlling the
 * compilation process
 *
 * G-API comes with a number of graph compilation options which can be
 * passed to cv::GComputation::apply() or
 * cv::GComputation::compile(). Known compilation options are listed
 * in this page, while extra backends may introduce their own
 * compilation options (G-API transparently accepts _everything_ which
 * can be passed to cv::compile_args(), it depends on underlying
 * backends if an option would be interpreted or not).
 *
 * For example, if an example computation is executed like this:
 *
 * @snippet samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp graph_decl_apply
 *
 * Extra parameter specifying which kernels to compile with can be
 * passed like this:
 *
 * @snippet samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp apply_with_param
 */

/**
 * @brief Represents an arbitrary compilation argument.
 *
 * Any value can be wrapped into cv::GCompileArg, but only known ones
 * (to G-API or its backends) can be interpreted correctly.
 *
 * Normally objects of this class shouldn't be created manually, use
 * cv::compile_args() function which automatically wraps everything
 * passed in (a variadic template parameter pack) into a vector of
 * cv::GCompileArg objects.
 */
struct GCompileArg
{
public:
    // NB: Required for pythnon bindings
    GCompileArg() = default;

    std::string tag;

    // FIXME: use decay in GArg/other trait-based wrapper before leg is shot!
    template<typename T, typename std::enable_if<!detail::is_compile_arg<T>::value, int>::type = 0>
    explicit GCompileArg(T &&t)
        : tag(detail::CompileArgTag<typename std::decay<T>::type>::tag())
        , serializeF(cv::gapi::s11n::detail::has_S11N_spec<T>::value ?
                     &cv::gapi::s11n::detail::wrap_serialize<T>::serialize :
                     nullptr)
        , arg(t)
    {
    }

    template<typename T> T& get()
    {
        return util::any_cast<T>(arg);
    }

    template<typename T> const T& get() const
    {
        return util::any_cast<T>(arg);
    }

    void serialize(cv::gapi::s11n::IOStream& os) const
    {
        if (serializeF)
        {
            serializeF(os, *this);
        }
    }

private:
    std::function<void(cv::gapi::s11n::IOStream&, const GCompileArg&)> serializeF;
    util::any arg;
};

using GCompileArgs = std::vector<GCompileArg>;

inline cv::GCompileArgs& operator += (      cv::GCompileArgs &lhs,
                                      const cv::GCompileArgs &rhs)
{
    lhs.reserve(lhs.size() + rhs.size());
    lhs.insert(lhs.end(), rhs.begin(), rhs.end());
    return lhs;
}

/**
 * @brief Wraps a list of arguments (a parameter pack) into a vector of
 *        compilation arguments (cv::GCompileArg).
 */
template<typename... Ts> GCompileArgs compile_args(Ts&&... args)
{
    return GCompileArgs{ GCompileArg(args)... };
}

namespace gapi
{
/**
 * @brief Retrieves particular compilation argument by its type from
 *        cv::GCompileArgs
 */
template<typename T>
inline cv::util::optional<T> getCompileArg(const cv::GCompileArgs &args)
{
    for (auto &compile_arg : args)
    {
        if (compile_arg.tag == cv::detail::CompileArgTag<T>::tag())
        {
            return cv::util::optional<T>(compile_arg.get<T>());
        }
    }
    return cv::util::optional<T>();
}

namespace s11n {
namespace detail {
template<typename T> struct wrap_serialize
{
    static void serialize(IOStream& os, const GCompileArg& arg)
    {
        using DT = typename std::decay<T>::type;
        S11N<DT>::serialize(os, arg.get<DT>());
    }
};
} // namespace detail
} // namespace s11n
} // namespace gapi

/** @} gapi_compile_args */

/**
 * @brief Ask G-API to dump compiled graph in Graphviz format under
 * the given file name.
 *
 * Specifies a graph dump path (path to .dot file to be generated).
 * G-API will dump a .dot file under specified path during a
 * compilation process if this flag is passed.
 */
struct graph_dump_path
{
    std::string m_dump_path;
};

/**
 * @brief Ask G-API to use threaded executor when cv::GComputation
 * is compiled via cv::GComputation::compile method.
 *
 * Specifies a number of threads that should be used by executor.
 */
struct GAPI_EXPORTS use_threaded_executor
{
    use_threaded_executor();
    explicit use_threaded_executor(const uint32_t nthreads);

    uint32_t num_threads;
};

namespace detail
{
    template<> struct CompileArgTag<cv::graph_dump_path>
    {
        static const char* tag() { return "gapi.graph_dump_path"; }
    };

    template<> struct CompileArgTag<cv::use_threaded_executor>
    {
        static const char* tag() { return "gapi.threaded_executor"; }
    };
}

} // namespace cv

// std::hash overload for GShape
namespace std
{
template<> struct hash<cv::GShape>
{
    size_t operator() (cv::GShape sh) const
    {
        return std::hash<int>()(static_cast<int>(sh));
    }
};
} // namespace std


#endif // OPENCV_GAPI_GCOMMON_HPP
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **GAPI_EXPORTS**: A class/struct defined in this file
- **CompileArgTag**: A class/struct defined in this file
- **GShape**: A class/struct defined in this file
- **hash**: A class/struct defined in this file
- **TransformTag**: A class/struct defined in this file
- **OpaqueKind**: A class/struct defined in this file
- **over**: A class/struct defined in this file
- **GOpaqueTraits**: A class/struct defined in this file
- **graph_dump_path**: A class/struct defined in this file
- **wrap_serialize**: A class/struct defined in this file
- **GCompileArg**: A class/struct defined in this file
- **shouldn**: A class/struct defined in this file
- **GMat**: A class/struct defined in this file
- **KernelTag**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GCOMMON_HPP()**: A function/method defined in this file
- **which()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `functional`
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/util/optional.hpp`
- `opencv2/gapi/own/exports.hpp`
- `vector`
- `opencv2/gapi/s11n/base.hpp`
- `opencv2/gapi/util/any.hpp`
- `opencv2/gapi/own/assert.hpp`
- `type_traits`
- `opencv2/gapi/render/render_types.hpp`

**Python Imports:**
- `type`


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

