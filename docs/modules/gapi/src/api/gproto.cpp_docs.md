# Documentation for `modules/gapi/src/api/gproto.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/gproto.cpp`
- **File Name**: `gproto.cpp`
- **File Size**: 12,869 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/gproto.cpp](../../../../modules/gapi/src/api/gproto.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "precomp.hpp"

#include <ade/util/algorithm.hpp>
#include <opencv2/gapi/util/throw.hpp>
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gproto.hpp>

#include "api/gorigin.hpp"
#include "api/gproto_priv.hpp"

// FIXME: it should be a visitor!
// FIXME: Reimplement with traits?

const cv::GOrigin& cv::gimpl::proto::origin_of(const cv::GProtoArg &arg)
{
    switch (arg.index())
    {
    case cv::GProtoArg::index_of<cv::GMat>():
        return util::get<cv::GMat>(arg).priv();

    case cv::GProtoArg::index_of<cv::GMatP>():
        return util::get<cv::GMatP>(arg).priv();

    case cv::GProtoArg::index_of<cv::GFrame>():
        return util::get<cv::GFrame>(arg).priv();

    case cv::GProtoArg::index_of<cv::GScalar>():
        return util::get<cv::GScalar>(arg).priv();

    case cv::GProtoArg::index_of<cv::detail::GArrayU>():
        return util::get<cv::detail::GArrayU>(arg).priv();

    case cv::GProtoArg::index_of<cv::detail::GOpaqueU>():
        return util::get<cv::detail::GOpaqueU>(arg).priv();

    default:
        util::throw_error(std::logic_error("Unsupported GProtoArg type"));
    }
}

const cv::GOrigin& cv::gimpl::proto::origin_of(const cv::GArg &arg)
{
    // Generic, but not very efficient implementation
    // FIXME: Walking a thin line here!!! Here we rely that GArg and
    // GProtoArg share the same object and this is true while objects
    // are reference-counted, so return value is not a reference to a tmp.
    return origin_of(rewrap(arg));
}

bool cv::gimpl::proto::is_dynamic(const cv::GArg& arg)
{
    // FIXME: refactor this method to be auto-generated from
    // - GProtoArg variant parameter pack, and
    // - traits over every type
    switch (arg.kind)
    {
    case detail::ArgKind::GMAT:
    case detail::ArgKind::GMATP:
    case detail::ArgKind::GFRAME:
    case detail::ArgKind::GSCALAR:
    case detail::ArgKind::GARRAY:
    case detail::ArgKind::GOPAQUE:
        return true;

    default:
        return false;
    }
}

cv::GRunArg cv::value_of(const cv::GOrigin &origin)
{
    switch (origin.shape)
    {
    case GShape::GSCALAR: return GRunArg(util::get<cv::Scalar>(origin.value));
    case GShape::GARRAY:  return GRunArg(util::get<cv::detail::VectorRef>(origin.value));
    case GShape::GMAT:    return GRunArg(util::get<cv::Mat>(origin.value));
    default: util::throw_error(std::logic_error("Unsupported shape for constant"));
    }
}

cv::GProtoArg cv::gimpl::proto::rewrap(const cv::GArg &arg)
{
    // FIXME: replace with a more generic any->variant
    // (or variant<T> -> variant<U>) conversion?
    switch (arg.kind)
    {
    case detail::ArgKind::GMAT:    return GProtoArg(arg.get<cv::GMat>());
    case detail::ArgKind::GMATP:   return GProtoArg(arg.get<cv::GMatP>());
    case detail::ArgKind::GFRAME:  return GProtoArg(arg.get<cv::GFrame>());
    case detail::ArgKind::GSCALAR: return GProtoArg(arg.get<cv::GScalar>());
    case detail::ArgKind::GARRAY:  return GProtoArg(arg.get<cv::detail::GArrayU>());
    case detail::ArgKind::GOPAQUE: return GProtoArg(arg.get<cv::detail::GOpaqueU>());
    default: util::throw_error(std::logic_error("Unsupported GArg type"));
    }
}

cv::GMetaArg cv::descr_of(const cv::GRunArg &arg)
{
    switch (arg.index())
    {
        case GRunArg::index_of<cv::Mat>():
            return cv::GMetaArg(cv::descr_of(util::get<cv::Mat>(arg)));

        case GRunArg::index_of<cv::Scalar>():
            return cv::GMetaArg(descr_of(util::get<cv::Scalar>(arg)));

        case GRunArg::index_of<cv::detail::VectorRef>():
            return cv::GMetaArg(util::get<cv::detail::VectorRef>(arg).descr_of());

        case GRunArg::index_of<cv::detail::OpaqueRef>():
            return cv::GMetaArg(util::get<cv::detail::OpaqueRef>(arg).descr_of());

        case GRunArg::index_of<cv::gapi::wip::IStreamSource::Ptr>():
            return cv::util::get<cv::gapi::wip::IStreamSource::Ptr>(arg)->descr_of();

        case GRunArg::index_of<cv::RMat>():
            return cv::GMetaArg(cv::util::get<cv::RMat>(arg).desc());

        case GRunArg::index_of<cv::MediaFrame>():
            return cv::GMetaArg(cv::util::get<cv::MediaFrame>(arg).desc());

        default: util::throw_error(std::logic_error("Unsupported GRunArg type"));
    }
}

cv::GMetaArgs cv::descr_of(const cv::GRunArgs &args)
{
    cv::GMetaArgs metas;
    ade::util::transform(args, std::back_inserter(metas), [](const cv::GRunArg &arg){ return descr_of(arg); });
    return metas;
}

// FIXME: Is it tested for all types?
cv::GMetaArg cv::descr_of(const cv::GRunArgP &argp)
{
    switch (argp.index())
    {
#if !defined(GAPI_STANDALONE)
    case GRunArgP::index_of<cv::UMat*>():              return GMetaArg(cv::descr_of(*util::get<cv::UMat*>(argp)));
#endif //  !defined(GAPI_STANDALONE)
    case GRunArgP::index_of<cv::Mat*>():               return GMetaArg(cv::descr_of(*util::get<cv::Mat*>(argp)));
    case GRunArgP::index_of<cv::Scalar*>():            return GMetaArg(descr_of(*util::get<cv::Scalar*>(argp)));
    case GRunArgP::index_of<cv::MediaFrame*>():        return GMetaArg(descr_of(*util::get<cv::MediaFrame*>(argp)));
    case GRunArgP::index_of<cv::detail::VectorRef>():  return GMetaArg(util::get<cv::detail::VectorRef>(argp).descr_of());
    case GRunArgP::index_of<cv::detail::OpaqueRef>():  return GMetaArg(util::get<cv::detail::OpaqueRef>(argp).descr_of());
    default: util::throw_error(std::logic_error("Unsupported GRunArgP type"));
    }
}

// FIXME: Is it tested for all types??
bool cv::can_describe(const GMetaArg& meta, const GRunArgP& argp)
{
    switch (argp.index())
    {
#if !defined(GAPI_STANDALONE)
    case GRunArgP::index_of<cv::UMat*>():              return meta == GMetaArg(cv::descr_of(*util::get<cv::UMat*>(argp)));
#endif //  !defined(GAPI_STANDALONE)
    case GRunArgP::index_of<cv::Mat*>():               return util::holds_alternative<GMatDesc>(meta) &&
                                                              util::get<GMatDesc>(meta).canDescribe(*util::get<cv::Mat*>(argp));
    case GRunArgP::index_of<cv::Scalar*>():            return meta == GMetaArg(cv::descr_of(*util::get<cv::Scalar*>(argp)));
    case GRunArgP::index_of<cv::MediaFrame*>():        return meta == GMetaArg(cv::descr_of(*util::get<cv::MediaFrame*>(argp)));
    case GRunArgP::index_of<cv::detail::VectorRef>():  return meta == GMetaArg(util::get<cv::detail::VectorRef>(argp).descr_of());
    case GRunArgP::index_of<cv::detail::OpaqueRef>():  return meta == GMetaArg(util::get<cv::detail::OpaqueRef>(argp).descr_of());
    default: util::throw_error(std::logic_error("Unsupported GRunArgP type"));
    }
}

// FIXME: Is it tested for all types??
bool cv::can_describe(const GMetaArg& meta, const GRunArg& arg)
{
    switch (arg.index())
    {
#if !defined(GAPI_STANDALONE)
    case GRunArg::index_of<cv::UMat>():              return meta == cv::GMetaArg(descr_of(util::get<cv::UMat>(arg)));
#endif //  !defined(GAPI_STANDALONE)
    case GRunArg::index_of<cv::Mat>():               return util::holds_alternative<GMatDesc>(meta) &&
                                                            util::get<GMatDesc>(meta).canDescribe(util::get<cv::Mat>(arg));
    case GRunArg::index_of<cv::Scalar>():            return meta == cv::GMetaArg(descr_of(util::get<cv::Scalar>(arg)));
    case GRunArg::index_of<cv::detail::VectorRef>(): return meta == cv::GMetaArg(util::get<cv::detail::VectorRef>(arg).descr_of());
    case GRunArg::index_of<cv::detail::OpaqueRef>(): return meta == cv::GMetaArg(util::get<cv::detail::OpaqueRef>(arg).descr_of());
    case GRunArg::index_of<cv::gapi::wip::IStreamSource::Ptr>(): return util::holds_alternative<GMatDesc>(meta); // FIXME(?) may be not the best option
    case GRunArg::index_of<cv::RMat>():              return util::holds_alternative<GMatDesc>(meta) &&
                                                            util::get<GMatDesc>(meta).canDescribe(cv::util::get<cv::RMat>(arg));
    case GRunArg::index_of<cv::MediaFrame>():        return meta == cv::GMetaArg(util::get<cv::MediaFrame>(arg).desc());
    default: util::throw_error(std::logic_error("Unsupported GRunArg type"));
    }
}

bool cv::can_describe(const GMetaArgs &metas, const GRunArgs &args)
{
    return metas.size() == args.size() &&
           std::equal(metas.begin(), metas.end(), args.begin(),
                     [](const GMetaArg& meta, const GRunArg& arg) {
                         return can_describe(meta, arg);
                     });
}

void cv::gimpl::proto::validate_input_meta_arg(const cv::GMetaArg& meta)
{
    switch (meta.index())
    {
        case cv::GMetaArg::index_of<cv::GMatDesc>():
        {
            cv::gimpl::proto::validate_input_meta(cv::util::get<GMatDesc>(meta)); //may throw
            break;
        }
        default:
            break;
    }
}

void cv::gimpl::proto::validate_input_meta(const cv::GMatDesc& meta)
{
    if (meta.dims.empty())
    {
        if (!(meta.size.height > 0 && meta.size.width > 0))
        {
            cv::util::throw_error
                (std::logic_error(
                 "Image format is invalid. Size must contain positive values"
                 ", got width: " + std::to_string(meta.size.width ) +
                 (", height: ") + std::to_string(meta.size.height)));
        }

        if (!(meta.chan > 0))
        {
            cv::util::throw_error
                (std::logic_error(
                 "Image format is invalid. Channel mustn't be negative value, got channel: " +
                 std::to_string(meta.chan)));
        }
    }

    if (!(meta.depth >= 0))
    {
        cv::util::throw_error
            (std::logic_error(
             "Image format is invalid. Depth must be positive value, got depth: " +
             std::to_string(meta.depth)));
    }
    // All checks are ok
}

// FIXME: Is it tested for all types?
// FIXME: Where does this validation happen??
void cv::validate_input_arg(const GRunArg& arg)
{
    // FIXME: It checks only Mat argument
    switch (arg.index())
    {
#if !defined(GAPI_STANDALONE)
    case GRunArg::index_of<cv::UMat>():
    {
        const auto desc = cv::descr_of(util::get<cv::UMat>(arg));
        cv::gimpl::proto::validate_input_meta(desc); //may throw
        break;
    }
#endif //  !defined(GAPI_STANDALONE)
    case GRunArg::index_of<cv::Mat>():
    {
        const auto desc = cv::descr_of(util::get<cv::Mat>(arg));
        cv::gimpl::proto::validate_input_meta(desc); //may throw
        break;
    }
    default:
        // No extra handling
        break;
    }
}

void cv::validate_input_args(const GRunArgs& args)
{
    for (const auto& arg : args)
    {
        validate_input_arg(arg);
    }
}

namespace cv {
std::ostream& operator<<(std::ostream& os, const cv::GMetaArg &arg)
{
    // FIXME: Implement via variant visitor
    switch (arg.index())
    {
    case cv::GMetaArg::index_of<util::monostate>():
        os << "(unresolved)";
        break;

    case cv::GMetaArg::index_of<cv::GMatDesc>():
        os << util::get<cv::GMatDesc>(arg);
        break;

    case cv::GMetaArg::index_of<cv::GScalarDesc>():
        os << util::get<cv::GScalarDesc>(arg);
        break;

    case cv::GMetaArg::index_of<cv::GArrayDesc>():
        os << util::get<cv::GArrayDesc>(arg);
        break;

    case cv::GMetaArg::index_of<cv::GOpaqueDesc>():
        os << util::get<cv::GOpaqueDesc>(arg);
        break;

    case cv::GMetaArg::index_of<cv::GFrameDesc>():
        os << util::get<cv::GFrameDesc>(arg);
        break;

    default:
        GAPI_Error("InternalError");
    }

    return os;
}
} // namespace cv

const void* cv::gimpl::proto::ptr(const GRunArgP &arg)
{
    switch (arg.index())
    {
#if !defined(GAPI_STANDALONE)
    case GRunArgP::index_of<cv::UMat*>():
        return static_cast<const void*>(cv::util::get<cv::UMat*>(arg));
#endif
    case GRunArgP::index_of<cv::Mat*>():
        return static_cast<const void*>(cv::util::get<cv::Mat*>(arg));
    case GRunArgP::index_of<cv::Scalar*>():
        return static_cast<const void*>(cv::util::get<cv::Scalar*>(arg));
    case GRunArgP::index_of<cv::RMat*>():
        return static_cast<const void*>(cv::util::get<cv::RMat*>(arg));
    case GRunArgP::index_of<cv::detail::VectorRef>():
        return cv::util::get<cv::detail::VectorRef>(arg).ptr();
    case GRunArgP::index_of<cv::detail::OpaqueRef>():
        return cv::util::get<cv::detail::OpaqueRef>(arg).ptr();
    case GRunArgP::index_of<cv::MediaFrame*>():
        return static_cast<const void*>(cv::util::get<cv::MediaFrame*>(arg));
    default:
        util::throw_error(std::logic_error("Unknown GRunArgP type!"));
    }
}
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
- `api/gorigin.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi/gproto.hpp`
- `api/gproto_priv.hpp`
- `opencv2/gapi/util/throw.hpp`
- `precomp.hpp`
- `ade/util/algorithm.hpp`


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

