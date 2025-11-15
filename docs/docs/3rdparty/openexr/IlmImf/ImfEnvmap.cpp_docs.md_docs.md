# Documentation for `docs/3rdparty/openexr/IlmImf/ImfEnvmap.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfEnvmap.cpp_docs.md`
- **File Name**: `ImfEnvmap.cpp_docs.md`
- **File Size**: 10,243 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfEnvmap.cpp_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfEnvmap.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfEnvmap.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfEnvmap.cpp`
- **File Name**: `ImfEnvmap.cpp`
- **File Size**: 7,229 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfEnvmap.cpp](../../../3rdparty/openexr/IlmImf/ImfEnvmap.cpp)

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
//	Environment maps
//
//-----------------------------------------------------------------------------

#include "ImfEnvmap.h"
#include "ImathFun.h"
#include "ImfNamespace.h"

#include <algorithm>
#include <math.h>

using namespace std;
using namespace IMATH_NAMESPACE;


OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_ENTER


namespace LatLongMap {

V2f	
latLong (const V3f &dir)
{
    float r = sqrt (dir.z * dir.z + dir.x * dir.x);

    float latitude = (r < abs (dir.y))?
			 acos (r / dir.length()) * sign (dir.y):
			 asin (dir.y / dir.length());

    float longitude = (dir.z == 0 && dir.x == 0)? 0: atan2 (dir.x, dir.z);

    return V2f (latitude, longitude);
}


V2f
latLong (const Box2i &dataWindow, const V2f &pixelPosition)
{
    float latitude, longitude;

    if (dataWindow.max.y > dataWindow.min.y)
    {
        latitude = -1 * float(M_PI) *
            ((pixelPosition.y  - dataWindow.min.y) /
            (dataWindow.max.y - dataWindow.min.y) - 0.5f);
    }
    else
    {
	latitude = 0;
    }

    if (dataWindow.max.x > dataWindow.min.x)
    {
	longitude = -2 * float(M_PI) *
		   ((pixelPosition.x  - dataWindow.min.x) /
		    (dataWindow.max.x - dataWindow.min.x) - 0.5f);
    }
    else
    {
	longitude = 0;
    }

    return V2f (latitude, longitude);
}


V2f
pixelPosition (const Box2i &dataWindow, const V2f &latLong)
{
    float x = latLong.y / (-2 * float(M_PI)) + 0.5f;
    float y = latLong.x / (-1 * float(M_PI)) + 0.5f;

    return V2f (x * (dataWindow.max.x - dataWindow.min.x) + dataWindow.min.x,
		y * (dataWindow.max.y - dataWindow.min.y) + dataWindow.min.y);
}


V2f
pixelPosition (const Box2i &dataWindow, const V3f &direction)
{
    return pixelPosition (dataWindow, latLong (direction));
}


V3f
direction (const Box2i &dataWindow, const V2f &pixelPosition)
{
    V2f ll = latLong (dataWindow, pixelPosition);

    return V3f (sin (ll.y) * cos (ll.x),
		sin (ll.x),
		cos (ll.y) * cos (ll.x));
}

} // namespace LatLongMap


namespace CubeMap {

int
sizeOfFace (const Box2i &dataWindow)
{
    return min ((dataWindow.max.x - dataWindow.min.x + 1),
		(dataWindow.max.y - dataWindow.min.y + 1) / 6);
}


Box2i
dataWindowForFace (CubeMapFace face, const Box2i &dataWindow)
{
    int sof = sizeOfFace (dataWindow);
    Box2i dwf;

    dwf.min.x = 0;
    dwf.min.y = int (face) * sof;

    dwf.max.x = dwf.min.x + sof - 1;
    dwf.max.y = dwf.min.y + sof - 1;

    return dwf;
}


V2f
pixelPosition (CubeMapFace face, const Box2i &dataWindow, V2f positionInFace)
{
    Box2i dwf = dataWindowForFace (face, dataWindow);
    V2f pos (0, 0);

    switch (face)
    {
      case CUBEFACE_POS_X:

	pos.x = dwf.min.x + positionInFace.y;
	pos.y = dwf.max.y - positionInFace.x;
	break;

      case CUBEFACE_NEG_X:

	pos.x = dwf.max.x - positionInFace.y;
	pos.y = dwf.max.y - positionInFace.x;
	break;

      case CUBEFACE_POS_Y:

	pos.x = dwf.min.x + positionInFace.x;
	pos.y = dwf.max.y - positionInFace.y;
	break;

      case CUBEFACE_NEG_Y:

	pos.x = dwf.min.x + positionInFace.x;
	pos.y = dwf.min.y + positionInFace.y;
	break;

      case CUBEFACE_POS_Z:

	pos.x = dwf.max.x - positionInFace.x;
	pos.y = dwf.max.y - positionInFace.y;
	break;

      case CUBEFACE_NEG_Z:

	pos.x = dwf.min.x + positionInFace.x;
	pos.y = dwf.max.y - positionInFace.y;
	break;
    }

    return pos;
}


void
faceAndPixelPosition (const V3f &direction,
		      const Box2i &dataWindow,
		      CubeMapFace &face,
		      V2f &pif)
{
    int sof = sizeOfFace (dataWindow);
    float absx = abs (direction.x);
    float absy = abs (direction.y);
    float absz = abs (direction.z);

    if (absx >= absy && absx >= absz)
    {
	if (absx == 0)
	{
	    //
	    // Special case - direction is (0, 0, 0)
	    //

	    face = CUBEFACE_POS_X;
	    pif = V2f (0, 0);
	    return;
	}

	pif.x = (direction.y / absx + 1) / 2 * (sof - 1);
	pif.y = (direction.z / absx + 1) / 2 * (sof - 1);

	if (direction.x > 0)
	    face = CUBEFACE_POS_X;
	else
	    face = CUBEFACE_NEG_X;
    }
    else if (absy >= absz)
    {
	pif.x = (direction.x / absy + 1) / 2 * (sof - 1);
	pif.y = (direction.z / absy + 1) / 2 * (sof - 1);

	if (direction.y > 0)
	    face = CUBEFACE_POS_Y;
	else
	    face = CUBEFACE_NEG_Y;
    }
    else
    {
	pif.x = (direction.x / absz + 1) / 2 * (sof - 1);
	pif.y = (direction.y / absz + 1) / 2 * (sof - 1);

	if (direction.z > 0)
	    face = CUBEFACE_POS_Z;
	else
	    face = CUBEFACE_NEG_Z;
    }
}


V3f
direction (CubeMapFace face, const Box2i &dataWindow, const V2f &positionInFace)
{
    int sof = sizeOfFace (dataWindow);

    V2f pos;
    
    if (sof > 1)
    {
	pos = V2f (positionInFace.x / (sof - 1) * 2 - 1,
		   positionInFace.y / (sof - 1) * 2 - 1);
    }
    else
    {
	pos = V2f (0, 0);
    }

    V3f dir (1, 0, 0);

    switch (face)
    {
      case CUBEFACE_POS_X:

	dir.x = 1;
	dir.y = pos.x;
	dir.z = pos.y;
	break;

      case CUBEFACE_NEG_X:

	dir.x = -1;
	dir.y = pos.x;
	dir.z = pos.y;
	break;

      case CUBEFACE_POS_Y:

	dir.x = pos.x;
	dir.y = 1;
	dir.z = pos.y;
	break;

      case CUBEFACE_NEG_Y:

	dir.x = pos.x;
	dir.y = -1;
	dir.z = pos.y;
	break;

      case CUBEFACE_POS_Z:

	dir.x = pos.x;
	dir.y = pos.y;
	dir.z = 1;
	break;

      case CUBEFACE_NEG_Z:

	dir.x = pos.x;
	dir.y = pos.y;
	dir.z = -1;
	break;
    }

    return dir;
}

} // namespace CubeMap


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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImathFun.h`
- `math.h`
- `ImfEnvmap.h`
- `algorithm`
- `ImfNamespace.h`

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

