# Documentation for `modules/imgcodecs/src/grfmt_exr.hpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/src/grfmt_exr.hpp`
- **File Name**: `grfmt_exr.hpp`
- **File Size**: 4,253 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/imgcodecs/src/grfmt_exr.hpp](../../../modules/imgcodecs/src/grfmt_exr.hpp)

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

#ifndef _GRFMT_EXR_H_
#define _GRFMT_EXR_H_

#ifdef HAVE_OPENEXR

#if defined __GNUC__ && defined __APPLE__
#  pragma GCC diagnostic ignored "-Wshadow"
#endif

#include <ImfChromaticities.h>
#include <ImfInputFile.h>
#include <ImfChannelList.h>
#include <ImathBox.h>
#include <ImfRgbaFile.h>
#include "grfmt_base.hpp"

namespace cv
{

using namespace Imf;
using namespace Imath;

/* libpng version only */

class ExrDecoder CV_FINAL : public BaseImageDecoder
{
public:

    ExrDecoder();
    ~ExrDecoder() CV_OVERRIDE;

    int   type() const CV_OVERRIDE;
    bool  readData( Mat& img ) CV_OVERRIDE;
    bool  readHeader() CV_OVERRIDE;
    void  close();

    ImageDecoder newDecoder() const CV_OVERRIDE;

protected:
    void  UpSample( uchar *data, int xstep, int ystep, int xsample, int ysample );
    void  UpSampleX( float *data, int xstep, int xsample );
    void  UpSampleY( uchar *data, int xstep, int ystep, int ysample );
    void  ChromaToBGR( float *data, int numlines, int xstep, int ystep );
    void  ChromaToRGB( float *data, int numlines, int xstep, int ystep );
    void  RGBToGray( float *in, float *out );

    InputFile      *m_file;
    Imf::PixelType  m_type;
    Box2i           m_datawindow;
    bool            m_ischroma;
    const Channel  *m_red;
    const Channel  *m_green;
    const Channel  *m_blue;
    const Channel  *m_alpha;
    Chromaticities  m_chroma;
    int             m_bit_depth;
    bool            m_native_depth;
    bool            m_iscolor;
    bool            m_isfloat;
    bool            m_hasalpha;
    bool            m_multispectral;
    int             m_channels;

private:
    ExrDecoder(const ExrDecoder &); // copy disabled
    ExrDecoder& operator=(const ExrDecoder &); // assign disabled
};


class ExrEncoder CV_FINAL : public BaseImageEncoder
{
public:
    ExrEncoder();
    ~ExrEncoder() CV_OVERRIDE;

    bool  isFormatSupported( int depth ) const CV_OVERRIDE;
    bool  write( const Mat& img, const std::vector<int>& params ) CV_OVERRIDE;
    ImageEncoder newEncoder() const CV_OVERRIDE;
};

}

#endif

#endif/*_GRFMT_EXR_H_*/
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

- **ExrDecoder**: A class/struct defined in this file
- **ExrEncoder**: A class/struct defined in this file

### Functions and Methods

- **HAVE_OPENEXR()**: A function/method defined in this file
- **_GRFMT_EXR_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfChromaticities.h`
- `ImfRgbaFile.h`
- `grfmt_base.hpp`
- `ImfInputFile.h`
- `ImathBox.h`
- `ImfChannelList.h`

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

