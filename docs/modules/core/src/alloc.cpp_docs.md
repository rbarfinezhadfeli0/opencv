# Documentation for `modules/core/src/alloc.cpp`

## File Metadata

- **Full Path**: `modules/core/src/alloc.cpp`
- **File Name**: `alloc.cpp`
- **File Size**: 7,590 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/alloc.cpp](../../../modules/core/src/alloc.cpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
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

#include "precomp.hpp"

#include <opencv2/core/utils/logger.defines.hpp>
#undef CV_LOG_STRIP_LEVEL
#define CV_LOG_STRIP_LEVEL CV_LOG_LEVEL_VERBOSE + 1
#include <opencv2/core/utils/logger.hpp>
#include <opencv2/core/utils/configuration.private.hpp>

#define CV__ALLOCATOR_STATS_LOG(...) CV_LOG_VERBOSE(NULL, 0, "alloc.cpp: " << __VA_ARGS__)
#include "opencv2/core/utils/allocator_stats.impl.hpp"
#undef CV__ALLOCATOR_STATS_LOG

//#define OPENCV_ALLOC_ENABLE_STATISTICS


#ifdef HAVE_POSIX_MEMALIGN
#include <stdlib.h>
#elif defined HAVE_MALLOC_H
#include <malloc.h>
#endif

#ifdef OPENCV_ALLOC_ENABLE_STATISTICS
#define OPENCV_ALLOC_STATISTICS_LIMIT 4096  // don't track buffers less than N bytes
#include <map>
#endif

namespace cv {

static void* OutOfMemoryError(size_t size)
{
    CV_Error_(cv::Error::StsNoMem, ("Failed to allocate %llu bytes", (unsigned long long)size));
}

CV_EXPORTS cv::utils::AllocatorStatisticsInterface& getAllocatorStatistics();

static cv::utils::AllocatorStatistics allocator_stats;

cv::utils::AllocatorStatisticsInterface& getAllocatorStatistics()
{
    return allocator_stats;
}

#if defined HAVE_POSIX_MEMALIGN || defined HAVE_MEMALIGN || defined HAVE_WIN32_ALIGNED_MALLOC
static bool readMemoryAlignmentParameter()
{
    bool value = true;
#if defined(__GLIBC__) && defined(__linux__) \
    && !defined(CV_STATIC_ANALYSIS) \
    && !defined(OPENCV_ENABLE_MEMORY_SANITIZER) \
    && !defined(FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION)  /* oss-fuzz */ \
    && !defined(_WIN32)  /* MinGW? */
    {
        // https://github.com/opencv/opencv/issues/15526
        value = false;
    }
#endif
    value = cv::utils::getConfigurationParameterBool("OPENCV_ENABLE_MEMALIGN", value);  // should not call fastMalloc() internally
    // TODO add checks for valgrind, ASAN if value == false
    return value;
}

#if defined _MSC_VER
#pragma warning(suppress:4714)  // preventive: const marked as __forceinline not inlined
static __forceinline
#else
static inline
#endif
bool isAlignedAllocationEnabled()
{
    // use construct on first use idiom https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use
    // details: https://github.com/opencv/opencv/issues/15691
    static bool useMemalign = readMemoryAlignmentParameter();
    return useMemalign;
}

// need for this static const is disputed; retaining as it doesn't cause harm
static const bool g_force_initialization_memalign_flag
#if defined __GNUC__
    __attribute__((unused))
#endif
    = isAlignedAllocationEnabled();
#endif

#ifdef OPENCV_ALLOC_ENABLE_STATISTICS
static inline
void* fastMalloc_(size_t size)
#else
void* fastMalloc(size_t size)
#endif
{
#ifdef HAVE_POSIX_MEMALIGN
    if (isAlignedAllocationEnabled())
    {
        void* ptr = NULL;
        if(posix_memalign(&ptr, CV_MALLOC_ALIGN, size))
            ptr = NULL;
        if(!ptr)
            return OutOfMemoryError(size);
        return ptr;
    }
#elif defined HAVE_MEMALIGN
    if (isAlignedAllocationEnabled())
    {
        void* ptr = memalign(CV_MALLOC_ALIGN, size);
        if(!ptr)
            return OutOfMemoryError(size);
        return ptr;
    }
#elif defined HAVE_WIN32_ALIGNED_MALLOC
    if (isAlignedAllocationEnabled())
    {
        void* ptr = _aligned_malloc(size, CV_MALLOC_ALIGN);
        if(!ptr)
            return OutOfMemoryError(size);
        return ptr;
    }
#endif
    uchar* udata = (uchar*)malloc(size + sizeof(void*) + CV_MALLOC_ALIGN);
    if(!udata)
        return OutOfMemoryError(size);
    uchar** adata = alignPtr((uchar**)udata + 1, CV_MALLOC_ALIGN);
    adata[-1] = udata;
    return adata;
}

#ifdef OPENCV_ALLOC_ENABLE_STATISTICS
static inline
void fastFree_(void* ptr)
#else
void fastFree(void* ptr)
#endif
{
#if defined HAVE_POSIX_MEMALIGN || defined HAVE_MEMALIGN
    if (isAlignedAllocationEnabled())
    {
        free(ptr);
        return;
    }
#elif defined HAVE_WIN32_ALIGNED_MALLOC
    if (isAlignedAllocationEnabled())
    {
        _aligned_free(ptr);
        return;
    }
#endif
    if(ptr)
    {
        uchar* udata = ((uchar**)ptr)[-1];
        CV_DbgAssert(udata < (uchar*)ptr &&
               ((uchar*)ptr - udata) <= (ptrdiff_t)(sizeof(void*)+CV_MALLOC_ALIGN));
        free(udata);
    }
}

#ifdef OPENCV_ALLOC_ENABLE_STATISTICS

static
Mutex& getAllocationStatisticsMutex()
{
    static Mutex* p_alloc_mutex = allocSingletonNew<Mutex>();
    CV_Assert(p_alloc_mutex);
    return *p_alloc_mutex;
}

static std::map<void*, size_t> allocated_buffers;  // guarded by getAllocationStatisticsMutex()

void* fastMalloc(size_t size)
{
    void* res = fastMalloc_(size);
    if (res && size >= OPENCV_ALLOC_STATISTICS_LIMIT)
    {
        cv::AutoLock lock(getAllocationStatisticsMutex());
        allocated_buffers.insert(std::make_pair(res, size));
        allocator_stats.onAllocate(size);
    }
    return res;
}

void fastFree(void* ptr)
{
    {
        cv::AutoLock lock(getAllocationStatisticsMutex());
        std::map<void*, size_t>::iterator i = allocated_buffers.find(ptr);
        if (i != allocated_buffers.end())
        {
            size_t size = i->second;
            allocator_stats.onFree(size);
            allocated_buffers.erase(i);
        }
    }
    fastFree_(ptr);
}

#endif // OPENCV_ALLOC_ENABLE_STATISTICS

} // namespace

CV_IMPL void* cvAlloc( size_t size )
{
    return cv::fastMalloc( size );
}

CV_IMPL void cvFree_( void* ptr )
{
    cv::fastFree( ptr );
}

/* End of file. */
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

### Classes and Structures

- **on**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_ALLOC_ENABLE_STATISTICS()**: A function/method defined in this file
- **CV__ALLOCATOR_STATS_LOG()**: A function/method defined in this file
- **HAVE_POSIX_MEMALIGN()**: A function/method defined in this file
- **CV_LOG_STRIP_LEVEL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `malloc.h`
- `opencv2/core/utils/logger.defines.hpp`
- `stdlib.h`
- `opencv2/core/utils/logger.hpp`
- `opencv2/core/utils/allocator_stats.impl.hpp`
- `opencv2/core/utils/configuration.private.hpp`
- `precomp.hpp`
- `map`

**Python Imports:**
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

