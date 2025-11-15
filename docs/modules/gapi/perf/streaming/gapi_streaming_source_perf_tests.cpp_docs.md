# Documentation for `modules/gapi/perf/streaming/gapi_streaming_source_perf_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/perf/streaming/gapi_streaming_source_perf_tests.cpp`
- **File Name**: `gapi_streaming_source_perf_tests.cpp`
- **File Size**: 15,035 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/perf/streaming/gapi_streaming_source_perf_tests.cpp](../../../../modules/gapi/perf/streaming/gapi_streaming_source_perf_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/perf/streaming` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifdef HAVE_ONEVPL

#include "../perf_precomp.hpp"
#include "../../test/common/gapi_tests_common.hpp"
#include <opencv2/gapi/streaming/onevpl/source.hpp>
#include <opencv2/gapi/streaming/cap.hpp>

#include "streaming/onevpl/engine/preproc/preproc_engine.hpp"
#include "streaming/onevpl/engine/preproc/preproc_session.hpp"
#include "streaming/onevpl/accelerators/accel_policy_interface.hpp"
#include "streaming/onevpl/cfg_param_device_selector.hpp"
#include "streaming/onevpl/accelerators/accel_policy_dx11.hpp"
#include "streaming/onevpl/accelerators/accel_policy_cpu.hpp"
#include "streaming/onevpl/accelerators/accel_policy_va_api.hpp"

namespace opencv_test
{
using namespace perf;

const std::string files[] = {
    "highgui/video/big_buck_bunny.h265",
    "highgui/video/big_buck_bunny.h264",
    "highgui/video/sample_322x242_15frames.yuv420p.libx265.mp4",
};

const std::string codec[] = {
    "MFX_CODEC_HEVC",
    "MFX_CODEC_AVC",
    "",
};

using source_t = std::string;
using codec_t = std::string;
using accel_mode_t = std::string;
using source_description_t = std::tuple<source_t, codec_t, accel_mode_t>;

class OneVPLSourcePerf_Test : public TestPerfParams<source_description_t> {};
class VideoCapSourcePerf_Test : public TestPerfParams<source_t> {};

PERF_TEST_P_(OneVPLSourcePerf_Test, TestPerformance)
{
    using namespace cv::gapi::wip::onevpl;

    const auto params = GetParam();
    source_t src = findDataFile(get<0>(params));
    codec_t type = get<1>(params);
    accel_mode_t mode = get<2>(params);

    std::vector<CfgParam> cfg_params {
        CfgParam::create_implementation("MFX_IMPL_TYPE_HARDWARE"),
    };

    if (!type.empty()) {
        cfg_params.push_back(CfgParam::create_decoder_id(type.c_str()));
    }

    if (!mode.empty()) {
        cfg_params.push_back(CfgParam::create_acceleration_mode(mode.c_str()));
    }

    auto source_ptr = cv::gapi::wip::make_onevpl_src(src, cfg_params);

    cv::gapi::wip::Data out;
    TEST_CYCLE()
    {
        source_ptr->pull(out);
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P_(VideoCapSourcePerf_Test, TestPerformance)
{
    using namespace cv::gapi::wip;

    source_t src = findDataFile(GetParam());
    auto source_ptr = make_src<GCaptureSource>(src);
    Data out;
    TEST_CYCLE()
    {
        source_ptr->pull(out);
    }

    SANITY_CHECK_NOTHING();
}

#ifdef __WIN32__
INSTANTIATE_TEST_CASE_P(Streaming, OneVPLSourcePerf_Test,
                        Values(source_description_t(files[0], codec[0], ""),
                               source_description_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11"),
                               source_description_t(files[1], codec[1], ""),
                               source_description_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11"),
                               source_description_t(files[2], codec[2], ""),
                               source_description_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11")));
#elif __linux__
INSTANTIATE_TEST_CASE_P(Streaming, OneVPLSourcePerf_Test,
                        Values(source_description_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI"),
                               source_description_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI")));
#endif

INSTANTIATE_TEST_CASE_P(Streaming, VideoCapSourcePerf_Test,
                        Values(files[0],
                               files[1],
                               files[2]));

using pp_out_param_t = cv::GFrameDesc;
using source_description_preproc_t = decltype(std::tuple_cat(std::declval<source_description_t>(),
                                                             std::declval<std::tuple<pp_out_param_t>>()));
class OneVPLSourcePerf_PP_Test : public TestPerfParams<source_description_preproc_t> {};

PERF_TEST_P_(OneVPLSourcePerf_PP_Test, TestPerformance)
{
    using namespace cv::gapi::wip::onevpl;

    const auto params = GetParam();
    source_t src = findDataFile(get<0>(params));
    codec_t type = get<1>(params);
    accel_mode_t mode = get<2>(params);
    pp_out_param_t res = get<3>(params);

    std::vector<CfgParam> cfg_params {
        CfgParam::create_implementation("MFX_IMPL_TYPE_HARDWARE"),
    };

    if (!type.empty()) {
        cfg_params.push_back(CfgParam::create_decoder_id(type.c_str()));
    }

    if (!mode.empty()) {
        cfg_params.push_back(CfgParam::create_acceleration_mode(mode.c_str()));
    }

    cfg_params.push_back(CfgParam::create_vpp_out_width(static_cast<uint16_t>(res.size.width)));
    cfg_params.push_back(CfgParam::create_vpp_out_height(static_cast<uint16_t>(res.size.height)));
    cfg_params.push_back(CfgParam::create_vpp_out_crop_x(0));
    cfg_params.push_back(CfgParam::create_vpp_out_crop_y(0));
    cfg_params.push_back(CfgParam::create_vpp_out_crop_w(static_cast<uint16_t>(res.size.width)));
    cfg_params.push_back(CfgParam::create_vpp_out_crop_h(static_cast<uint16_t>(res.size.height)));

    auto source_ptr = cv::gapi::wip::make_onevpl_src(src, cfg_params);

    cv::gapi::wip::Data out;
    TEST_CYCLE()
    {
        source_ptr->pull(out);
    }

    SANITY_CHECK_NOTHING();
}
static pp_out_param_t full_hd = pp_out_param_t {cv::MediaFormat::NV12,
                                                {1920, 1080}};

static pp_out_param_t cif = pp_out_param_t {cv::MediaFormat::NV12,
                                            {352, 288}};


#ifdef __WIN32__
INSTANTIATE_TEST_CASE_P(Streaming_Source_PP, OneVPLSourcePerf_PP_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "", full_hd),
                               source_description_preproc_t(files[0], codec[0], "", cif),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11", full_hd),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11", cif),
                               source_description_preproc_t(files[1], codec[1], "", full_hd),
                               source_description_preproc_t(files[1], codec[1], "", cif),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11",full_hd),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11",cif),
                               source_description_preproc_t(files[2], codec[2], "", full_hd),
                               source_description_preproc_t(files[2], codec[2], "", cif),
                               source_description_preproc_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11", full_hd),
                               source_description_preproc_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11", cif)));
#elif __linux__
INSTANTIATE_TEST_CASE_P(Streaming_Source_PP, OneVPLSourcePerf_PP_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI", full_hd),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI", cif),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI",full_hd),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI",cif)));
#endif

class OneVPLSourcePerf_PP_Engine_Test : public TestPerfParams<source_description_preproc_t> {};

PERF_TEST_P_(OneVPLSourcePerf_PP_Engine_Test, TestPerformance)
{
    using namespace cv::gapi::wip;
    using namespace cv::gapi::wip::onevpl;

    const auto params = GetParam();
    source_t src = findDataFile(get<0>(params));
    codec_t type = get<1>(params);
    accel_mode_t mode = get<2>(params);
    const pp_out_param_t &required_frame_param = get<3>(params);

    std::vector<CfgParam> cfg_params {
        CfgParam::create_implementation("MFX_IMPL_TYPE_HARDWARE"),
    };

    if (!type.empty()) {
        cfg_params.push_back(CfgParam::create_decoder_id(type.c_str()));
    }

    if (!mode.empty()) {
        cfg_params.push_back(CfgParam::create_acceleration_mode(mode.c_str()));
    }

    auto device_selector = std::make_shared<CfgParamDeviceSelector>(cfg_params);
    auto source_ptr = cv::gapi::wip::make_onevpl_src(src, cfg_params, device_selector);

    // create VPP preproc engine
    std::unique_ptr<VPLAccelerationPolicy> policy;
    if (mode == "MFX_ACCEL_MODE_VIA_D3D11") {
        policy.reset(new VPLDX11AccelerationPolicy(device_selector));
    } else if (mode == "MFX_ACCEL_MODE_VIA_VAAPI") {
        policy.reset(new VPLVAAPIAccelerationPolicy(device_selector));
    } else if (mode.empty()){
        policy.reset(new VPLCPUAccelerationPolicy(device_selector));
    } else {
        ASSERT_TRUE(false && "Unsupported acceleration policy type");
    }
    VPPPreprocEngine preproc_engine(std::move(policy));
    cv::gapi::wip::Data out;
    cv::util::optional<cv::Rect> empty_roi;
    TEST_CYCLE()
    {
        source_ptr->pull(out);
        cv::MediaFrame frame = cv::util::get<cv::MediaFrame>(out);
        cv::util::optional<pp_params> param = preproc_engine.is_applicable(frame);
        pp_session sess = preproc_engine.initialize_preproc(param.value(),
                                                            required_frame_param);
        (void)preproc_engine.run_sync(sess, frame, empty_roi);
    }

    SANITY_CHECK_NOTHING();
}

#ifdef __WIN32__
INSTANTIATE_TEST_CASE_P(Streaming_Engine_PP, OneVPLSourcePerf_PP_Engine_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "", full_hd),
                               source_description_preproc_t(files[0], codec[0], "", cif),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11", full_hd),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11", cif),
                               source_description_preproc_t(files[1], codec[1], "", full_hd),
                               source_description_preproc_t(files[1], codec[1], "", cif),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11",full_hd),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11",cif),
                               source_description_preproc_t(files[2], codec[2], "", full_hd),
                               source_description_preproc_t(files[2], codec[2], "", cif),
                               source_description_preproc_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11", full_hd),
                               source_description_preproc_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11", cif)));
#elif __linux__
INSTANTIATE_TEST_CASE_P(Streaming_Engine_PP, OneVPLSourcePerf_PP_Engine_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI", full_hd),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI", cif),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI",full_hd),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI",cif)));
#endif

class OneVPLSourcePerf_PP_Engine_Bypass_Test : public TestPerfParams<source_description_preproc_t> {};

PERF_TEST_P_(OneVPLSourcePerf_PP_Engine_Bypass_Test, TestPerformance)
{
    using namespace cv::gapi::wip;
    using namespace cv::gapi::wip::onevpl;

    const auto params = GetParam();
    source_t src = findDataFile(get<0>(params));
    codec_t type = get<1>(params);
    accel_mode_t mode = get<2>(params);
    const pp_out_param_t &required_frame_param = get<3>(params);

    std::vector<CfgParam> cfg_params {
        CfgParam::create_implementation("MFX_IMPL_TYPE_HARDWARE"),
    };

    if (!type.empty()) {
        cfg_params.push_back(CfgParam::create_decoder_id(type.c_str()));
    }

    if (!mode.empty()) {
        cfg_params.push_back(CfgParam::create_acceleration_mode(mode.c_str()));
    }

    auto device_selector = std::make_shared<CfgParamDeviceSelector>(cfg_params);
    auto source_ptr = cv::gapi::wip::make_onevpl_src(src, cfg_params, device_selector);

    // create VPP preproc engine
    std::unique_ptr<VPLAccelerationPolicy> policy;
    if (mode == "MFX_ACCEL_MODE_VIA_D3D11") {
        policy.reset(new VPLDX11AccelerationPolicy(device_selector));
    } else if (mode == "MFX_ACCEL_MODE_VIA_VAAPI") {
        policy.reset(new VPLVAAPIAccelerationPolicy(device_selector));
    } else if (mode.empty()){
        policy.reset(new VPLCPUAccelerationPolicy(device_selector));
    } else {
        ASSERT_TRUE(false && "Unsupported acceleration policy type");
    }
    VPPPreprocEngine preproc_engine(std::move(policy));
    cv::gapi::wip::Data out;
    cv::util::optional<cv::Rect> empty_roi;
    TEST_CYCLE()
    {
        source_ptr->pull(out);
        cv::MediaFrame frame = cv::util::get<cv::MediaFrame>(out);
        cv::util::optional<pp_params> param = preproc_engine.is_applicable(frame);
        pp_session sess = preproc_engine.initialize_preproc(param.value(),
                                                            required_frame_param);
        (void)preproc_engine.run_sync(sess, frame, empty_roi);
    }

    SANITY_CHECK_NOTHING();
}

static pp_out_param_t res_672x384 = pp_out_param_t {cv::MediaFormat::NV12,
                                                    {672, 384}};
static pp_out_param_t res_336x256 = pp_out_param_t {cv::MediaFormat::NV12,
                                                    {336, 256}};

#ifdef __WIN32__
INSTANTIATE_TEST_CASE_P(Streaming_Engine_PP_Bypass, OneVPLSourcePerf_PP_Engine_Bypass_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "", res_672x384),
                               source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_D3D11", res_672x384),
                               source_description_preproc_t(files[1], codec[1], "", res_672x384),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_D3D11", res_672x384),
                               source_description_preproc_t(files[2], codec[2], "", res_336x256),
                               source_description_preproc_t(files[2], codec[2], "MFX_ACCEL_MODE_VIA_D3D11", res_336x256)));
#elif __linux__
INSTANTIATE_TEST_CASE_P(Streaming_Engine_PP_Bypass, OneVPLSourcePerf_PP_Engine_Bypass_Test,
                        Values(source_description_preproc_t(files[0], codec[0], "MFX_ACCEL_MODE_VIA_VAAPI", res_672x384),
                               source_description_preproc_t(files[1], codec[1], "MFX_ACCEL_MODE_VIA_VAAPI", res_672x384)));
#endif
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

- **OneVPLSourcePerf_Test**: A class/struct defined in this file
- **OneVPLSourcePerf_PP_Engine_Test**: A class/struct defined in this file
- **OneVPLSourcePerf_PP_Test**: A class/struct defined in this file
- **OneVPLSourcePerf_PP_Engine_Bypass_Test**: A class/struct defined in this file
- **VideoCapSourcePerf_Test**: A class/struct defined in this file

### Functions and Methods

- **__WIN32__()**: A function/method defined in this file
- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `streaming/onevpl/accelerators/accel_policy_interface.hpp`
- `../../test/common/gapi_tests_common.hpp`
- `opencv2/gapi/streaming/cap.hpp`
- `streaming/onevpl/cfg_param_device_selector.hpp`
- `streaming/onevpl/engine/preproc/preproc_session.hpp`
- `streaming/onevpl/accelerators/accel_policy_va_api.hpp`
- `streaming/onevpl/engine/preproc/preproc_engine.hpp`
- `streaming/onevpl/accelerators/accel_policy_dx11.hpp`
- `opencv2/gapi/streaming/onevpl/source.hpp`
- `streaming/onevpl/accelerators/accel_policy_cpu.hpp`
- `../perf_precomp.hpp`


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

