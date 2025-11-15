# Documentation for `modules/gapi/include/opencv2/gapi/ocl/goclkernel.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/ocl/goclkernel.hpp`
- **File Name**: `goclkernel.hpp`
- **File Size**: 7,664 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/ocl/goclkernel.hpp](../../../../../../modules/gapi/include/opencv2/gapi/ocl/goclkernel.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/ocl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#ifndef OPENCV_GAPI_GOCLKERNEL_HPP
#define OPENCV_GAPI_GOCLKERNEL_HPP

#include <vector>
#include <functional>
#include <map>
#include <unordered_map>

#include <opencv2/core/mat.hpp>
#include <opencv2/gapi/gcommon.hpp>
#include <opencv2/gapi/gkernel.hpp>
#include <opencv2/gapi/garg.hpp>

// FIXME: namespace scheme for backends?
namespace cv {

namespace gimpl
{
    // Forward-declare an internal class
    class GOCLExecutable;
} // namespace gimpl

namespace gapi
{
/**
 * @brief This namespace contains G-API OpenCL backend functions, structures, and symbols.
 */
namespace ocl
{
    /**
     * \addtogroup gapi_std_backends G-API Standard Backends
     * @{
     */
    /**
     * @brief Get a reference to OCL backend.
     *
     * At the moment, the OCL backend is built atop of OpenCV
     * "Transparent API" (T-API), see cv::UMat for details.
     *
     * @sa gapi_std_backends
     */
    GAPI_EXPORTS cv::gapi::GBackend backend();
    /** @} */
} // namespace ocl
} // namespace gapi


// Represents arguments which are passed to a wrapped OCL function
// FIXME: put into detail?
class GAPI_EXPORTS GOCLContext
{
public:
    // Generic accessor API
    template<typename T>
    const T& inArg(int input) { return m_args.at(input).get<T>(); }

    // Syntax sugar
    const cv::UMat&  inMat(int input);
    cv::UMat&  outMatR(int output); // FIXME: Avoid cv::Mat m = ctx.outMatR()

    const cv::Scalar& inVal(int input);
    cv::Scalar& outValR(int output); // FIXME: Avoid cv::Scalar s = ctx.outValR()
    template<typename T> std::vector<T>& outVecR(int output) // FIXME: the same issue
    {
        return outVecRef(output).wref<T>();
    }
    template<typename T> T& outOpaqueR(int output) // FIXME: the same issue
    {
        return outOpaqueRef(output).wref<T>();
    }

protected:
    detail::VectorRef& outVecRef(int output);
    detail::OpaqueRef& outOpaqueRef(int output);

    std::vector<GArg> m_args;
    std::unordered_map<std::size_t, GRunArgP> m_results;


    friend class gimpl::GOCLExecutable;
};

class GAPI_EXPORTS GOCLKernel
{
public:
    // This function is kernel's execution entry point (does the processing work)
    using F = std::function<void(GOCLContext &)>;

    GOCLKernel();
    explicit GOCLKernel(const F& f);

    void apply(GOCLContext &ctx);

protected:
    F m_f;
};

// FIXME: This is an ugly ad-hoc implementation. TODO: refactor

namespace detail
{
template<class T> struct ocl_get_in;
template<> struct ocl_get_in<cv::GMat>
{
    static cv::UMat    get(GOCLContext &ctx, int idx) { return ctx.inMat(idx); }
};
template<> struct ocl_get_in<cv::GScalar>
{
    static cv::Scalar get(GOCLContext &ctx, int idx) { return ctx.inVal(idx); }
};
template<typename U> struct ocl_get_in<cv::GArray<U> >
{
    static const std::vector<U>& get(GOCLContext &ctx, int idx) { return ctx.inArg<VectorRef>(idx).rref<U>(); }
};
template<> struct ocl_get_in<cv::GFrame>
{
    static cv::MediaFrame get(GOCLContext &ctx, int idx) { return ctx.inArg<cv::MediaFrame>(idx); }
};
template<typename U> struct ocl_get_in<cv::GOpaque<U> >
{
    static const U& get(GOCLContext &ctx, int idx) { return ctx.inArg<OpaqueRef>(idx).rref<U>(); }
};
template<class T> struct ocl_get_in
{
    static T get(GOCLContext &ctx, int idx) { return ctx.inArg<T>(idx); }
};

struct tracked_cv_umat{
    //TODO Think if T - API could reallocate UMat to a proper size - how do we handle this ?
    //tracked_cv_umat(cv::UMat& m) : r{(m)}, original_data{m.getMat(ACCESS_RW).data} {}
    tracked_cv_umat(cv::UMat& m) : r(m), original_data{ nullptr } {}
    cv::UMat &r; // FIXME: It was a value (not a reference) before.
                 // Actually OCL backend should allocate its internal data!
    uchar* original_data;

    operator cv::UMat& (){ return r;}
    void validate() const{
        //if (r.getMat(ACCESS_RW).data != original_data)
        //{
        //    util::throw_error
        //        (std::logic_error
        //         ("OpenCV kernel output parameter was reallocated. \n"
        //          "Incorrect meta data was provided ?"));
        //}

    }
};

#ifdef _MSC_VER
#pragma warning(push)
#pragma warning(disable: 4702)  // unreachable code
#endif
template<typename... Outputs>
void postprocess_ocl(Outputs&... outs)
{
    struct
    {
        void operator()(tracked_cv_umat* bm) { bm->validate(); }
        void operator()(...) {                  }

    } validate;
    //dummy array to unfold parameter pack
    int dummy[] = { 0, (validate(&outs), 0)... };
    cv::util::suppress_unused_warning(dummy);
}
#ifdef _MSC_VER
#pragma warning(pop)
#endif

template<class T> struct ocl_get_out;
template<> struct ocl_get_out<cv::GMat>
{
    static tracked_cv_umat get(GOCLContext &ctx, int idx)
    {
        auto& r = ctx.outMatR(idx);
        return{ r };
    }
};
template<> struct ocl_get_out<cv::GScalar>
{
    static cv::Scalar& get(GOCLContext &ctx, int idx)
    {
        return ctx.outValR(idx);
    }
};
template<typename U> struct ocl_get_out<cv::GArray<U> >
{
    static std::vector<U>& get(GOCLContext &ctx, int idx) { return ctx.outVecR<U>(idx);  }
};
template<typename U> struct ocl_get_out<cv::GOpaque<U> >
{
    static U& get(GOCLContext &ctx, int idx) { return ctx.outOpaqueR<U>(idx);  }
};

template<typename, typename, typename>
struct OCLCallHelper;

// FIXME: probably can be simplified with std::apply or analogue.
template<typename Impl, typename... Ins, typename... Outs>
struct OCLCallHelper<Impl, std::tuple<Ins...>, std::tuple<Outs...> >
{
    template<typename... Inputs>
    struct call_and_postprocess
    {
        template<typename... Outputs>
        static void call(Inputs&&... ins, Outputs&&... outs)
        {
            //not using a std::forward on outs is deliberate in order to
            //cause compilation error, by trying to bind rvalue references to lvalue references
            Impl::run(std::forward<Inputs>(ins)..., outs...);

            postprocess_ocl(outs...);
        }
    };

    template<int... IIs, int... OIs>
    static void call_impl(GOCLContext &ctx, detail::Seq<IIs...>, detail::Seq<OIs...>)
    {
        //TODO: Make sure that OpenCV kernels do not reallocate memory for output parameters
        //by comparing it's state (data ptr) before and after the call.
        //Convert own::Scalar to cv::Scalar before call kernel and run kernel
        //convert cv::Scalar to own::Scalar after call kernel and write back results
        call_and_postprocess<decltype(ocl_get_in<Ins>::get(ctx, IIs))...>::call(ocl_get_in<Ins>::get(ctx, IIs)..., ocl_get_out<Outs>::get(ctx, OIs)...);
    }

    static void call(GOCLContext &ctx)
    {
        call_impl(ctx,
            typename detail::MkSeq<sizeof...(Ins)>::type(),
            typename detail::MkSeq<sizeof...(Outs)>::type());
    }
};

} // namespace detail

template<class Impl, class K>
class GOCLKernelImpl: public cv::detail::OCLCallHelper<Impl, typename K::InArgs, typename K::OutArgs>,
                      public cv::detail::KernelTag
{
    using P = detail::OCLCallHelper<Impl, typename K::InArgs, typename K::OutArgs>;

public:
    using API = K;

    static cv::gapi::GBackend backend()  { return cv::gapi::ocl::backend(); }
    static cv::GOCLKernel     kernel()   { return GOCLKernel(&P::call);     }
};

#define GAPI_OCL_KERNEL(Name, API) struct Name: public cv::GOCLKernelImpl<Name, API>

} // namespace cv

#endif // OPENCV_GAPI_GOCLKERNEL_HPP
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

- **ocl_get_out**: A class/struct defined in this file
- **GAPI_EXPORTS**: A class/struct defined in this file
- **class**: A class/struct defined in this file
- **call_and_postprocess**: A class/struct defined in this file
- **K**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **Impl**: A class/struct defined in this file
- **Name**: A class/struct defined in this file
- **tracked_cv_umat**: A class/struct defined in this file
- **GOCLKernelImpl**: A class/struct defined in this file
- **gimpl**: A class/struct defined in this file
- **OCLCallHelper**: A class/struct defined in this file
- **ocl_get_in**: A class/struct defined in this file

### Functions and Methods

- **is()**: A function/method defined in this file
- **OPENCV_GAPI_GOCLKERNEL_HPP()**: A function/method defined in this file
- **_MSC_VER()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `unordered_map`
- `functional`
- `opencv2/gapi/gkernel.hpp`
- `vector`
- `opencv2/core/mat.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi/gcommon.hpp`
- `map`


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

