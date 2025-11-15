# Documentation for `modules/dnn/src/cuda4dnn/csl/memory.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/csl/memory.hpp`
- **File Name**: `memory.hpp`
- **File Size**: 10,882 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/csl/memory.hpp](../../../../../modules/dnn/src/cuda4dnn/csl/memory.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/csl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_CSL_MEMORY_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_CSL_MEMORY_HPP

#include "error.hpp"
#include "pointer.hpp"

#include <opencv2/core.hpp>

#include <cuda_runtime_api.h>

#include <cstddef>
#include <type_traits>
#include <memory>
#include <utility>

namespace cv { namespace dnn { namespace cuda4dnn { namespace csl {

    /* @brief smart device pointer with allocation/deallocation methods
     *
     * ManagedPtr is a smart shared device pointer which also handles memory allocation.
     */
    template <class T>
    class ManagedPtr {
        static_assert(!std::is_const<T>::value && !std::is_volatile<T>::value, "T cannot be cv-qualified");
        static_assert(std::is_standard_layout<T>::value, "T must satisfy StandardLayoutType");

    public:
        using element_type = T;

        using pointer = DevicePtr<element_type>;
        using const_pointer = DevicePtr<typename std::add_const<element_type>::type>;

        using size_type = std::size_t;

        ManagedPtr() noexcept : wrapped{ nullptr }, n{ 0 }, capacity{ 0 } { }
        ManagedPtr(const ManagedPtr&) noexcept = default;
        ManagedPtr(ManagedPtr&& other) noexcept
            : wrapped{ std::move(other.wrapped) }, n{ other.n }, capacity { other.capacity }
        {
            other.reset();
        }

        /** allocates device memory for \p count number of element */
        ManagedPtr(size_type count) {
            if (count <= 0) {
                CV_Error(Error::StsBadArg, "number of elements is zero or negative");
            }

            void* temp = nullptr;
            CUDA4DNN_CHECK_CUDA(cudaMalloc(&temp, count * sizeof(element_type)));

            auto ptr = typename pointer::pointer(static_cast<element_type*>(temp));
            wrapped.reset(ptr, [](element_type* ptr) {
                if (ptr != nullptr) {
                    /* contract violation for std::shared_ptr if cudaFree throws */
                    try {
                        CUDA4DNN_CHECK_CUDA(cudaFree(ptr));
                    } catch (const CUDAException& ex) {
                        std::ostringstream os;
                        os << "Device memory deallocation failed in deleter.\n";
                        os << ex.what();
                        os << "Exception will be ignored.\n";
                        CV_LOG_WARNING(0, os.str().c_str());
                    }
                }
            });
            /* std::shared_ptr<T>::reset invokes the deleter if an exception occurs; hence, we don't
             * need to have a try-catch block to free the allocated device memory
             */

            n = capacity = count;
        }

        ManagedPtr& operator=(ManagedPtr&& other) noexcept {
            wrapped = std::move(other.wrapped);
            n = other.n;
            capacity = other.capacity;

            other.reset();
            return *this;
        }

        size_type size() const noexcept { return n; }

        void reset() noexcept { wrapped.reset(); n = capacity = 0; }

        /**
         * deallocates any previously allocated memory and allocates device memory
         * for \p count number of elements
         *
         * @note no reallocation if the previously allocated memory has no owners and the requested memory size fits in it
         * @note use move constructor to guarantee a deallocation of the previously allocated memory
         *
         * Exception Guarantee: Strong
         */
        void reset(size_type count) {
            /* we need to fully own the memory to perform optimizations */
            if (wrapped.use_count() == 1) {
                /* avoid reallocation if the existing capacity is sufficient */
                if (count <= capacity) {
                    n = count;
                    return;
                }
            }

            /* no optimization performed; allocate memory */
            ManagedPtr tmp(count);
            swap(tmp, *this);
        }

        pointer get() const noexcept { return pointer(wrapped.get()); }

        explicit operator bool() const noexcept { return wrapped; }

        friend bool operator==(const ManagedPtr& lhs, const ManagedPtr& rhs) noexcept { return lhs.wrapped == rhs.wrapped; }
        friend bool operator!=(const ManagedPtr& lhs, const ManagedPtr& rhs) noexcept { return lhs.wrapped != rhs.wrapped; }

        friend void swap(ManagedPtr& lhs, ManagedPtr& rhs) noexcept {
            using std::swap;
            swap(lhs.wrapped, rhs.wrapped);
            swap(lhs.n, rhs.n);
            swap(lhs.capacity, rhs.capacity);
        }

    private:
        std::shared_ptr<element_type> wrapped;
        size_type n, capacity;
    };

    /** copies entire memory block pointed by \p src to \p dest
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    host pointer
     *
     * Pre-conditions:
     * - memory pointed by \p dest must be large enough to hold the entire block of memory held by \p src
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(T *dest, const ManagedPtr<T>& src) {
        memcpy<T>(dest, src.get(), src.size());
    }

    /** copies data from memory pointed by \p src to fully fill \p dest
     *
     * \param[in]   src     host pointer
     * \param[out]  dest    device pointer
     *
     * Pre-conditions:
     * - memory pointed by \p src must be at least as big as the memory block held by \p dest
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(const ManagedPtr<T>& dest, const T* src) {
        memcpy<T>(dest.get(), src, dest.size());
    }

    /** copies data from memory pointed by \p src to \p dest
     *
     * if the two \p src and \p  dest have different sizes, the number of elements copied is
     * equal to the size of the smaller memory block
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    device pointer
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(const ManagedPtr<T>& dest, const ManagedPtr<T>& src) {
        memcpy<T>(dest.get(), src.get(), std::min(dest.size(), src.size()));
    }

    /** sets device memory block to a specific 8-bit value
     *
     * \param[in]   src     device pointer
     * \param[out]  ch      8-bit value to fill the device memory with
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memset(const ManagedPtr<T>& dest, std::int8_t ch) {
        memset<T>(dest.get(), ch, dest.size());
    }

    /** copies entire memory block pointed by \p src to \p dest asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    host pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * Pre-conditions:
     * - memory pointed by \p dest must be large enough to hold the entire block of memory held by \p src
     * - \p dest points to page-locked memory
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(T *dest, const ManagedPtr<T>& src, const Stream& stream) {
        CV_Assert(stream);
        memcpy<T>(dest, src.get(), src.size(), stream);
    }

    /** copies data from memory pointed by \p src to \p dest asynchronously
     *
     * \param[in]   src     host pointer
     * \param[out]  dest    device pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * Pre-conditions:
     * - memory pointed by \p dest must be large enough to hold the entire block of memory held by \p src
     * - \p src points to page-locked memory
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(const ManagedPtr<T>& dest, const T* src, const Stream& stream) {
        CV_Assert(stream);
        memcpy<T>(dest.get(), src, dest.size(), stream);
    }

    /** copies data from memory pointed by \p src to \p dest asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    device pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * if the two \p src and \p  dest have different sizes, the number of elements copied is
     * equal to the size of the smaller memory block
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(ManagedPtr<T>& dest, const ManagedPtr<T>& src, const Stream& stream) {
        CV_Assert(stream);
        memcpy<T>(dest.get(), src.get(), std::min(dest.size(), src.size()), stream);
    }

    /** sets device memory block to a specific 8-bit value asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  ch      8-bit value to fill the device memory with
     * \param       stream  CUDA stream that has to be used for the memory operation
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memset(const ManagedPtr<T>& dest, int ch, const Stream& stream) {
        CV_Assert(stream);
        memset<T>(dest.get(), ch, dest.size(), stream);
    }

    /** @brief registers host memory as page-locked and unregisters on destruction */
    class MemoryLockGuard {
    public:
        MemoryLockGuard() noexcept : ptr { nullptr } { }
        MemoryLockGuard(const MemoryLockGuard&) = delete;
        MemoryLockGuard(MemoryLockGuard&& other) noexcept : ptr{ other.ptr } {
            other.ptr = nullptr;
        }

        /** page-locks \p size_in_bytes bytes of memory starting from \p ptr_
         *
         * Pre-conditions:
         * - host memory should be unregistered
         */
        MemoryLockGuard(void* ptr_, std::size_t size_in_bytes) {
            CUDA4DNN_CHECK_CUDA(cudaHostRegister(ptr_, size_in_bytes, cudaHostRegisterPortable));
            ptr = ptr_;
        }

        MemoryLockGuard& operator=(const MemoryLockGuard&) = delete;
        MemoryLockGuard& operator=(MemoryLockGuard&& other) noexcept {
            if (&other != this) {
                if(ptr != nullptr) {
                    /* cudaHostUnregister does not throw for a valid ptr */
                    CUDA4DNN_CHECK_CUDA(cudaHostUnregister(ptr));
                }
                ptr = other.ptr;
                other.ptr = nullptr;
            }
            return *this;
        }

        ~MemoryLockGuard() {
            if(ptr != nullptr) {
                /* cudaHostUnregister does not throw for a valid ptr */
                CUDA4DNN_CHECK_CUDA(cudaHostUnregister(ptr));
            }
        }

    private:
        void *ptr;
    };

}}}} /* namespace cv::dnn::cuda4dnn::csl */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_CSL_MEMORY_HPP */
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

- **ManagedPtr**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **MemoryLockGuard**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_CSL_MEMORY_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `error.hpp`
- `cstddef`
- `memory`
- `pointer.hpp`
- `type_traits`
- `opencv2/core.hpp`
- `cuda_runtime_api.h`

**Python Imports:**
- `memory`


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

