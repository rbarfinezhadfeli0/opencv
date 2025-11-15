# Documentation for `docs/3rdparty/openexr/IlmImf/ImfPizCompressor.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfPizCompressor.cpp_docs.md`
- **File Name**: `ImfPizCompressor.cpp_docs.md`
- **File Size**: 18,300 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfPizCompressor.cpp_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfPizCompressor.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfPizCompressor.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfPizCompressor.cpp`
- **File Name**: `ImfPizCompressor.cpp`
- **File Size**: 14,901 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfPizCompressor.cpp](../../../3rdparty/openexr/IlmImf/ImfPizCompressor.cpp)

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


//-----------------------------------------------------------------------------
//
//	class PizCompressor
//
//-----------------------------------------------------------------------------

#include "ImfPizCompressor.h"
#include "ImfHeader.h"
#include "ImfChannelList.h"
#include "ImfHuf.h"
#include "ImfWav.h"
#include "ImfMisc.h"
#include "ImfCheckedArithmetic.h"
#include <ImathFun.h>
#include <ImathBox.h>
#include <Iex.h>
#include "ImfIO.h"
#include "ImfXdr.h"
#include "ImfAutoArray.h"
#include <string.h>
#include <assert.h>
#include "ImfNamespace.h"

OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_ENTER

using IMATH_NAMESPACE::divp;
using IMATH_NAMESPACE::modp;
using IMATH_NAMESPACE::Box2i;
using IMATH_NAMESPACE::V2i;
using IEX_NAMESPACE::InputExc;

namespace {

//
// Functions to compress the range of values in the pixel data
//

const int USHORT_RANGE = (1 << 16);
const int BITMAP_SIZE  = (USHORT_RANGE >> 3);

void
bitmapFromData (const unsigned short data[/*nData*/],
		int nData,
		unsigned char bitmap[BITMAP_SIZE],
		unsigned short &minNonZero,
		unsigned short &maxNonZero)
{
    for (int i = 0; i < BITMAP_SIZE; ++i)
	bitmap[i] = 0;

    for (int i = 0; i < nData; ++i)
	bitmap[data[i] >> 3] |= (1 << (data[i] & 7));

    bitmap[0] &= ~1;			// zero is not explicitly stored in
					// the bitmap; we assume that the
					// data always contain zeroes
    minNonZero = BITMAP_SIZE - 1;
    maxNonZero = 0;

    for (int i = 0; i < BITMAP_SIZE; ++i)
    {
	if (bitmap[i])
	{
	    if (minNonZero > i)
		minNonZero = i;
	    if (maxNonZero < i)
		maxNonZero = i;
	}
    }
}


unsigned short
forwardLutFromBitmap (const unsigned char bitmap[BITMAP_SIZE],
		      unsigned short lut[USHORT_RANGE])
{
    int k = 0;

    for (int i = 0; i < USHORT_RANGE; ++i)
    {
	if ((i == 0) || (bitmap[i >> 3] & (1 << (i & 7))))
	    lut[i] = k++;
	else
	    lut[i] = 0;
    }

    return k - 1;	// maximum value stored in lut[],
}			// i.e. number of ones in bitmap minus 1


unsigned short
reverseLutFromBitmap (const unsigned char bitmap[BITMAP_SIZE],
		      unsigned short lut[USHORT_RANGE])
{
    int k = 0;

    for (int i = 0; i < USHORT_RANGE; ++i)
    {
	if ((i == 0) || (bitmap[i >> 3] & (1 << (i & 7))))
	    lut[k++] = i;
    }

    int n = k - 1;

    while (k < USHORT_RANGE)
	lut[k++] = 0;

    return n;		// maximum k where lut[k] is non-zero,
}			// i.e. number of ones in bitmap minus 1


void
applyLut (const unsigned short lut[USHORT_RANGE],
	  unsigned short data[/*nData*/],
	  int nData)
{
    for (int i = 0; i < nData; ++i)
	data[i] = lut[data[i]];
}


} // namespace


struct PizCompressor::ChannelData
{
    unsigned short *	start;
    unsigned short *	end;
    int			nx;
    int			ny;
    int			ys;
    int			size;
};


PizCompressor::PizCompressor
    (const Header &hdr,
     size_t maxScanLineSize,
     size_t numScanLines)
:
    Compressor (hdr),
    _maxScanLineSize (maxScanLineSize),
    _format (XDR),
    _numScanLines (numScanLines),
    _tmpBuffer (0),
    _outBuffer (0),
    _numChans (0),
    _channels (hdr.channels()),
    _channelData (0)
{
    size_t tmpBufferSize =
                uiMult (maxScanLineSize, numScanLines) / 2;

    size_t outBufferSize =
                uiAdd (uiMult (maxScanLineSize, numScanLines),
                       size_t (65536 + 8192));

    _tmpBuffer = new unsigned short
            [checkArraySize (tmpBufferSize, sizeof (unsigned short))];

    _outBuffer = new char [outBufferSize];

    const ChannelList &channels = header().channels();
    bool onlyHalfChannels = true;

    for (ChannelList::ConstIterator c = channels.begin();
	 c != channels.end();
	 ++c)
    {
	_numChans++;

	assert (pixelTypeSize (c.channel().type) % pixelTypeSize (HALF) == 0);

	if (c.channel().type != HALF)
	    onlyHalfChannels = false;
    }

    _channelData = new ChannelData[_numChans];

    const Box2i &dataWindow = hdr.dataWindow();

    _minX = dataWindow.min.x;
    _maxX = dataWindow.max.x;
    _maxY = dataWindow.max.y;

    //
    // We can support uncompressed data in the machine's native format
    // if all image channels are of type HALF, and if the Xdr and the
    // native represenations of a half have the same size.
    //

    if (onlyHalfChannels && (sizeof (half) == pixelTypeSize (HALF)))
	_format = NATIVE;
}


PizCompressor::~PizCompressor ()
{
    delete [] _tmpBuffer;
    delete [] _outBuffer;
    delete [] _channelData;
}


int
PizCompressor::numScanLines () const
{
    return _numScanLines;
}


Compressor::Format
PizCompressor::format () const
{
    return _format;
}


int
PizCompressor::compress (const char *inPtr,
			 int inSize,
			 int minY,
			 const char *&outPtr)
{
    return compress (inPtr,
		     inSize,
		     Box2i (V2i (_minX, minY),
			    V2i (_maxX, minY + numScanLines() - 1)),
		     outPtr);
}


int
PizCompressor::compressTile (const char *inPtr,
			     int inSize,
			     IMATH_NAMESPACE::Box2i range,
			     const char *&outPtr)
{
    return compress (inPtr, inSize, range, outPtr);
}


int
PizCompressor::uncompress (const char *inPtr,
			   int inSize,
			   int minY,
			   const char *&outPtr)
{
    return uncompress (inPtr,
		       inSize,
		       Box2i (V2i (_minX, minY),
			      V2i (_maxX, minY + numScanLines() - 1)),
		       outPtr);
}


int
PizCompressor::uncompressTile (const char *inPtr,
			       int inSize,
			       IMATH_NAMESPACE::Box2i range,
			       const char *&outPtr)
{
    return uncompress (inPtr, inSize, range, outPtr);
}


int
PizCompressor::compress (const char *inPtr,
			 int inSize,
			 IMATH_NAMESPACE::Box2i range,
			 const char *&outPtr)
{
    //
    // This is the compress function which is used by both the tiled and
    // scanline compression routines.
    //

    //
    // Special case �- empty input buffer
    //

    if (inSize == 0)
    {
	outPtr = _outBuffer;
	return 0;
    }

    //
    // Rearrange the pixel data so that the wavelet
    // and Huffman encoders can process them easily.
    //
    // The wavelet and Huffman encoders both handle only
    // 16-bit data, so 32-bit data must be split into smaller
    // pieces.  We treat each 32-bit channel (UINT, FLOAT) as
    // two interleaved 16-bit channels.
    //

    int minX = range.min.x;
    int maxX = range.max.x;
    int minY = range.min.y;
    int maxY = range.max.y;
    
    if (maxY > _maxY)
        maxY = _maxY;
    
    if (maxX > _maxX)
        maxX = _maxX;

    unsigned short *tmpBufferEnd = _tmpBuffer;
    int i = 0;

    for (ChannelList::ConstIterator c = _channels.begin();
	 c != _channels.end();
	 ++c, ++i)
    {
	ChannelData &cd = _channelData[i];

	cd.start = tmpBufferEnd;
	cd.end = cd.start;

	cd.nx = numSamples (c.channel().xSampling, minX, maxX);
	cd.ny = numSamples (c.channel().ySampling, minY, maxY);
	cd.ys = c.channel().ySampling;

	cd.size = pixelTypeSize (c.channel().type) / pixelTypeSize (HALF);

	tmpBufferEnd += cd.nx * cd.ny * cd.size;
    }

    if (_format == XDR)
    {
	//
	// Machine-independent (Xdr) data format
	//

	for (int y = minY; y <= maxY; ++y)
	{
	    for (int i = 0; i < _numChans; ++i)
	    {
		ChannelData &cd = _channelData[i];

		if (modp (y, cd.ys) != 0)
		    continue;

		for (int x = cd.nx * cd.size; x > 0; --x)
		{
		    Xdr::read <CharPtrIO> (inPtr, *cd.end);
		    ++cd.end;
		}
	    }
	}
    }
    else
    {
	//
	// Native, machine-dependent data format
	//

	for (int y = minY; y <= maxY; ++y)
	{
	    for (int i = 0; i < _numChans; ++i)
	    {
		ChannelData &cd = _channelData[i];

		if (modp (y, cd.ys) != 0)
		    continue;

		int n = cd.nx * cd.size;
		memcpy (cd.end, inPtr, n * sizeof (unsigned short));
		inPtr  += n * sizeof (unsigned short);
		cd.end += n;
	    }
	}
    }

    #if defined (DEBUG)

	for (int i = 1; i < _numChans; ++i)
	    assert (_channelData[i-1].end == _channelData[i].start);

	assert (_channelData[_numChans-1].end == tmpBufferEnd);

    #endif

    //
    // Compress the range of the pixel data
    //

    AutoArray <unsigned char, BITMAP_SIZE> bitmap;
    unsigned short minNonZero;
    unsigned short maxNonZero;

    bitmapFromData (_tmpBuffer,
		    tmpBufferEnd - _tmpBuffer,
		    bitmap,
		    minNonZero, maxNonZero);

    AutoArray <unsigned short, USHORT_RANGE> lut;
    unsigned short maxValue = forwardLutFromBitmap (bitmap, lut);
    applyLut (lut, _tmpBuffer, tmpBufferEnd - _tmpBuffer);

    //
    // Store range compression info in _outBuffer
    //

    char *buf = _outBuffer;

    Xdr::write <CharPtrIO> (buf, minNonZero);
    Xdr::write <CharPtrIO> (buf, maxNonZero);

    if (minNonZero <= maxNonZero)
    {
	Xdr::write <CharPtrIO> (buf, (char *) &bitmap[0] + minNonZero,
				maxNonZero - minNonZero + 1);
    }

    //
    // Apply wavelet encoding
    //

    for (int i = 0; i < _numChans; ++i)
    {
	ChannelData &cd = _channelData[i];

	for (int j = 0; j < cd.size; ++j)
	{
	    wav2Encode (cd.start + j,
			cd.nx, cd.size,
			cd.ny, cd.nx * cd.size,
			maxValue);
	}
    }

    //
    // Apply Huffman encoding; append the result to _outBuffer
    //

    char *lengthPtr = buf;
    Xdr::write <CharPtrIO> (buf, int(0));

    int length = hufCompress (_tmpBuffer, tmpBufferEnd - _tmpBuffer, buf);
    Xdr::write <CharPtrIO> (lengthPtr, length);

    outPtr = _outBuffer;
    return buf - _outBuffer + length;
}


int
PizCompressor::uncompress (const char *inPtr,
			   int inSize,
			   IMATH_NAMESPACE::Box2i range,
			   const char *&outPtr)
{
    //
    // This is the cunompress function which is used by both the tiled and
    // scanline decompression routines.
    //
    
    //
    // Special case - empty input buffer
    //

    if (inSize == 0)
    {
	outPtr = _outBuffer;
	return 0;
    }

    //
    // Determine the layout of the compressed pixel data
    //

    int minX = range.min.x;
    int maxX = range.max.x;
    int minY = range.min.y;
    int maxY = range.max.y;
    
    if (maxY > _maxY)
        maxY = _maxY;
    
    if (maxX > _maxX)
        maxX = _maxX;

    unsigned short *tmpBufferEnd = _tmpBuffer;
    int i = 0;

    for (ChannelList::ConstIterator c = _channels.begin();
	 c != _channels.end();
	 ++c, ++i)
    {
	ChannelData &cd = _channelData[i];

	cd.start = tmpBufferEnd;
	cd.end = cd.start;

	cd.nx = numSamples (c.channel().xSampling, minX, maxX);
	cd.ny = numSamples (c.channel().ySampling, minY, maxY);
	cd.ys = c.channel().ySampling;

	cd.size = pixelTypeSize (c.channel().type) / pixelTypeSize (HALF);

	tmpBufferEnd += cd.nx * cd.ny * cd.size;
    }

    //
    // Read range compression data
    //

    unsigned short minNonZero;
    unsigned short maxNonZero;

    AutoArray <unsigned char, BITMAP_SIZE> bitmap;
    memset (bitmap, 0, sizeof (unsigned char) * BITMAP_SIZE);

    Xdr::read <CharPtrIO> (inPtr, minNonZero);
    Xdr::read <CharPtrIO> (inPtr, maxNonZero);

    if (maxNonZero >= BITMAP_SIZE)
    {
	throw InputExc ("Error in header for PIZ-compressed data "
			"(invalid bitmap size).");
    }

    if (minNonZero <= maxNonZero)
    {
	Xdr::read <CharPtrIO> (inPtr, (char *) &bitmap[0] + minNonZero,
			       maxNonZero - minNonZero + 1);
    }

    AutoArray <unsigned short, USHORT_RANGE> lut;
    unsigned short maxValue = reverseLutFromBitmap (bitmap, lut);

    //
    // Huffman decoding
    //

    int length;
    Xdr::read <CharPtrIO> (inPtr, length);

    if (length > inSize)
    {
	throw InputExc ("Error in header for PIZ-compressed data "
			"(invalid array length).");
    }

    hufUncompress (inPtr, length, _tmpBuffer, tmpBufferEnd - _tmpBuffer);

    //
    // Wavelet decoding
    //

    for (int i = 0; i < _numChans; ++i)
    {
	ChannelData &cd = _channelData[i];

	for (int j = 0; j < cd.size; ++j)
	{
	    wav2Decode (cd.start + j,
			cd.nx, cd.size,
			cd.ny, cd.nx * cd.size,
			maxValue);
	}
    }

    //
    // Expand the pixel data to their original range
    //

    applyLut (lut, _tmpBuffer, tmpBufferEnd - _tmpBuffer);
    
    //
    // Rearrange the pixel data into the format expected by the caller.
    //

    char *outEnd = _outBuffer;

    if (_format == XDR)
    {
	//
	// Machine-independent (Xdr) data format
	//

	for (int y = minY; y <= maxY; ++y)
	{
	    for (int i = 0; i < _numChans; ++i)
	    {
		ChannelData &cd = _channelData[i];

		if (modp (y, cd.ys) != 0)
		    continue;

		for (int x = cd.nx * cd.size; x > 0; --x)
		{
		    Xdr::write <CharPtrIO> (outEnd, *cd.end);
		    ++cd.end;
		}
	    }
	}
    }
    else
    {
	//
	// Native, machine-dependent data format
	//

	for (int y = minY; y <= maxY; ++y)
	{
	    for (int i = 0; i < _numChans; ++i)
	    {
		ChannelData &cd = _channelData[i];

		if (modp (y, cd.ys) != 0)
		    continue;

		int n = cd.nx * cd.size;
		memcpy (outEnd, cd.end, n * sizeof (unsigned short));
		outEnd += n * sizeof (unsigned short);
		cd.end += n;
	    }
	}
    }

    #if defined (DEBUG)

	for (int i = 1; i < _numChans; ++i)
	    assert (_channelData[i-1].end == _channelData[i].start);

	assert (_channelData[_numChans-1].end == tmpBufferEnd);

    #endif

    outPtr = _outBuffer;
    return outEnd - _outBuffer;
}


OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_EXIT
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

- **PizCompressor**: A class/struct defined in this file

### Functions and Methods

- **which()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfAutoArray.h`
- `ImfHuf.h`
- `ImfIO.h`
- `ImfMisc.h`
- `ImathFun.h`
- `ImfHeader.h`
- `ImfPizCompressor.h`
- `ImathBox.h`
- `ImfCheckedArithmetic.h`
- `Iex.h`
- `string.h`
- `ImfWav.h`
- `ImfChannelList.h`
- `ImfXdr.h`
- `ImfNamespace.h`
- `assert.h`

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

