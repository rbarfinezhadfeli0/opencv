# Documentation for `modules/gapi/src/streaming/onevpl/cfg_params_parser.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/streaming/onevpl/cfg_params_parser.cpp`
- **File Name**: `cfg_params_parser.cpp`
- **File Size**: 10,564 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/streaming/onevpl/cfg_params_parser.cpp](../../../../../modules/gapi/src/streaming/onevpl/cfg_params_parser.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/streaming/onevpl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2021 Intel Corporation

#include <stdio.h>

#include <algorithm>
#include <sstream>

#include "streaming/onevpl/cfg_params_parser.hpp"
#include "streaming/onevpl/utils.hpp"
#include "logger.hpp"

#ifdef HAVE_ONEVPL
namespace cv {
namespace gapi {
namespace wip {
namespace onevpl {

template <>
struct ParamCreator<CfgParam> {
    template<typename ValueType>
    CfgParam create (const std::string& name, ValueType&& value, bool is_major_flag = false) {
        return CfgParam::create(name, std::forward<ValueType>(value), is_major_flag);
    }
};

template <>
struct ParamCreator<mfxVariant> {
    template<typename ValueType>
    mfxVariant create (const std::string& name, ValueType&& value, bool is_major_flag = false) {
        cv::util::suppress_unused_warning(is_major_flag);
        return create_impl(name, value);
    }
private:
    mfxVariant create_impl(const std::string&, mfxU16 value) {
        mfxVariant ret;
        ret.Type = MFX_VARIANT_TYPE_U16;
        ret.Data.U16 = value;
        return ret;
    }
    mfxVariant create_impl(const std::string&, mfxU32 value) {
        mfxVariant ret;
        ret.Type = MFX_VARIANT_TYPE_U32;
        ret.Data.U32 = value;
        return ret;
    }
    mfxVariant create_impl(const std::string&, mfxI64 value) {
        mfxVariant ret;
        ret.Type = MFX_VARIANT_TYPE_I64;
        ret.Data.I64 = value;
        return ret;
    }
    mfxVariant create_impl(const std::string&, mfxU64 value) {
        mfxVariant ret;
        ret.Type = MFX_VARIANT_TYPE_U64;
        ret.Data.U64 = value;
        return ret;
    }
    mfxVariant create_impl(const std::string&, const std::string&) {
        GAPI_Error("Something wrong: you should not create mfxVariant "
                             "from string directly - native type is lost in this case");
    }
};

template<typename ValueType>
std::vector<ValueType> get_params_from_string(const std::string& str) {
    std::vector<ValueType> ret;
    std::string::size_type pos = 0;
    std::string::size_type endline_pos = std::string::npos;
    do
    {
        endline_pos = str.find_first_of("\r\n", pos);
        std::string line = str.substr(pos, endline_pos == std::string::npos ? std::string::npos : endline_pos - pos);
        if (line.empty()) break;

        std::string::size_type name_endline_pos = line.find(':');
        if (name_endline_pos == std::string::npos) {
            throw std::runtime_error("Cannot parse param from string: " + line +
                                     ". Name and value should be separated by \":\"" );
        }

        std::string name = line.substr(0, name_endline_pos);
        std::string value = line.substr(name_endline_pos + 2);

        ParamCreator<ValueType> creator;
        if (name == CfgParam::implementation_name()) {
            ret.push_back(creator.template create<mfxU32>(name, cstr_to_mfx_impl(value.c_str())));
        } else if (name == CfgParam::decoder_id_name()) {
            ret.push_back(creator.template create<mfxU32>(name, cstr_to_mfx_codec_id(value.c_str())));
        } else if (name == CfgParam::acceleration_mode_name()) {
            ret.push_back(creator.template create<mfxU32>(name, cstr_to_mfx_accel_mode(value.c_str())));
        } else if (name == "mfxImplDescription.ApiVersion.Version") {
            ret.push_back(creator.template create<mfxU32>(name, cstr_to_mfx_version(value.c_str())));
        } else if ((name == CfgParam::frames_pool_size_name()) || (name == CfgParam::vpp_frames_pool_size_name())) {
            ret.push_back(creator.create(name, static_cast<mfxU32>(strtoull_or_throw(value.c_str()), false)));
        } else if ((name == CfgParam::vpp_in_width_name()) || (name == CfgParam::vpp_in_height_name()) ||
                   (name == CfgParam::vpp_in_crop_w_name()) || (name == CfgParam::vpp_in_crop_h_name()) ||
                   (name == CfgParam::vpp_in_crop_x_name()) || (name == CfgParam::vpp_in_crop_y_name()) ||
                   (name == CfgParam::vpp_out_chroma_format_name()) ||
                   (name == CfgParam::vpp_out_width_name()) || (name == CfgParam::vpp_out_height_name()) ||
                   (name == CfgParam::vpp_out_crop_w_name()) || (name == CfgParam::vpp_out_crop_h_name()) ||
                   (name == CfgParam::vpp_out_crop_x_name()) || (name == CfgParam::vpp_out_crop_y_name()) ||
                   (name == CfgParam::vpp_out_pic_struct_name())) {
            ret.push_back(creator.create(name,
                                         static_cast<uint16_t>(strtoul_or_throw(value.c_str())),
                                         false));
        } else if ((name == CfgParam::vpp_out_fourcc_name()) ||
                   (name == CfgParam::vpp_out_framerate_n_name()) ||
                   (name == CfgParam::vpp_out_framerate_d_name())) {
            ret.push_back(creator.create(name,
                                         static_cast<uint32_t>(strtoul_or_throw(value.c_str())),
                                         false));
        } else {
            GAPI_LOG_DEBUG(nullptr, "Cannot parse configuration param, name: " << name <<
                                    ", value: " << value);
        }

        pos = endline_pos + 1;
    }
    while (endline_pos != std::string::npos);

    return ret;
}

template
std::vector<CfgParam> get_params_from_string(const std::string& str);
template
std::vector<mfxVariant> get_params_from_string(const std::string& str);

mfxVariant cfg_param_to_mfx_variant(const CfgParam& cfg_val) {
    const CfgParam::name_t& name = cfg_val.get_name();
    mfxVariant ret;
    cv::util::visit(cv::util::overload_lambdas(
            [&ret](uint8_t value)   { ret.Type = MFX_VARIANT_TYPE_U8;   ret.Data.U8 = value;    },
            [&ret](int8_t value)    { ret.Type = MFX_VARIANT_TYPE_I8;   ret.Data.I8 = value;    },
            [&ret](uint16_t value)  { ret.Type = MFX_VARIANT_TYPE_U16;  ret.Data.U16 = value;   },
            [&ret](int16_t value)   { ret.Type = MFX_VARIANT_TYPE_I16;  ret.Data.I16 = value;   },
            [&ret](uint32_t value)  { ret.Type = MFX_VARIANT_TYPE_U32;  ret.Data.U32 = value;   },
            [&ret](int32_t value)   { ret.Type = MFX_VARIANT_TYPE_I32;  ret.Data.I32 = value;   },
            [&ret](uint64_t value)  { ret.Type = MFX_VARIANT_TYPE_U64;  ret.Data.U64 = value;   },
            [&ret](int64_t value)   { ret.Type = MFX_VARIANT_TYPE_I64;  ret.Data.I64 = value;   },
            [&ret](float_t value)   { ret.Type = MFX_VARIANT_TYPE_F32;  ret.Data.F32 = value;   },
            [&ret](double_t value)  { ret.Type = MFX_VARIANT_TYPE_F64;  ret.Data.F64 = value;   },
            [&ret](void* value)     { ret.Type = MFX_VARIANT_TYPE_PTR;  ret.Data.Ptr = value;   },
            [&ret, &name] (const std::string& value) {
                auto parsed = get_params_from_string<mfxVariant>(name + ": " + value + "\n");
                if (parsed.empty()) {
                    throw std::logic_error("Unsupported parameter, name: " + name + ", value: " + value);
                }
                ret = *parsed.begin();
            }), cfg_val.get_value());
    return ret;
}

void extract_optional_param_by_name(const std::string &name,
                                    const std::vector<CfgParam> &in_params,
                                    cv::util::optional<size_t> &out_param) {
    auto it = std::find_if(in_params.begin(), in_params.end(), [&name] (const CfgParam& value) {
        return value.get_name() == name;
    });
    if (it != in_params.end()) {
        cv::util::visit(cv::util::overload_lambdas(
            [&out_param](uint8_t value)   { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](int8_t value)    { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](uint16_t value)  { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](int16_t value)   { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](uint32_t value)  { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](int32_t value)   { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](uint64_t value)  { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](int64_t value)   { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](float_t value)   { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](double_t value)  { out_param = cv::util::make_optional(static_cast<size_t>(value));   },
            [&out_param](void*)     { GAPI_Error("`void*` is unsupported type");  },
            [&out_param](const std::string& value) {
                out_param = cv::util::make_optional(strtoull_or_throw(value.c_str()));
            }),
            it->get_value());
    }
}

unsigned long strtoul_or_throw(const char* str) {
    char *end_ptr = nullptr;
    errno = 0;
    unsigned long ret = strtoul(str, &end_ptr, 10);
    if ((end_ptr == str) ||
        ((ret == ULONG_MAX) && errno == ERANGE)) {
            // nothing parsed from the string, handle errors or exit
        GAPI_LOG_WARNING(nullptr, "strtoul failed for: " << str);
        GAPI_Error("strtoul_or_throw");
    }
    return ret;
}

size_t strtoull_or_throw(const char* str) {
    char *end_ptr = nullptr;
    errno = 0;
    size_t ret = strtoull(str, &end_ptr, 10);
    if ((end_ptr == str) ||
        ((ret == ULLONG_MAX) && errno == ERANGE)) {
            // nothing parsed from the string, handle errors or exit
        GAPI_LOG_WARNING(nullptr, "strtoull failed for: " << str);
        GAPI_Error("strtoull_or_throw");
    }
    return ret;
}

int64_t strtoll_or_throw(const char* str) {
    char *end_ptr = nullptr;
    errno = 0;
    int64_t ret = strtoll(str, &end_ptr, 10);
    if ((end_ptr == str) ||
        ((ret == LONG_MAX || ret == LONG_MIN) && errno == ERANGE)) {
            // nothing parsed from the string, handle errors or exit
        GAPI_LOG_WARNING(nullptr, "strtoll failed for: " << str);
        GAPI_Error("strtoll_or_throw");
    }
    return ret;
}
} // namespace onevpl
} // namespace wip
} // namespace gapi
} // namespace cv
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

- **ParamCreator**: A class/struct defined in this file

### Functions and Methods

- **HAVE_ONEVPL()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `stdio.h`
- `streaming/onevpl/cfg_params_parser.hpp`
- `sstream`
- `algorithm`
- `logger.hpp`
- `streaming/onevpl/utils.hpp`

**Python Imports:**
- `string`
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

