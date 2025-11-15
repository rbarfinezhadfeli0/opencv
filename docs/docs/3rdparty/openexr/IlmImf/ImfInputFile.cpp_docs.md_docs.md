# Documentation for `docs/3rdparty/openexr/IlmImf/ImfInputFile.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmImf/ImfInputFile.cpp_docs.md`
- **File Name**: `ImfInputFile.cpp_docs.md`
- **File Size**: 27,300 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmImf/ImfInputFile.cpp_docs.md](../../../../docs/3rdparty/openexr/IlmImf/ImfInputFile.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmImf/ImfInputFile.cpp`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfInputFile.cpp`
- **File Name**: `ImfInputFile.cpp`
- **File Size**: 23,786 bytes
- **File Type**: .cpp
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfInputFile.cpp](../../../3rdparty/openexr/IlmImf/ImfInputFile.cpp)

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
//	class InputFile
//
//-----------------------------------------------------------------------------

#include "ImfInputFile.h"
#include "ImfScanLineInputFile.h"
#include "ImfTiledInputFile.h"
#include "ImfChannelList.h"
#include "ImfMisc.h"
#include "ImfStdIO.h"
#include "ImfVersion.h"
#include "ImfPartType.h"
#include "ImfInputPartData.h"
#include "ImfMultiPartInputFile.h"

#include <ImfCompositeDeepScanLine.h>
#include <ImfDeepScanLineInputFile.h>

#include "ImathFun.h"
#include "IlmThreadMutex.h"
#include "Iex.h"
#include "half.h"

#include <fstream>
#include <algorithm>

#include "ImfNamespace.h"

OPENEXR_IMF_INTERNAL_NAMESPACE_SOURCE_ENTER


using IMATH_NAMESPACE::Box2i;
using IMATH_NAMESPACE::divp;
using IMATH_NAMESPACE::modp;
using ILMTHREAD_NAMESPACE::Mutex;
using ILMTHREAD_NAMESPACE::Lock;


//
// Struct InputFile::Data stores things that will be
// needed between calls to readPixels
//

struct InputFile::Data : public Mutex
{
    Header              header;
    int                 version;
    bool                isTiled;

    TiledInputFile *	tFile;
    ScanLineInputFile *	sFile;
    DeepScanLineInputFile * dsFile;

    LineOrder		lineOrder;      // the file's lineorder
    int			minY;           // data window's min y coord
    int			maxY;           // data window's max x coord
    
    FrameBuffer		tFileBuffer; 
    FrameBuffer *	cachedBuffer;
    CompositeDeepScanLine * compositor; // for loading deep files
    
    int			cachedTileY;
    int                 offset;
    
    int                 numThreads;

    int                 partNumber;
    InputPartData*      part;

    bool                multiPartBackwardSupport;
    MultiPartInputFile* multiPartFile;
    InputStreamMutex    * _streamData;
    bool                _deleteStream;

     Data (int numThreads);
    ~Data ();

    void		deleteCachedBuffer();
};


InputFile::Data::Data (int numThreads):
    isTiled (false),
    tFile (0),
    sFile (0),
    dsFile(0),
    cachedBuffer (0),
    compositor(0),
    cachedTileY (-1),
    numThreads (numThreads),
    partNumber (-1),
    part(NULL),
    multiPartBackwardSupport (false),
    multiPartFile (0),
    _streamData(0),
    _deleteStream(false)
           
{
    // empty
}


InputFile::Data::~Data ()
{
    if (tFile)
        delete tFile;
    if (sFile)
        delete sFile;
    if (dsFile)
        delete dsFile;
    if (compositor)
        delete compositor;

    deleteCachedBuffer();

    if (multiPartBackwardSupport && multiPartFile)
        delete multiPartFile;
}


