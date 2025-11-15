# Documentation for `3rdparty/openexr/IlmImf/ImfTileOffsets.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfTileOffsets.cpp`
- **File Name**: `ImfTileOffsets.cpp`
- **File Size**: 14,626 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfTileOffsets.cpp](../../../3rdparty/openexr/IlmImf/ImfTileOffsets.cpp)

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
//	class TileOffsets
//
//-----------------------------------------------------------------------------

#include <ImfTileOffsets.h>
#include <ImfXdr.h>
#include <ImfIO.h>
#include "Iex.h"
#include "ImfNamespace.h"
#include <algorithm>

OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_ENTER


TileOffsets::TileOffsets (LevelMode mode,
			  int numXLevels, int numYLevels,
			  const int *numXTiles, const int *numYTiles)
:
    _mode (mode),
    _numXLevels (numXLevels),
    _numYLevels (numYLevels)
{
    switch (_mode)
    {
      case ONE_LEVEL:
      case MIPMAP_LEVELS:

        _offsets.resize (_numXLevels);

        for (unsigned int l = 0; l < _offsets.size(); ++l)
        {
            _offsets[l].resize (numYTiles[l]);

            for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	    {
                _offsets[l][dy].resize (numXTiles[l]);
            }
        }
        break;

      case RIPMAP_LEVELS:

        _offsets.resize (_numXLevels * _numYLevels);

        for (int ly = 0; ly < _numYLevels; ++ly)
        {
            for (int lx = 0; lx < _numXLevels; ++lx)
            {
                int l = ly * _numXLevels + lx;
                _offsets[l].resize (numYTiles[ly]);

                for (size_t dy = 0; dy < _offsets[l].size(); ++dy)
                {
                    _offsets[l][dy].resize (numXTiles[lx]);
                }
            }
        }
        break;

      case NUM_LEVELMODES :
          throw IEX_NAMESPACE::ArgExc("Bad initialisation of TileOffsets object");
    }
}


bool
TileOffsets::anyOffsetsAreInvalid () const
{
    for (unsigned int l = 0; l < _offsets.size(); ++l)
	for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	    for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
		if (_offsets[l][dy][dx] <= 0)
		    return true;
    
    return false;
}


void
TileOffsets::findTiles (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is, bool isMultiPartFile, bool isDeep, bool skipOnly)
{
    for (unsigned int l = 0; l < _offsets.size(); ++l)
    {
	for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	{
	    for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
	    {
		Int64 tileOffset = is.tellg();

		if (isMultiPartFile)
		{
		    int partNumber;
		    OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, partNumber);
		}

		int tileX;
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, tileX);

		int tileY;
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, tileY);

		int levelX;
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, levelX);

		int levelY;
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, levelY);

                if(isDeep)
                {
                     Int64 packed_offset_table_size;
                     Int64 packed_sample_size;
                     
                     OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, packed_offset_table_size);
                     OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, packed_sample_size);
                     
                     // next Int64 is unpacked sample size - skip that too
                     Xdr::skip <StreamIO> (is, packed_offset_table_size+packed_sample_size+8);
                    
                }else{
                    
		     int dataSize;
		     OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, dataSize);

		     Xdr::skip <StreamIO> (is, dataSize);
                }
		if (skipOnly) continue;

		if (!isValidTile(tileX, tileY, levelX, levelY))
		    return;

		operator () (tileX, tileY, levelX, levelY) = tileOffset;
	    }
	}
    }
}


void
TileOffsets::reconstructFromFile (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is,bool isMultiPart,bool isDeep)
{
    //
    // Try to reconstruct a missing tile offset table by sequentially
    // scanning through the file, and recording the offsets in the file
    // of the tiles we find.
    //

    Int64 position = is.tellg();

    try
    {
	findTiles (is,isMultiPart,isDeep,false);
    }
    catch (...)
    {
        //
        // Suppress all exceptions.  This function is called only to
	// reconstruct the tile offset table for incomplete files,
	// and exceptions are likely.
        //
    }

    is.clear();
    is.seekg (position);
}


