# Documentation for `modules/dnn/include/opencv2/dnn/dict.hpp`

## File Metadata

- **Full Path**: `modules/dnn/include/opencv2/dnn/dict.hpp`
- **File Name**: `dict.hpp`
- **File Size**: 6,264 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/include/opencv2/dnn/dict.hpp](../../../../../modules/dnn/include/opencv2/dnn/dict.hpp)

## Purpose and Role

This file is located in the `modules/dnn/include/opencv2/dnn` directory and serves as part of the OpenCV library infrastructure.

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
//                           License Agreement
//                For Open Source Computer Vision Library
//
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

#include <opencv2/core.hpp>
#include <map>
#include <ostream>

#include <opencv2/dnn/dnn.hpp>

#ifndef OPENCV_DNN_DNN_DICT_HPP
#define OPENCV_DNN_DNN_DICT_HPP

namespace cv {
namespace dnn {
CV__DNN_INLINE_NS_BEGIN
//! @addtogroup dnn
//! @{

/** @brief This struct stores the scalar value (or array) of one of the following type: double, cv::String or int64.
 *  @todo Maybe int64 is useless because double type exactly stores at least 2^52 integers.
 */
struct CV_EXPORTS_W DictValue
{
    DictValue(const DictValue &r);
    explicit DictValue(bool i)           : type(Param::INT), pi(new AutoBuffer<int64,1>) { (*pi)[0] = i ? 1 : 0; }       //!< Constructs integer scalar
    explicit DictValue(int64 i = 0)      : type(Param::INT), pi(new AutoBuffer<int64,1>) { (*pi)[0] = i; }       //!< Constructs integer scalar
    CV_WRAP explicit DictValue(int i)    : type(Param::INT), pi(new AutoBuffer<int64,1>) { (*pi)[0] = i; }       //!< Constructs integer scalar
    explicit DictValue(unsigned p)       : type(Param::INT), pi(new AutoBuffer<int64,1>) { (*pi)[0] = p; }       //!< Constructs integer scalar
    CV_WRAP explicit DictValue(double p)         : type(Param::REAL), pd(new AutoBuffer<double,1>) { (*pd)[0] = p; }     //!< Constructs floating point scalar
    CV_WRAP explicit DictValue(const String &s)  : type(Param::STRING), ps(new AutoBuffer<String,1>) { (*ps)[0] = s; }   //!< Constructs string scalar
    explicit DictValue(const char *s)            : type(Param::STRING), ps(new AutoBuffer<String,1>) { (*ps)[0] = s; }   //!< @overload

    template<typename TypeIter>
    static DictValue arrayInt(TypeIter begin, int size);    //!< Constructs integer array
    template<typename TypeIter>
    static DictValue arrayReal(TypeIter begin, int size);   //!< Constructs floating point array
    template<typename TypeIter>
    static DictValue arrayString(TypeIter begin, int size); //!< Constructs array of strings

    template<typename T>
    T get(int idx = -1) const; //!< Tries to convert array element with specified index to requested type and returns its.

    int size() const;

    CV_WRAP bool isInt() const;
    CV_WRAP bool isString() const;
    CV_WRAP bool isReal() const;

    CV_WRAP int getIntValue(int idx = -1) const;
    CV_WRAP double getRealValue(int idx = -1) const;
    CV_WRAP String getStringValue(int idx = -1) const;

    DictValue &operator=(const DictValue &r);

    friend std::ostream &operator<<(std::ostream &stream, const DictValue &dictv);

    ~DictValue();

private:

    Param type;

    union
    {
        AutoBuffer<int64, 1> *pi;
        AutoBuffer<double, 1> *pd;
        AutoBuffer<String, 1> *ps;
        void *pv;
    };

    DictValue(Param _type, void *_p) : type(_type), pv(_p) {}
    void release();
};

/** @brief This class implements name-value dictionary, values are instances of DictValue. */
class CV_EXPORTS Dict
{
    typedef std::map<String, DictValue> _Dict;
    _Dict dict;

public:

    //! Checks a presence of the @p key in the dictionary.
    bool has(const String &key) const;

    //! If the @p key in the dictionary then returns pointer to its value, else returns NULL.
    DictValue *ptr(const String &key);

    /** @overload */
    const DictValue *ptr(const String &key) const;

    //! If the @p key in the dictionary then returns its value, else an error will be generated.
    const DictValue &get(const String &key) const;

    /** @overload */
    template <typename T>
    T get(const String &key) const;

    //! If the @p key in the dictionary then returns its value, else returns @p defaultValue.
    template <typename T>
    T get(const String &key, const T &defaultValue) const;

    //! Sets new @p value for the @p key, or adds new key-value pair into the dictionary.
    template<typename T>
    const T &set(const String &key, const T &value);

    //! Erase @p key from the dictionary.
    void erase(const String &key);

    friend std::ostream &operator<<(std::ostream &stream, const Dict &dict);

    std::map<String, DictValue>::const_iterator begin() const;

    std::map<String, DictValue>::const_iterator end() const;
};

//! @}
CV__DNN_INLINE_NS_END
}
}

#endif
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

- **CV_EXPORTS_W**: A class/struct defined in this file
- **CV_EXPORTS**: A class/struct defined in this file
- **stores**: A class/struct defined in this file
- **implements**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **OPENCV_DNN_DNN_DICT_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/dnn/dnn.hpp`
- `ostream`
- `map`

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

