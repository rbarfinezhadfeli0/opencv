# Documentation for `modules/gapi/src/backends/onnx/dml_ep.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/onnx/dml_ep.cpp`
- **File Name**: `dml_ep.cpp`
- **File Size**: 10,620 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/onnx/dml_ep.cpp](../../../../../modules/gapi/src/backends/onnx/dml_ep.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/onnx` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2023 Intel Corporation

#include "backends/onnx/dml_ep.hpp"
#include "logger.hpp"

#ifdef HAVE_ONNX
#include <onnxruntime_cxx_api.h>

#ifdef HAVE_ONNX_DML
#include "../providers/dml/dml_provider_factory.h"

#ifdef HAVE_DIRECTML

#undef WINVER
#define WINVER 0x0A00
#undef _WIN32_WINNT
#define _WIN32_WINNT 0x0A00

#include <initguid.h>

#include <d3d11.h>
#include <dxgi1_2.h>
#include <dxgi1_4.h>
#include <dxgi.h>
#include <dxcore.h>
#include <dxcore_interface.h>
#include <d3d12.h>
#include <directml.h>

#pragma comment (lib, "d3d11.lib")
#pragma comment (lib, "d3d12.lib")
#pragma comment (lib, "dxgi.lib")
#pragma comment (lib, "dxcore.lib")
#pragma comment (lib, "directml.lib")

#endif  // HAVE_DIRECTML

static void addDMLExecutionProviderWithAdapterName(Ort::SessionOptions *session_options,
                                                   const std::string &adapter_name);

void cv::gimpl::onnx::addDMLExecutionProvider(Ort::SessionOptions *session_options,
                                              const cv::gapi::onnx::ep::DirectML &dml_ep) {
    namespace ep = cv::gapi::onnx::ep;
    switch (dml_ep.ddesc.index()) {
        case ep::DirectML::DeviceDesc::index_of<int>(): {
            const int device_id = cv::util::get<int>(dml_ep.ddesc);
            try {
                OrtSessionOptionsAppendExecutionProvider_DML(*session_options, device_id);
            } catch (const std::exception &e) {
                std::stringstream ss;
                ss << "ONNX Backend: Failed to enable DirectML"
                   << " Execution Provider: " << e.what();
                cv::util::throw_error(std::runtime_error(ss.str()));
            }
            break;
        }
        case ep::DirectML::DeviceDesc::index_of<std::string>(): {
            const std::string adapter_name = cv::util::get<std::string>(dml_ep.ddesc);
            addDMLExecutionProviderWithAdapterName(session_options, adapter_name);
            break;
        }
        default:
            GAPI_Assert(false && "Invalid DirectML device description");
    }
}

#ifdef HAVE_DIRECTML

#define THROW_IF_FAILED(hr, error_msg)       \
{                                            \
    if ((hr) != S_OK)                        \
        throw std::runtime_error(error_msg); \
}

template <typename T>
void release(T *ptr) {
    if (ptr) {
        ptr->Release();
    }
}

template <typename T>
using ComPtrGuard = std::unique_ptr<T, decltype(&release<T>)>;

template <typename T>
ComPtrGuard<T> make_com_ptr(T *ptr) {
    return ComPtrGuard<T>{ptr, &release<T>};
}

struct AdapterDesc {
    ComPtrGuard<IDXCoreAdapter> ptr;
    std::string description;
};

static std::vector<AdapterDesc> getAvailableAdapters() {
        std::vector<AdapterDesc> all_adapters;

        IDXCoreAdapterFactory* factory_ptr;
        GAPI_LOG_DEBUG(nullptr, "Create IDXCoreAdapterFactory");
        THROW_IF_FAILED(
            DXCoreCreateAdapterFactory(
                __uuidof(IDXCoreAdapterFactory), (void**)&factory_ptr),
            "Failed to create IDXCoreAdapterFactory");
        auto factory = make_com_ptr<IDXCoreAdapterFactory>(factory_ptr);

        IDXCoreAdapterList* adapter_list_ptr;
        const GUID dxGUIDs[] = { DXCORE_ADAPTER_ATTRIBUTE_D3D12_CORE_COMPUTE };
        GAPI_LOG_DEBUG(nullptr, "CreateAdapterList");
        THROW_IF_FAILED(
            factory->CreateAdapterList(
                ARRAYSIZE(dxGUIDs), dxGUIDs, __uuidof(IDXCoreAdapterList), (void**)&adapter_list_ptr),
            "Failed to create IDXCoreAdapterList");
        auto adapter_list = make_com_ptr<IDXCoreAdapterList>(adapter_list_ptr);

        for (UINT i = 0; i < adapter_list->GetAdapterCount(); i++)
        {
            IDXCoreAdapter* curr_adapter_ptr;
            GAPI_LOG_DEBUG(nullptr, "GetAdapter");
            THROW_IF_FAILED(
                adapter_list->GetAdapter(
                    i, __uuidof(IDXCoreAdapter), (void**)&curr_adapter_ptr),
                "Failed to obtain IDXCoreAdapter"
            );
            auto curr_adapter = make_com_ptr<IDXCoreAdapter>(curr_adapter_ptr);

            bool is_hardware = false;
            curr_adapter->GetProperty(DXCoreAdapterProperty::IsHardware, &is_hardware);
            // NB: Filter out if not hardware adapter.
            if (!is_hardware) {
                continue;
            }

            size_t desc_size = 0u;
            char description[256];
            curr_adapter->GetPropertySize(DXCoreAdapterProperty::DriverDescription, &desc_size);
            curr_adapter->GetProperty(DXCoreAdapterProperty::DriverDescription, desc_size, &description);
            all_adapters.push_back(AdapterDesc{std::move(curr_adapter), description});
        }
        return all_adapters;
};

struct DMLDeviceInfo {
    ComPtrGuard<IDMLDevice> device;
    ComPtrGuard<ID3D12CommandQueue> cmd_queue;
};

static DMLDeviceInfo createDMLInfo(IDXCoreAdapter* adapter) {
    auto pAdapter = make_com_ptr<IUnknown>(adapter);
    D3D_FEATURE_LEVEL d3dFeatureLevel = D3D_FEATURE_LEVEL_1_0_CORE;
    if (adapter->IsAttributeSupported(DXCORE_ADAPTER_ATTRIBUTE_D3D12_GRAPHICS))
    {
        GAPI_LOG_INFO(nullptr, "DXCORE_ADAPTER_ATTRIBUTE_D3D12_GRAPHICS is supported");
        d3dFeatureLevel = D3D_FEATURE_LEVEL::D3D_FEATURE_LEVEL_11_0;

        IDXGIFactory4* dxgiFactory4;
        GAPI_LOG_DEBUG(nullptr, "CreateDXGIFactory2");
        THROW_IF_FAILED(
            CreateDXGIFactory2(0, __uuidof(IDXGIFactory4), (void**)&dxgiFactory4),
            "Failed to create IDXGIFactory4"
        );
        // If DXGI factory creation was successful then get the IDXGIAdapter from the LUID
        // acquired from the selectedAdapter
        LUID adapterLuid;
        IDXGIAdapter* spDxgiAdapter;

        GAPI_LOG_DEBUG(nullptr, "Get DXCoreAdapterProperty::InstanceLuid property");
        THROW_IF_FAILED(
            adapter->GetProperty(DXCoreAdapterProperty::InstanceLuid, &adapterLuid),
            "Failed to get DXCoreAdapterProperty::InstanceLuid property");

        GAPI_LOG_DEBUG(nullptr, "Get IDXGIAdapter by luid");
        THROW_IF_FAILED(
            dxgiFactory4->EnumAdapterByLuid(
                adapterLuid, __uuidof(IDXGIAdapter), (void**)&spDxgiAdapter),
            "Failed to get IDXGIAdapter");
        pAdapter = make_com_ptr<IUnknown>(spDxgiAdapter);
    } else {
        GAPI_LOG_INFO(nullptr, "DXCORE_ADAPTER_ATTRIBUTE_D3D12_GRAPHICS isn't supported");
    }

    ID3D12Device* d3d12_device_ptr;
    GAPI_LOG_DEBUG(nullptr, "Create D3D12Device");
    THROW_IF_FAILED(
        D3D12CreateDevice(
            pAdapter.get(), d3dFeatureLevel, __uuidof(ID3D12Device), (void**)&d3d12_device_ptr),
        "Failed to create ID3D12Device");
    auto d3d12_device = make_com_ptr<ID3D12Device>(d3d12_device_ptr);

    D3D12_COMMAND_LIST_TYPE commandQueueType = D3D12_COMMAND_LIST_TYPE_COMPUTE;
    ID3D12CommandQueue* cmd_queue_ptr;
    D3D12_COMMAND_QUEUE_DESC commandQueueDesc = {};
    commandQueueDesc.Type = commandQueueType;
    GAPI_LOG_DEBUG(nullptr, "Create D3D12CommandQueue");
    THROW_IF_FAILED(
        d3d12_device->CreateCommandQueue(
            &commandQueueDesc, __uuidof(ID3D12CommandQueue), (void**)&cmd_queue_ptr),
        "Failed to create D3D12CommandQueue"
    );
    GAPI_LOG_DEBUG(nullptr, "Create D3D12CommandQueue - successful");
    auto cmd_queue = make_com_ptr<ID3D12CommandQueue>(cmd_queue_ptr);

    IDMLDevice* dml_device_ptr;
    GAPI_LOG_DEBUG(nullptr, "Create DirectML device");
    THROW_IF_FAILED(
        DMLCreateDevice(
            d3d12_device.get(), DML_CREATE_DEVICE_FLAG_NONE, IID_PPV_ARGS(&dml_device_ptr)),
        "Failed to create IDMLDevice");
    GAPI_LOG_DEBUG(nullptr, "Create DirectML device - successful");
    auto dml_device = make_com_ptr<IDMLDevice>(dml_device_ptr);

    return {std::move(dml_device), std::move(cmd_queue)};
};

static void addDMLExecutionProviderWithAdapterName(Ort::SessionOptions *session_options,
                                                   const std::string &adapter_name) {
    auto all_adapters = getAvailableAdapters();

    std::vector<AdapterDesc> selected_adapters;
    std::stringstream log_msg;
    for (auto&& adapter : all_adapters) {
        log_msg << adapter.description << std::endl;
        if (std::strstr(adapter.description.c_str(), adapter_name.c_str())) {
            selected_adapters.emplace_back(std::move(adapter));
        }
    }
    GAPI_LOG_INFO(NULL, "\nAvailable DirectML adapters:\n" << log_msg.str());

    if (selected_adapters.empty()) {
        std::stringstream error_msg;
        error_msg << "ONNX Backend: No DirectML adapters found match to \"" << adapter_name << "\"";
        cv::util::throw_error(std::runtime_error(error_msg.str()));
    } else if (selected_adapters.size() > 1) {
        std::stringstream error_msg;
        error_msg << "ONNX Backend: More than one adapter matches to \"" << adapter_name << "\":\n";
        for (const auto &selected_adapter : selected_adapters) {
            error_msg << selected_adapter.description << "\n";
        }
        cv::util::throw_error(std::runtime_error(error_msg.str()));
    }

    GAPI_LOG_INFO(NULL, "Selected device: " << selected_adapters.front().description);
    auto dml = createDMLInfo(selected_adapters.front().ptr.get());
    try {
        OrtSessionOptionsAppendExecutionProviderEx_DML(
            *session_options, dml.device.release(), dml.cmd_queue.release());
    } catch (const std::exception &e) {
        std::stringstream ss;
        ss << "ONNX Backend: Failed to enable DirectML"
           << " Execution Provider: " << e.what();
        cv::util::throw_error(std::runtime_error(ss.str()));
    }
}

#else  // HAVE_DIRECTML

static void addDMLExecutionProviderWithAdapterName(Ort::SessionOptions*, const std::string&) {
    std::stringstream ss;
    ss << "ONNX Backend: Failed to add DirectML Execution Provider with adapter name."
       << " DirectML support is required.";
    cv::util::throw_error(std::runtime_error(ss.str()));
}

#endif  // HAVE_DIRECTML
#else  // HAVE_ONNX_DML

void cv::gimpl::onnx::addDMLExecutionProvider(Ort::SessionOptions*,
                                              const cv::gapi::onnx::ep::DirectML&) {
     util::throw_error(std::runtime_error("G-API has been compiled with ONNXRT"
                                          " without DirectML support"));
}

#endif  // HAVE_ONNX_DML
#endif  // HAVE_ONNX
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

- **DMLDeviceInfo**: A class/struct defined in this file
- **AdapterDesc**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONNX()**: A function/method defined in this file
- **WINVER()**: A function/method defined in this file
- **HAVE_ONNX_DML()**: A function/method defined in this file
- **HAVE_DIRECTML()**: A function/method defined in this file
- **_WIN32_WINNT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `dxgi1_4.h`
- `d3d12.h`
- `dxcore_interface.h`
- `d3d11.h`
- `dxcore.h`
- `dxgi.h`
- `onnxruntime_cxx_api.h`
- `directml.h`
- `dxgi1_2.h`
- `initguid.h`
- `logger.hpp`
- `backends/onnx/dml_ep.hpp`
- `../providers/dml/dml_provider_factory.h`

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

