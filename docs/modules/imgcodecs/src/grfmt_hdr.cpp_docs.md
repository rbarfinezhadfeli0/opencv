# Documentation for `modules/imgcodecs/src/grfmt_hdr.cpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/src/grfmt_hdr.cpp`
- **File Name**: `grfmt_hdr.cpp`
- **File Size**: 6,675 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgcodecs/src/grfmt_hdr.cpp](../../../modules/imgcodecs/src/grfmt_hdr.cpp)

## Purpose and Role

This file is located in the `modules/imgcodecs/src` directory and serves as part of the OpenCV library infrastructure.

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
#include "grfmt_hdr.hpp"
#include "rgbe.hpp"
#include "opencv2/core/utils/logger.hpp"

#ifdef HAVE_IMGCODEC_HDR

namespace cv
{

HdrDecoder::HdrDecoder()
{
    m_signature = "#?RGBE";
    m_signature_alt = "#?RADIANCE";
    file = NULL;
    m_type = CV_32FC3;
}

HdrDecoder::~HdrDecoder()
{
    if(file) {
        fclose(file);
    }
}

size_t HdrDecoder::signatureLength() const
{
    return m_signature.size() > m_signature_alt.size() ?
           m_signature.size() : m_signature_alt.size();
}

bool  HdrDecoder::readHeader()
{
    file = fopen(m_filename.c_str(), "rb");
    if(!file) {
        return false;
    }
    RGBE_ReadHeader(file, &m_width, &m_height, NULL);
    if(m_width <= 0 || m_height <= 0) {
        fclose(file);
        file = NULL;
        return false;
    }
    return true;
}

bool HdrDecoder::readData(Mat& _img)
{
    Mat img(m_height, m_width, CV_32FC3);
    if(!file) {
        if(!readHeader()) {
            return false;
        }
    }
    RGBE_ReadPixels_RLE(file, const_cast<float*>(img.ptr<float>()), img.cols, img.rows);
    fclose(file); file = NULL;

    // NOTE: 'img' has type CV32FC3
    switch (_img.depth())
    {
        case CV_8U: img.convertTo(img, _img.depth(), 255); break;
        case CV_32F: break;
        default: CV_Error(Error::StsError, "Wrong expected image depth, allowed: CV_8U and CV_32F");
    }
    switch (_img.channels())
    {
        case 1: cvtColor(img, _img, COLOR_BGR2GRAY); break;
        case 3:
        // TODO, try to modify RGBE_ReadPixels_RLE to load rgb data directly.
        if (m_use_rgb)
            cv::cvtColor(img, _img, cv::COLOR_BGR2RGB);
        else
            img.copyTo(_img);
        break;
        default: CV_Error(Error::StsError, "Wrong expected image channels, allowed: 1 and 3");
    }
    return true;
}

bool HdrDecoder::checkSignature( const String& signature ) const
{
    if (signature.size() >= m_signature.size() &&
        0 == memcmp(signature.c_str(), m_signature.c_str(), m_signature.size())
    )
        return true;
    if (signature.size() >= m_signature_alt.size() &&
        0 == memcmp(signature.c_str(), m_signature_alt.c_str(), m_signature_alt.size())
    )
        return true;
    return false;
}

ImageDecoder HdrDecoder::newDecoder() const
{
    return makePtr<HdrDecoder>();
}

HdrEncoder::HdrEncoder()
{
    m_description = "Radiance HDR (*.hdr;*.pic)";
    m_supported_encode_key = {IMWRITE_HDR_COMPRESSION};
}

HdrEncoder::~HdrEncoder()
{
}

bool HdrEncoder::write( const Mat& input_img, const std::vector<int>& params )
{
    Mat img;
    CV_Assert(input_img.channels() == 3 || input_img.channels() == 1);
    if(input_img.channels() == 1) {
         std::vector<Mat> splitted(3, input_img);
         merge(splitted, img);
    } else {
        input_img.copyTo(img);
    }
    if(img.depth() != CV_32F) {
        img.convertTo(img, CV_32FC3, 1/255.0f);
    }

    int compression = IMWRITE_HDR_COMPRESSION_RLE;
    for (size_t i = 0; i + 1 < params.size(); i += 2)
    {
        const int value = params[i+1];
        switch (params[i])
        {
        case IMWRITE_HDR_COMPRESSION:
            switch(value)
            {
                case IMWRITE_HDR_COMPRESSION_NONE:
                case IMWRITE_HDR_COMPRESSION_RLE:
                    compression = value;
                    break;
                default:
                    compression = IMWRITE_HDR_COMPRESSION_RLE;
                    CV_LOG_WARNING(nullptr, cv::format("The value(%d) for IMWRITE_HDR_COMPRESSION must be one of ImwriteHDRCompressionFlags. It is fallbacked to IMWRITE_HDR_COMPRESSION_RLE", value));
                    break;
            }
            break;
        default:
            break;
        }
    }
    CV_Check(compression, compression == IMWRITE_HDR_COMPRESSION_NONE || compression == IMWRITE_HDR_COMPRESSION_RLE, "");

    FILE *fout = fopen(m_filename.c_str(), "wb");
    if(!fout) {
        return false;
    }

    RGBE_WriteHeader(fout, img.cols, img.rows, NULL);
    if (compression == IMWRITE_HDR_COMPRESSION_RLE) {
        RGBE_WritePixels_RLE(fout, const_cast<float*>(img.ptr<float>()), img.cols, img.rows);
    } else {
        RGBE_WritePixels(fout, const_cast<float*>(img.ptr<float>()), img.cols * img.rows);
    }

    fclose(fout);
    return true;
}

ImageEncoder HdrEncoder::newEncoder() const
{
    return makePtr<HdrEncoder>();
}

bool HdrEncoder::isFormatSupported( int depth ) const {
    return depth != CV_64F;
}

}

#endif // HAVE_IMGCODEC_HDR
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

- **HAVE_IMGCODEC_HDR()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `rgbe.hpp`
- `precomp.hpp`
- `opencv2/core/utils/logger.hpp`
- `grfmt_hdr.hpp`

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