void	
InputFile::Data::deleteCachedBuffer()
{
    //
    // Delete the cached frame buffer, and all memory
    // allocated for the slices in the cached frameBuffer.
    //

    if (cachedBuffer)
    {
	for (FrameBuffer::Iterator k = cachedBuffer->begin();
	     k != cachedBuffer->end();
	     ++k)
	{
	    Slice &s = k.slice();

	    switch (s.type)
	    {
	      case OPENEXR_IMF_INTERNAL_NAMESPACE::UINT:

		delete [] (((unsigned int *)s.base) + offset);
		break;

	      case OPENEXR_IMF_INTERNAL_NAMESPACE::HALF:

		delete [] ((half *)s.base + offset);
		break;

	      case OPENEXR_IMF_INTERNAL_NAMESPACE::FLOAT:

		delete [] (((float *)s.base) + offset);
		break;
              case NUM_PIXELTYPES :
                  throw(IEX_NAMESPACE::ArgExc("Invalid pixel type"));
	    }                
	}

	//
	// delete the cached frame buffer
	//

	delete cachedBuffer;
	cachedBuffer = 0;
    }
}


namespace {

void
bufferedReadPixels (InputFile::Data* ifd, int scanLine1, int scanLine2)
{
    //
    // bufferedReadPixels reads each row of tiles that intersect the
    // scan-line range (scanLine1 to scanLine2). The previous row of
    // tiles is cached in order to prevent redundent tile reads when
    // accessing scanlines sequentially.
    //

    int minY = std::min (scanLine1, scanLine2);
    int maxY = std::max (scanLine1, scanLine2);

    if (minY < ifd->minY || maxY >  ifd->maxY)
    {
        throw IEX_NAMESPACE::ArgExc ("Tried to read scan line outside "
			   "the image file's data window.");
    }

    //
    // The minimum and maximum y tile coordinates that intersect this
    // scanline range
    //

    int minDy = (minY - ifd->minY) / ifd->tFile->tileYSize();
    int maxDy = (maxY - ifd->minY) / ifd->tFile->tileYSize();

    //
    // Figure out which one is first in the file so we can read without seeking
    //

    int yStart, yEnd, yStep;

    if (ifd->lineOrder == DECREASING_Y)
    {
        yStart = maxDy;
        yEnd = minDy - 1;
        yStep = -1;
    }
    else
    {
        yStart = minDy;
        yEnd = maxDy + 1;
        yStep = 1;
    }

    //
    // the number of pixels in a row of tiles
    //

    Box2i levelRange = ifd->tFile->dataWindowForLevel(0);
    
    //
    // Read the tiles into our temporary framebuffer and copy them into
    // the user's buffer
    //

    for (int j = yStart; j != yEnd; j += yStep)
    {
        Box2i tileRange = ifd->tFile->dataWindowForTile (0, j, 0);

        int minYThisRow = std::max (minY, tileRange.min.y);
        int maxYThisRow = std::min (maxY, tileRange.max.y);

        if (j != ifd->cachedTileY)
        {
            //
            // We don't have any valid buffered info, so we need to read in
            // from the file.
            //

            ifd->tFile->readTiles (0, ifd->tFile->numXTiles (0) - 1, j, j);
            ifd->cachedTileY = j;
        }

        //
        // Copy the data from our cached framebuffer into the user's
        // framebuffer.
        //

        for (FrameBuffer::ConstIterator k = ifd->cachedBuffer->begin();
             k != ifd->cachedBuffer->end();
             ++k)
        {
            Slice fromSlice = k.slice();		// slice to write from
            Slice toSlice = ifd->tFileBuffer[k.name()];	// slice to write to

            char *fromPtr, *toPtr;
            int size = pixelTypeSize (toSlice.type);

	    int xStart = levelRange.min.x;
	    int yStart = minYThisRow;

	    while (modp (xStart, toSlice.xSampling) != 0)
		++xStart;

	    while (modp (yStart, toSlice.ySampling) != 0)
		++yStart;

            for (int y = yStart;
		 y <= maxYThisRow;
		 y += toSlice.ySampling)
            {
		//
                // Set the pointers to the start of the y scanline in
                // this row of tiles
		//
                
                fromPtr = fromSlice.base +
                          (y - tileRange.min.y) * fromSlice.yStride +
                          xStart * fromSlice.xStride;

                toPtr = toSlice.base +
                        divp (y, toSlice.ySampling) * toSlice.yStride +
                        divp (xStart, toSlice.xSampling) * toSlice.xStride;

		//
                // Copy all pixels for the scanline in this row of tiles
		//

                for (int x = xStart;
		     x <= levelRange.max.x;
		     x += toSlice.xSampling)
                {
		    for (int i = 0; i < size; ++i)
			toPtr[i] = fromPtr[i];

		    fromPtr += fromSlice.xStride * toSlice.xSampling;
		    toPtr += toSlice.xStride;
                }
            }
        }
    }
}

} // namespace



