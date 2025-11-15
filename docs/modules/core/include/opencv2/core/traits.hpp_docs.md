# Documentation for `modules/core/include/opencv2/core/traits.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/traits.hpp`
- **File Name**: `traits.hpp`
- **File Size**: 12,293 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/traits.hpp](../../../../../modules/core/include/opencv2/core/traits.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                          License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_CORE_TRAITS_HPP
#define OPENCV_CORE_TRAITS_HPP

#include "opencv2/core/cvdef.h"

namespace cv
{

//#define OPENCV_TRAITS_ENABLE_DEPRECATED

//! @addtogroup core_basic
//! @{

/** @brief Template "trait" class for OpenCV primitive data types.

@note Deprecated. This is replaced by "single purpose" traits: traits::Type and traits::Depth

A primitive OpenCV data type is one of unsigned char, bool, signed char, unsigned short, signed
short, int, float, double, or a tuple of values of one of these types, where all the values in the
tuple have the same type. Any primitive type from the list can be defined by an identifier in the
form CV_\<bit-depth\>{U|S|F}C(\<number_of_channels\>), for example: uchar \~ CV_8UC1, 3-element
floating-point tuple \~ CV_32FC3, and so on. A universal OpenCV structure that is able to store a
single instance of such a primitive data type is Vec. Multiple instances of such a type can be
stored in a std::vector, Mat, Mat_, SparseMat, SparseMat_, or any other container that is able to
store Vec instances.

The DataType class is basically used to provide a description of such primitive data types without
adding any fields or methods to the corresponding classes (and it is actually impossible to add
anything to primitive C/C++ data types). This technique is known in C++ as class traits. It is not
DataType itself that is used but its specialized versions, such as:
@code
    template<> class DataType<uchar>
    {
        typedef uchar value_type;
        typedef int work_type;
        typedef uchar channel_type;
        enum { channel_type = CV_8U, channels = 1, fmt='u', type = CV_8U };
    };
    ...
    template<typename _Tp> DataType<std::complex<_Tp> >
    {
        typedef std::complex<_Tp> value_type;
        typedef std::complex<_Tp> work_type;
        typedef _Tp channel_type;
        // DataDepth is another helper trait class
        enum { depth = DataDepth<_Tp>::value, channels=2,
            fmt=(channels-1)*256+DataDepth<_Tp>::fmt,
            type=CV_MAKETYPE(depth, channels) };
    };
    ...
@endcode
The main purpose of this class is to convert compilation-time type information to an
OpenCV-compatible data type identifier, for example:
@code
    // allocates a 30x40 floating-point matrix
    Mat A(30, 40, DataType<float>::type);

    Mat B = Mat_<std::complex<double> >(3, 3);
    // the statement below will print 6, 2 , that is depth == CV_64F, channels == 2
    cout << B.depth() << ", " << B.channels() << endl;
@endcode
So, such traits are used to tell OpenCV which data type you are working with, even if such a type is
not native to OpenCV. For example, the matrix B initialization above is compiled because OpenCV
defines the proper specialized template class DataType\<complex\<_Tp\> \> . This mechanism is also
useful (and used in OpenCV this way) for generic algorithms implementations.

@note Default values were dropped to stop confusing developers about using of unsupported types (see #7599)
*/
template<typename _Tp> class DataType
{
public:
#ifdef OPENCV_TRAITS_ENABLE_DEPRECATED
    typedef _Tp         value_type;
    typedef value_type  work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 1,
           depth        = -1,
           channels     = 1,
           fmt          = 0,
           type = CV_MAKETYPE(depth, channels)
         };
#endif
};

template<> class DataType<bool>
{
public:
    typedef bool        value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_8U,
           channels     = 1,
           fmt          = (int)'u',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<uchar>
{
public:
    typedef uchar       value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_8U,
           channels     = 1,
           fmt          = (int)'u',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<schar>
{
public:
    typedef schar       value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_8S,
           channels     = 1,
           fmt          = (int)'c',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<char>
{
public:
    typedef schar       value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_8S,
           channels     = 1,
           fmt          = (int)'c',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<ushort>
{
public:
    typedef ushort      value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_16U,
           channels     = 1,
           fmt          = (int)'w',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<short>
{
public:
    typedef short       value_type;
    typedef int         work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_16S,
           channels     = 1,
           fmt          = (int)'s',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<int>
{
public:
    typedef int         value_type;
    typedef value_type  work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_32S,
           channels     = 1,
           fmt          = (int)'i',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<float>
{
public:
    typedef float       value_type;
    typedef value_type  work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_32F,
           channels     = 1,
           fmt          = (int)'f',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<double>
{
public:
    typedef double      value_type;
    typedef value_type  work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_64F,
           channels     = 1,
           fmt          = (int)'d',
           type         = CV_MAKETYPE(depth, channels)
         };
};

template<> class DataType<hfloat>
{
public:
    typedef hfloat   value_type;
    typedef float       work_type;
    typedef value_type  channel_type;
    typedef value_type  vec_type;
    enum { generic_type = 0,
           depth        = CV_16F,
           channels     = 1,
           fmt          = (int)'h',
           type         = CV_MAKETYPE(depth, channels)
         };
};

/** @brief A helper class for cv::DataType

The class is specialized for each fundamental numerical data type supported by OpenCV. It provides
DataDepth<T>::value constant.
*/
template<typename _Tp> class DataDepth
{
public:
    enum
    {
        value = DataType<_Tp>::depth,
        fmt   = DataType<_Tp>::fmt
    };
};


#ifdef OPENCV_TRAITS_ENABLE_DEPRECATED

template<int _depth> class TypeDepth
{
#ifdef OPENCV_TRAITS_ENABLE_LEGACY_DEFAULTS
    enum { depth = CV_USRTYPE1 };
    typedef void value_type;
#endif
};

template<> class TypeDepth<CV_8U>
{
    enum { depth = CV_8U };
    typedef uchar value_type;
};

template<> class TypeDepth<CV_8S>
{
    enum { depth = CV_8S };
    typedef schar value_type;
};

template<> class TypeDepth<CV_16U>
{
    enum { depth = CV_16U };
    typedef ushort value_type;
};

template<> class TypeDepth<CV_16S>
{
    enum { depth = CV_16S };
    typedef short value_type;
};

template<> class TypeDepth<CV_32S>
{
    enum { depth = CV_32S };
    typedef int value_type;
};

template<> class TypeDepth<CV_32F>
{
    enum { depth = CV_32F };
    typedef float value_type;
};

template<> class TypeDepth<CV_64F>
{
    enum { depth = CV_64F };
    typedef double value_type;
};

template<> class TypeDepth<CV_16F>
{
    enum { depth = CV_16F };
    typedef hfloat value_type;
};

#endif

//! @}

namespace traits {

namespace internal {
#define CV_CREATE_MEMBER_CHECK(X) \
template<typename T> class CheckMember_##X { \
    struct Fallback { int X; }; \
    struct Derived : T, Fallback { }; \
    template<typename U, U> struct Check; \
    typedef char CV_NO[1]; \
    typedef char CV_YES[2]; \
    template<typename U> static CV_NO & func(Check<int Fallback::*, &U::X> *); \
    template<typename U> static CV_YES & func(...); \
public: \
    typedef CheckMember_##X type; \
    enum { value = sizeof(func<Derived>(0)) == sizeof(CV_YES) }; \
};

CV_CREATE_MEMBER_CHECK(fmt)
CV_CREATE_MEMBER_CHECK(type)

} // namespace internal


template<typename T>
struct Depth
{ enum { value = DataType<T>::depth }; };

template<typename T>
struct Type
{ enum { value = DataType<T>::type }; };

/** Similar to traits::Type<T> but has value = -1 in case of unknown type (instead of compiler error) */
template<typename T, bool available = internal::CheckMember_type< DataType<T> >::value >
struct SafeType {};

template<typename T>
struct SafeType<T, false>
{ enum { value = -1 }; };

template<typename T>
struct SafeType<T, true>
{ enum { value = Type<T>::value }; };


template<typename T, bool available = internal::CheckMember_fmt< DataType<T> >::value >
struct SafeFmt {};

template<typename T>
struct SafeFmt<T, false>
{ enum { fmt = 0 }; };

template<typename T>
struct SafeFmt<T, true>
{ enum { fmt = DataType<T>::fmt }; };


} // namespace

} // cv

#endif // OPENCV_CORE_TRAITS_HPP
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

- **is**: A class/struct defined in this file
- **SafeFmt**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **DataType**: A class/struct defined in this file
- **enum**: A class/struct defined in this file
- **Check**: A class/struct defined in this file
- **Depth**: A class/struct defined in this file
- **traits**: A class/struct defined in this file
- **SafeType**: A class/struct defined in this file
- **Fallback**: A class/struct defined in this file
- **CheckMember_**: A class/struct defined in this file
- **DataDepth**: A class/struct defined in this file
- **Derived**: A class/struct defined in this file
- **Type**: A class/struct defined in this file
- **TypeDepth**: A class/struct defined in this file

### Functions and Methods

- **ushort()**: A function/method defined in this file
- **OPENCV_TRAITS_ENABLE_DEPRECATED()**: A function/method defined in this file
- **double()**: A function/method defined in this file
- **uchar()**: A function/method defined in this file
- **std()**: A function/method defined in this file
- **OPENCV_CORE_TRAITS_HPP()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **OPENCV_TRAITS_ENABLE_LEGACY_DEFAULTS()**: A function/method defined in this file
- **schar()**: A function/method defined in this file
- **_Tp()**: A function/method defined in this file
- **char()**: A function/method defined in this file
- **CheckMember_()**: A function/method defined in this file
- **bool()**: A function/method defined in this file
- **float()**: A function/method defined in this file
- **int()**: A function/method defined in this file
- **hfloat()**: A function/method defined in this file
- **value_type()**: A function/method defined in this file
- **short()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/cvdef.h`

**Python Imports:**
- `the`
- `this`


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

