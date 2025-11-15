# Documentation for `modules/dnn/src/vkcom/src/tensor.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/vkcom/src/tensor.cpp`
- **File Name**: `tensor.cpp`
- **File Size**: 2,725 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/vkcom/src/tensor.cpp](../../../../../modules/dnn/src/vkcom/src/tensor.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/vkcom/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.

#include "../../precomp.hpp"
#include "internal.hpp"

namespace cv { namespace dnn { namespace vkcom {

#ifdef HAVE_VULKAN

Tensor::Tensor(Format fmt, VkBufferUsageFlags usageFlag) : size_in_byte_(0), format_(fmt), usageFlag_(usageFlag)
{
}

Tensor::Tensor(const char* data, std::vector<int>& shape, Format fmt, VkBufferUsageFlags usageFlag)
               : size_in_byte_(0), format_(fmt), usageFlag_(usageFlag)
{
    reshape(data, shape);
}

void* Tensor::map()
{
    void *p;

    VK_CHECK_RESULT(vkMapMemory(kDevice, buffer_->getVkMemory(),
                                0, size_in_byte_, 0, (void **)&p));

    return p;
}

void Tensor::unMap()
{
    vkUnmapMemory(kDevice, buffer_->getVkMemory());
}

Shape Tensor::getShape() const
{
    return shape_;
}

int Tensor::count(const int start_axis, const int end_axis) const
{
    return shapeCount(shape_, start_axis, end_axis);
}

int Tensor::dimSize(const int axis) const
{
    CV_Assert(axis >= 0);
    CV_Assert(axis < shape_.size());

    return shape_[axis];
}

int Tensor::dimNum() const
{
    return shape_.size();
}

Tensor Tensor::reshape(const char* data, const std::vector<int>& shape, bool alloc, Format fmt)
{
    if (kDevice == VK_NULL_HANDLE)
    {
        CV_Error(Error::StsError, "device is NULL!");
        return *this;
    }

    CV_Assert(shape.size() > 0 && shape.size() <= 6);

    if (shape_ != shape) shape_ = shape;
    if (checkFormat(fmt) && fmt != format_) format_ = fmt;

    size_t new_size = shapeCount(shape_) * elementSize(format_);
    if (alloc || new_size > size_in_byte_)
        alloc = true;
    size_in_byte_ = new_size;

    if (alloc)
    {
        buffer_.reset(new Buffer(size_in_byte_, data, usageFlag_));
    }
    else if (data)
    {
        void* p = map();
        memcpy(p, data, size_in_byte_);
        unMap();
    }

    return *this;
}

void Tensor::setTo(float val)
{
    if (kDevice == VK_NULL_HANDLE)
    {
        CV_Error(Error::StsError, "device is NULL!");
        return;
    }

    CV_Assert(format_ == kFormatFp32);

    float* p = (float *)map();
    int cnt = count();
    for (int i = 0; i < cnt; i++)
        *p++ = val;
    unMap();
}

int Tensor::getFormat() const
{
    return format_;
}

void Tensor::copyTo(Tensor& dst)
{
    void* p = map();
    dst.reshape((const char*)p, shape_, format_);
    unMap();
}

#endif // HAVE_VULKAN

}}} // namespace cv::dnn::vkcom
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

### Functions and Methods

- **HAVE_VULKAN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../precomp.hpp`
- `internal.hpp`


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