InputFile::InputFile (const char fileName[], int numThreads):
    _data (new Data (numThreads))
{
    _data->_streamData = NULL;
    _data->_deleteStream=true;
    
    OPENEXR_IMF_INTERNAL_NAMESPACE::IStream* is = 0;
    try
    {
        is = new StdIFStream (fileName);
        readMagicNumberAndVersionField(*is, _data->version);

        //
        // compatibility to read multipart file.
        //
        if (isMultiPart(_data->version))
        {
            compatibilityInitialize(*is);
        }
        else
        {
            _data->_streamData = new InputStreamMutex();
            _data->_streamData->is = is;
            _data->header.readFrom (*_data->_streamData->is, _data->version);
            
            // fix type attribute in single part regular image types
            // (may be wrong if an old version of OpenEXR converts
            // a tiled image to scanline or vice versa)
            if(!isNonImage(_data->version)  && 
               !isMultiPart(_data->version) && 
               _data->header.hasType())
            {
                _data->header.setType(isTiled(_data->version) ? TILEDIMAGE : SCANLINEIMAGE);
            }
            
            _data->header.sanityCheck (isTiled (_data->version));

            initialize();
        }
    }
    catch (IEX_NAMESPACE::BaseExc &e)
    {
        if (is)          delete is;
         
        if ( _data && !_data->multiPartBackwardSupport  && _data->_streamData)
        {
            delete _data->_streamData;
            _data->_streamData=NULL;
        }
        
        if (_data)       delete _data;
        _data=NULL;

        REPLACE_EXC (e, "Cannot read image file "
                     "\"" << fileName << "\". " << e.what());
        throw;
    }
    catch (...)
    {
        if (is)          delete is;
        if (_data && !_data->multiPartBackwardSupport && _data->_streamData)
        {
            delete _data->_streamData;
        }
        if (_data)       delete _data;

        throw;
    }
}


InputFile::InputFile (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream &is, int numThreads):
    _data (new Data (numThreads))
{
    _data->_streamData=NULL;
    _data->_deleteStream=false;
    try
    {
        readMagicNumberAndVersionField(is, _data->version);

        //
        // Backward compatibility to read multpart file.
        //
        if (isMultiPart(_data->version))
        {
            compatibilityInitialize(is);
        }
        else
        {
            _data->_streamData = new InputStreamMutex();
            _data->_streamData->is = &is;
            _data->header.readFrom (*_data->_streamData->is, _data->version);
            
            // fix type attribute in single part regular image types
            // (may be wrong if an old version of OpenEXR converts
            // a tiled image to scanline or vice versa)
            if(!isNonImage(_data->version)  && 
               !isMultiPart(_data->version) &&  
               _data->header.hasType())
            {
                _data->header.setType(isTiled(_data->version) ? TILEDIMAGE : SCANLINEIMAGE);
            }
            
            _data->header.sanityCheck (isTiled (_data->version));

            initialize();
        }
    }
    catch (IEX_NAMESPACE::BaseExc &e)
    {
        if (_data && !_data->multiPartBackwardSupport && _data->_streamData) delete _data->_streamData;
        if (_data)       delete _data;
        _data=NULL; 

        REPLACE_EXC (e, "Cannot read image file "
                     "\"" << is.fileName() << "\". " << e.what());
        throw;
    }
    catch (...)
    {
        if (_data &&  !_data->multiPartBackwardSupport  && _data->_streamData) delete _data->_streamData;
        if (_data)       delete _data;
        _data=NULL;
        throw;
    }
}


InputFile::InputFile (InputPartData* part) :
    _data (new Data (part->numThreads))
{
    _data->_deleteStream=false;
    multiPartInitialize (part);
}


