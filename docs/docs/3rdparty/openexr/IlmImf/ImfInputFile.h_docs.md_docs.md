# Documentation for `docs/3rdparty/openexr/IlmImf/ImfInputFile.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfInputFile.h_docs.md`
- **File Name**: `ImfInputFile.h_docs.md`
- **File Size**: 13,040 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfInputFile.h_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfInputFile.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfInputFile.h`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfInputFile.h`
- **File Name**: `ImfInputFile.h`
- **File Size**: 9,470 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfInputFile.h](../../../3rdparty/openexr/IlmImf/ImfInputFile.h)

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


#ifndef INCLUDED_IMF_INPUT_FILE_H
#define INCLUDED_IMF_INPUT_FILE_H

//-----------------------------------------------------------------------------
//
//	class InputFile -- a scanline-based interface that can be used
//	to read both scanline-based and tiled OpenEXR image files.
//
//-----------------------------------------------------------------------------

#include "ImfHeader.h"
#include "ImfFrameBuffer.h"
#include "ImfTiledOutputFile.h"
#include "ImfThreading.h"
#include "ImfGenericInputFile.h"
#include "ImfNamespace.h"
#include "ImfForward.h"
#include "ImfExport.h"

#include <fstream>


OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_ENTER


class InputFile : public GenericInputFile
{
  public:

    //-----------------------------------------------------------
    // A constructor that opens the file with the specified name.
    // Destroying the InputFile object will close the file.
    //
    // numThreads determines the number of threads that will be
    // used to read the file (see ImfThreading.h).
    //-----------------------------------------------------------

    IMF_EXPORT
    InputFile (const char fileName[], int numThreads = globalThreadCount());


    //-------------------------------------------------------------
    // A constructor that attaches the new InputFile object to a
    // file that has already been opened.  Destroying the InputFile
    // object will not close the file.
    //
    // numThreads determines the number of threads that will be
    // used to read the file (see ImfThreading.h).
    //-------------------------------------------------------------

    IMF_EXPORT
    InputFile (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is, int numThreads = globalThreadCount());


    //-----------
    // Destructor
    //-----------

    IMF_EXPORT
    virtual ~InputFile ();


    //------------------------
    // Access to the file name
    //------------------------

    IMF_EXPORT
    const char *	fileName () const;


    //--------------------------
    // Access to the file header
    //--------------------------

    IMF_EXPORT
    const Header &	header () const;


    //----------------------------------
    // Access to the file format version
    //----------------------------------

    IMF_EXPORT
    int			version () const;


    //-----------------------------------------------------------
    // Set the current frame buffer -- copies the FrameBuffer
    // object into the InputFile object.
    //
    // The current frame buffer is the destination for the pixel
    // data read from the file.  The current frame buffer must be
    // set at least once before readPixels() is called.
    // The current frame buffer can be changed after each call
    // to readPixels().
    //-----------------------------------------------------------

    IMF_EXPORT
    void		setFrameBuffer (const FrameBuffer &frameBuffer);


    //-----------------------------------
    // Access to the current frame buffer
    //-----------------------------------

    IMF_EXPORT
    const FrameBuffer &	frameBuffer () const;


    //---------------------------------------------------------------
    // Check if the file is complete:
    //
    // isComplete() returns true if all pixels in the data window are
    // present in the input file, or false if any pixels are missing.
    // (Another program may still be busy writing the file, or file
    // writing may have been aborted prematurely.)
    //---------------------------------------------------------------

    IMF_EXPORT
    bool		isComplete () const;

    
    //---------------------------------------------------------------
    // Check if SSE optimization is enabled
    //
    // Call after setFrameBuffer() to query whether optimized file decoding
    // is available - decode times will be faster if returns true
    //
    // Optimization depends on:
    //   the file type (only scanline data is supported),
    //   the framebuffer channels (RGB/RGBA mono or stereo)
    //   the framebuffer channel types (all channels half-float format only)
    //   the file channels (RGB/RGBA mono or stereo)
    //   the file channel types (all channel half-float format only)
    //   whether SSE2 instruction support was detected at compile time
    //
    // Calling isOptimizationEnabled before setFrameBuffer will throw an exception
    //
    //---------------------------------------------------------------
    
