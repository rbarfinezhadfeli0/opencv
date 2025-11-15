# Documentation for `3rdparty/openexr/IlmImf/ImfDwaCompressor.h`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfDwaCompressor.h`
- **File Name**: `ImfDwaCompressor.h`
- **File Size**: 7,386 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfDwaCompressor.h](../../../3rdparty/openexr/IlmImf/ImfDwaCompressor.h)

## Purpose and Role

This file is located in the `3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////
//
// Copyright (c) 2009-2014 DreamWorks Animation LLC. 
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
// *       Neither the name of DreamWorks Animation nor the names of
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

#ifndef INCLUDED_IMF_DWA_COMRESSOR_H
#define INCLUDED_IMF_DWA_COMRESSOR_H

//------------------------------------------------------------------------------
//
// class DwaCompressor -- Store lossy RGB data by quantizing DCT components.
//
//------------------------------------------------------------------------------

#include <vector>
#include <half.h>

#include "ImfInt64.h"
#include "ImfZip.h"
#include "ImfChannelList.h"
#include "ImfCompressor.h"
#include "ImfNamespace.h"

OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_ENTER

class DwaCompressor: public Compressor
{
  public:

    enum AcCompression 
    {
        STATIC_HUFFMAN,
        DEFLATE,
    };


    IMF_EXPORT
    DwaCompressor (const Header &hdr, 
                   int           maxScanLineSize,
                   int           numScanLines,    // ideally is a multiple of 8
                   AcCompression acCompression);

    IMF_EXPORT
    virtual ~DwaCompressor ();

    IMF_EXPORT
    virtual int numScanLines () const;

    IMF_EXPORT
    virtual OPENEXR_IMF_NAMESPACE::Compressor::Format format () const;

    IMF_EXPORT
    virtual int compress (const char *inPtr,
                          int         inSize,
                          int         minY,
                          const char *&outPtr);

    IMF_EXPORT
    virtual int compressTile (const char              *inPtr,
                              int                     inSize,
                              IMATH_NAMESPACE::Box2i  range,
                              const char              *&outPtr);

    IMF_EXPORT
    virtual int uncompress (const char *inPtr,
                            int         inSize,
                            int         minY,
                            const char *&outPtr);

    IMF_EXPORT
    virtual int uncompressTile (const char             *inPtr,
                                int                    inSize,
                                IMATH_NAMESPACE::Box2i range,
                                const char             *&outPtr);

    IMF_EXPORT
    static void initializeFuncs ();

  private:

    struct ChannelData;
    struct CscChannelSet;
    struct Classifier;
    
    class LossyDctDecoderBase;
    class LossyDctDecoder;
    class LossyDctDecoderCsc;

    class LossyDctEncoderBase;
    class LossyDctEncoder;
    class LossyDctEncoderCsc;

    enum CompressorScheme 
    {
        UNKNOWN = 0,
        LOSSY_DCT,
        RLE,
        
        NUM_COMPRESSOR_SCHEMES
    };

    //
    // Per-chunk compressed data sizes, one value per chunk
    //

    enum DataSizesSingle 
    {
        VERSION = 0,                  // Version number:
                                      //   0: classic
                                      //   1: adds "end of block" to the AC RLE

        UNKNOWN_UNCOMPRESSED_SIZE,    // Size of leftover data, uncompressed.
        UNKNOWN_COMPRESSED_SIZE,      // Size of leftover data, zlib compressed.

        AC_COMPRESSED_SIZE,           // AC RLE + Huffman size
        DC_COMPRESSED_SIZE,           // DC + Deflate size
        RLE_COMPRESSED_SIZE,          // RLE + Deflate data size
        RLE_UNCOMPRESSED_SIZE,        // RLE'd data size 
        RLE_RAW_SIZE,                 // Un-RLE'd data size

        AC_UNCOMPRESSED_COUNT,        // AC RLE number of elements
        DC_UNCOMPRESSED_COUNT,        // DC number of elements

        AC_COMPRESSION,               // AC compression strategy
        NUM_SIZES_SINGLE
    };

    AcCompression     _acCompression;

    int               _maxScanLineSize;
    int               _numScanLines;
    int               _min[2], _max[2];

    ChannelList                _channels;
    std::vector<ChannelData>   _channelData;
    std::vector<CscChannelSet> _cscSets;
    std::vector<Classifier>    _channelRules;

    char             *_packedAcBuffer;
    size_t            _packedAcBufferSize;
    char             *_packedDcBuffer;
    size_t            _packedDcBufferSize;
    char             *_rleBuffer;
    size_t            _rleBufferSize;
    char             *_outBuffer;
    size_t            _outBufferSize;
    char             *_planarUncBuffer[NUM_COMPRESSOR_SCHEMES];
    size_t            _planarUncBufferSize[NUM_COMPRESSOR_SCHEMES];

    Zip              *_zip;
    float             _dwaCompressionLevel;

    int compress (const char              *inPtr,
                  int                     inSize,
                  IMATH_NAMESPACE::Box2i  range,
                  const char              *&outPtr);

    int uncompress (const char             *inPtr,
                    int                    inSize,
                    IMATH_NAMESPACE::Box2i range,
                    const char             *&outPtr);

    void initializeBuffers (size_t&);
    void initializeDefaultChannelRules ();
    void initializeLegacyChannelRules ();

    void relevantChannelRules( std::vector<Classifier> &) const;

    //
    // Populate our cached version of the channel data with
    // data from the real channel list. We want to 
    // copy over attributes, determine compression schemes
    // releveant for the channel type, and find sets of
    // channels to be compressed from Y'CbCr data instead 
    // of R'G'B'.
    //

    void classifyChannels (ChannelList                  channels,
                           std::vector<ChannelData>    &chanData, 
                           std::vector<CscChannelSet>  &cscData);

    // 
    // Compute various buffer pointers for each channel
    //

    void setupChannelData (int minX, int minY, int maxX, int maxY);
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

- **ChannelData**: A class/struct defined in this file
- **LossyDctEncoderBase**: A class/struct defined in this file
- **DwaCompressor**: A class/struct defined in this file
- **LossyDctEncoder**: A class/struct defined in this file
- **LossyDctEncoderCsc**: A class/struct defined in this file
- **CscChannelSet**: A class/struct defined in this file
- **LossyDctDecoderCsc**: A class/struct defined in this file
- **LossyDctDecoder**: A class/struct defined in this file
- **Classifier**: A class/struct defined in this file
- **LossyDctDecoderBase**: A class/struct defined in this file

### Functions and Methods

- **INCLUDED_IMF_DWA_COMRESSOR_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfInt64.h`
- `ImfZip.h`
- `vector`
- `ImfCompressor.h`
- `ImfNamespace.h`
- `ImfChannelList.h`
- `half.h`

**Python Imports:**
- `Y`
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

