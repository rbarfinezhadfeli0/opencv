# Documentation for `modules/gapi/test/streaming/gapi_streaming_vpl_device_selector.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/streaming/gapi_streaming_vpl_device_selector.cpp`
- **File Name**: `gapi_streaming_vpl_device_selector.cpp`
- **File Size**: 14,671 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/streaming/gapi_streaming_vpl_device_selector.cpp](../../../../modules/gapi/test/streaming/gapi_streaming_vpl_device_selector.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation


#include "../test_precomp.hpp"

#include "../common/gapi_tests_common.hpp"

#include <opencv2/gapi/cpu/core.hpp>
#include <opencv2/gapi/ocl/core.hpp>

#include <opencv2/gapi/streaming/onevpl/source.hpp>

#ifdef HAVE_DIRECTX
#ifdef HAVE_D3D11
#pragma comment(lib,"d3d11.lib")

// get rid of generate macro max/min/etc from DX side
#define D3D11_NO_HELPERS
#define NOMINMAX
#include <d3d11.h>
#include <codecvt>
#include "opencv2/core/directx.hpp"
#undef D3D11_NO_HELPERS
#undef NOMINMAX
#endif // HAVE_D3D11
#endif // HAVE_DIRECTX

#ifdef __linux__
#if defined(HAVE_VA) || defined(HAVE_VA_INTEL)
#include "va/va.h"
#include "va/va_drm.h"

#include <fcntl.h>
#include <unistd.h>
#endif // defined(HAVE_VA) || defined(HAVE_VA_INTEL)
#endif // __linux__

#ifdef HAVE_ONEVPL
#include "streaming/onevpl/onevpl_export.hpp"
#include "streaming/onevpl/cfg_param_device_selector.hpp"

namespace opencv_test
{
namespace
{

void test_dev_eq(const typename cv::gapi::wip::onevpl::IDeviceSelector::DeviceScoreTable::value_type &scored_device,
                 cv::gapi::wip::onevpl::IDeviceSelector::Score expected_score,
                 cv::gapi::wip::onevpl::AccelType expected_type,
                 cv::gapi::wip::onevpl::Device::Ptr expected_ptr) {
    EXPECT_EQ(std::get<0>(scored_device), expected_score);
    EXPECT_EQ(std::get<1>(scored_device).get_type(), expected_type);
    EXPECT_EQ(std::get<1>(scored_device).get_ptr(), expected_ptr);
}

void test_ctx_eq(const typename cv::gapi::wip::onevpl::IDeviceSelector::DeviceContexts::value_type &ctx,
                 cv::gapi::wip::onevpl::AccelType expected_type,
                 cv::gapi::wip::onevpl::Context::Ptr expected_ptr) {
    EXPECT_EQ(ctx.get_type(), expected_type);
    EXPECT_EQ(ctx.get_ptr(), expected_ptr);
}

void test_host_dev_eq(const typename cv::gapi::wip::onevpl::IDeviceSelector::DeviceScoreTable::value_type &scored_device,
                      cv::gapi::wip::onevpl::IDeviceSelector::Score expected_score) {
    test_dev_eq(scored_device, expected_score,
                cv::gapi::wip::onevpl::AccelType::HOST, nullptr);
}

void test_host_ctx_eq(const typename cv::gapi::wip::onevpl::IDeviceSelector::DeviceContexts::value_type &ctx) {
    test_ctx_eq(ctx, cv::gapi::wip::onevpl::AccelType::HOST, nullptr);
}

TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDevice)
{
    using namespace cv::gapi::wip::onevpl;
    CfgParamDeviceSelector selector;
    IDeviceSelector::DeviceScoreTable devs = selector.select_devices();
    EXPECT_TRUE(devs.size() == 1);
    test_host_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority);

    IDeviceSelector::DeviceContexts ctxs = selector.select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    test_host_ctx_eq(*ctxs.begin());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithEmptyCfgParam)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> empty_params;
    CfgParamDeviceSelector selector(empty_params);
    IDeviceSelector::DeviceScoreTable devs = selector.select_devices();
    EXPECT_TRUE(devs.size() == 1);
    test_host_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority);
    IDeviceSelector::DeviceContexts ctxs = selector.select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    test_host_ctx_eq(*ctxs.begin());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithAccelNACfgParam)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_no_accel;
    cfg_params_w_no_accel.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_NA));
    CfgParamDeviceSelector selector(cfg_params_w_no_accel);
    IDeviceSelector::DeviceScoreTable devs = selector.select_devices();
    EXPECT_TRUE(devs.size() == 1);
    test_host_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority);

    IDeviceSelector::DeviceContexts ctxs = selector.select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    test_host_ctx_eq(*ctxs.begin());
}