void
TileOffsets::readFrom (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is, bool &complete,bool isMultiPartFile, bool isDeep)
{
    //
    // Read in the tile offsets from the file's tile offset table
    //

    for (unsigned int l = 0; l < _offsets.size(); ++l)
	for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	    for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::read <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (is, _offsets[l][dy][dx]);

    //
    // Check if any tile offsets are invalid.
    //
    // Invalid offsets mean that the file is probably incomplete
    // (the offset table is the last thing written to the file).
    // Either some process is still busy writing the file, or
    // writing the file was aborted.
    //
    // We should still be able to read the existing parts of the
    // file.  In order to do this, we have to make a sequential
    // scan over the scan tile to reconstruct the tile offset
    // table.
    //

    if (anyOffsetsAreInvalid())
    {
	complete = false;
	reconstructFromFile (is,isMultiPartFile,isDeep);
    }
    else
    {
	complete = true;
    }

}


void
TileOffsets::readFrom (std::vector<Int64> chunkOffsets,bool &complete)
{
    size_t totalSize = 0;
 
    for (unsigned int l = 0; l < _offsets.size(); ++l)
        for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
            totalSize += _offsets[l][dy].size();

    if (chunkOffsets.size() != totalSize)
        throw IEX_NAMESPACE::ArgExc ("Wrong offset count, not able to read from this array");



    int pos = 0;
    for (size_t l = 0; l < _offsets.size(); ++l)
        for (size_t dy = 0; dy < _offsets[l].size(); ++dy)
            for (size_t dx = 0; dx < _offsets[l][dy].size(); ++dx)
            {
                _offsets[l][dy][dx] = chunkOffsets[pos];
                pos++;
            }

    complete = !anyOffsetsAreInvalid();

}


Int64
TileOffsets::writeTo (OPENEXR_IMF_INTERNAL_NAMESPACE::OStream &os) const
{
    //
    // Write the tile offset table to the file, and
    // return the position of the start of the table
    // in the file.
    //
    
    Int64 pos = os.tellp();

    if (pos == -1)
	IEX_NAMESPACE::throwErrnoExc ("Cannot determine current file position (%T).");

    for (unsigned int l = 0; l < _offsets.size(); ++l)
	for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	    for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
		OPENEXR_IMF_INTERNAL_NAMESPACE::Xdr::write <OPENEXR_IMF_INTERNAL_NAMESPACE::StreamIO> (os, _offsets[l][dy][dx]);

    return pos;
}

namespace {
struct tilepos{
    Int64 filePos;
    int dx;
    int dy;
    int l;
    bool operator <(const tilepos & other) const
    {
        return filePos < other.filePos;
    }
};
}
//-------------------------------------
// fill array with tile coordinates in the order they appear in the file
//
// each input array must be of size (totalTiles)
// 
//
// if the tile order is not RANDOM_Y, it is more efficient to compute the
// tile ordering rather than using this function
//
//-------------------------------------
void TileOffsets::getTileOrder(int dx_table[],int dy_table[],int lx_table[],int ly_table[]) const
{
    // 
    // helper class
    // 

    // how many entries?
    size_t entries=0;
    for (unsigned int l = 0; l < _offsets.size(); ++l)
        for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
           entries+=_offsets[l][dy].size();
        
    std::vector<struct tilepos> table(entries);
    
    size_t i = 0;
    for (unsigned int l = 0; l < _offsets.size(); ++l)
        for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
            for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
            {
                table[i].filePos = _offsets[l][dy][dx];
                table[i].dx = dx;
                table[i].dy = dy;
                table[i].l = l;

                ++i;
                
            }
              
    std::sort(table.begin(),table.end());
    
    //
    // write out the values
    //
    
    // pass 1: write out dx and dy, since these are independent of level mode
    
    for(size_t i=0;i<entries;i++)
    {
        dx_table[i] = table[i].dx;
        dy_table[i] = table[i].dy;
    }

    // now write out the levels, which depend on the level mode
    
    switch (_mode)
    {
        case ONE_LEVEL:
        {
            for(size_t i=0;i<entries;i++)
            {
                lx_table[i] = 0;
                ly_table[i] = 0;               
            }
            break;            
        }
        case MIPMAP_LEVELS:
        {
            for(size_t i=0;i<entries;i++)
            {
                lx_table[i]= table[i].l;
                ly_table[i] =table[i].l;               
                
            }
            break;
        }
            
        case RIPMAP_LEVELS:
        {
            for(size_t i=0;i<entries;i++)
            {
                lx_table[i]= table[i].l % _numXLevels;
                ly_table[i] = table[i].l / _numXLevels; 
                
            }
            break;
        }
        case NUM_LEVELMODES :
            throw IEX_NAMESPACE::LogicExc("Bad level mode getting tile order");
    }
    
    
    
}


