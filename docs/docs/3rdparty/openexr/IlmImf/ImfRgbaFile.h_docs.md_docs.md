# Documentation for `docs/3rdparty/openexr/IlmImf/ImfRgbaFile.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfRgbaFile.h_docs.md`
- **File Name**: `ImfRgbaFile.h_docs.md`
- **File Size**: 15,823 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfRgbaFile.h_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfRgbaFile.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfRgbaFile.h`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfRgbaFile.h`
- **File Name**: `ImfRgbaFile.h`
- **File Size**: 12,222 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfRgbaFile.h](../../../3rdparty/openexr/IlmImf/ImfRgbaFile.h)

## Purpose and Role

This file is located in the `3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////
//
// Copyright (c) 2004, Industrial Light & Magic, a division of Lucas
// Digital Ltd. LLC
// 
// All rights reserved.
// 
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions are
// met:
// *       Redistributions of source code must retain the above copyright
// notice, this list of conditions and the following disclaimer.
// *       Redistributions in binary form must reproduce the above
// copyright notice, this list of conditions and the following disclaimer
// in the documentation and/or other materials provided with the
// distribution.
// *       Neither the name of Industrial Light & Magic nor the names of
// its contributors may be used to endorse or promote products derived
// from this software without specific prior written permission. 
// 
// THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
// "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
// LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
// A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
// OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
// SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
// LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
// DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
// THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
// (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
// OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
//
///////////////////////////////////////////////////////////////////////////



#ifndef INCLUDED_IMF_RGBA_FILE_H
#define INCLUDED_IMF_RGBA_FILE_H


//-----------------------------------------------------------------------------
//
//	Simplified RGBA image I/O
//
//	class RgbaOutputFile
//	class RgbaInputFile
//
//-----------------------------------------------------------------------------

#include "ImfHeader.h"
#include "ImfFrameBuffer.h"
#include "ImfRgba.h"
#include "ImathVec.h"
#include "ImathBox.h"
#include "half.h"
#include "ImfThreading.h"
#include <string>
#include "ImfNamespace.h"
#include "ImfForward.h"

OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_ENTER


//
// RGBA output file.
//

class RgbaOutputFile
{
  public:

    //---------------------------------------------------
    // Constructor -- header is constructed by the caller
    //---------------------------------------------------

    IMF_EXPORT
    RgbaOutputFile (const char name[],
		    const Header &header,
		    RgbaChannels rgbaChannels = WRITE_RGBA,
                    int numThreads = globalThreadCount());


    //----------------------------------------------------
    // Constructor -- header is constructed by the caller,
    // file is opened by the caller, destructor will not
    // automatically close the file.
    //----------------------------------------------------

    IMF_EXPORT
    RgbaOutputFile (OPENEXR_IMF_INTERNAL_NAMESPACE::OStream &os,
		    const Header &header,
		    RgbaChannels rgbaChannels = WRITE_RGBA,
                    int numThreads = globalThreadCount());


    //----------------------------------------------------------------
    // Constructor -- header data are explicitly specified as function
    // call arguments (empty dataWindow means "same as displayWindow")
    //----------------------------------------------------------------

    IMF_EXPORT
    RgbaOutputFile (const char name[],
		    const IMATH_NAMESPACE::Box2i &displayWindow,
		    const IMATH_NAMESPACE::Box2i &dataWindow = IMATH_NAMESPACE::Box2i(),
		    RgbaChannels rgbaChannels = WRITE_RGBA,
		    float pixelAspectRatio = 1,
		    const IMATH_NAMESPACE::V2f screenWindowCenter = IMATH_NAMESPACE::V2f (0, 0),
		    float screenWindowWidth = 1,
		    LineOrder lineOrder = INCREASING_Y,
		    Compression compression = PIZ_COMPRESSION,
                    int numThreads = globalThreadCount());


    //-----------------------------------------------
    // Constructor -- like the previous one, but both
    // the display window and the data window are
    // Box2i (V2i (0, 0), V2i (width - 1, height -1))
    //-----------------------------------------------

    IMF_EXPORT
    RgbaOutputFile (const char name[],
		    int width,
		    int height,
		    RgbaChannels rgbaChannels = WRITE_RGBA,
		    float pixelAspectRatio = 1,
		    const IMATH_NAMESPACE::V2f screenWindowCenter = IMATH_NAMESPACE::V2f (0, 0),
		    float screenWindowWidth = 1,
		    LineOrder lineOrder = INCREASING_Y,
		    Compression compression = PIZ_COMPRESSION,
                    int numThreads = globalThreadCount());


    //-----------
    // Destructor
    //-----------

    IMF_EXPORT
    virtual ~RgbaOutputFile ();


    //------------------------------------------------
    // Define a frame buffer as the pixel data source:
    // Pixel (x, y) is at address
    //
    //  base + x * xStride + y * yStride
    //
    //------------------------------------------------

    IMF_EXPORT
    void			setFrameBuffer (const Rgba *base,
						size_t xStride,
						size_t yStride);


    //---------------------------------------------
    // Write pixel data (see class Imf::OutputFile)
    //---------------------------------------------

    IMF_EXPORT
    void			writePixels (int numScanLines = 1);
    IMF_EXPORT
    int				currentScanLine () const;


    //--------------------------
    // Access to the file header
    //--------------------------

    IMF_EXPORT
    const Header &		header () const;
    IMF_EXPORT
    const FrameBuffer &		frameBuffer () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::Box2i &	displayWindow () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::Box2i &	dataWindow () const;
    IMF_EXPORT
    float			pixelAspectRatio () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::V2f		screenWindowCenter () const;
    IMF_EXPORT
    float			screenWindowWidth () const;
    IMF_EXPORT
    LineOrder			lineOrder () const;
    IMF_EXPORT
    Compression			compression () const;
    IMF_EXPORT
    RgbaChannels		channels () const;


    // --------------------------------------------------------------------
    // Update the preview image (see Imf::OutputFile::updatePreviewImage())
    // --------------------------------------------------------------------

    IMF_EXPORT
    void			updatePreviewImage (const PreviewRgba[]);


    //-----------------------------------------------------------------------
    // Rounding control for luminance/chroma images:
    //
    // If the output file contains luminance and chroma channels (WRITE_YC
    // or WRITE_YCA), then the the significands of the luminance and
    // chroma values are rounded to roundY and roundC bits respectively (see
    // function half::round()).  Rounding improves compression with minimal
    // image degradation, usually much less than the degradation caused by
    // chroma subsampling.  By default, roundY is 7, and roundC is 5.
    //
    // If the output file contains RGB channels or a luminance channel,
    // without chroma, then no rounding is performed.
    //-----------------------------------------------------------------------

    IMF_EXPORT
    void			setYCRounding (unsigned int roundY,
					       unsigned int roundC);


    //----------------------------------------------------
    // Break a scan line -- for testing and debugging only
    // (see Imf::OutputFile::updatePreviewImage()
    //
    // Warning: Calling this function usually results in a
    // broken image file.  The file or parts of it may not
    // be readable, or the file may contain bad data.
    //
    //----------------------------------------------------

    IMF_EXPORT
    void			breakScanLine  (int y,
						int offset,
						int length,
						char c);
  private:

    RgbaOutputFile (const RgbaOutputFile &);		  // not implemented
    RgbaOutputFile & operator = (const RgbaOutputFile &); // not implemented

    class ToYca;

    OutputFile *		_outputFile;
    ToYca *			_toYca;
};


//
// RGBA input file
//

class RgbaInputFile
{
  public:

    //-------------------------------------------------------
    // Constructor -- opens the file with the specified name,
    // destructor will automatically close the file.
    //-------------------------------------------------------

    IMF_EXPORT
    RgbaInputFile (const char name[], int numThreads = globalThreadCount());


    //-----------------------------------------------------------
    // Constructor -- attaches the new RgbaInputFile object to a
    // file that has already been opened by the caller.
    // Destroying the RgbaInputFile object will not automatically
    // close the file.
    //-----------------------------------------------------------

    IMF_EXPORT
    RgbaInputFile (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is, int numThreads = globalThreadCount());


    //--------------------------------------------------------------
    // Constructors -- the same as the previous two, but the names
    // of the red, green, blue, alpha, luminance and chroma channels
    // are expected to be layerName.R, layerName.G, etc.
    //--------------------------------------------------------------

    IMF_EXPORT
    RgbaInputFile (const char name[],
		   const std::string &layerName,
		   int numThreads = globalThreadCount());

    IMF_EXPORT
    RgbaInputFile (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is,
		   const std::string &layerName,
		   int numThreads = globalThreadCount());


    //-----------
    // Destructor
    //-----------

    IMF_EXPORT
    virtual ~RgbaInputFile ();


    //-----------------------------------------------------
    // Define a frame buffer as the pixel data destination:
    // Pixel (x, y) is at address
    //
    //  base + x * xStride + y * yStride
    //
    //-----------------------------------------------------

    IMF_EXPORT
    void			setFrameBuffer (Rgba *base,
						size_t xStride,
						size_t yStride);


    //----------------------------------------------------------------
    // Switch to a different layer -- subsequent calls to readPixels()
    // will read channels layerName.R, layerName.G, etc.
    // After each call to setLayerName(), setFrameBuffer() must be
    // called at least once before the next call to readPixels().
    //----------------------------------------------------------------

    IMF_EXPORT
    void			setLayerName (const std::string &layerName);


    //-------------------------------------------
    // Read pixel data (see class Imf::InputFile)
    //-------------------------------------------

    IMF_EXPORT
    void			readPixels (int scanLine1, int scanLine2);
    IMF_EXPORT
    void			readPixels (int scanLine);


    //--------------------------
    // Access to the file header
    //--------------------------

    IMF_EXPORT
    const Header &		header () const;
    IMF_EXPORT
    const FrameBuffer &		frameBuffer () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::Box2i &	displayWindow () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::Box2i &	dataWindow () const;
    IMF_EXPORT
    float			pixelAspectRatio () const;
    IMF_EXPORT
    const IMATH_NAMESPACE::V2f		screenWindowCenter () const;
    IMF_EXPORT
    float			screenWindowWidth () const;
    IMF_EXPORT
    LineOrder			lineOrder () const;
    IMF_EXPORT
    Compression			compression () const;
    IMF_EXPORT
    RgbaChannels		channels () const;
    IMF_EXPORT
    const char *                fileName () const;
    IMF_EXPORT
    bool			isComplete () const;


    //----------------------------------
    // Access to the file format version
    //----------------------------------

    IMF_EXPORT
    int				version () const;

  private:

    RgbaInputFile (const RgbaInputFile &);		  // not implemented
    RgbaInputFile & operator = (const RgbaInputFile &);   // not implemented

    class FromYca;

    InputFile *			_inputFile;
    FromYca *			_fromYca;
    std::string			_channelNamePrefix;
};


OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_EXIT





#endif
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

- **RgbaInputFile**: A class/struct defined in this file
- **FromYca**: A class/struct defined in this file
- **Imf**: A class/struct defined in this file
- **ToYca**: A class/struct defined in this file
- **RgbaOutputFile**: A class/struct defined in this file

### Functions and Methods

- **INCLUDED_IMF_RGBA_FILE_H()**: A function/method defined in this file
- **usually()**: A function/method defined in this file
- **half()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfNamespace.h`
- `ImfThreading.h`
- `ImfRgba.h`
- `ImfFrameBuffer.h`
- `ImfHeader.h`
- `ImathBox.h`
- `string`
- `ImathVec.h`
- `half.h`
- `ImfForward.h`

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