void
InputFile::compatibilityInitialize (OPENEXR_IMF_INTERNAL_NAMESPACE::IStream& is)
{
    is.seekg(0);

    //
    // Construct a MultiPartInputFile, initialize InputFile
    // with the part 0 data.
    // (TODO) may want to have a way to set the reconstruction flag.
    //
    _data->multiPartBackwardSupport = true;
    _data->multiPartFile = new MultiPartInputFile(is, _data->numThreads);
    InputPartData* part = _data->multiPartFile->getPart(0);

    multiPartInitialize (part);
}


void
InputFile::multiPartInitialize (InputPartData* part)
{
    _data->_streamData = part->mutex;
    _data->version = part->version;
    _data->header = part->header;
    _data->partNumber = part->partNumber;
    _data->part = part;

    initialize();
}


void
InputFile::initialize ()
{
    if (!_data->part)
    {
        if(_data->header.hasType() && _data->header.type()==DEEPSCANLINE)
        {
            _data->isTiled=false;
            const Box2i &dataWindow = _data->header.dataWindow();
            _data->minY = dataWindow.min.y;
            _data->maxY = dataWindow.max.y;
            
            _data->dsFile = new DeepScanLineInputFile (_data->header,
                                               _data->_streamData->is,
                                               _data->version,
                                               _data->numThreads);
            _data->compositor = new CompositeDeepScanLine;
            _data->compositor->addSource(_data->dsFile);
        }
        
        else if (isTiled (_data->version))
        {
            _data->isTiled = true;
            _data->lineOrder = _data->header.lineOrder();

            //
            // Save the dataWindow information
            //
    
            const Box2i &dataWindow = _data->header.dataWindow();
            _data->minY = dataWindow.min.y;
            _data->maxY = dataWindow.max.y;

            _data->tFile = new TiledInputFile (_data->header,
                                               _data->_streamData->is,
                                               _data->version,
                                               _data->numThreads);
        }
        
        else if(!_data->header.hasType() || _data->header.type()==SCANLINEIMAGE)
        {
            _data->sFile = new ScanLineInputFile (_data->header,
                                                  _data->_streamData->is,
                                                  _data->numThreads);
        }else{
            // type set but not recognised
            
            THROW(IEX_NAMESPACE::ArgExc, "InputFile cannot handle parts of type " << _data->header.type());
        }
    }
    else
    {
        if(_data->header.hasType() && _data->header.type()==DEEPSCANLINE)
        {
            _data->isTiled=false;
            const Box2i &dataWindow = _data->header.dataWindow();
            _data->minY = dataWindow.min.y;
            _data->maxY = dataWindow.max.y;
            
            _data->dsFile = new DeepScanLineInputFile (_data->part);
            _data->compositor = new CompositeDeepScanLine;
            _data->compositor->addSource(_data->dsFile);
        }
        else if (isTiled (_data->header.type()))
        {
            _data->isTiled = true;
            _data->lineOrder = _data->header.lineOrder();

            //
            // Save the dataWindow information
            //

            const Box2i &dataWindow = _data->header.dataWindow();
            _data->minY = dataWindow.min.y;
            _data->maxY = dataWindow.max.y;

            _data->tFile = new TiledInputFile (_data->part);
        }
        else if(!_data->header.hasType() || _data->header.type()==SCANLINEIMAGE)
        {
            _data->sFile = new ScanLineInputFile (_data->part);
        }else{
            THROW(IEX_NAMESPACE::ArgExc, "InputFile cannot handle parts of type " << _data->header.type());
            
        }
    }
}

#include <iostream>
InputFile::~InputFile ()
{
    if (_data->_deleteStream)
        delete _data->_streamData->is;

    // unless this file was opened via the multipart API,
    // delete the streamData object too
    if (_data->partNumber==-1 && _data->_streamData)
        delete _data->_streamData;

    if (_data)  delete _data;
}

const char *
InputFile::fileName () const
{
    return _data->_streamData->is->fileName();
}


const Header &
InputFile::header () const
{
    return _data->header;
}