bool
TileOffsets::isEmpty () const
{
    for (unsigned int l = 0; l < _offsets.size(); ++l)
	for (unsigned int dy = 0; dy < _offsets[l].size(); ++dy)
	    for (unsigned int dx = 0; dx < _offsets[l][dy].size(); ++dx)
		if (_offsets[l][dy][dx] != 0)
		    return false;
    return true;
}


bool
TileOffsets::isValidTile (int dx, int dy, int lx, int ly) const
{
    if(lx<0 || ly < 0 || dx<0 || dy < 0) return false;
    switch (_mode)
    {
      case ONE_LEVEL:

        if (lx == 0 &&
	    ly == 0 &&
	    _offsets.size() > 0 &&
            int(_offsets[0].size()) > dy &&
            int(_offsets[0][dy].size()) > dx)
	{
            return true;
	}

        break;

      case MIPMAP_LEVELS:

        if (lx < _numXLevels &&
	    ly < _numYLevels &&
            int(_offsets.size()) > lx &&
            int(_offsets[lx].size()) > dy &&
            int(_offsets[lx][dy].size()) > dx)
	{
            return true;
	}

        break;

      case RIPMAP_LEVELS:

        if (lx < _numXLevels &&
	    ly < _numYLevels &&
	    (_offsets.size() > (size_t) lx+  ly *  (size_t) _numXLevels) &&
            int(_offsets[lx + ly * _numXLevels].size()) > dy &&
            int(_offsets[lx + ly * _numXLevels][dy].size()) > dx)
	{
            return true;
	}

        break;

      default:

        return false;
    }
    
    return false;
}


Int64 &
TileOffsets::operator () (int dx, int dy, int lx, int ly)
{
    //
    // Looks up the value of the tile with tile coordinate (dx, dy)
    // and level number (lx, ly) in the _offsets array, and returns
    // the cooresponding offset.
    //

    switch (_mode)
    {
      case ONE_LEVEL:

        return _offsets[0][dy][dx];
        break;

      case MIPMAP_LEVELS:

        return _offsets[lx][dy][dx];
        break;

      case RIPMAP_LEVELS:

        return _offsets[lx + ly * _numXLevels][dy][dx];
        break;

      default:

        throw IEX_NAMESPACE::ArgExc ("Unknown LevelMode format.");
    }
}


Int64 &
TileOffsets::operator () (int dx, int dy, int l)
{
    return operator () (dx, dy, l, l);
}


const Int64 &
TileOffsets::operator () (int dx, int dy, int lx, int ly) const
{
    //
    // Looks up the value of the tile with tile coordinate (dx, dy)
    // and level number (lx, ly) in the _offsets array, and returns
    // the cooresponding offset.
    //

    switch (_mode)
    {
      case ONE_LEVEL:

        return _offsets[0][dy][dx];
        break;

      case MIPMAP_LEVELS:

        return _offsets[lx][dy][dx];
        break;

      case RIPMAP_LEVELS:

        return _offsets[lx + ly * _numXLevels][dy][dx];
        break;

      default:

        throw IEX_NAMESPACE::ArgExc ("Unknown LevelMode format.");
    }
}


const Int64 &
TileOffsets::operator () (int dx, int dy, int l) const
{
    return operator () (dx, dy, l, l);
}

const std::vector<std::vector<std::vector <Int64> > >&
TileOffsets::getOffsets() const
{
    return _offsets;
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

- **a**: A class/struct defined in this file
- **tilepos**: A class/struct defined in this file
- **the**: A class/struct defined in this file
- **TileOffsets**: A class/struct defined in this file

### Functions and Methods

- **is()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfIO.h`
- `ImfTileOffsets.h`
- `Iex.h`
- `algorithm`
- `ImfXdr.h`
- `ImfNamespace.h`

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

