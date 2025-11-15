# Documentation for `3rdparty/openexr/IlmImf/ImfPartHelper.h`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmImf/ImfPartHelper.h`
- **File Name**: `ImfPartHelper.h`
- **File Size**: 7,841 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/openexr/IlmImf/ImfPartHelper.h](../../../3rdparty/openexr/IlmImf/ImfPartHelper.h)

## Purpose and Role

This file is located in the `3rdparty/openexr/IlmImf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
///////////////////////////////////////////////////////////////////////////
//
// Copyright (c) 2012, Weta Digital Ltd
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
// *       Neither the name of Weta Digital nor the names of
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


#ifndef INCLUDED_IMF_PARTHELPER_H
#define INCLUDED_IMF_PARTHELPER_H

//-----------------------------------------------------------------------------
//
//	Functions to help split channels into separate parts: provide a list of
//      channels, with desired views. call SplitChannels to assign a part to each
//      layer, or correct the name of the channel.
//      Also can enumerate the parts in a file and list which parts channels are in
//
//      This is a good way to offer a 'create Multipart file' checkbox to the user in a
//      write dialog box: Populate a list of MultiViewChannelName objects,
//      call SplitChannels with whether single or multipart files are required.
//      Then write the number of parts it specifies, using internal_name for the channel
//      names in the ChannelList and FrameBuffer objects. There should be no need
//      for different codepaths for single part and multipart files
//
//      Similarly, on reading a file as a MultiPartInputFile, use GetChannelsInMultiPartFile to
//      enumerate all channels in the file, using internal_name in FrameBuffer objects
//      to read the channel
//   
//
//-----------------------------------------------------------------------------

#include "ImfForward.h"
#include "ImfNamespace.h"
#include "ImfExport.h"
#include "ImfMultiPartInputFile.h"
#include "ImfChannelList.h"
#include "ImfStringVectorAttribute.h"
#include "ImfStandardAttributes.h"
#include "ImfMultiView.h"

#include <string>
#include <map>
#include <set>

OPENEXR_IMF_INTERNAL_NAMESPACE_HEADER_ENTER

struct MultiViewChannelName{
  
public:
  std::string name;         ///< name of channel
  std::string view;         ///< view for channel
  
  int part_number;          ///< part number: updated by SplitChannels
  std::string internal_name;///< name used in headers: in singlepart mode, may contain viewname
  
  virtual ~MultiViewChannelName() {}
    
    //return layer for this channel, or "" if no layer
    std::string getLayer() const 
    {
        std::size_t q=name.rfind('.');
	if(  q==name.npos  )
	{
	    return "";
	}
	return name.substr(0,q);
    
  }

  std::string getSuffix() const
  {
        std::size_t q=name.rfind('.');
	if(  q==name.npos  )
	{
	    return name;
	}
	return name.substr(q+1);
      
  }
  
};



//
///\brief assigns individual channels to different parts based on their layer and view name
///       input is an array, list, vector etc of MultiViewChannelName objects
///       on entry, each MultiViewChannelName name/view must be set (view can be empty if not multiview)
///
///       if singlepart set, then on exit part_number will be zero, and internal_name will have view name inserted
///       otherwise, each channel will be assigned to a different part based on its layer name and view name
///
/// @param begin pointer to first MultiViewChannelName item
/// @param end   pointer to end of MultiViewChannelName item array
/// @return      total number of parts required
//

template<typename T> int 
SplitChannels(const T & begin,const T & end,bool multipart=true,const std::string & heroView="")
{
    if(!multipart)
    {
	for(T i=begin;i!=end;i++)
	{
	    i->part_number=0;
	
	    //does this have a view name set?
	    if(i->view=="")
	    {
		i->internal_name=i->name;
	    }else{
		
		std::string lname = i->getLayer();
	   
		// no layer, only non-hero views get view name in layer name
	
	    
		if(lname=="")
		{
		    if(i->view==heroView)
		    {
			i->internal_name = i->name;
		    }else{
			i->internal_name = i->view+"."+i->name;
		    }
		}else{
		    i->internal_name = lname+"."+i->view+"."+i->getSuffix();
		}
	    }
	}
	// single part created
	return 1;
    }else{
	// step 1: extract individual layers and parts
	// for each layer, enumerate which views are active
	
	std::map< std::string , std::set< std::string > > viewsInLayers;
	for(T i=begin;i!=end;i++)
	{
	    viewsInLayers[i->getLayer()].insert(i->view);
	}
	
	// step 2: assign a part number to each layer/view
	
	std::map< std::pair<std::string,std::string> , int > layerToPart;
	
	int partCount=0;
	
	for(std::map< std::string , std::set< std::string > >::const_iterator layer=viewsInLayers.begin();
	    layer!=viewsInLayers.end();layer++)
	{
	    // if this layer has a heroView, insert that first
	    bool layer_has_hero = layer->second.find(heroView)!=layer->second.end();
	    if( layer_has_hero )
	    {
		layerToPart[ std::make_pair(layer->first,heroView) ] = partCount++;
	    }
	    
	    
	    // insert other layers which aren't the hero view
	    for(std::set< std::string >::const_iterator view=layer->second.begin();
		view!=layer->second.end();view++)
	    {
		if(*view!=heroView)
		{
		    layerToPart[ std::make_pair(layer->first,*view) ] = partCount++;
		}
	    }
		
	}
	
	// step 3: update part number of each provided channel
	
	for( T i=begin;i!=end;i++)
	{
	    i->internal_name=i->name;
	    i->part_number = layerToPart[ std::make_pair(i->getLayer(),i->view) ];
	}
	
	
	// return number of parts created
	return partCount;
    }
}

//
// populate the chans vector<MultiViewChannelName> with a list of channels in the file
// and their corresponding part number
//
template<class T> void
GetChannelsInMultiPartFile(const MultiPartInputFile & file,T & chans)
{
    bool has_multiview=false;
    StringVector mview; 
    if(file.parts()==1)
    {
	if(hasMultiView(file.header(0)))
	{
	    mview=multiView(file.header(0));
	    has_multiview=true;
	}
    }
    
    for(int p=0;p<file.parts();p++)
    {
	const ChannelList & c=file.header(p).channels();
	
	std::string view="";
	if(file.header(p).hasView())
	{
	    view=file.header(p).view();
	}
	for(ChannelList::ConstIterator i=c.begin();i!=c.end();i++)
	{
	    MultiViewChannelName m;
            m.name=std::string(i.name());
	    m.internal_name=m.name;

	    if(has_multiview)
	    {
		m.view=viewFromChannelName(m.name,mview);
		m.name=removeViewName(m.internal_name,m.view);
	    }else{
		m.view=view;
	    }
            m.part_number=p;
	    chans.push_back(m);
		
	}
    }
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

- **T**: A class/struct defined in this file
- **MultiViewChannelName**: A class/struct defined in this file

### Functions and Methods

- **INCLUDED_IMF_PARTHELPER_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `set`
- `ImfMultiPartInputFile.h`
- `ImfStandardAttributes.h`
- `ImfMultiView.h`
- `ImfStringVectorAttribute.h`
- `string`
- `ImfExport.h`
- `ImfChannelList.h`
- `ImfNamespace.h`
- `ImfForward.h`
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