int
InputFile::version () const
{
    return _data->version;
}


void
InputFile::setFrameBuffer (const FrameBuffer &frameBuffer)
{
    if (_data->isTiled)
    {
	Lock lock (*_data);

	//
        // We must invalidate the cached buffer if the new frame
	// buffer has a different set of channels than the old
	// frame buffer, or if the type of a channel has changed.
	//

	const FrameBuffer &oldFrameBuffer = _data->tFileBuffer;

	FrameBuffer::ConstIterator i = oldFrameBuffer.begin();
	FrameBuffer::ConstIterator j = frameBuffer.begin();

	while (i != oldFrameBuffer.end() && j != frameBuffer.end())
	{
	    if (strcmp (i.name(), j.name()) || i.slice().type != j.slice().type)
		break;

	    ++i;
	    ++j;
	}

	if (i != oldFrameBuffer.end() || j != frameBuffer.end())
        {
	    //
	    // Invalidate the cached buffer.
	    //

            _data->deleteCachedBuffer ();
	    _data->cachedTileY = -1;

	    //
	    // Create new a cached frame buffer.  It can hold a single
	    // row of tiles.  The cached buffer can be reused for each
	    // row of tiles because we set the yTileCoords parameter of
	    // each Slice to true.
	    //

	    const Box2i &dataWindow = _data->header.dataWindow();
	    _data->cachedBuffer = new FrameBuffer();
	    _data->offset = dataWindow.min.x;
	    
	    int tileRowSize = (dataWindow.max.x - dataWindow.min.x + 1) *
			      _data->tFile->tileYSize();

	    for (FrameBuffer::ConstIterator k = frameBuffer.begin();
		 k != frameBuffer.end();
		 ++k)
	    {
		Slice s = k.slice();

		switch (s.type)
		{
		  case OPENEXR_IMF_INTERNAL_NAMESPACE::UINT:

		    _data->cachedBuffer->insert
			(k.name(),
			 Slice (UINT,
				(char *)(new unsigned int[tileRowSize] - 
					_data->offset),
				sizeof (unsigned int),
				sizeof (unsigned int) *
				    _data->tFile->levelWidth(0),
				1, 1,
				s.fillValue,
				false, true));
		    break;

		  case OPENEXR_IMF_INTERNAL_NAMESPACE::HALF:

		    _data->cachedBuffer->insert
			(k.name(),
			 Slice (HALF,
				(char *)(new half[tileRowSize] - 
					_data->offset),
				sizeof (half),
				sizeof (half) *
				    _data->tFile->levelWidth(0),
				1, 1,
				s.fillValue,
				false, true));
		    break;

		  case OPENEXR_IMF_INTERNAL_NAMESPACE::FLOAT:

		    _data->cachedBuffer->insert
			(k.name(),
			 Slice (OPENEXR_IMF_INTERNAL_NAMESPACE::FLOAT,
				(char *)(new float[tileRowSize] - 
					_data->offset),
				sizeof(float),
				sizeof(float) *
				    _data->tFile->levelWidth(0),
				1, 1,
				s.fillValue,
				false, true));
		    break;

		  default:

		    throw IEX_NAMESPACE::ArgExc ("Unknown pixel data type.");
		}
	    }

	    _data->tFile->setFrameBuffer (*_data->cachedBuffer);
        }

	_data->tFileBuffer = frameBuffer;
    }
    else if(_data->compositor)
    {
        _data->compositor->setFrameBuffer(frameBuffer);
    }else {
        _data->sFile->setFrameBuffer(frameBuffer);
        _data->tFileBuffer = frameBuffer;
    }
}


const FrameBuffer &
InputFile::frameBuffer () const
{
    if(_data->compositor)
    {
        return _data->compositor->frameBuffer();
    }
    else if(_data->isTiled)
    {
	Lock lock (*_data);
	return _data->tFileBuffer;
    }
    else
    {
	return _data->sFile->frameBuffer();
    }
}


bool
InputFile::isComplete () const
{
    if (_data->dsFile)
        return _data->dsFile->isComplete();
    else if (_data->isTiled)
	return _data->tFile->isComplete();
    else
	return _data->sFile->isComplete();
}

