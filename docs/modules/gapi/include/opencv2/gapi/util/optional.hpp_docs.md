# Documentation for `modules/gapi/include/opencv2/gapi/util/optional.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/util/optional.hpp`
- **File Name**: `optional.hpp`
- **File Size**: 4,643 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/util/optional.hpp](../../../../../../modules/gapi/include/opencv2/gapi/util/optional.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/util` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#ifndef OPENCV_GAPI_UTIL_OPTIONAL_HPP
#define OPENCV_GAPI_UTIL_OPTIONAL_HPP

#include <opencv2/gapi/util/variant.hpp>

// A poor man's `optional` implementation, incompletely modeled against C++17 spec.
namespace cv
{
namespace util
{
    class bad_optional_access: public std::exception
    {
    public:
        virtual const char *what() const noexcept override
        {
            return "Bad optional access";
        }
    };

    // TODO: nullopt_t

    // Interface ///////////////////////////////////////////////////////////////
    template<typename T> class optional
    {
    public:
        // Constructors
        // NB.: there were issues with Clang 3.8 when =default() was used
        // instead {}
        optional() {}
        optional(const optional&) = default;
        explicit optional(T&&) noexcept;
        explicit optional(const T&) noexcept;
        optional(optional&&) noexcept;
        // TODO: optional(nullopt_t) noexcept;
        // TODO: optional(const optional<U> &)
        // TODO: optional(optional<U> &&)
        // TODO: optional(Args&&...)
        // TODO: optional(initializer_list<U>)
        // TODO: optional(U&& value);

        // Assignment
        optional& operator=(const optional&) = default;
        optional& operator=(optional&&);

        // Observers
        T* operator-> ();
        const T* operator-> () const;
        T& operator* ();
        const T& operator* () const;
        // TODO: && versions

        operator bool() const noexcept;
        bool has_value() const noexcept;

        T& value();
        const T& value() const;
        // TODO: && versions

        template<class U>
        T value_or(U &&default_value) const;

        void swap(optional &other) noexcept;
        void reset() noexcept;
        // TODO: emplace

        // TODO: operator==, !=, <, <=, >, >=

    private:
        struct nothing {};
        util::variant<nothing, T> m_holder;
    };

    template<class T>
    optional<typename std::decay<T>::type> make_optional(T&& value);

    // TODO: Args... and initializer_list versions

    // Implementation //////////////////////////////////////////////////////////
    template<class T> optional<T>::optional(T &&v) noexcept
        : m_holder(std::move(v))
    {
    }

    template<class T> optional<T>::optional(const T &v) noexcept
        : m_holder(v)
    {
    }

    template<class T> optional<T>::optional(optional&& rhs) noexcept
        : m_holder(std::move(rhs.m_holder))
    {
        rhs.reset();
    }

    template<class T> optional<T>& optional<T>::operator=(optional&& rhs)
    {
        m_holder = std::move(rhs.m_holder);
        rhs.reset();
        return *this;
    }

    template<class T> T* optional<T>::operator-> ()
    {
        return & *(*this);
    }

    template<class T> const T* optional<T>::operator-> () const
    {
        return & *(*this);
    }

    template<class T> T& optional<T>::operator* ()
    {
        return this->value();
    }

    template<class T> const T& optional<T>::operator* () const
    {
        return this->value();
    }

    template<class T> optional<T>::operator bool() const noexcept
    {
        return this->has_value();
    }

    template<class T> bool optional<T>::has_value() const noexcept
    {
        return util::holds_alternative<T>(m_holder);
    }

    template<class T> T& optional<T>::value()
    {
        if (!this->has_value())
            throw_error(bad_optional_access());
        return util::get<T>(m_holder);
    }

    template<class T> const T& optional<T>::value() const
    {
        if (!this->has_value())
            throw_error(bad_optional_access());
        return util::get<T>(m_holder);
    }

    template<class T>
    template<class U> T optional<T>::value_or(U &&default_value) const
    {
        return (this->has_value() ? this->value() : T(default_value));
    }

    template<class T> void optional<T>::swap(optional<T> &other) noexcept
    {
        m_holder.swap(other.m_holder);
    }

    template<class T> void optional<T>::reset() noexcept
    {
        if (this->has_value())
            m_holder = nothing{};
    }

    template<class T>
    optional<typename std::decay<T>::type> make_optional(T&& value)
    {
        return optional<typename std::decay<T>::type>(std::forward<T>(value));
    }
} // namespace util
} // namespace cv

#endif // OPENCV_GAPI_UTIL_OPTIONAL_HPP
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

- **nothing**: A class/struct defined in this file
- **bad_optional_access**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **U**: A class/struct defined in this file
- **optional**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_UTIL_OPTIONAL_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/util/variant.hpp`


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

