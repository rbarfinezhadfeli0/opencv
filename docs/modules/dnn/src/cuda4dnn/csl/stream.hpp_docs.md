# Documentation for `modules/dnn/src/cuda4dnn/csl/stream.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/csl/stream.hpp`
- **File Name**: `stream.hpp`
- **File Size**: 5,561 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/csl/stream.hpp](../../../../../modules/dnn/src/cuda4dnn/csl/stream.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/csl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_CSL_STREAM_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_CSL_STREAM_HPP

#include "error.hpp"

#include <opencv2/core.hpp>
#include <opencv2/core/utils/logger.hpp>

#include <cuda_runtime_api.h>

#include <memory>
#include <sstream>
#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn { namespace csl {

    /** \file stream.hpp
     *
     * Default streams are not supported as they limit flexiblity. All operations are always
     * carried out in non-default streams in the CUDA backend. The stream classes sacrifice
     * the ability to support default streams in exchange for better error detection. That is,
     * a default constructed stream represents no stream and any attempt to use it will throw an
     * exception.
     */

    /** @brief non-copyable smart CUDA stream
     *
     * UniqueStream is a smart non-sharable wrapper for CUDA stream handle which ensures that
     * the handle is destroyed after use. Unless explicitly specified by a constructor argument,
     * the stream object does not represent any stream by default.
     */
    class UniqueStream {
    public:
        UniqueStream() noexcept : stream{ 0 } { }
        UniqueStream(UniqueStream&) = delete;
        UniqueStream(UniqueStream&& other) noexcept {
            stream = other.stream;
            other.stream = 0;
        }

        /** creates a non-default stream if `create` is true; otherwise, no stream is created */
        UniqueStream(bool create) : stream{ 0 } {
            if (create) {
                /* we create non-blocking streams to avoid inrerruptions from users using the default stream */
                CUDA4DNN_CHECK_CUDA(cudaStreamCreateWithFlags(&stream, cudaStreamNonBlocking));
            }
        }

        ~UniqueStream() {
            try {
                /* cudaStreamDestroy does not throw if a valid stream is passed unless a previous
                 * asynchronous operation errored.
                 */
                if (stream != 0)
                    CUDA4DNN_CHECK_CUDA(cudaStreamDestroy(stream));
            } catch (const CUDAException& ex) {
                std::ostringstream os;
                os << "Asynchronous exception caught during CUDA stream destruction.\n";
                os << ex.what();
                os << "Exception will be ignored.\n";
                CV_LOG_WARNING(0, os.str().c_str());
            }
        }

        UniqueStream& operator=(const UniqueStream&) = delete;
        UniqueStream& operator=(UniqueStream&& other) noexcept {
            CV_Assert(other);
            if (&other != this) {
                UniqueStream(std::move(*this)); /* destroy current stream */
                stream = other.stream;
                other.stream = 0;
            }
            return *this;
        }

        /** returns the raw CUDA stream handle */
        cudaStream_t get() const noexcept {
            CV_Assert(stream);
            return stream;
        }

        /** blocks the calling thread until all pending operations in the stream finish */
        void synchronize() const {
            CV_Assert(stream);
            CUDA4DNN_CHECK_CUDA(cudaStreamSynchronize(stream));
        }

        /** returns true if there are pending operations in the stream */
        bool busy() const {
            CV_Assert(stream);

            auto status = cudaStreamQuery(stream);
            if (status == cudaErrorNotReady)
                return true;
            CUDA4DNN_CHECK_CUDA(status);
            return false;
        }

        /** returns true if the stream is valid */
        explicit operator bool() const noexcept { return static_cast<bool>(stream); }

    private:
        cudaStream_t stream;
    };

    /** @brief sharable smart CUDA stream
     *
     * Stream is a smart sharable wrapper for CUDA stream handle which ensures that
     * the handle is destroyed after use. Unless explicitly specified in the constructor,
     * the stream object represents no stream.
     */
    class Stream {
    public:
        Stream() { }
        Stream(const Stream&) = default;
        Stream(Stream&&) = default;

        /** if \p create is `true`, a new stream will be created; otherwise, no stream is created */
        Stream(bool create) {
            if (create)
                stream = std::make_shared<UniqueStream>(create);
        }

        Stream& operator=(const Stream&) = default;
        Stream& operator=(Stream&&) = default;

        /** blocks the caller thread until all operations in the stream are complete */
        void synchronize() const {
            CV_Assert(stream);
            stream->synchronize();
        }

        /** returns true if there are operations pending in the stream */
        bool busy() const {
            CV_Assert(stream);
            return stream->busy();
        }

        /** returns true if the object points has a valid stream */
        explicit operator bool() const noexcept {
            if (!stream)
                return false;
            return stream->operator bool();
        }

        cudaStream_t get() const noexcept {
            CV_Assert(stream);
            return stream->get();
        }

    private:
        std::shared_ptr<UniqueStream> stream;
    };

}}}} /* namespace cv::dnn::cuda4dnn::csl */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_CSL_STREAM_HPP */
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

- **Stream**: A class/struct defined in this file
- **UniqueStream**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_CSL_STREAM_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `error.hpp`
- `opencv2/core/utils/logger.hpp`
- `memory`
- `sstream`
- `opencv2/core.hpp`
- `cuda_runtime_api.h`

**Python Imports:**
- `users`


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