bool
InputFile::isOptimizationEnabled() const
{
   if(_data->sFile)
   {
       return _data->sFile->isOptimizationEnabled();
   }else{
       return false;
   }
}


void
InputFile::readPixels (int scanLine1, int scanLine2)
{
    if (_data->compositor)
    {
        _data->compositor->readPixels(scanLine1,scanLine2);
    }
    else if (_data->isTiled)
    {
	Lock lock (*_data);
        bufferedReadPixels (_data, scanLine1, scanLine2);
    }
    else
    {
        _data->sFile->readPixels (scanLine1, scanLine2);
    }
}


void
InputFile::readPixels (int scanLine)
{
    readPixels (scanLine, scanLine);
}


void
InputFile::rawPixelData (int firstScanLine,
			 const char *&pixelData,
			 int &pixelDataSize)
{
    try
    {
        if (_data->dsFile)
        {
            throw IEX_NAMESPACE::ArgExc ("Tried to read a raw scanline "
            "from a deep image.");
        }
        
	else if (_data->isTiled)
	{
	    throw IEX_NAMESPACE::ArgExc ("Tried to read a raw scanline "
			       "from a tiled image.");
	}
        
        _data->sFile->rawPixelData (firstScanLine, pixelData, pixelDataSize);
    }
    catch (IEX_NAMESPACE::BaseExc &e)
    {
	REPLACE_EXC (e, "Error reading pixel data from image "
                 "file \"" << fileName() << "\". " << e.what());
	throw;
    }
}




void
InputFile::rawPixelDataToBuffer (int scanLine,
                                 char *pixelData,
                                 int &pixelDataSize) const
{
    try
    {
        if (_data->dsFile)
        {
            throw IEX_NAMESPACE::ArgExc ("Tried to read a raw scanline "
                                         "from a deep image.");
        }
        
        else if (_data->isTiled)
        {
            throw IEX_NAMESPACE::ArgExc ("Tried to read a raw scanline "
                                         "from a tiled image.");
        }
        
        _data->sFile->rawPixelDataToBuffer(scanLine, pixelData, pixelDataSize);
    }
    catch (IEX_NAMESPACE::BaseExc &e)
    {
        REPLACE_EXC (e, "Error reading pixel data from image "
                     "file \"" << fileName() << "\". " << e.what());
        throw;
    }
}



void
InputFile::rawTileData (int &dx, int &dy,
			int &lx, int &ly,
			const char *&pixelData,
			int &pixelDataSize)
{
    try
    {
	if (!_data->isTiled)
	{
	    throw IEX_NAMESPACE::ArgExc ("Tried to read a raw tile "
			       "from a scanline-based image.");
	}
        
        _data->tFile->rawTileData (dx, dy, lx, ly, pixelData, pixelDataSize);
    }
    catch (IEX_NAMESPACE::BaseExc &e)
    {
	REPLACE_EXC (e, "Error reading tile data from image "
                 "file \"" << fileName() << "\". " << e.what());
	throw;
    }
}


TiledInputFile*
InputFile::tFile()
{
    if (!_data->isTiled)
    {
	throw IEX_NAMESPACE::ArgExc ("Cannot get a TiledInputFile pointer "
			   "from an InputFile that is not tiled.");
    }

    return _data->tFile;
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
- **InputFile**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ImfPartType.h`
- `ImfStdIO.h`
- `ImathFun.h`
- `ImfCompositeDeepScanLine.h`
- `iostream`
- `ImfInputFile.h`
- `half.h`
- `ImfVersion.h`
- `ImfMultiPartInputFile.h`
- `ImfTiledInputFile.h`
- `IlmThreadMutex.h`
- `ImfScanLineInputFile.h`
- `ImfChannelList.h`
- `algorithm`
- `fstream`
- `ImfMisc.h`
- `ImfInputPartData.h`
- `ImfDeepScanLineInputFile.h`
- `Iex.h`
- `ImfNamespace.h`

**Python Imports:**
- `the`
- `image`
- `an`
- `Slice`
- `this`
- `our`
- `a`


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

