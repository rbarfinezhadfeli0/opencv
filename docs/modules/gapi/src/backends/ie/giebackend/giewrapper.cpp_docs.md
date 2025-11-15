# Documentation for `modules/gapi/src/backends/ie/giebackend/giewrapper.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/ie/giebackend/giewrapper.cpp`
- **File Name**: `giewrapper.cpp`
- **File Size**: 5,262 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/ie/giebackend/giewrapper.cpp](../../../../../../modules/gapi/src/backends/ie/giebackend/giewrapper.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/ie/giebackend` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2020-2024 Intel Corporation

#if defined HAVE_INF_ENGINE && INF_ENGINE_RELEASE < 2023010000

#include <vector>
#include <string>
#include <tuple>

#include "backends/ie/giebackend/giewrapper.hpp"

#include <ade/util/range.hpp>
#include <ade/util/zip_range.hpp>

#include <opencv2/core/utility.hpp>
#include <opencv2/core/utils/logger.hpp>

#include <opencv2/core/utils/configuration.private.hpp>

namespace IE = InferenceEngine;
namespace giewrap = cv::gimpl::ie::wrap;
using GIEParam = cv::gapi::ie::detail::ParamDesc;

#if INF_ENGINE_RELEASE < 2020000000  // < 2020.1
// Load extensions (taken from DNN module)
std::vector<std::string> giewrap::getExtensions(const GIEParam& params) {
    std::vector<std::string> candidates;
    if (params.device_id == "CPU" || params.device_id == "FPGA")
    {
        const std::string suffixes[] = { "_avx2", "_sse4", ""};
        const bool haveFeature[] = {
            cv::checkHardwareSupport(CPU_AVX2),
            cv::checkHardwareSupport(CPU_SSE4_2),
            true
        };
        for (auto &&it : ade::util::zip(ade::util::toRange(suffixes),
                                        ade::util::toRange(haveFeature)))
        {
            std::string suffix;
            bool available = false;
            std::tie(suffix, available) = it;
            if (!available) continue;
#ifdef _WIN32
            candidates.push_back("cpu_extension" + suffix + ".dll");
#elif defined(__APPLE__)
            candidates.push_back("libcpu_extension" + suffix + ".so");  // built as loadable module
            candidates.push_back("libcpu_extension" + suffix + ".dylib");  // built as shared library
#else
            candidates.push_back("libcpu_extension" + suffix + ".so");
#endif  // _WIN32
        }
    }
    return candidates;
}

IE::CNNNetwork giewrap::readNetwork(const GIEParam& params) {
    IE::CNNNetReader reader;
    reader.ReadNetwork(params.model_path);
    reader.ReadWeights(params.weights_path);
    return reader.getNetwork();
}
#else // >= 2020.1
std::vector<std::string> giewrap::getExtensions(const GIEParam&) {
    return std::vector<std::string>();
}

IE::CNNNetwork giewrap::readNetwork(const GIEParam& params) {
    auto core = giewrap::getCore();
    return core.ReadNetwork(params.model_path, params.weights_path);
}
#endif // INF_ENGINE_RELEASE < 2020000000

#if INF_ENGINE_RELEASE < 2019020000  // < 2019.R2
IE::InferencePlugin giewrap::getPlugin(const GIEParam& params) {
    auto plugin = IE::PluginDispatcher().getPluginByDevice(params.device_id);
    if (params.device_id == "CPU" || params.device_id == "FPGA")
    {
        for (auto &&extlib : giewrap::getExtensions(params))
        {
            try
            {
                plugin.AddExtension(IE::make_so_pointer<IE::IExtension>(extlib));
                CV_LOG_INFO(NULL, "DNN-IE: Loaded extension plugin: " << extlib);
                break;
            }
            catch(...)
            {
                CV_LOG_INFO(NULL, "Failed to load IE extension: " << extlib);
            }
        }
    }
    return plugin;
}
#else // >= 2019.R2

// NB: Some of IE plugins fail during IE::Core destroying in specific cases.
// Solution is allocate IE::Core in heap and doesn't destroy it, which cause
// leak, but fixes tests on CI. This behaviour is configurable by using
// OPENCV_GAPI_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND=0
static IE::Core create_IE_Core_pointer() {
    // NB: 'delete' is never called
    static IE::Core* core = new IE::Core();
    return *core;
}

static IE::Core create_IE_Core_instance() {
    static IE::Core core;
    return core;
}

IE::Core giewrap::getCore() {
    // NB: to make happy memory leak tools use:
    // - OPENCV_GAPI_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND=0
    static bool param_GAPI_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND =
        utils::getConfigurationParameterBool(
                "OPENCV_GAPI_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND",
#if defined(_WIN32) || defined(__APPLE__)
                true
#else
                false
#endif
                );
    return param_GAPI_INFERENCE_ENGINE_CORE_LIFETIME_WORKAROUND
        ? create_IE_Core_pointer() : create_IE_Core_instance();
}

IE::Core giewrap::getPlugin(const GIEParam& params) {
    auto plugin = giewrap::getCore();
    if (params.device_id == "CPU" || params.device_id == "FPGA")
    {
        for (auto &&extlib : giewrap::getExtensions(params))
        {
            try
            {
#if INF_ENGINE_RELEASE >= 2021040000
                plugin.AddExtension(std::make_shared<IE::Extension>(extlib), params.device_id);
#else
                plugin.AddExtension(IE::make_so_pointer<IE::IExtension>(extlib), params.device_id);
#endif
                CV_LOG_INFO(NULL, "DNN-IE: Loaded extension plugin: " << extlib);
                break;
            }
            catch(...)
            {
                CV_LOG_INFO(NULL, "Failed to load IE extension: " << extlib);
            }
        }
    }
    return plugin;
}
#endif // INF_ENGINE_RELEASE < 2019020000

#endif //HAVE_INF_ENGINE
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

- **_WIN32()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/utils/configuration.private.hpp`
- `ade/util/zip_range.hpp`
- `vector`
- `opencv2/core/utils/logger.hpp`
- `string`
- `tuple`
- `opencv2/core/utility.hpp`
- `ade/util/range.hpp`
- `backends/ie/giebackend/giewrapper.hpp`

**Python Imports:**
- `DNN`


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

