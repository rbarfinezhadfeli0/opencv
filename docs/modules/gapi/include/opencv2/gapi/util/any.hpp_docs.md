# Documentation for `modules/gapi/include/opencv2/gapi/util/any.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/util/any.hpp`
- **File Name**: `any.hpp`
- **File Size**: 4,533 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/util/any.hpp](../../../../../../modules/gapi/include/opencv2/gapi/util/any.hpp)

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


#ifndef OPENCV_GAPI_UTIL_ANY_HPP
#define OPENCV_GAPI_UTIL_ANY_HPP

#include <memory>
#include <type_traits>
#include <typeinfo>
#include <utility>

#include <opencv2/gapi/util/throw.hpp>

#if defined(_MSC_VER)
   // disable MSVC warning on "multiple copy constructors specified"
#  pragma warning(disable: 4521)
#endif

namespace cv
{

namespace internal
{
    template <class T, class Source>
    T down_cast(Source operand)
    {
#if defined(__GXX_RTTI) || defined(_CPPRTTI)
       return dynamic_cast<T>(operand);
#else
#ifdef __GNUC__
#warning used static cast instead of dynamic because RTTI is disabled
#else
#pragma message("WARNING: used static cast instead of dynamic because RTTI is disabled")
#endif
       return static_cast<T>(operand);
#endif
    }
}

namespace util
{
   class bad_any_cast : public std::bad_cast
   {
   public:
       virtual const char* what() const noexcept override
       {
           return "Bad any cast";
       }
   };

   //modeled against C++17 std::any

   class any
   {
   private:
      struct holder;
      using holder_ptr = std::unique_ptr<holder>;
      struct holder
      {
         virtual holder_ptr clone() = 0;
         virtual ~holder() = default;
      };

      template <typename value_t>
      struct holder_impl : holder
      {
         value_t v;
         template<typename arg_t>
         holder_impl(arg_t&& a) : v(std::forward<arg_t>(a)) {}
         holder_ptr clone() override { return holder_ptr(new holder_impl (v));}
      };

      holder_ptr hldr;
   public:
      template<class value_t>
      any(value_t&& arg) :  hldr(new holder_impl<typename std::decay<value_t>::type>( std::forward<value_t>(arg))) {}

      any(any const& src) : hldr( src.hldr ? src.hldr->clone() : nullptr) {}
      //simple hack in order not to write enable_if<not any> for the template constructor
      any(any & src) : any (const_cast<any const&>(src)) {}

      any()       = default;
      any(any&& ) = default;

      any& operator=(any&&) = default;

      any& operator=(any const& src)
      {
         any copy(src);
         swap(*this, copy);
         return *this;
      }

      template<class value_t>
      friend value_t* any_cast(any* operand);

      template<class value_t>
      friend const value_t* any_cast(const any* operand);

      template<class value_t>
      friend value_t& unsafe_any_cast(any& operand);

      template<class value_t>
      friend const value_t& unsafe_any_cast(const any& operand);

      friend void swap(any & lhs, any& rhs)
      {
         swap(lhs.hldr, rhs.hldr);
      }

   };

   template<class value_t>
   value_t* any_cast(any* operand)
   {
      auto casted = internal::down_cast<any::holder_impl<typename std::decay<value_t>::type> *>(operand->hldr.get());
      if (casted){
         return & (casted->v);
      }
      return nullptr;
   }

   template<class value_t>
   const value_t* any_cast(const any* operand)
   {
      auto casted = internal::down_cast<any::holder_impl<typename std::decay<value_t>::type> *>(operand->hldr.get());
      if (casted){
         return & (casted->v);
      }
      return nullptr;
   }

   template<class value_t>
   value_t& any_cast(any& operand)
   {
      auto ptr = any_cast<value_t>(&operand);
      if (ptr)
      {
         return *ptr;
      }

      throw_error(bad_any_cast());
   }


   template<class value_t>
   const value_t& any_cast(const any& operand)
   {
      auto ptr = any_cast<value_t>(&operand);
      if (ptr)
      {
         return *ptr;
      }

      throw_error(bad_any_cast());
   }

   template<class value_t>
   inline value_t& unsafe_any_cast(any& operand)
   {
#ifdef DEBUG
      return any_cast<value_t>(operand);
#else
      return static_cast<any::holder_impl<typename std::decay<value_t>::type> *>(operand.hldr.get())->v;
#endif
   }

   template<class value_t>
   inline const value_t& unsafe_any_cast(const any& operand)
   {
#ifdef DEBUG
      return any_cast<value_t>(operand);
#else
      return static_cast<any::holder_impl<typename std::decay<value_t>::type> *>(operand.hldr.get())->v;
#endif
   }

} // namespace util
} // namespace cv

#if defined(_MSC_VER)
   // Enable "multiple copy constructors specified" back
#  pragma warning(default: 4521)
#endif

#endif // OPENCV_GAPI_UTIL_ANY_HPP
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

- **holder**: A class/struct defined in this file
- **any**: A class/struct defined in this file
- **holder_impl**: A class/struct defined in this file
- **value_t**: A class/struct defined in this file
- **Source**: A class/struct defined in this file
- **T**: A class/struct defined in this file
- **bad_any_cast**: A class/struct defined in this file

### Functions and Methods

- **__GNUC__()**: A function/method defined in this file
- **OPENCV_GAPI_UTIL_ANY_HPP()**: A function/method defined in this file
- **DEBUG()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `utility`
- `typeinfo`
- `memory`
- `type_traits`
- `opencv2/gapi/util/throw.hpp`


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