#ifdef HAVE_DIRECTX
#ifdef HAVE_D3D11
TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithEmptyCfgParam_DX11_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> empty_params;
    CfgParamDeviceSelector selector(empty_params);
    IDeviceSelector::DeviceScoreTable devs = selector.select_devices();
    EXPECT_TRUE(devs.size() == 1);
    test_host_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority);

    IDeviceSelector::DeviceContexts ctxs = selector.select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    test_host_ctx_eq(*ctxs.begin());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithDX11AccelCfgParam_DX11_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_dx11;
    cfg_params_w_dx11.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_D3D11));
    std::unique_ptr<CfgParamDeviceSelector> selector_ptr;
    EXPECT_NO_THROW(selector_ptr.reset(new CfgParamDeviceSelector(cfg_params_w_dx11)));
    IDeviceSelector::DeviceScoreTable devs = selector_ptr->select_devices();

    EXPECT_TRUE(devs.size() == 1);
    test_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority,
                AccelType::DX11,
                std::get<1>(*devs.begin()).get_ptr() /* compare just type */);

    IDeviceSelector::DeviceContexts ctxs = selector_ptr->select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    EXPECT_TRUE(ctxs.begin()->get_ptr());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, NULLDeviceWithDX11AccelCfgParam_DX11_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_dx11;
    cfg_params_w_dx11.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_D3D11));
    Device::Ptr empty_device_ptr = nullptr;
    Context::Ptr empty_ctx_ptr = nullptr;
    EXPECT_THROW(CfgParamDeviceSelector sel(empty_device_ptr, "GPU",
                                            empty_ctx_ptr,
                                            cfg_params_w_dx11),
                 std::logic_error); // empty_device_ptr must be invalid
}

TEST(OneVPL_Source_Device_Selector_CfgParam, ExternalDeviceWithDX11AccelCfgParam_DX11_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    ID3D11Device *device = nullptr;
    ID3D11DeviceContext* device_context = nullptr;
    {
        UINT flags = 0;
        D3D_FEATURE_LEVEL features[] = { D3D_FEATURE_LEVEL_11_1,
                                         D3D_FEATURE_LEVEL_11_0,
                                       };
        D3D_FEATURE_LEVEL feature_level;

        // Create the Direct3D 11 API device object and a corresponding context.
        HRESULT err = D3D11CreateDevice(nullptr, D3D_DRIVER_TYPE_HARDWARE,
                                        nullptr, flags,
                                        features,
                                        ARRAYSIZE(features), D3D11_SDK_VERSION,
                                        &device, &feature_level, &device_context);
        EXPECT_FALSE(FAILED(err));
    }

    std::unique_ptr<CfgParamDeviceSelector> selector_ptr;
    std::vector<CfgParam> cfg_params_w_dx11;
    cfg_params_w_dx11.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_D3D11));
    EXPECT_NO_THROW(selector_ptr.reset(new CfgParamDeviceSelector(device, "GPU",
                                                                  device_context,
                                                                  cfg_params_w_dx11)));
    IDeviceSelector::DeviceScoreTable devs = selector_ptr->select_devices();

    EXPECT_TRUE(devs.size() == 1);
    test_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority,
                AccelType::DX11, device);

    IDeviceSelector::DeviceContexts ctxs = selector_ptr->select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    EXPECT_EQ(reinterpret_cast<ID3D11DeviceContext*>(ctxs.begin()->get_ptr()),
              device_context);
}

