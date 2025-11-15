# Documentation for `3rdparty/openexr/IlmImf/ImfTimeCode.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfTimeCode.cpp`
- **File Name**: `ImfTimeCode.cpp`
- **File Size**: 8,773 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfTimeCode.cpp](../../../3rdparty/openexr/IlmImf/ImfTimeCode.cpp)

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
//	class TimeCode
// 	
//-----------------------------------------------------------------------------

#include <ImfTimeCode.h>
#include "Iex.h"
#include "ImfNamespace.h"

OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_ENTER

   
TimeCode::TimeCode ()
{
    _time = 0;
    _user = 0;
}


TimeCode::TimeCode
    (int hours,
     int minutes,
     int seconds,
     int frame,
     bool dropFrame,
     bool colorFrame,
     bool fieldPhase,
     bool bgf0,
     bool bgf1,
     bool bgf2,
     int binaryGroup1,
     int binaryGroup2,
     int binaryGroup3,
     int binaryGroup4,
     int binaryGroup5,
     int binaryGroup6,
     int binaryGroup7,
     int binaryGroup8)
{
    setHours (hours);
    setMinutes (minutes);
    setSeconds (seconds);
    setFrame (frame);
    setDropFrame (dropFrame);
    setColorFrame (colorFrame);
    setFieldPhase (fieldPhase);
    setBgf0 (bgf0);
    setBgf1 (bgf1);
    setBgf2 (bgf2);
    setBinaryGroup (1, binaryGroup1);
    setBinaryGroup (2, binaryGroup2);
    setBinaryGroup (3, binaryGroup3);
    setBinaryGroup (4, binaryGroup4);
    setBinaryGroup (5, binaryGroup5);
    setBinaryGroup (6, binaryGroup6);
    setBinaryGroup (7, binaryGroup7);
    setBinaryGroup (8, binaryGroup8);
}


TimeCode::TimeCode
    (unsigned int timeAndFlags,
     unsigned int userData,
     Packing packing)
{
    setTimeAndFlags (timeAndFlags, packing);
    setUserData (userData);
}


TimeCode::TimeCode (const TimeCode &other)
{
    _time = other._time;
    _user = other._user;
}


TimeCode &
TimeCode::operator = (const TimeCode &other)
{
    _time = other._time;
    _user = other._user;
    return *this;
}

    
bool
TimeCode::operator == (const TimeCode & c) const
{
    return (_time == c._time && _user == c._user);
}


bool
TimeCode::operator != (const TimeCode & c) const
{
    return (_time != c._time || _user != c._user);
}
    
    

namespace {

unsigned int
bitField (unsigned int value, int minBit, int maxBit)
{
    int shift = minBit;
    unsigned int mask = (~(~0U << (maxBit - minBit + 1)) << minBit);
    return (value & mask) >> shift;
}


void
setBitField (unsigned int &value, int minBit, int maxBit, unsigned int field)
{
    int shift = minBit;
    unsigned int mask = (~(~0U << (maxBit - minBit + 1)) << minBit);
    value = ((value & ~mask) | ((field << shift) & mask));
}


int
bcdToBinary (unsigned int bcd)
{
    return int ((bcd & 0x0f) + 10 * ((bcd >> 4) & 0x0f));
}


unsigned int
binaryToBcd (int binary)
{
    int units = binary % 10;
    int tens = (binary / 10) % 10;
    return (unsigned int) (units | (tens << 4));
}


} // namespace


int
TimeCode::hours () const
{
    return bcdToBinary (bitField (_time, 24, 29));
}


void
TimeCode::setHours (int value)
{
    if (value < 0 || value > 23)
	throw IEX_NAMESPACE::ArgExc ("Cannot set hours field in time code. "
			   "New value is out of range.");

    setBitField (_time, 24, 29, binaryToBcd (value));
}


int
TimeCode::minutes () const
{
    return bcdToBinary (bitField (_time, 16, 22));
}


void
TimeCode::setMinutes (int value)
{
    if (value < 0 || value > 59)
	throw IEX_NAMESPACE::ArgExc ("Cannot set minutes field in time code. "
			   "New value is out of range.");

    setBitField (_time, 16, 22, binaryToBcd (value));
}


int
TimeCode::seconds () const
{
    return bcdToBinary (bitField (_time, 8, 14));
}


