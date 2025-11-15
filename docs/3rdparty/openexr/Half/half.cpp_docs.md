# Documentation for `3rdparty/openexr/Half/half.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/Half/half.cpp`
- **File Name**: `half.cpp`
- **File Size**: 7,550 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/Half/half.cpp](../../../3rdparty/openexr/Half/half.cpp)

## Purpose and Role

This file is located in the `3rdparty/openexr/Half` directory and serves as part of the OpenCV library infrastructure.

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

// Primary authors:
//     Florian Kainz <kainz@ilm.com>
//     Rod Bogart <rgb@ilm.com>


//---------------------------------------------------------------------------
//
//	class half --
//	implementation of non-inline members
//
//---------------------------------------------------------------------------

#include <assert.h>
#include "half.h"

using namespace std;

//-------------------------------------------------------------
// Lookup tables for half-to-float and float-to-half conversion
//-------------------------------------------------------------

HALF_EXPORT const half::uif half::_toFloat[1 << 16] =
    #include "toFloat.h"
HALF_EXPORT const unsigned short half::_eLut[1 << 9] =
    #include "eLut.h"

//-----------------------------------------------
// Overflow handler for float-to-half conversion;
// generates a hardware floating-point overflow,
// which may be trapped by the operating system.
//-----------------------------------------------

HALF_EXPORT float
half::overflow ()
{
    volatile float f = 1e10;

    for (int i = 0; i < 10; i++)	
	f *= f;				// this will overflow before
					// the forloop terminates
    return f;
}


//-----------------------------------------------------
// Float-to-half conversion -- general case, including
// zeroes, denormalized numbers and exponent overflows.
//-----------------------------------------------------

HALF_EXPORT short
half::convert (int i)
{
    //
    // Our floating point number, f, is represented by the bit
    // pattern in integer i.  Disassemble that bit pattern into
    // the sign, s, the exponent, e, and the significand, m.
    // Shift s into the position where it will go in in the
    // resulting half number.
    // Adjust e, accounting for the different exponent bias
    // of float and half (127 versus 15).
    //

    int s =  (i >> 16) & 0x00008000;
    int e = ((i >> 23) & 0x000000ff) - (127 - 15);
    int m =   i        & 0x007fffff;

    //
    // Now reassemble s, e and m into a half:
    //

    if (e <= 0)
    {
	if (e < -10)
	{
	    //
	    // E is less than -10.  The absolute value of f is
	    // less than HALF_MIN (f may be a small normalized
	    // float, a denormalized float or a zero).
	    //
	    // We convert f to a half zero with the same sign as f.
	    //

	    return s;
	}

	//
	// E is between -10 and 0.  F is a normalized float
	// whose magnitude is less than HALF_NRM_MIN.
	//
	// We convert f to a denormalized half.
	//

	//
	// Add an explicit leading 1 to the significand.
	// 

	m = m | 0x00800000;

	//
	// Round to m to the nearest (10+e)-bit value (with e between
	// -10 and 0); in case of a tie, round to the nearest even value.
	//
	// Rounding may cause the significand to overflow and make
	// our number normalized.  Because of the way a half's bits
	// are laid out, we don't have to treat this case separately;
	// the code below will handle it correctly.
	// 

	int t = 14 - e;
	int a = (1 << (t - 1)) - 1;
	int b = (m >> t) & 1;

	m = (m + a + b) >> t;

	//
	// Assemble the half from s, e (zero) and m.
	//

	return s | m;
    }
    else if (e == 0xff - (127 - 15))
    {
	if (m == 0)
	{
	    //
	    // F is an infinity; convert f to a half
	    // infinity with the same sign as f.
	    //

	    return s | 0x7c00;
	}
	else
	{
	    //
	    // F is a NAN; we produce a half NAN that preserves
	    // the sign bit and the 10 leftmost bits of the
	    // significand of f, with one exception: If the 10
	    // leftmost bits are all zero, the NAN would turn 
	    // into an infinity, so we have to set at least one
	    // bit in the significand.
	    //

	    m >>= 13;
	    return s | 0x7c00 | m | (m == 0);
	}
    }
    else
    {
	//
	// E is greater than zero.  F is a normalized float.
	// We try to convert f to a normalized half.
	//

	//
	// Round to m to the nearest 10-bit value.  In case of
	// a tie, round to the nearest even value.
	//

	m = m + 0x00000fff + ((m >> 13) & 1);

	if (m & 0x00800000)
	{
	    m =  0;		// overflow in significand,
	    e += 1;		// adjust exponent
	}

	//
	// Handle exponent overflow
	//

	if (e > 30)
	{
	    overflow ();	// Cause a hardware floating point overflow;
	    return s | 0x7c00;	// if this returns, the half becomes an
	}   			// infinity with the same sign as f.

	//
	// Assemble the half from s, e and m.
	//

	return s | (e << 10) | (m >> 13);
    }
}


//---------------------
// Stream I/O operators
//---------------------

HALF_EXPORT ostream &
operator << (ostream &os, half h)
{
    os << float (h);
    return os;
}


HALF_EXPORT istream &
operator >> (istream &is, half &h)
{
    float f;
    is >> f;
    h = half (f);
    return is;
}


//---------------------------------------
// Functions to print the bit-layout of
// floats and halfs, mostly for debugging
//---------------------------------------

HALF_EXPORT void
printBits (ostream &os, half h)
{
    unsigned short b = h.bits();

    for (int i = 15; i >= 0; i--)
    {
	os << (((b >> i) & 1)? '1': '0');

	if (i == 15 || i == 10)
	    os << ' ';
    }
}


HALF_EXPORT void
printBits (ostream &os, float f)
{
    half::uif x;
    x.f = f;

    for (int i = 31; i >= 0; i--)
    {
	os << (((x.i >> i) & 1)? '1': '0');

	if (i == 31 || i == 23)
	    os << ' ';
    }
}


HALF_EXPORT void
printBits (char c[19], half h)
{
    unsigned short b = h.bits();

    for (int i = 15, j = 0; i >= 0; i--, j++)
    {
	c[j] = (((b >> i) & 1)? '1': '0');

	if (i == 15 || i == 10)
	    c[++j] = ' ';
    }
    
    c[18] = 0;
}


HALF_EXPORT void
printBits (char c[35], float f)
{
    half::uif x;
    x.f = f;

    for (int i = 31, j = 0; i >= 0; i--, j++)
    {
	c[j] = (((x.i >> i) & 1)? '1': '0');

	if (i == 31 || i == 23)
	    c[++j] = ' ';
    }

    c[34] = 0;
}
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

- **half**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `toFloat.h`
- `half.h`
- `assert.h`
- `eLut.h`

**Python Imports:**
- `s`
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

