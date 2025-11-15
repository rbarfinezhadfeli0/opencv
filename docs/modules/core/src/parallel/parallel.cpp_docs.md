# Documentation for `modules/core/src/parallel/parallel.cpp`

## File Metadata

- **Full Path**: `modules/core/src/parallel/parallel.cpp`
- **File Name**: `parallel.cpp`
- **File Size**: 6,554 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/parallel/parallel.cpp](../../../../modules/core/src/parallel/parallel.cpp)

## Purpose and Role

This file is located in the `modules/core/src/parallel` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
#include "../precomp.hpp"
#include "parallel.hpp"

#include <opencv2/core/utils/configuration.private.hpp>
#include <opencv2/core/utils/logger.defines.hpp>
#ifdef NDEBUG
#define CV_LOG_STRIP_LEVEL CV_LOG_LEVEL_DEBUG + 1
#else
#define CV_LOG_STRIP_LEVEL CV_LOG_LEVEL_VERBOSE + 1
#endif
#include <opencv2/core/utils/logger.hpp>


#include "registry_parallel.hpp"
#include "registry_parallel.impl.hpp"

#include "plugin_parallel_api.hpp"
#include "plugin_parallel_wrapper.impl.hpp"


namespace cv { namespace parallel {

int numThreads = -1;

ParallelForAPI::~ParallelForAPI()
{
    // nothing
}

static
std::string& getParallelBackendName()
{
    static std::string g_backendName = toUpperCase(cv::utils::getConfigurationParameterString("OPENCV_PARALLEL_BACKEND", ""));
    return g_backendName;
}

static bool g_initializedParallelForAPI = false;

static
std::shared_ptr<ParallelForAPI> createParallelForAPI()
{
    const std::string& name = getParallelBackendName();
    bool isKnown = false;
    const auto& backends = getParallelBackendsInfo();
    if (!name.empty())
    {
        CV_LOG_INFO(NULL, "core(parallel): requested backend name: " << name);
    }
    for (size_t i = 0; i < backends.size(); i++)
    {
        const auto& info = backends[i];
        if (!name.empty())
        {
            if (name != info.name)
            {
                continue;
            }
            isKnown = true;
        }
        try
        {
            CV_LOG_DEBUG(NULL, "core(parallel): trying backend: " << info.name << " (priority=" << info.priority << ")");
            if (!info.backendFactory)
            {
                CV_LOG_DEBUG(NULL, "core(parallel): factory is not available (plugins require filesystem support): " << info.name);
                continue;
            }
            std::shared_ptr<ParallelForAPI> backend = info.backendFactory->create();
            if (!backend)
            {
                CV_LOG_DEBUG(NULL, "core(parallel): not available: " << info.name);
                continue;
            }
            CV_LOG_INFO(NULL, "core(parallel): using backend: " << info.name << " (priority=" << info.priority << ")");
            g_initializedParallelForAPI = true;
            getParallelBackendName() = info.name;
            return backend;
        }
        catch (const std::exception& e)
        {
            CV_LOG_WARNING(NULL, "core(parallel): can't initialize " << info.name << " backend: " << e.what());
        }
        catch (...)
        {
            CV_LOG_WARNING(NULL, "core(parallel): can't initialize " << info.name << " backend: Unknown C++ exception");
        }
    }
    if (name.empty())
    {
        CV_LOG_DEBUG(NULL, "core(parallel): fallback on builtin code");
    }
    else
    {
        if (!isKnown)
        {
            CV_LOG_INFO(NULL, "core(parallel): unknown backend: " << name << ", fallback on builtin code");
        }
        else
        {
            CV_LOG_INFO(NULL, "core(parallel): backend=" << name << " is not available, fallback on builtin code");
        }
    }
    g_initializedParallelForAPI = true;
    getParallelBackendName() = std::string();
    return std::shared_ptr<ParallelForAPI>();
}

static inline
std::shared_ptr<ParallelForAPI> createDefaultParallelForAPI()
{
    CV_LOG_DEBUG(NULL, "core(parallel): Initializing parallel backend...");
    return createParallelForAPI();
}

std::shared_ptr<ParallelForAPI>& getCurrentParallelForAPI()
{
    static std::shared_ptr<ParallelForAPI> g_currentParallelForAPI = createDefaultParallelForAPI();
    return g_currentParallelForAPI;
}

void setParallelForBackend(const std::shared_ptr<ParallelForAPI>& api, bool propagateNumThreads)
{
    getCurrentParallelForAPI() = api;
    if (propagateNumThreads && api)
    {
        setNumThreads(numThreads);
    }
}

bool setParallelForBackend(const std::string& backendName, bool propagateNumThreads)
{
    CV_TRACE_FUNCTION();

    bool saved_initialized = g_initializedParallelForAPI;
    std::string saved_backendName = getParallelBackendName();
    std::shared_ptr<ParallelForAPI> saved_backend;  // don't call getCurrentParallelForAPI() if g_initializedParallelForAPI = false to avoid unnecessary "default" initialization

    std::string backendName_u = toUpperCase(backendName);
    if (g_initializedParallelForAPI)
    {
        // ... already initialized
        if (getParallelBackendName() == backendName_u)
        {
            CV_LOG_INFO(NULL, "core(parallel): backend is already activated: " << (backendName.empty() ? "builtin(legacy)" : backendName));
            return true;
        }
        else
        {
            saved_backend = getCurrentParallelForAPI();
            // ... re-create new
            CV_LOG_DEBUG(NULL, "core(parallel): replacing parallel backend (old=" << saved_backendName << " new=" << backendName << ")...");
            getParallelBackendName() = backendName_u;
            getCurrentParallelForAPI() = createParallelForAPI();
        }
    }
    else
    {
        // ... no backend exists, just specify the name (initialization is triggered by getCurrentParallelForAPI() call)
        getParallelBackendName() = backendName_u;
    }
    std::shared_ptr<ParallelForAPI> api = getCurrentParallelForAPI();
    if (!api)
    {
        if (!backendName.empty())
        {
            // restore previous backend or build default
            getParallelBackendName() = saved_backendName;
            if (saved_initialized)
            {
                CV_LOG_WARNING(NULL, "core(parallel): backend is not available: " << backendName << " (keep previous)");
                getCurrentParallelForAPI() = saved_backend;
            }
            else
            {
                CV_LOG_WARNING(NULL, "core(parallel): backend is not available: " << backendName << " (use default)");
                g_initializedParallelForAPI = false;
                getCurrentParallelForAPI() = createDefaultParallelForAPI();
            }
            return false;
        }
        else
        {
            CV_LOG_WARNING(NULL, "core(parallel): switched to builtin code (legacy)");
        }
    }
    if (!backendName_u.empty())
    {
        CV_Assert(backendName_u == getParallelBackendName());  // data race?
    }

    if (propagateNumThreads)
    {
        setNumThreads(numThreads);
    }
    return true;
}

}}  // namespace
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

- **NDEBUG()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `registry_parallel.hpp`
- `registry_parallel.impl.hpp`
- `plugin_parallel_wrapper.impl.hpp`
- `opencv2/core/utils/logger.defines.hpp`
- `plugin_parallel_api.hpp`
- `../precomp.hpp`
- `opencv2/core/utils/logger.hpp`
- `opencv2/core/utils/configuration.private.hpp`
- `parallel.hpp`


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

