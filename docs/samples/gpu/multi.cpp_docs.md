# Documentation for `samples/gpu/multi.cpp`

## File Metadata

- **Full Path**: `samples/gpu/multi.cpp`
- **File Name**: `multi.cpp`
- **File Size**: 2,240 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/gpu/multi.cpp](../../samples/gpu/multi.cpp)

## Purpose and Role

This file is located in the `samples/gpu` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/* This sample demonstrates the way you can perform independent tasks
   on the different GPUs */

// Disable some warnings which are caused with CUDA headers
#if defined(_MSC_VER)
#pragma warning(disable: 4201 4408 4100)
#endif

#include <iostream>
#include "opencv2/core.hpp"
#include "opencv2/cudaarithm.hpp"

#if !defined(HAVE_CUDA)

int main()
{
    std::cout << "CUDA support is required (OpenCV CMake parameter 'WITH_CUDA' must be true)." << std::endl;
    return 0;
}

#else

using namespace std;
using namespace cv;
using namespace cv::cuda;

struct Worker : public cv::ParallelLoopBody
{
    void operator()(const Range& r) const CV_OVERRIDE
    {
        for (int i = r.start; i < r.end; ++i) { this->operator()(i); }
    }
    void operator()(int device_id) const;
};

int main()
{
    int num_devices = getCudaEnabledDeviceCount();
    if (num_devices < 2)
    {
        std::cout << "Two or more GPUs are required\n";
        return -1;
    }
    for (int i = 0; i < num_devices; ++i)
    {
        cv::cuda::printShortCudaDeviceInfo(i);

        DeviceInfo dev_info(i);
        if (!dev_info.isCompatible())
        {
            std::cout << "CUDA module isn't built for GPU #" << i << " ("
                 << dev_info.name() << ", CC " << dev_info.majorVersion()
                 << dev_info.minorVersion() << "\n";
            return -1;
        }
    }

    // Execute calculation in two threads using two GPUs
    cv::Range devices(0, 2);
    cv::parallel_for_(devices, Worker(), devices.size());

    return 0;
}


void Worker::operator()(int device_id) const
{
    setDevice(device_id);

    Mat src(1000, 1000, CV_32F);
    Mat dst;

    RNG rng(0);
    rng.fill(src, RNG::UNIFORM, 0, 1);

    // CPU works
    cv::transpose(src, dst);

    // GPU works
    GpuMat d_src(src);
    GpuMat d_dst;
    cuda::transpose(d_src, d_dst);

    // Check results
    bool passed = cv::norm(dst - Mat(d_dst), NORM_INF) < 1e-3;
    std::cout << "GPU #" << device_id << " (" << DeviceInfo().name() << "): "
        << (passed ? "passed" : "FAILED") << endl;

    // Deallocate data here, otherwise deallocation will be performed
    // after context is extracted from the stack
    d_src.release();
    d_dst.release();
}

#endif
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

- **Worker**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `iostream`
- `opencv2/cudaarithm.hpp`

**Python Imports:**
- `the`


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

