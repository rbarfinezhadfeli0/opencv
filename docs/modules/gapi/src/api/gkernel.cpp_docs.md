# Documentation for `modules/gapi/src/api/gkernel.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/api/gkernel.cpp`
- **File Name**: `gkernel.cpp`
- **File Size**: 3,566 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/api/gkernel.cpp](../../../../modules/gapi/src/api/gkernel.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/api` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2021 Intel Corporation


#include "precomp.hpp"
#include <iostream> // cerr
#include <functional> // hash
#include <numeric> // accumulate

#include <ade/util/algorithm.hpp>

#include "logger.hpp"
#include <opencv2/gapi/gkernel.hpp>

#include "api/gbackend_priv.hpp"

// GKernelPackage public implementation ////////////////////////////////////////
void cv::GKernelPackage::remove(const cv::gapi::GBackend& backend)
{
    std::vector<std::string> id_deleted_kernels;
    for (const auto& p : m_id_kernels)
    {
        if (p.second.first == backend)
        {
            id_deleted_kernels.push_back(p.first);
        }
    }

    for (const auto& kernel_id : id_deleted_kernels)
    {
        m_id_kernels.erase(kernel_id);
    }
}

void cv::GKernelPackage::include(const cv::gapi::GFunctor& functor)
{
    m_id_kernels[functor.id()] = std::make_pair(functor.backend(), functor.impl());
}

void cv::GKernelPackage::include(const cv::gapi::GBackend& backend, const std::string& kernel_id)
{
    removeAPI(kernel_id);
    m_id_kernels[kernel_id] = std::make_pair(backend, GKernelImpl{{}, {}});
}

bool cv::GKernelPackage::includesAPI(const std::string &id) const
{
    return ade::util::contains(m_id_kernels, id);
}

void cv::GKernelPackage::removeAPI(const std::string &id)
{
    m_id_kernels.erase(id);
}

std::size_t cv::GKernelPackage::size() const
{
    return m_id_kernels.size();
}

const std::vector<cv::GTransform> &cv::GKernelPackage::get_transformations() const
{
    return m_transformations;
}

std::vector<std::string> cv::GKernelPackage::get_kernel_ids() const
{
    std::vector<std::string> ids;
    for (auto &&id : m_id_kernels)
    {
        ids.emplace_back(id.first);
    }
    return ids;
}

cv::GKernelPackage cv::gapi::combine(const cv::GKernelPackage  &lhs,
                                     const cv::GKernelPackage  &rhs)
{

        // If there is a collision, prefer RHS to LHS
        // since RHS package has a precedense, start with its copy
        cv::GKernelPackage result(rhs);
        // now iterate over LHS package and put kernel if and only
        // if there's no such one
        for (const auto& kernel : lhs.m_id_kernels)
        {
            if (!result.includesAPI(kernel.first))
            {
                result.m_id_kernels.emplace(kernel.first, kernel.second);
            }
        }
        for (const auto &transforms : lhs.m_transformations){
            result.m_transformations.push_back(transforms);
        }
        return result;
}

std::pair<cv::gapi::GBackend, cv::GKernelImpl>
cv::GKernelPackage::lookup(const std::string &id) const
{
    auto kernel_it = m_id_kernels.find(id);
    if (kernel_it != m_id_kernels.end())
    {
        return kernel_it->second;
    }
    // If reached here, kernel was not found.
    util::throw_error(std::logic_error("Kernel " + id + " was not found"));
}

std::vector<cv::gapi::GBackend> cv::GKernelPackage::backends() const
{
    using kernel_type = std::pair<std::string, std::pair<cv::gapi::GBackend, cv::GKernelImpl>>;
    std::unordered_set<cv::gapi::GBackend> unique_set;
    ade::util::transform(m_id_kernels, std::inserter(unique_set, unique_set.end()),
                                       [](const kernel_type& k) { return k.second.first; });

    return std::vector<cv::gapi::GBackend>(unique_set.begin(), unique_set.end());
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
- `functional`
- `opencv2/gapi/gkernel.hpp`
- `iostream`
- `api/gbackend_priv.hpp`
- `precomp.hpp`
- `logger.hpp`
- `ade/util/algorithm.hpp`
- `numeric`


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

