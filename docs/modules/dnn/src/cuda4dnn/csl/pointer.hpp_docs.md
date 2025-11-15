# Documentation for `modules/dnn/src/cuda4dnn/csl/pointer.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/cuda4dnn/csl/pointer.hpp`
- **File Name**: `pointer.hpp`
- **File Size**: 16,644 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/cuda4dnn/csl/pointer.hpp](../../../../../modules/dnn/src/cuda4dnn/csl/pointer.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/cuda4dnn/csl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_DNN_SRC_CUDA4DNN_CSL_POINTER_HPP
#define OPENCV_DNN_SRC_CUDA4DNN_CSL_POINTER_HPP

#include "nvcc_defs.hpp"
#include "error.hpp"
#include "stream.hpp"

#include <opencv2/core.hpp>

#include <cuda_runtime_api.h>

#include <cstddef>
#include <type_traits>
#include <ostream>

namespace cv { namespace dnn { namespace cuda4dnn { namespace csl {

    /** @brief provides a type-safe device pointer
     *
     * DevicePtr wraps a raw pointer and mimics its behaviour. It does not implicitly convert
     * to a raw pointer. This ensures that accidental mixing of host and device pointers do not happen.
     *
     * It is meant to point to locations in device memory. Hence, it provides dereferencing or
     * array subscript capability for device code only.
     *
     * A `const DevicePtr<T>` represents an immutable pointer to a mutable memory.
     * A `DevicePtr<const T>` represents a mutable pointer to an immutable memory.
     * A `const DevicePtr<const T>` represents an immutable pointer to an immutable memory.
     *
     * A `DevicePtr<T>` can implicitly convert to `DevicePtr<const T>`.
     *
     * Specializations:
     * - DevicePtr<void>/DevicePtr<const void> do not support pointer arithmetic (but relational operators are provided)
     * - any device pointer pointing to mutable memory is implicitly convertible to DevicePtr<void>
     * - any device pointer is implicitly convertible to DevicePtr<const void>
     * - DevicePtr<void> can be explicitly converted to any device pointer
     * - DevicePtr<const void> can be explicitly converted to any device pointer pointing to immutable memory
     */
    template <class T>
    class DevicePtr {
        static_assert(std::is_standard_layout<T>::value, "T must satisfy StandardLayoutType");

    public:
        using element_type = T;
        using difference_type = std::ptrdiff_t;
        using pointer = typename std::add_pointer<element_type>::type;
        using reference = typename std::add_lvalue_reference<element_type>::type;

        DevicePtr() = default;
        CUDA4DNN_HOST_DEVICE explicit DevicePtr(pointer ptr_) noexcept : ptr{ ptr_ } { }

        CUDA4DNN_HOST_DEVICE DevicePtr operator=(pointer ptr_) noexcept { ptr = ptr_; return *this; }

        CUDA4DNN_HOST_DEVICE pointer get() const noexcept { return ptr; };

        CUDA4DNN_DEVICE reference operator[](difference_type idx) const noexcept { return get()[idx]; }
        CUDA4DNN_DEVICE reference operator*() const noexcept { return *get(); }
        CUDA4DNN_DEVICE pointer operator->() const noexcept { return get(); }

        template<class U = T, typename std::enable_if<!std::is_const<U>::value, bool>::type = true>
        CUDA4DNN_HOST_DEVICE operator DevicePtr<typename std::add_const<U>::type>() const noexcept {
            return DevicePtr<typename std::add_const<U>::type>{ptr};
        }

        CUDA4DNN_HOST_DEVICE explicit operator bool() const noexcept { return ptr; }

        CUDA4DNN_HOST_DEVICE DevicePtr operator++() noexcept {
            ++ptr;
            return *this;
        }

        CUDA4DNN_HOST_DEVICE DevicePtr operator++(int) noexcept {
            auto tmp = DevicePtr(*this);
            ptr++;
            return tmp;
        }

        CUDA4DNN_HOST_DEVICE DevicePtr operator--() noexcept {
            --ptr;
            return *this;
        }

        CUDA4DNN_HOST_DEVICE DevicePtr operator--(int) noexcept {
            auto tmp = DevicePtr(*this);
            ptr--;
            return tmp;
        }

        CUDA4DNN_HOST_DEVICE DevicePtr operator+=(std::ptrdiff_t offset) noexcept {
            ptr += offset;
            return *this;
        }

        CUDA4DNN_HOST_DEVICE DevicePtr operator-=(std::ptrdiff_t offset) noexcept {
            ptr -= offset;
            return *this;
        }

        CUDA4DNN_HOST_DEVICE friend DevicePtr operator+(DevicePtr lhs, std::ptrdiff_t offset) noexcept {
            return lhs += offset;
        }

        CUDA4DNN_HOST_DEVICE friend DevicePtr operator-(DevicePtr lhs, std::ptrdiff_t offset) noexcept {
            return lhs -= offset;
        }

        CUDA4DNN_HOST_DEVICE friend difference_type operator-(DevicePtr lhs, DevicePtr rhs) noexcept {
            return lhs.ptr - rhs.ptr;
        }

        CUDA4DNN_HOST_DEVICE friend bool operator==(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr == rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator!=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs == rhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator<(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr < rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator>(DevicePtr lhs, DevicePtr rhs) noexcept { return rhs < lhs; }
        CUDA4DNN_HOST_DEVICE friend bool operator<=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(rhs < lhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator>=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs < rhs); }

        CUDA4DNN_HOST_DEVICE explicit operator pointer() const noexcept { return ptr; }

        CUDA4DNN_HOST friend void swap(DevicePtr& lhs, DevicePtr& rhs) noexcept {
            using std::swap;
            swap(lhs.ptr, rhs.ptr);
        }

        template <class U, class V>
        CUDA4DNN_HOST friend std::basic_ostream<U, V>& operator<<(std::basic_ostream<U, V>& os, DevicePtr other) {
            os << other.get() << " (device)";
            return os;
        }

    private:
        pointer ptr;
    };

    template <>
    class DevicePtr<const void> {
    public:
        using element_type = const void;
        using pointer = typename std::add_pointer<element_type>::type;

        DevicePtr() = default;

        /* host const void pointer to const void device pointer */
        CUDA4DNN_HOST_DEVICE explicit DevicePtr(pointer ptr_) noexcept : ptr{ ptr_ } { }

        /* allow any device pointer to be implicitly converted to void device pointer */
        template <class T>
        CUDA4DNN_HOST_DEVICE DevicePtr(DevicePtr<T> ptr_) noexcept : ptr{ ptr_.get() } { }

        CUDA4DNN_HOST_DEVICE DevicePtr operator=(pointer ptr_) noexcept { ptr = ptr_; return *this; }

        CUDA4DNN_HOST_DEVICE pointer get() const noexcept { return ptr; };

        CUDA4DNN_HOST_DEVICE explicit operator bool() const noexcept { return ptr; }

        CUDA4DNN_HOST_DEVICE friend bool operator==(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr == rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator!=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs == rhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator<(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr < rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator>(DevicePtr lhs, DevicePtr rhs) noexcept { return rhs < lhs; }
        CUDA4DNN_HOST_DEVICE friend bool operator<=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(rhs < lhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator>=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs < rhs); }

        /* explicit conversion into host void pointer */
        CUDA4DNN_HOST_DEVICE explicit operator pointer() const noexcept { return ptr; }

        /* const void device pointer can be explicitly casted into any const device pointer type */
        template <class T, typename std::enable_if<std::is_const<T>::value, bool>::type = true>
        CUDA4DNN_HOST_DEVICE explicit operator DevicePtr<T>() const noexcept {
            return static_cast<T*>(ptr);
        }

        CUDA4DNN_HOST friend void swap(DevicePtr& lhs, DevicePtr& rhs) noexcept {
            using std::swap;
            swap(lhs.ptr, rhs.ptr);
        }

        template <class U, class V>
        CUDA4DNN_HOST friend std::basic_ostream<U, V>& operator<<(std::basic_ostream<U, V>& os, DevicePtr other) {
            os << other.get() << " (device)";
            return os;
        }

    private:
        pointer ptr;
    };

    template <>
    class DevicePtr<void> {
    public:
        using element_type = void;
        using pointer = typename std::add_pointer<element_type>::type;

        DevicePtr() = default;

        /* host pointer to device pointer */
        CUDA4DNN_HOST_DEVICE explicit DevicePtr(pointer ptr_) noexcept : ptr{ ptr_ } { }

        /* allow any device pointer to mutable memory to be implicitly converted to void device pointer */
        template <class T, typename std::enable_if<!std::is_const<T>::value, bool>::type = false>
        CUDA4DNN_HOST_DEVICE DevicePtr(DevicePtr<T> ptr_) noexcept : ptr { ptr_.get() } { }

        CUDA4DNN_HOST_DEVICE DevicePtr operator=(pointer ptr_) noexcept { ptr = ptr_; return *this; }

        CUDA4DNN_HOST_DEVICE pointer get() const noexcept { return ptr; };

        CUDA4DNN_HOST_DEVICE operator DevicePtr<const void>() const noexcept { return DevicePtr<const void>{ptr}; }

        CUDA4DNN_HOST_DEVICE explicit operator bool() const noexcept { return ptr; }

        CUDA4DNN_HOST_DEVICE friend bool operator==(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr == rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator!=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs == rhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator<(DevicePtr lhs, DevicePtr rhs) noexcept { return lhs.ptr < rhs.ptr; }
        CUDA4DNN_HOST_DEVICE friend bool operator>(DevicePtr lhs, DevicePtr rhs) noexcept { return rhs < lhs; }
        CUDA4DNN_HOST_DEVICE friend bool operator<=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(rhs < lhs); }
        CUDA4DNN_HOST_DEVICE friend bool operator>=(DevicePtr lhs, DevicePtr rhs) noexcept { return !(lhs < rhs); }

        /* explicit conversion into host void pointer */
        CUDA4DNN_HOST_DEVICE explicit operator pointer() const noexcept { return ptr; }

        /* void device pointer can be explicitly casted into any device pointer type */
        template <class T>
        CUDA4DNN_HOST_DEVICE explicit operator DevicePtr<T>() const noexcept {
            return DevicePtr<T>(static_cast<T*>(ptr));
        }

        CUDA4DNN_HOST friend void swap(DevicePtr& lhs, DevicePtr& rhs) noexcept {
            using std::swap;
            swap(lhs.ptr, rhs.ptr);
        }

        template <class U, class V>
        CUDA4DNN_HOST friend std::basic_ostream<U, V>& operator<<(std::basic_ostream<U, V>& os, DevicePtr other) {
            os << other.get() << " (device)";
            return os;
        }

    private:
        pointer ptr;
    };

    template <class T>
    bool is_aligned(DevicePtr<const T> ptr, std::size_t alignment) {
        auto addr = reinterpret_cast<std::intptr_t>(ptr.get());
        return addr % alignment == 0;
    }

    /** copies \p n elements from \p src to \p dest4
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    host pointer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(T *dest, DevicePtr<const T> src, std::size_t n) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpy(dest, src.get(), n * sizeof(T), cudaMemcpyDefault));
    }

    /** copies \p n elements from \p src to \p dest
     *
     * \param[in]   src     host pointer
     * \param[out]  dest    device pointer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(DevicePtr<T> dest, const T* src, std::size_t n) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpy(dest.get(), src, n * sizeof(T), cudaMemcpyDefault));
    }

    /** copies \p n elements from \p src to \p dest
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    device pointer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(DevicePtr<T> dest, DevicePtr<const T> src, std::size_t n) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpy(dest.get(), src.get(), n * sizeof(T), cudaMemcpyDefault));
    }

    /** sets \p n elements to \p ch in \p dest
     *
     * \param[in]   src     device pointer
     * \param[out]  ch      8-bit value to fill the device memory with
     *
     * Pre-conditions:
     * - memory pointed by \p dest must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memset(DevicePtr<T> dest, std::int8_t ch, std::size_t n) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemset(dest.get(), ch, n * sizeof(T)));
    }

    /** copies \p n elements from \p src to \p dest asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    host pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     * - \p dest points to page-locked memory
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(T *dest, DevicePtr<const T> src, std::size_t n, const Stream& stream) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpyAsync(dest, src.get(), n * sizeof(T), cudaMemcpyDefault, stream.get()));
    }

    /** copies data from memory pointed by \p src to \p dest asynchronously
     *
     * \param[in]   src     host pointer
     * \param[out]  dest    device pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     * - \p src points to page-locked memory
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(DevicePtr<T> dest, const T *src, std::size_t n, const Stream& stream) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpyAsync(dest.get(), src, n * sizeof(T), cudaMemcpyDefault, stream.get()));
    }

    /** copies \p n elements from \p src to \p dest asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  dest    device pointer
     * \param       stream  CUDA stream that has to be used for the memory transfer
     *
     * Pre-conditions:
     * - memory pointed by \p dest and \p src must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memcpy(DevicePtr<T> dest, DevicePtr<const T> src, std::size_t n, const Stream& stream) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemcpyAsync(dest.get(), src.get(), n * sizeof(T), cudaMemcpyDefault, stream.get()));
    }

    /** sets \p n elements to \p ch in \p dest asynchronously
     *
     * \param[in]   src     device pointer
     * \param[out]  ch      8-bit value to fill the device memory with
     * \param       stream  CUDA stream that has to be used for the memory operation
     *
     * Pre-conditions:
     * - memory pointed by \p dest must be large enough to hold \p n elements
     *
     * Exception Guarantee: Basic
     */
    template <class T>
    void memset(DevicePtr<T> dest, std::int8_t ch, std::size_t n, const Stream& stream) {
        if (n <= 0) {
            CV_Error(Error::StsBadArg, "number of elements to copy is zero or negtaive");
        }

        CUDA4DNN_CHECK_CUDA(cudaMemsetAsync(dest.get(), ch, n * sizeof(T), stream.get()));
    }

}}}} /* namespace cv::dnn::cuda4dnn::csl */

#endif /* OPENCV_DNN_SRC_CUDA4DNN_CSL_POINTER_HPP */
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

- **V**: A class/struct defined in this file
- **DevicePtr**: A class/struct defined in this file
- **U**: A class/struct defined in this file
- **T**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_DNN_SRC_CUDA4DNN_CSL_POINTER_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `error.hpp`
- `stream.hpp`
- `ostream`
- `cstddef`
- `type_traits`
- `opencv2/core.hpp`
- `cuda_runtime_api.h`
- `nvcc_defs.hpp`

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

