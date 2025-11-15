# Documentation for `modules/core/include/opencv2/core/opencl/opencl_info.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/opencl/opencl_info.hpp`
- **File Name**: `opencl_info.hpp`
- **File Size**: 8,431 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/opencl/opencl_info.hpp](../../../../../../modules/core/include/opencv2/core/opencl/opencl_info.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include <iostream>
#include <sstream>

#include <opencv2/core.hpp>
#include <opencv2/core/ocl.hpp>

#ifndef DUMP_CONFIG_PROPERTY
#define DUMP_CONFIG_PROPERTY(...)
#endif

#ifndef DUMP_MESSAGE_STDOUT
#define DUMP_MESSAGE_STDOUT(...) do { std::cout << __VA_ARGS__ << std::endl; } while (false)
#endif

namespace cv {

namespace {
static std::string bytesToStringRepr(size_t value)
{
    size_t b = value % 1024;
    value /= 1024;

    size_t kb = value % 1024;
    value /= 1024;

    size_t mb = value % 1024;
    value /= 1024;

    size_t gb = value;

    std::ostringstream stream;

    if (gb > 0)
        stream << gb << " GB ";
    if (mb > 0)
        stream << mb << " MB ";
    if (kb > 0)
        stream << kb << " KB ";
    if (b > 0)
        stream << b << " B";

    std::string s = stream.str();
    if (s[s.size() - 1] == ' ')
        s = s.substr(0, s.size() - 1);
    return s;
}

static String getDeviceTypeString(const cv::ocl::Device& device)
{
    if (device.type() == cv::ocl::Device::TYPE_CPU) {
        return "CPU";
    }

    if (device.type() == cv::ocl::Device::TYPE_GPU) {
        if (device.hostUnifiedMemory()) {
            return "iGPU";
        } else {
            return "dGPU";
        }
    }

    return "unknown";
}
} // namespace

static void dumpOpenCLInformation()
{
    using namespace cv::ocl;

    try
    {
        if (!haveOpenCL() || !useOpenCL())
        {
            DUMP_MESSAGE_STDOUT("OpenCL is disabled");
            DUMP_CONFIG_PROPERTY("cv_ocl", "disabled");
            return;
        }

        std::vector<PlatformInfo> platforms;
        cv::ocl::getPlatfomsInfo(platforms);
        if (platforms.empty())
        {
            DUMP_MESSAGE_STDOUT("OpenCL is not available");
            DUMP_CONFIG_PROPERTY("cv_ocl", "not available");
            return;
        }

        DUMP_MESSAGE_STDOUT("OpenCL Platforms: ");
        for (size_t i = 0; i < platforms.size(); i++)
        {
            const PlatformInfo* platform = &platforms[i];
            DUMP_MESSAGE_STDOUT("    " << platform->name());
            Device current_device;
            for (int j = 0; j < platform->deviceNumber(); j++)
            {
                platform->getDevice(current_device, j);
                String deviceTypeStr = getDeviceTypeString(current_device);
                DUMP_MESSAGE_STDOUT( "        " << deviceTypeStr << ": " << current_device.name() << " (" << current_device.version() << ")");
                DUMP_CONFIG_PROPERTY( cv::format("cv_ocl_platform_%d_device_%d", (int)i, j ),
                    cv::format("(Platform=%s)(Type=%s)(Name=%s)(Version=%s)",
                    platform->name().c_str(), deviceTypeStr.c_str(), current_device.name().c_str(), current_device.version().c_str()) );
            }
        }
        const Device& device = Device::getDefault();
        if (!device.available())
            CV_Error(Error::OpenCLInitError, "OpenCL device is not available");

        DUMP_MESSAGE_STDOUT("Current OpenCL device: ");

        String deviceTypeStr = getDeviceTypeString(device);
        DUMP_MESSAGE_STDOUT("    Type = " << deviceTypeStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_deviceType", deviceTypeStr);

        DUMP_MESSAGE_STDOUT("    Name = " << device.name());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_deviceName", device.name());

        DUMP_MESSAGE_STDOUT("    Version = " << device.version());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_deviceVersion", device.version());

        DUMP_MESSAGE_STDOUT("    Driver version = " << device.driverVersion());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_driverVersion", device.driverVersion());

        DUMP_MESSAGE_STDOUT("    Address bits = " << device.addressBits());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_addressBits", device.addressBits());

        DUMP_MESSAGE_STDOUT("    Compute units = " << device.maxComputeUnits());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_maxComputeUnits", device.maxComputeUnits());

        DUMP_MESSAGE_STDOUT("    Max work group size = " << device.maxWorkGroupSize());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_maxWorkGroupSize", device.maxWorkGroupSize());

        std::string localMemorySizeStr = bytesToStringRepr(device.localMemSize());
        DUMP_MESSAGE_STDOUT("    Local memory size = " << localMemorySizeStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_localMemSize", device.localMemSize());

        std::string maxMemAllocSizeStr = bytesToStringRepr(device.maxMemAllocSize());
        DUMP_MESSAGE_STDOUT("    Max memory allocation size = " << maxMemAllocSizeStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_maxMemAllocSize", device.maxMemAllocSize());

        const char* doubleSupportStr = device.hasFP64() ? "Yes" : "No";
        DUMP_MESSAGE_STDOUT("    Double support = " << doubleSupportStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_haveDoubleSupport", device.hasFP64());

        const char* halfSupportStr = device.hasFP16() ? "Yes" : "No";
        DUMP_MESSAGE_STDOUT("    Half support = " << halfSupportStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_haveHalfSupport", device.hasFP16());

        const char* isUnifiedMemoryStr = device.hostUnifiedMemory() ? "Yes" : "No";
        DUMP_MESSAGE_STDOUT("    Host unified memory = " << isUnifiedMemoryStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_hostUnifiedMemory", device.hostUnifiedMemory());

        DUMP_MESSAGE_STDOUT("    Device extensions:");
        String extensionsStr = device.extensions();
        size_t pos = 0;
        while (pos < extensionsStr.size())
        {
            size_t pos2 = extensionsStr.find(' ', pos);
            if (pos2 == String::npos)
                pos2 = extensionsStr.size();
            if (pos2 > pos)
            {
                String extensionName = extensionsStr.substr(pos, pos2 - pos);
                DUMP_MESSAGE_STDOUT("        " << extensionName);
            }
            pos = pos2 + 1;
        }
        DUMP_CONFIG_PROPERTY("cv_ocl_current_extensions", extensionsStr);

        const char* haveAmdBlasStr = haveAmdBlas() ? "Yes" : "No";
        DUMP_MESSAGE_STDOUT("    Has AMD Blas = " << haveAmdBlasStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_AmdBlas", haveAmdBlas());

        const char* haveAmdFftStr = haveAmdFft() ? "Yes" : "No";
        DUMP_MESSAGE_STDOUT("    Has AMD Fft = " << haveAmdFftStr);
        DUMP_CONFIG_PROPERTY("cv_ocl_current_AmdFft", haveAmdFft());


        DUMP_MESSAGE_STDOUT("    Preferred vector width char = " << device.preferredVectorWidthChar());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthChar", device.preferredVectorWidthChar());

        DUMP_MESSAGE_STDOUT("    Preferred vector width short = " << device.preferredVectorWidthShort());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthShort", device.preferredVectorWidthShort());

        DUMP_MESSAGE_STDOUT("    Preferred vector width int = " << device.preferredVectorWidthInt());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthInt", device.preferredVectorWidthInt());

        DUMP_MESSAGE_STDOUT("    Preferred vector width long = " << device.preferredVectorWidthLong());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthLong", device.preferredVectorWidthLong());

        DUMP_MESSAGE_STDOUT("    Preferred vector width float = " << device.preferredVectorWidthFloat());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthFloat", device.preferredVectorWidthFloat());

        DUMP_MESSAGE_STDOUT("    Preferred vector width double = " << device.preferredVectorWidthDouble());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthDouble", device.preferredVectorWidthDouble());

        DUMP_MESSAGE_STDOUT("    Preferred vector width half = " << device.preferredVectorWidthHalf());
        DUMP_CONFIG_PROPERTY("cv_ocl_current_preferredVectorWidthHalf", device.preferredVectorWidthHalf());
    }
    catch (...)
    {
        DUMP_MESSAGE_STDOUT("Exception. Can't dump OpenCL info");
        DUMP_MESSAGE_STDOUT("OpenCL device not available");
        DUMP_CONFIG_PROPERTY("cv_ocl", "not available");
    }
}
#undef DUMP_MESSAGE_STDOUT
#undef DUMP_CONFIG_PROPERTY

} // namespace
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

### Functions and Methods

- **DUMP_MESSAGE_STDOUT()**: A function/method defined in this file
- **DUMP_CONFIG_PROPERTY()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `iostream`
- `sstream`
- `opencv2/core/ocl.hpp`


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

