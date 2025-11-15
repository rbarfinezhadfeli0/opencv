# Documentation for `docs/3rdparty/openexr/IlmImf/ImfFrameBuffer.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfFrameBuffer.h_docs.md`
- **File Name**: `ImfFrameBuffer.h_docs.md`
- **File Size**: 14,023 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfFrameBuffer.h_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfFrameBuffer.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfFrameBuffer.h`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfFrameBuffer.h`
- **File Name**: `ImfFrameBuffer.h`
- **File Size**: 10,581 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfFrameBuffer.h](../../../3rdparty/openexr/IlmImf/ImfFrameBuffer.h)

## Purpose and Role

This file is located in the `3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////
//
// Copyright (c) 2002, Industrial Light & Magic, a division of Lucas
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



#ifndef INCLUDED_IMF_FRAME_BUFFER_H
#define INCLUDED_IMF_FRAME_BUFFER_H

//-----------------------------------------------------------------------------
//
//      class Slice
//      class FrameBuffer
//
//-----------------------------------------------------------------------------

#include "ImfName.h"
#include "ImfPixelType.h"
#include "ImfExport.h"
#include "ImfNamespace.h"

#include <map>
#include <string>


OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_ENTER


//-------------------------------------------------------
// Description of a single slice of the frame buffer:
//
// Note -- terminology: as part of a file, a component of
// an image (e.g. red, green, blue, depth etc.) is called
// a "channel".  As part of a frame buffer, an image
// component is called a "slice".
//-------------------------------------------------------

struct Slice
{
    //------------------------------
    // Data type; see ImfPixelType.h
    //------------------------------

    PixelType           type;


    //---------------------------------------------------------------------
    // Memory layout:  The address of pixel (x, y) is
    //
    //  base + (xp / xSampling) * xStride + (yp / ySampling) * yStride
    //
    // where xp and yp are computed as follows:
    //
    //  * If we are reading or writing a scanline-based file:
    //
    //      xp = x
    //      yp = y
    //
    //  * If we are reading a tile whose upper left coorner is at (xt, yt):
    //
    //      if xTileCoords is true then xp = x - xt, else xp = x
    //      if yTileCoords is true then yp = y - yt, else yp = y
    //
    //---------------------------------------------------------------------

    char *              base;
    size_t              xStride;
    size_t              yStride;


    //--------------------------------------------
    // Subsampling: pixel (x, y) is present in the
    // slice only if 
    //
    //  x % xSampling == 0 && y % ySampling == 0
    //
    //--------------------------------------------

    int                 xSampling;
    int                 ySampling;


    //----------------------------------------------------------
    // Default value, used to fill the slice when a file without
    // a channel that corresponds to this slice is read.
    //----------------------------------------------------------

    double              fillValue;
    

    //-------------------------------------------------------
    // For tiled files, the xTileCoords and yTileCoords flags
    // determine whether pixel addressing is performed using
    // absolute coordinates or coordinates relative to a
    // tile's upper left corner.  (See the comment on base,
    // xStride and yStride, above.)
    //
    // For scanline-based files these flags have no effect;
    // pixel addressing is always done using absolute
    // coordinates.
    //-------------------------------------------------------

    bool                xTileCoords;
    bool                yTileCoords;


    //------------
    // Constructor
    //------------

    IMF_EXPORT
    Slice (PixelType type = HALF,
           char * base = 0,
           size_t xStride = 0,
           size_t yStride = 0,
           int xSampling = 1,
           int ySampling = 1,
           double fillValue = 0.0,
           bool xTileCoords = false,
           bool yTileCoords = false);
};


class FrameBuffer
{
  public:

    //------------
    // Add a slice
    //------------

    IMF_EXPORT
    void                        insert (const char name[],
                                        const Slice &slice);

    IMF_EXPORT
    void                        insert (const std::string &name,
                                        const Slice &slice);

    //----------------------------------------------------------------
    // Access to existing slices:
    //
    // [n]              Returns a reference to the slice with name n.
    //                  If no slice with name n exists, an IEX_NAMESPACE::ArgExc
    //                  is thrown.
    //
    // findSlice(n)     Returns a pointer to the slice with name n,
    //                  or 0 if no slice with name n exists.
    //
    //----------------------------------------------------------------

    IMF_EXPORT
    Slice &                     operator [] (const char name[]);
    IMF_EXPORT
    const Slice &               operator [] (const char name[]) const;

    IMF_EXPORT
    Slice &                     operator [] (const std::string &name);
    IMF_EXPORT
    const Slice &               operator [] (const std::string &name) const;

    IMF_EXPORT
    Slice *                     findSlice (const char name[]);
    IMF_EXPORT
    const Slice *               findSlice (const char name[]) const;

    IMF_EXPORT
    Slice *                     findSlice (const std::string &name);
    IMF_EXPORT
    const Slice *               findSlice (const std::string &name) const;


    //-----------------------------------------
    // Iterator-style access to existing slices
    //-----------------------------------------

    typedef std::map <Name, Slice> SliceMap;

    class Iterator;
    class ConstIterator;

    IMF_EXPORT
    Iterator                    begin ();
    IMF_EXPORT
    ConstIterator               begin () const;

    IMF_EXPORT
    Iterator                    end ();
    IMF_EXPORT
    ConstIterator               end () const;

    IMF_EXPORT
    Iterator                    find (const char name[]);
    IMF_EXPORT
    ConstIterator               find (const char name[]) const;

    IMF_EXPORT
    Iterator                    find (const std::string &name);
    IMF_EXPORT
    ConstIterator               find (const std::string &name) const;

  private:

    SliceMap                    _map;
};


//----------
// Iterators
//----------

class FrameBuffer::Iterator
{
  public:

    IMF_EXPORT
    Iterator ();
    IMF_EXPORT
    Iterator (const FrameBuffer::SliceMap::iterator &i);

    IMF_EXPORT
    Iterator &                  operator ++ ();
    IMF_EXPORT
    Iterator                    operator ++ (int);

    IMF_EXPORT
    const char *                name () const;
    IMF_EXPORT
    Slice &                     slice () const;

  private:

    friend class FrameBuffer::ConstIterator;

    FrameBuffer::SliceMap::iterator _i;
};


class FrameBuffer::ConstIterator
{
  public:

    IMF_EXPORT
    ConstIterator ();
    IMF_EXPORT
    ConstIterator (const FrameBuffer::SliceMap::const_iterator &i);
    IMF_EXPORT
    ConstIterator (const FrameBuffer::Iterator &other);

    IMF_EXPORT
    ConstIterator &             operator ++ ();
    IMF_EXPORT
    ConstIterator               operator ++ (int);

    IMF_EXPORT
    const char *                name () const;
    IMF_EXPORT
    const Slice &               slice () const;

  private:

    friend bool operator == (const ConstIterator &, const ConstIterator &);
    friend bool operator != (const ConstIterator &, const ConstIterator &);

    FrameBuffer::SliceMap::const_iterator _i;
};


//-----------------
// Inline Functions
//-----------------

inline
FrameBuffer::Iterator::Iterator (): _i()
{
    // empty
}


inline
FrameBuffer::Iterator::Iterator (const FrameBuffer::SliceMap::iterator &i):
    _i (i)
{
    // empty
}


inline FrameBuffer::Iterator &
FrameBuffer::Iterator::operator ++ ()
{
    ++_i;
    return *this;
}


inline FrameBuffer::Iterator
FrameBuffer::Iterator::operator ++ (int)
{
    Iterator tmp = *this;
    ++_i;
    return tmp;
}


inline const char *
FrameBuffer::Iterator::name () const
{
    return *_i->first;
}


inline Slice &
FrameBuffer::Iterator::slice () const
{
    return _i->second;
}


inline
FrameBuffer::ConstIterator::ConstIterator (): _i()
{
    // empty
}

inline
FrameBuffer::ConstIterator::ConstIterator
    (const FrameBuffer::SliceMap::const_iterator &i): _i (i)
{
    // empty
}


inline
FrameBuffer::ConstIterator::ConstIterator (const FrameBuffer::Iterator &other):
    _i (other._i)
{
    // empty
}

inline FrameBuffer::ConstIterator &
FrameBuffer::ConstIterator::operator ++ ()
{
    ++_i;
    return *this;
}


inline FrameBuffer::ConstIterator
FrameBuffer::ConstIterator::operator ++ (int)
{
    ConstIterator tmp = *this;
    ++_i;
    return tmp;
}


inline const char *
FrameBuffer::ConstIterator::name () const
{
    return *_i->first;
}

inline const Slice &
FrameBuffer::ConstIterator::slice () const
{
    return _i->second;
}


inline bool
operator == (const FrameBuffer::ConstIterator &x,
             const FrameBuffer::ConstIterator &y)
{
    return x._i == y._i;
}


inline bool
operator != (const FrameBuffer::ConstIterator &x,
             const FrameBuffer::ConstIterator &y)
{
    return !(x == y);
}


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

- **FrameBuffer**: A class/struct defined in this file
- **ConstIterator**: A class/struct defined in this file
- **Slice**: A class/struct defined in this file
- **Iterator**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **INCLUDED_IMF_FRAME_BUFFER_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfPixelType.h`
- `ImfName.h`
- `string`
- `ImfExport.h`
- `ImfNamespace.h`
- `map`

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

