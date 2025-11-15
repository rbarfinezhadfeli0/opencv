# Documentation for `modules/gapi/src/compiler/gcompiled.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/compiler/gcompiled.cpp`
- **File Name**: `gcompiled.cpp`
- **File Size**: 4,544 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/compiler/gcompiled.cpp](../../../../modules/gapi/src/compiler/gcompiled.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/compiler` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2020 Intel Corporation


#include "precomp.hpp"

#include <ade/graph.hpp>

#include <opencv2/gapi/gproto.hpp> // can_describe
#include <opencv2/gapi/gcompiled.hpp>

#include "compiler/gcompiled_priv.hpp"
#include "backends/common/gbackend.hpp"
#include "executor/gexecutor.hpp"

// GCompiled private implementation ////////////////////////////////////////////
void cv::GCompiled::Priv::setup(const GMetaArgs &_metaArgs,
                                const GMetaArgs &_outMetas,
                                std::unique_ptr<cv::gimpl::GAbstractExecutor> &&_pE)
{
    m_metas    = _metaArgs;
    m_outMetas = _outMetas;
    m_exec     = std::move(_pE);
}

bool cv::GCompiled::Priv::isEmpty() const
{
    return !m_exec;
}

void cv::GCompiled::Priv::run(cv::gimpl::GRuntimeArgs &&args)
{
    // Strip away types since ADE knows nothing about that
    // args will be taken by specific GBackendExecutables
    checkArgs(args);
    m_exec->run(std::move(args));
}

const cv::GMetaArgs& cv::GCompiled::Priv::metas() const
{
    return m_metas;
}

const cv::GMetaArgs& cv::GCompiled::Priv::outMetas() const
{
    return m_outMetas;
}

void cv::GCompiled::Priv::checkArgs(const cv::gimpl::GRuntimeArgs &args) const
{
    if (!can_describe(m_metas, args.inObjs))
    {
        util::throw_error(std::logic_error("This object was compiled "
                                           "for different metadata!"));
        // FIXME: Add details on what is actually wrong
    }
    validate_input_args(args.inObjs);
    // FIXME: Actually, the passed parameter vector is never checked
    // against its shapes - so if you compile with GScalarDesc passed
    // for GMat argument, you will get your compilation right (!!)
    // Probably it was there but somehow that olds checks (if they
    // exist) are bypassed now.
}

bool cv::GCompiled::Priv::canReshape() const
{
    GAPI_Assert(m_exec);
    return m_exec->canReshape();
}

void cv::GCompiled::Priv::reshape(const GMetaArgs& inMetas, const GCompileArgs& args)
{
    GAPI_Assert(m_exec);
    m_exec->reshape(inMetas, args);
    m_metas = inMetas;
}

void cv::GCompiled::Priv::prepareForNewStream()
{
    GAPI_Assert(m_exec);
    m_exec->prepareForNewStream();
}

const cv::gimpl::GModel::Graph& cv::GCompiled::Priv::model() const
{
    GAPI_Assert(nullptr != m_exec);
    return m_exec->model();
}

// GCompiled public implementation /////////////////////////////////////////////
cv::GCompiled::GCompiled()
    : m_priv(new Priv())
{
}

cv::GCompiled::operator bool() const
{
    return !m_priv->isEmpty();
}

void cv::GCompiled::operator() (GRunArgs &&ins, GRunArgsP &&outs)
{
    // FIXME: Check that <ins> matches the protocol!!!
    // FIXME: Check that <outs> matches the protocol
    m_priv->run(cv::gimpl::GRuntimeArgs{std::move(ins),std::move(outs)});
}

#if !defined(GAPI_STANDALONE)
void cv::GCompiled::operator ()(cv::Mat in, cv::Mat &out)
{
    (*this)(cv::gin(in), cv::gout(out));
}

void cv::GCompiled::operator() (cv::Mat in, cv::Scalar &out)
{
    (*this)(cv::gin(in), cv::gout(out));
}

void cv::GCompiled::operator() (cv::Mat in1, cv::Mat in2, cv::Mat &out)
{
    (*this)(cv::gin(in1, in2), cv::gout(out));
}

void cv::GCompiled::operator() (cv::Mat in1, cv::Mat in2, cv::Scalar &out)
{
    (*this)(cv::gin(in1, in2), cv::gout(out));
}

void cv::GCompiled::operator ()(const std::vector<cv::Mat> &ins,
                                const std::vector<cv::Mat> &outs)
{
    GRunArgs call_ins;
    GRunArgsP call_outs;

    // Make a temporary copy of vector outs - cv::Mats are copies anyway
    auto tmp = outs;
    for (const cv::Mat &m : ins) { call_ins.emplace_back(m);   }
    for (      cv::Mat &m : tmp) { call_outs.emplace_back(&m); }

    (*this)(std::move(call_ins), std::move(call_outs));
}
#endif // !defined(GAPI_STANDALONE)

const cv::GMetaArgs& cv::GCompiled::metas() const
{
    return m_priv->metas();
}

const cv::GMetaArgs& cv::GCompiled::outMetas() const
{
    return m_priv->outMetas();
}

cv::GCompiled::Priv& cv::GCompiled::priv()
{
    return *m_priv;
}

bool cv::GCompiled::canReshape() const
{
    return m_priv->canReshape();
}

void cv::GCompiled::reshape(const GMetaArgs& inMetas, const GCompileArgs& args)
{
    m_priv->reshape(inMetas, args);
}

void cv::GCompiled::prepareForNewStream()
{
    m_priv->prepareForNewStream();
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
- `backends/common/gbackend.hpp`
- `executor/gexecutor.hpp`
- `compiler/gcompiled_priv.hpp`
- `opencv2/gapi/gproto.hpp`
- `precomp.hpp`
- `ade/graph.hpp`
- `opencv2/gapi/gcompiled.hpp`


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