#endif // HAVE_D3D11
#endif // HAVE_DIRECTX

#ifndef HAVE_DIRECTX
#ifndef HAVE_D3D11
TEST(OneVPL_Source_Device_Selector_CfgParam, DX11DeviceFromCfgParamWithDX11Disabled)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_non_existed_dx11;
    cfg_params_w_non_existed_dx11.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_D3D11));
    EXPECT_THROW(CfgParamDeviceSelector{cfg_params_w_non_existed_dx11},
                 std::logic_error);
}
#endif // HAVE_D3D11
#endif // HAVE_DIRECTX

#ifdef __linux__
#if defined(HAVE_VA) || defined(HAVE_VA_INTEL)
TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithEmptyCfgParam_VAAPI_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> empty_params;
    CfgParamDeviceSelector selector(empty_params);
    IDeviceSelector::DeviceScoreTable devs = selector.select_devices();
    EXPECT_TRUE(devs.size() == 1);
    test_host_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority);

    IDeviceSelector::DeviceContexts ctxs = selector.select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    test_host_ctx_eq(*ctxs.begin());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, DefaultDeviceWithVAAPIAccelCfgParam_VAAPI_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_vaapi;
    cfg_params_w_vaapi.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_VAAPI));
    std::unique_ptr<CfgParamDeviceSelector> selector_ptr;
    EXPECT_NO_THROW(selector_ptr.reset(new CfgParamDeviceSelector(cfg_params_w_vaapi)));
    IDeviceSelector::DeviceScoreTable devs = selector_ptr->select_devices();

    EXPECT_TRUE(devs.size() == 1);
    test_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority,
                AccelType::VAAPI,
                std::get<1>(*devs.begin()).get_ptr() /* compare just type */);

    IDeviceSelector::DeviceContexts ctxs = selector_ptr->select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    EXPECT_FALSE(ctxs.begin()->get_ptr());
}

TEST(OneVPL_Source_Device_Selector_CfgParam, NULLDeviceWithVAAPIAccelCfgParam_VAAPI_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> cfg_params_w_vaapi;
    cfg_params_w_vaapi.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_VAAPI));
    Device::Ptr empty_device_ptr = nullptr;
    Context::Ptr empty_ctx_ptr = nullptr;
    EXPECT_THROW(CfgParamDeviceSelector sel(empty_device_ptr, "GPU",
                                            empty_ctx_ptr,
                                            cfg_params_w_vaapi),
                 std::logic_error); // empty_device_ptr must be invalid
}