    IMF_EXPORT
    bool                isOptimizationEnabled () const;
    
    
    

    //---------------------------------------------------------------
    // Read pixel data:
    //
    // readPixels(s1,s2) reads all scan lines with y coordinates
    // in the interval [min (s1, s2), max (s1, s2)] from the file,
    // and stores them in the current frame buffer.
    //
    // Both s1 and s2 must be within the interval
    // [header().dataWindow().min.y, header().dataWindow().max.y]
    //
    // The scan lines can be read from the file in random order, and
    // individual scan lines may be skipped or read multiple times.
    // For maximum efficiency, the scan lines should be read in the
    // order in which they were written to the file.
    //
    // readPixels(s) calls readPixels(s,s).
    //
    //---------------------------------------------------------------

    IMF_EXPORT
    void		readPixels (int scanLine1, int scanLine2);
    IMF_EXPORT
    void		readPixels (int scanLine);


    //----------------------------------------------
    // Read a block of raw pixel data from the file,
    // without uncompressing it (this function is
    // used to implement OutputFile::copyPixels()).
    //----------------------------------------------

    IMF_EXPORT
    void		rawPixelData (int firstScanLine,
				      const char *&pixelData,
				      int &pixelDataSize);


    //----------------------------------------------
    // Read a scanline's worth of raw pixel data 
    // from the file, without uncompressing it, and 
    // store in an external buffer, pixelData. 
    // pixelData should be pre-allocated with space 
    // for pixelDataSize chars. 
    //
    // This function can be used to separate the 
    // reading of a raw scan line from the 
    // decompression of that scan line, for
    // example to allow multiple scan lines to be
    // decompressed in parallel by an application's
    // own threads, where it is not convenient to 
    // use the threading within the library.
    //----------------------------------------------

    IMF_EXPORT
    void		rawPixelDataToBuffer (int scanLine,
					      char *pixelData,
					      int &pixelDataSize) const;   
    
 

    //--------------------------------------------------
    // Read a tile of raw pixel data from the file,
    // without uncompressing it (this function is
    // used to implement TiledOutputFile::copyPixels()).
    //--------------------------------------------------

    IMF_EXPORT
    void		rawTileData (int &dx, int &dy,
				     int &lx, int &ly,
				     const char *&pixelData,
				     int &pixelDataSize);

    struct Data;
    
  private:

    InputFile (InputPartData* part);
    InputFile (const InputFile &);			// not implemented
    InputFile & operator = (const InputFile &);		// not implemented

    void		initialize ();
    void                multiPartInitialize(InputPartData* part);
    void                compatibilityInitialize(OPENEXR_IMF_INTERNAL_NAMESPACE::IStream& is);
    TiledInputFile *	tFile ();
    
    friend void TiledOutputFile::copyPixels (InputFile &);
    
    Data *		_data;


    friend class MultiPartInputFile;
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

- **MultiPartInputFile**: A class/struct defined in this file
- **that**: A class/struct defined in this file
- **Data**: A class/struct defined in this file
- **InputFile**: A class/struct defined in this file

### Functions and Methods

- **INCLUDED_IMF_INPUT_FILE_H()**: A function/method defined in this file
- **is()**: A function/method defined in this file
- **can()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `fstream`
- `ImfThreading.h`
- `ImfTiledOutputFile.h`
- `ImfFrameBuffer.h`
- `ImfGenericInputFile.h`
- `ImfHeader.h`
- `ImfExport.h`
- `ImfNamespace.h`
- `ImfForward.h`

**Python Imports:**
- `the`
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

