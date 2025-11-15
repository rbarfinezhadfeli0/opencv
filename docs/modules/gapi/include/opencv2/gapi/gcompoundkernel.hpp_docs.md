# Documentation for `modules/gapi/include/opencv2/gapi/gcompoundkernel.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/gcompoundkernel.hpp`
- **File Name**: `gcompoundkernel.hpp`
- **File Size**: 3,790 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/gcompoundkernel.hpp](../../../../../modules/gapi/include/opencv2/gapi/gcompoundkernel.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2019 Intel Corporation


#ifndef OPENCV_GAPI_GCOMPOUNDKERNEL_HPP
#define OPENCV_GAPI_GCOMPOUNDKERNEL_HPP

#include <opencv2/gapi/opencv_includes.hpp>
#include <opencv2/gapi/gcommon.hpp>
#include <opencv2/gapi/gkernel.hpp>
#include <opencv2/gapi/garg.hpp>

namespace cv {
namespace gapi
{
namespace compound
{
    // FIXME User does not need to know about this function
    // Needs that user may define compound kernels(as cpu kernels)
    GAPI_EXPORTS cv::gapi::GBackend backend();
} // namespace compound
} // namespace gapi

namespace detail
{

struct GCompoundContext
{
    explicit GCompoundContext(const GArgs& in_args);
    template<typename T>
    const T& inArg(int input) { return m_args.at(input).get<T>(); }

    GArgs m_args;
    GArgs m_results;
};

class GAPI_EXPORTS GCompoundKernel
{
// Compound kernel must use all of it's inputs
public:
    using F = std::function<void(GCompoundContext& ctx)>;

    explicit GCompoundKernel(const F& f);
    void apply(GCompoundContext& ctx);

protected:
    F m_f;
};

template<typename T> struct get_compound_in
{
    static T get(GCompoundContext &ctx, int idx) { return ctx.inArg<T>(idx); }
};

template<typename U> struct get_compound_in<cv::GArray<U>>
{
    static cv::GArray<U> get(GCompoundContext &ctx, int idx)
    {
        auto array = cv::GArray<U>();
        ctx.m_args[idx] = GArg(array);
        return array;
    }
};

template<typename U> struct get_compound_in<cv::GOpaque<U>>
{
    static cv::GOpaque<U> get(GCompoundContext &ctx, int idx)
    {
        auto opaq = cv::GOpaque<U>();
        ctx.m_args[idx] = GArg(opaq);
        return opaq;
    }
};

template<> struct get_compound_in<cv::GMatP>
{
    static cv::GMatP get(GCompoundContext &ctx, int idx)
    {
        auto mat = cv::GMatP();
        ctx.m_args[idx] = GArg(mat);
        return mat;
    }
};

template<typename, typename, typename>
struct GCompoundCallHelper;

template<typename Impl, typename... Ins, typename... Outs>
struct GCompoundCallHelper<Impl, std::tuple<Ins...>, std::tuple<Outs...> >
{
    template<int... IIs, int... OIs>
    static void expand_impl(GCompoundContext &ctx, detail::Seq<IIs...>, detail::Seq<OIs...>)
    {
        auto result = Impl::expand(get_compound_in<Ins>::get(ctx, IIs)...);
        auto tuple_return = tuple_wrap_helper<decltype(result)>::get(std::move(result));
        ctx.m_results = { cv::GArg(std::get<OIs>(tuple_return))... };
    }

    static void expand(GCompoundContext &ctx)
    {
        expand_impl(ctx,
                    typename detail::MkSeq<sizeof...(Ins)>::type(),
                    typename detail::MkSeq<sizeof...(Outs)>::type());
    }
};

template<class Impl, class K>
class GCompoundKernelImpl: public cv::detail::GCompoundCallHelper<Impl, typename K::InArgs, typename K::OutArgs>,
                           public cv::detail::KernelTag
{
    using P = cv::detail::GCompoundCallHelper<Impl, typename K::InArgs, typename K::OutArgs>;

public:
    using API = K;

    static cv::gapi::GBackend backend() { return cv::gapi::compound::backend(); }
    static GCompoundKernel    kernel()  { return GCompoundKernel(&P::expand);   }
};

} // namespace detail


/**
 * Declares a new compound kernel. See this
 * [documentation chapter](@ref gapi_kernel_compound)
 * on compound kernels for more details.
 *
 * @param Name type name for new kernel
 * @param API the interface this kernel implements
 */
#define GAPI_COMPOUND_KERNEL(Name, API) \
    struct Name: public cv::detail::GCompoundKernelImpl<Name, API>

} // namespace cv

#endif // OPENCV_GAPI_GCOMPOUNDKERNEL_HPP
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
- **GCompoundCallHelper**: A class/struct defined in this file
- **GCompoundContext**: A class/struct defined in this file
- **get_compound_in**: A class/struct defined in this file
- **K**: A class/struct defined in this file
- **GCompoundKernelImpl**: A class/struct defined in this file
- **Impl**: A class/struct defined in this file
- **Name**: A class/struct defined in this file
- **this**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_GCOMPOUNDKERNEL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gkernel.hpp`
- `opencv2/gapi/opencv_includes.hpp`
- `opencv2/gapi/gcommon.hpp`
- `opencv2/gapi/garg.hpp`


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

