# Documentation for `modules/gapi/src/api/gmat.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/gmat.cpp`
- **File Name**: `gmat.cpp`
- **File Size**: 4,609 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/gmat.cpp](../../../../modules/gapi/src/api/gmat.cpp)

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

#include <ade/util/iota_range.hpp>
#include <ade/util/algorithm.hpp>

#include <opencv2/gapi/own/mat.hpp> //gapi::own::Mat
#include <opencv2/gapi/gmat.hpp>

#include "api/gorigin.hpp"

// cv::GMat public implementation //////////////////////////////////////////////
cv::GMat::GMat()
    : m_priv(new GOrigin(GShape::GMAT, GNode::Param()))
{
}

cv::GMat::GMat(const GNode &n, std::size_t out)
    : m_priv(new GOrigin(GShape::GMAT, n, out))
{
}

cv::GMat::GMat(cv::Mat m)
    : m_priv(new GOrigin(GShape::GMAT, cv::gimpl::ConstVal(m))) {
}

cv::GOrigin& cv::GMat::priv()
{
    return *m_priv;
}

const cv::GOrigin& cv::GMat::priv() const
{
    return *m_priv;
}

static std::vector<int> checkVectorImpl(const int width, const int height, const int chan,
                                        const int n)
{
    if (width == 1 && (n == -1 || n == chan))
    {
        return {height, chan};
    }
    else if (height == 1 && (n == -1 || n == chan))
    {
        return {width, chan};
    }
    else if (chan == 1 && (n == -1 || n == width))
    {
        return {height, width};
    }
    else // input Mat can't be described as vector of points of given dimensionality
    {
        return {-1, -1};
    }
}

int cv::gapi::detail::checkVector(const cv::GMatDesc& in, const size_t n)
{
    GAPI_Assert(n != 0u);
    return checkVectorImpl(in.size.width, in.size.height, in.chan, static_cast<int>(n))[0];
}

std::vector<int> cv::gapi::detail::checkVector(const cv::GMatDesc& in)
{
    return checkVectorImpl(in.size.width, in.size.height, in.chan, -1);
}

namespace{
    template <typename T> cv::GMetaArgs vec_descr_of(const std::vector<T> &vec)
        {
        cv::GMetaArgs vec_descr;
        vec_descr.reserve(vec.size());
        for(auto& mat : vec){
            vec_descr.emplace_back(descr_of(mat));
        }
        return vec_descr;
    }
}

#if !defined(GAPI_STANDALONE)
cv::GMatDesc cv::descr_of(const cv::Mat &mat)
{
    const auto mat_dims = mat.size.dims();

    if (mat_dims == 2)
        return GMatDesc{mat.depth(), mat.channels(), {mat.cols, mat.rows}};

    std::vector<int> dims(mat_dims);
    for (auto i : ade::util::iota(mat_dims)) {
        // Note: cv::MatSize is not iterable
        dims[i] = mat.size[i];
    }
    return GMatDesc{mat.depth(), std::move(dims)};
}
#endif

cv::GMatDesc cv::gapi::own::descr_of(const Mat &mat)
{
    return (mat.dims.empty())
        ? GMatDesc{mat.depth(), mat.channels(), {mat.cols, mat.rows}}
        : GMatDesc{mat.depth(), mat.dims};
}

#if !defined(GAPI_STANDALONE)
cv::GMatDesc cv::descr_of(const cv::UMat &mat)
{
    GAPI_Assert(mat.size.dims() == 2);
    return GMatDesc{ mat.depth(), mat.channels(),{ mat.cols, mat.rows } };
}

cv::GMetaArgs cv::descrs_of(const std::vector<cv::UMat> &vec)
{
    return vec_descr_of(vec);
}
#endif

cv::GMetaArgs cv::descrs_of(const std::vector<cv::Mat> &vec)
{
    return vec_descr_of(vec);
}

cv::GMetaArgs cv::gapi::own::descrs_of(const std::vector<Mat> &vec)
{
    return vec_descr_of(vec);
}

cv::GMatDesc cv::descr_of(const cv::RMat &mat)
{
    return mat.desc();
}

namespace cv {
std::ostream& operator<<(std::ostream& os, const cv::GMatDesc &desc)
{
    switch (desc.depth)
    {
#define TT(X) case CV_##X: os << #X; break;
        TT(8U);
        TT(8S);
        TT(16U);
        TT(16S);
        TT(32S);
        TT(32F);
        TT(64F);
#undef TT
    default:
        os << "(user type "
           << std::hex << desc.depth << std::dec
           << ")";
        break;
    }

    if (desc.isND()) {
        os << " [";
        for (size_t i = 0; i < desc.dims.size() - 1; ++i) {
            os << desc.dims[i] << "x";
        }
        os << desc.dims.back() << "]";
    } else {
        os << "C" << desc.chan;
        if (desc.planar) os << "p";
        os << " ";
        os << desc.size.width << "x" << desc.size.height;
    }

    return os;
}

namespace {
template<typename M> inline bool canDescribeHelper(const GMatDesc& desc, const M& mat)
{
    const auto mat_desc = desc.planar ? cv::descr_of(mat).asPlanar(desc.chan) : cv::descr_of(mat);
    return desc == mat_desc;
}
} // anonymous namespace

bool GMatDesc::canDescribe(const cv::Mat& mat) const
{
    return canDescribeHelper(*this, mat);
}

bool GMatDesc::canDescribe(const cv::RMat& mat) const
{
    return canDescribeHelper(*this, mat);
}

}// namespace cv
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

- **TT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ade/util/iota_range.hpp`
- `api/gorigin.hpp`
- `opencv2/gapi/gmat.hpp`
- `opencv2/gapi/own/mat.hpp`
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