void
TimeCode::setSeconds (int value)
{
    if (value < 0 || value > 59)
	throw IEX_NAMESPACE::ArgExc ("Cannot set seconds field in time code. "
			   "New value is out of range.");

    setBitField (_time, 8, 14, binaryToBcd (value));
}


int
TimeCode::frame () const
{
    return bcdToBinary (bitField (_time, 0, 5));
}


void
TimeCode::setFrame (int value)
{
    if (value < 0 || value > 59)
	throw IEX_NAMESPACE::ArgExc ("Cannot set frame field in time code. "
			   "New value is out of range.");

    setBitField (_time, 0, 5, binaryToBcd (value));
}


bool
TimeCode::dropFrame () const
{
    return !!bitField (_time, 6, 6);
}


void
TimeCode::setDropFrame (bool value)
{
    setBitField (_time, 6, 6, (unsigned int) !!value);
}


bool
TimeCode::colorFrame () const
{
    return !!bitField (_time, 7, 7);
}


void
TimeCode::setColorFrame (bool value)
{
    setBitField (_time, 7, 7, (unsigned int) !!value);
}


bool
TimeCode::fieldPhase () const
{
    return !!bitField (_time, 15, 15);
}


void
TimeCode::setFieldPhase (bool value)
{
    setBitField (_time, 15, 15, (unsigned int) !!value);
}


bool
TimeCode::bgf0 () const
{
    return !!bitField (_time, 23, 23);
}


void
TimeCode::setBgf0 (bool value)
{
    setBitField (_time, 23, 23, (unsigned int) !!value);
}


bool
TimeCode::bgf1 () const
{
    return!!bitField (_time, 30, 30);
}


void
TimeCode::setBgf1 (bool value)
{
    setBitField (_time, 30, 30, (unsigned int) !!value);
}


bool
TimeCode::bgf2 () const
{
    return !!bitField (_time, 31, 31);
}


void
TimeCode::setBgf2 (bool value)
{
    setBitField (_time, 31, 31, (unsigned int) !!value);
}


int
TimeCode::binaryGroup (int group) const
{
    if (group < 1 || group > 8)
	throw IEX_NAMESPACE::ArgExc ("Cannot extract binary group from time code "
		           "user data.  Group number is out of range.");

    int minBit = 4 * (group - 1);
    int maxBit = minBit + 3;
    return int (bitField (_user, minBit, maxBit));
}


void
TimeCode::setBinaryGroup (int group, int value)
{
    if (group < 1 || group > 8)
	throw IEX_NAMESPACE::ArgExc ("Cannot extract binary group from time code "
		           "user data.  Group number is out of range.");

    int minBit = 4 * (group - 1);
    int maxBit = minBit + 3;
    setBitField (_user, minBit, maxBit, (unsigned int) value);
}


unsigned int
TimeCode::timeAndFlags (Packing packing) const
{
    if (packing == TV50_PACKING)
    {
	unsigned int t = _time;

	t &= ~((1 << 6) | (1 << 15) | (1 << 23) | (1 << 30) | (1 << 31));

	t |= ((unsigned int) bgf0() << 15);
	t |= ((unsigned int) bgf2() << 23);
	t |= ((unsigned int) bgf1() << 30);
	t |= ((unsigned int) fieldPhase() << 31);

	return t;
    }
    if (packing == FILM24_PACKING)
    {
	return _time & ~((1 << 6) | (1 << 7));
    }
    else // packing == TV60_PACKING
    {
	return _time;
    }
}


void
TimeCode::setTimeAndFlags (unsigned int value, Packing packing)
{
    if (packing == TV50_PACKING)
    {
	_time = value &
		 ~((1 << 6) | (1 << 15) | (1 << 23) | (1 << 30) | (1 << 31));

	if (value & (1 << 15))
	    setBgf0 (true);

	if (value & (1 << 23))
	    setBgf2 (true);

	if (value & (1 << 30))
	    setBgf1 (true);

	if (value & (1 << 31))
	    setFieldPhase (true);
    }
    else if (packing == FILM24_PACKING)
    {
	_time = value & ~((1 << 6) | (1 << 7));
    }
    else // packing == TV60_PACKING
    {
	_time = value;
    }
}


unsigned int
TimeCode::userData () const
{
    return _user;
}


void
TimeCode::setUserData (unsigned int value)
{
    _user = value;
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

- **TimeCode**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfTimeCode.h`
- `Iex.h`
- `ImfNamespace.h`

**Python Imports:**
- `time`
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

