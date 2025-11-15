# Documentation for `modules/gapi/include/opencv2/gapi/streaming/queue_source.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/queue_source.hpp`
- **File Name**: `queue_source.hpp`
- **File Size**: 1,905 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/queue_source.hpp](../../../../../../modules/gapi/include/opencv2/gapi/streaming/queue_source.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2023 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_QUEUE_SOURCE_HPP
#define OPENCV_GAPI_STREAMING_QUEUE_SOURCE_HPP

#include <memory>                      // shared_ptr
#include <type_traits>                 // is_base_of

#include <opencv2/gapi/garg.hpp>       // GRunArgs
#include <opencv2/gapi/gmetaarg.hpp>   // GMetaArg + all descr_of
#include <opencv2/gapi/streaming/source.hpp> // IStreamSource

namespace cv {
namespace gapi {
namespace wip {
struct Data; // fwd-declare to avoid circular? header dependencies

class GAPI_EXPORTS QueueSourceBase: public cv::gapi::wip::IStreamSource {
    class Priv;
    std::shared_ptr<Priv> m_priv;
    // FIXME: Need to understand how it works with IStreamSource's shared_from_this
    // Can we avoid having too many shared_ptrs here?

public:
    explicit QueueSourceBase(const cv::GMetaArg &m);
    void push(Data &&data);
    virtual bool pull(Data &data) override;
    virtual void halt() override;
    virtual GMetaArg descr_of() const override;
    virtual ~QueueSourceBase() = default;
};

/**
 * @brief Queued streaming pipeline source.
 *
 */
template<class T>
class QueueSource final: public QueueSourceBase
{
public:
    using Meta = decltype(cv::descr_of(T{}));
    explicit QueueSource(Meta m) : QueueSourceBase(GMetaArg{m}) {
    }
    void push(T t) {
        QueueSourceBase::push(Data{t});
    }
};

class GAPI_EXPORTS QueueInput {
    std::vector<std::shared_ptr<QueueSourceBase> > m_sources;

public:
    explicit QueueInput(const cv::GMetaArgs &args);

    void push(cv::GRunArgs &&ins);
    operator cv::GRunArgs();
};

} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STREAMING_SOURCE_HPP
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
- **Priv**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **QueueSource**: A class/struct defined in this file
- **Data**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_QUEUE_SOURCE_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/gmetaarg.hpp`
- `opencv2/gapi/garg.hpp`
- `memory`
- `type_traits`
- `opencv2/gapi/streaming/source.hpp`


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

