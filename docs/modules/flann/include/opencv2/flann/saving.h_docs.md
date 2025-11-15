# Documentation for `modules/flann/include/opencv2/flann/saving.h`

## File Metadata

- **Full Path**: `modules/flann/include/opencv2/flann/saving.h`
- **File Name**: `saving.h`
- **File Size**: 5,910 bytes
- **File Type**: .h
- **Link to Source**: [modules/flann/include/opencv2/flann/saving.h](../../../../../modules/flann/include/opencv2/flann/saving.h)

## Purpose and Role

This file is located in the `modules/flann/include/opencv2/flann` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/***********************************************************************
 * Software License Agreement (BSD License)
 *
 * Copyright 2008-2009  Marius Muja (mariusm@cs.ubc.ca). All rights reserved.
 * Copyright 2008-2009  David G. Lowe (lowe@cs.ubc.ca). All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
 * IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
 * OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
 * IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
 * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
 * NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE NNIndexGOODS OR SERVICES; LOSS OF USE,
 * DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
 * THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
 * THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *************************************************************************/

#ifndef OPENCV_FLANN_SAVING_H_
#define OPENCV_FLANN_SAVING_H_

//! @cond IGNORED

#include <cstring>
#include <vector>

#include "general.h"
#include "nn_index.h"

#ifdef FLANN_SIGNATURE_
#undef FLANN_SIGNATURE_
#endif
#define FLANN_SIGNATURE_ "FLANN_INDEX"

namespace cvflann
{

template <typename T>
struct Datatype {};
template<>
struct Datatype<char> { static flann_datatype_t type() { return FLANN_INT8; } };
template<>
struct Datatype<short> { static flann_datatype_t type() { return FLANN_INT16; } };
template<>
struct Datatype<int> { static flann_datatype_t type() { return FLANN_INT32; } };
template<>
struct Datatype<unsigned char> { static flann_datatype_t type() { return FLANN_UINT8; } };
template<>
struct Datatype<unsigned short> { static flann_datatype_t type() { return FLANN_UINT16; } };
template<>
struct Datatype<unsigned int> { static flann_datatype_t type() { return FLANN_UINT32; } };
template<>
struct Datatype<float> { static flann_datatype_t type() { return FLANN_FLOAT32; } };
template<>
struct Datatype<double> { static flann_datatype_t type() { return FLANN_FLOAT64; } };


/**
 * Structure representing the index header.
 */
struct IndexHeader
{
    char signature[16];
    char version[16];
    flann_datatype_t data_type;
    flann_algorithm_t index_type;
    size_t rows;
    size_t cols;
};

/**
 * Saves index header to stream
 *
 * @param stream - Stream to save to
 * @param index - The index to save
 */
template<typename Distance>
void save_header(FILE* stream, const NNIndex<Distance>& index)
{
    IndexHeader header;
    memset(header.signature, 0, sizeof(header.signature));
    strcpy(header.signature, FLANN_SIGNATURE_);
    memset(header.version, 0, sizeof(header.version));
    strcpy(header.version, FLANN_VERSION_);
    header.data_type = Datatype<typename Distance::ElementType>::type();
    header.index_type = index.getType();
    header.rows = index.size();
    header.cols = index.veclen();

    std::fwrite(&header, sizeof(header),1,stream);
}


/**
 *
 * @param stream - Stream to load from
 * @return Index header
 */
inline IndexHeader load_header(FILE* stream)
{
    IndexHeader header;
    size_t read_size = fread(&header,sizeof(header),1,stream);

    if (read_size!=(size_t)1) {
        FLANN_THROW(cv::Error::StsError, "Invalid index file, cannot read");
    }

    if (strcmp(header.signature,FLANN_SIGNATURE_)!=0) {
        FLANN_THROW(cv::Error::StsError, "Invalid index file, wrong signature");
    }

    return header;

}


template<typename T>
void save_value(FILE* stream, const T& value, size_t count = 1)
{
    fwrite(&value, sizeof(value),count, stream);
}

template<typename T>
void save_value(FILE* stream, const cvflann::Matrix<T>& value)
{
    fwrite(&value, sizeof(value),1, stream);
    fwrite(value.data, sizeof(T),value.rows*value.cols, stream);
}

template<typename T>
void save_value(FILE* stream, const std::vector<T>& value)
{
    size_t size = value.size();
    fwrite(&size, sizeof(size_t), 1, stream);
    fwrite(&value[0], sizeof(T), size, stream);
}

template<typename T>
void load_value(FILE* stream, T& value, size_t count = 1)
{
    size_t read_cnt = fread(&value, sizeof(value), count, stream);
    if (read_cnt != count) {
        FLANN_THROW(cv::Error::StsParseError, "Cannot read from file");
    }
}

template<typename T>
void load_value(FILE* stream, cvflann::Matrix<T>& value)
{
    size_t read_cnt = fread(&value, sizeof(value), 1, stream);
    if (read_cnt != 1) {
        FLANN_THROW(cv::Error::StsParseError, "Cannot read from file");
    }
    value.data = new T[value.rows*value.cols];
    read_cnt = fread(value.data, sizeof(T), value.rows*value.cols, stream);
    if (read_cnt != (size_t)(value.rows*value.cols)) {
        FLANN_THROW(cv::Error::StsParseError, "Cannot read from file");
    }
}


template<typename T>
void load_value(FILE* stream, std::vector<T>& value)
{
    size_t size;
    size_t read_cnt = fread(&size, sizeof(size_t), 1, stream);
    if (read_cnt!=1) {
        FLANN_THROW(cv::Error::StsError, "Cannot read from file");
    }
    value.resize(size);
    read_cnt = fread(&value[0], sizeof(T), size, stream);
    if (read_cnt != size) {
        FLANN_THROW(cv::Error::StsError, "Cannot read from file");
    }
}

}

//! @endcond

#endif /* OPENCV_FLANN_SAVING_H_ */
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

- **IndexHeader**: A class/struct defined in this file
- **Datatype**: A class/struct defined in this file

### Functions and Methods

- **FLANN_SIGNATURE_()**: A function/method defined in this file
- **OPENCV_FLANN_SAVING_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cstring`
- `nn_index.h`
- `general.h`
- `vector`

**Python Imports:**
- `file`


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