TEST(OneVPL_Source_Device_Selector_CfgParam, ExternalDeviceWithVAAPIAccelCfgParam_VAAPI_ENABLED)
{
    using namespace cv::gapi::wip::onevpl;
    VADisplay va_handle = nullptr;
    struct FileDescriptorRAII {
        FileDescriptorRAII() :fd (-1) {}
        ~FileDescriptorRAII() { reset(-1); }
        void reset(int d) {
            if (fd != -1) {
                close(fd);
            }
            fd = d;
        }
        operator int() { return fd; }
    private:
        FileDescriptorRAII(FileDescriptorRAII& src) = delete;
        FileDescriptorRAII& operator=(FileDescriptorRAII& src) = delete;
        FileDescriptorRAII(FileDescriptorRAII&& src) = delete;
        FileDescriptorRAII& operator=(FileDescriptorRAII&& src) = delete;
        int fd = -1;
    };
    static const char *predefined_vaapi_devices_list[] {"/dev/dri/renderD128",
                                                        "/dev/dri/renderD129",
                                                        "/dev/dri/card0",
                                                        "/dev/dri/card1",
                                                        nullptr};

    FileDescriptorRAII device_fd;
    for (const char **device_path = predefined_vaapi_devices_list;
        *device_path != nullptr; device_path++) {
        device_fd.reset(open(*device_path, O_RDWR));
        if (device_fd < 0) {
            continue;
        }
        va_handle = vaGetDisplayDRM(device_fd);
        if (!va_handle) {
            continue;
        }
        int major_version = 0, minor_version = 0;
        VAStatus status {};
        status = vaInitialize(va_handle, &major_version, &minor_version);
        if (VA_STATUS_SUCCESS != status) {
            close(device_fd);
            va_handle = nullptr;
            continue;
        }
        break;
    }
    EXPECT_TRUE(device_fd != -1);
    EXPECT_TRUE(va_handle);
    auto device = cv::util::make_optional(
                            cv::gapi::wip::onevpl::create_vaapi_device(reinterpret_cast<void*>(va_handle),
                                                                       "GPU", device_fd));
    auto device_context = cv::util::make_optional(
                            cv::gapi::wip::onevpl::create_vaapi_context(nullptr));

    std::unique_ptr<CfgParamDeviceSelector> selector_ptr;
    std::vector<CfgParam> cfg_params_w_vaapi;
    cfg_params_w_vaapi.push_back(CfgParam::create_acceleration_mode(MFX_ACCEL_MODE_VIA_VAAPI));
    EXPECT_NO_THROW(selector_ptr.reset(new CfgParamDeviceSelector(device.value(),
                                                                  device_context.value(),
                                                                  cfg_params_w_vaapi)));
    IDeviceSelector::DeviceScoreTable devs = selector_ptr->select_devices();

    EXPECT_TRUE(devs.size() == 1);
    test_dev_eq(*devs.begin(), IDeviceSelector::Score::MaxActivePriority,
                AccelType::VAAPI, device.value().get_ptr());

    IDeviceSelector::DeviceContexts ctxs = selector_ptr->select_context();
    EXPECT_TRUE(ctxs.size() == 1);
    EXPECT_EQ(reinterpret_cast<void*>(ctxs.begin()->get_ptr()),
              device_context.value().get_ptr());
}
#endif // defined(HAVE_VA) || defined(HAVE_VA_INTEL)
#endif // #ifdef __linux__

TEST(OneVPL_Source_Device_Selector_CfgParam, UnknownPtrDeviceFromCfgParam)
{
    using namespace cv::gapi::wip::onevpl;
    std::vector<CfgParam> empty_params;
    Device::Ptr empty_device_ptr = nullptr;
    Context::Ptr empty_ctx_ptr = nullptr;
    EXPECT_THROW(CfgParamDeviceSelector sel(empty_device_ptr, "",
                                            empty_ctx_ptr,
                                            empty_params),
                 std::logic_error); // params must describe device_ptr explicitly
}
}
} // namespace opencv_test
#endif // HAVE_ONEVPL
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

- **FileDescriptorRAII**: A class/struct defined in this file

### Functions and Methods

- **D3D11_NO_HELPERS()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file
- **NOMINMAX()**: A function/method defined in this file
- **HAVE_D3D11()**: A function/method defined in this file
- **__linux__()**: A function/method defined in this file
- **HAVE_DIRECTX()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/directx.hpp`
- `opencv2/gapi/ocl/core.hpp`
- `streaming/onevpl/cfg_param_device_selector.hpp`
- `unistd.h`
- `d3d11.h`
- `streaming/onevpl/onevpl_export.hpp`
- `../test_precomp.hpp`
- `va/va.h`
- `opencv2/gapi/streaming/onevpl/source.hpp`
- `va/va_drm.h`
- `opencv2/gapi/cpu/core.hpp`
- `../common/gapi_tests_common.hpp`
- `codecvt`
- `fcntl.h`

**Python Imports:**
- `DX`


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

