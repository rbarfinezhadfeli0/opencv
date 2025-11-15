# Documentation for `modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp`

## File Metadata

- **Full Path**: `modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp`
- **File Name**: `cfg_params.hpp`
- **File Size**: 9,047 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp](../../../../../../../modules/gapi/include/opencv2/gapi/streaming/onevpl/cfg_params.hpp)

## Purpose and Role

This file is located in the `modules/gapi/include/opencv2/gapi/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#ifndef OPENCV_GAPI_STREAMING_ONEVPL_CFG_PARAMS_HPP
#define OPENCV_GAPI_STREAMING_ONEVPL_CFG_PARAMS_HPP

#include <map>
#include <memory>
#include <string>

#include <opencv2/gapi/streaming/source.hpp>
#include <opencv2/gapi/util/variant.hpp>

namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

/**
 * @brief Public class is using for creation of onevpl::GSource instances.
 *
 * Class members available through methods @ref CfgParam::get_name() and @ref CfgParam::get_value() are used by
 * onevpl::GSource inner logic to create or find oneVPL particular implementation
 * (software/hardware, specific API version and etc.).
 *
 * @note Because oneVPL may provide several implementations which are satisfying with multiple (or single one) @ref CfgParam
 * criteria therefore it is possible to configure `preferred` parameters. This kind of CfgParams are created
 * using `is_major = false` argument in @ref CfgParam::create method and are not used by creating oneVPL particular implementations.
 * Instead they fill out a "score table" to select preferable implementation from available list. Implementation are satisfying
 * with most of these optional params would be chosen.
 * If no one optional CfgParam params were present then first of available oneVPL implementation would be applied.
 * Please get on https://spec.oneapi.io/versions/latest/elements/oneVPL/source/API_ref/VPL_disp_api_func.html?highlight=mfxcreateconfig#mfxsetconfigfilterproperty
 * for using OneVPL configuration. In this schema `mfxU8 *name` represents @ref CfgParam::get_name() and
 * `mfxVariant value` is @ref CfgParam::get_value()
 */
struct GAPI_EXPORTS CfgParam {
    using name_t = std::string;
    using value_t = cv::util::variant<uint8_t, int8_t,
                                      uint16_t, int16_t,
                                      uint32_t, int32_t,
                                      uint64_t, int64_t,
                                      float_t,
                                      double_t,
                                      void*,
                                      std::string>;
    /**
     * @brief frames_pool_size_name
     *
     * Special configuration parameter name for onevp::GSource:
     *
     * @note frames_pool_size_name allows to allocate surfaces pool appropriate size to keep
     * decoded frames in accelerator memory ready before
     * they would be consumed by onevp::GSource::pull operation. If you see
     * a lot of WARNING about lack of free surface then it's time to increase
     * frames_pool_size_name but be aware of accelerator free memory volume.
     * If not set then MFX implementation use
     * mfxFrameAllocRequest::NumFrameSuggested behavior
     *
     */
    static constexpr const char *frames_pool_size_name() { return "frames_pool_size"; }
    static CfgParam create_frames_pool_size(size_t value);

    /**
     * @brief acceleration_mode_name
     *
     * Special configuration parameter names for onevp::GSource:
     *
     * @note acceleration_mode_name allows to activate hardware acceleration &
     * device memory management.
     * Supported values:
     * - MFX_ACCEL_MODE_VIA_D3D11   Will activate DX11 acceleration and will produces
     * MediaFrames with data allocated in DX11 device memory
     *
     * If not set then MFX implementation will use default acceleration behavior:
     * all decoding operation uses default GPU resources but MediaFrame produces
     * data allocated by using host RAM
     *
     */
    static constexpr const char *acceleration_mode_name() { return "mfxImplDescription.AccelerationMode"; }
    static CfgParam create_acceleration_mode(uint32_t value);
    static CfgParam create_acceleration_mode(const char* value);

    /**
     * @brief decoder_id_name
     *
     * Special configuration parameter names for onevp::GSource:
     *
     * @note decoder_id_name allows to specify VPL decoder type which MUST present
     * in case of RAW video input data and MUST NOT present as CfgParam if video
     * stream incapsulated into container(*.mp4, *.mkv and so on). In latter case
     * onevp::GSource will determine it automatically
     * Supported values:
     * - MFX_CODEC_AVC
     * - MFX_CODEC_HEVC
     * - MFX_CODEC_MPEG2
     * - MFX_CODEC_VC1
     * - MFX_CODEC_CAPTURE
     * - MFX_CODEC_VP9
     * - MFX_CODEC_AV1
     *
     */
    static constexpr const char *decoder_id_name() { return "mfxImplDescription.mfxDecoderDescription.decoder.CodecID"; }
    static CfgParam create_decoder_id(uint32_t value);
    static CfgParam create_decoder_id(const char* value);

    static constexpr const char *implementation_name() { return "mfxImplDescription.Impl"; }
    static CfgParam create_implementation(uint32_t value);
    static CfgParam create_implementation(const char* value);


    static constexpr const char *vpp_frames_pool_size_name() { return "vpp_frames_pool_size"; }
    static CfgParam create_vpp_frames_pool_size(size_t value);

    static constexpr const char *vpp_in_width_name() { return "vpp.In.Width"; }
    static CfgParam create_vpp_in_width(uint16_t value);

    static constexpr const char *vpp_in_height_name() { return "vpp.In.Height"; }
    static CfgParam create_vpp_in_height(uint16_t value);

    static constexpr const char *vpp_in_crop_x_name() { return "vpp.In.CropX"; }
    static CfgParam create_vpp_in_crop_x(uint16_t value);

    static constexpr const char *vpp_in_crop_y_name() { return "vpp.In.CropY"; }
    static CfgParam create_vpp_in_crop_y(uint16_t value);

    static constexpr const char *vpp_in_crop_w_name() { return "vpp.In.CropW"; }
    static CfgParam create_vpp_in_crop_w(uint16_t value);

    static constexpr const char *vpp_in_crop_h_name() { return "vpp.In.CropH"; }
    static CfgParam create_vpp_in_crop_h(uint16_t value);


    static constexpr const char *vpp_out_fourcc_name() { return "vpp.Out.FourCC"; }
    static CfgParam create_vpp_out_fourcc(uint32_t value);

    static constexpr const char *vpp_out_chroma_format_name() { return "vpp.Out.ChromaFormat"; }
    static CfgParam create_vpp_out_chroma_format(uint16_t value);

    static constexpr const char *vpp_out_width_name() { return "vpp.Out.Width"; }
    static CfgParam create_vpp_out_width(uint16_t value);

    static constexpr const char *vpp_out_height_name() { return "vpp.Out.Height"; }
    static CfgParam create_vpp_out_height(uint16_t value);

    static constexpr const char *vpp_out_crop_x_name() { return "vpp.Out.CropX"; }
    static CfgParam create_vpp_out_crop_x(uint16_t value);

    static constexpr const char *vpp_out_crop_y_name() { return "vpp.Out.CropY"; }
    static CfgParam create_vpp_out_crop_y(uint16_t value);

    static constexpr const char *vpp_out_crop_w_name() { return "vpp.Out.CropW"; }
    static CfgParam create_vpp_out_crop_w(uint16_t value);

    static constexpr const char *vpp_out_crop_h_name() { return "vpp.Out.CropH"; }
    static CfgParam create_vpp_out_crop_h(uint16_t value);

    static constexpr const char *vpp_out_pic_struct_name() { return "vpp.Out.PicStruct"; }
    static CfgParam create_vpp_out_pic_struct(uint16_t value);

    static constexpr const char *vpp_out_framerate_n_name() { return "vpp.Out.FrameRateExtN"; }
    static CfgParam create_vpp_out_framerate_n(uint32_t value);

    static constexpr const char *vpp_out_framerate_d_name() { return "vpp.Out.FrameRateExtD"; }
    static CfgParam create_vpp_out_framerate_d(uint32_t value);

    /**
     * Create generic onevp::GSource configuration parameter.
     *
     *@param name           name of parameter.
     *@param value          value of parameter.
     *@param is_major       TRUE if parameter MUST be provided by OneVPL inner implementation, FALSE for optional (for resolve multiple available implementations).
     *
     */
    template<typename ValueType>
    static CfgParam create(const std::string& name, ValueType&& value, bool is_major = true) {
        CfgParam param(name, CfgParam::value_t(std::forward<ValueType>(value)), is_major);
        return param;
    }

    struct Priv;

    const name_t& get_name() const;
    const value_t& get_value() const;
    bool is_major() const;
    std::string to_string() const;

    bool operator==(const CfgParam& rhs) const;
    bool operator< (const CfgParam& rhs) const;
    bool operator!=(const CfgParam& rhs) const;

    CfgParam& operator=(const CfgParam& src);
    CfgParam& operator=(CfgParam&& src);
    CfgParam(const CfgParam& src);
    CfgParam(CfgParam&& src);
    ~CfgParam();
private:
    CfgParam(const std::string& param_name, value_t&& param_value, bool is_major_param);
    std::shared_ptr<Priv> m_priv;
};

} //namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv

#endif // OPENCV_GAPI_STREAMING_ONEVPL_CFG_PARAMS_HPP
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

- **GAPI_EXPORTS**: A class/struct defined in this file
- **is**: A class/struct defined in this file
- **Priv**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_GAPI_STREAMING_ONEVPL_CFG_PARAMS_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/util/variant.hpp`
- `string`
- `memory`
- `opencv2/gapi/streaming/source.hpp`
- `map`

**Python Imports:**
- `available`


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

