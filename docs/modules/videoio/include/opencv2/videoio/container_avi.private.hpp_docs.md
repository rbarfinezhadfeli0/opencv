# Documentation for `modules/videoio/include/opencv2/videoio/container_avi.private.hpp`

## File Metadata

- **Full Path**: `modules/videoio/include/opencv2/videoio/container_avi.private.hpp`
- **File Name**: `container_avi.private.hpp`
- **File Size**: 6,013 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/videoio/include/opencv2/videoio/container_avi.private.hpp](../../../../../modules/videoio/include/opencv2/videoio/container_avi.private.hpp)

## Purpose and Role

This file is located in the `modules/videoio/include/opencv2/videoio` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef CONTAINER_AVI_HPP
#define CONTAINER_AVI_HPP

#ifndef __OPENCV_BUILD
#  error this is a private header which should not be used from outside of the OpenCV library
#endif

#include "opencv2/core/types.hpp"
#include <deque>
#include <vector>
#include <string>
#include <memory>

namespace cv
{

/*
AVI struct:

RIFF ('AVI '
      LIST ('hdrl'
            'avih'(<Main AVI Header>)
            LIST ('strl'
                  'strh'(<Stream header>)
                  'strf'(<Stream format>)
                  [ 'strd'(<Additional header data>) ]
                  [ 'strn'(<Stream name>) ]
                  [ 'indx'(<Odml index data>) ]
                  ...
                 )
            [LIST ('strl' ...)]
            [LIST ('strl' ...)]
            ...
            [LIST ('odml'
                  'dmlh'(<ODML header data>)
                  ...
                 )
            ]
            ...
           )
      [LIST ('INFO' ...)]
      [JUNK]
      LIST ('movi'
            {{xxdb|xxdc|xxpc|xxwb}(<Data>) | LIST ('rec '
                              {xxdb|xxdc|xxpc|xxwb}(<Data>)
                              {xxdb|xxdc|xxpc|xxwb}(<Data>)
                              ...
                             )
               ...
            }
            ...
           )
      ['idx1' (<AVI Index>) ]
     )

     {xxdb|xxdc|xxpc|xxwb}
     xx - stream number: 00, 01, 02, ...
     db - uncompressed video frame
     dc - compressed video frame
     pc - palette change
     wb - audio frame

     JUNK section may pad any data section and must be ignored
*/

typedef std::deque< std::pair<uint64_t, uint32_t> > frame_list;
typedef frame_list::iterator frame_iterator;
struct RiffChunk;
struct RiffList;
class VideoInputStream;
enum Codecs { MJPEG };

//Represents single MJPEG video stream within single AVI/AVIX entry
//Multiple video streams within single AVI/AVIX entry are not supported
//ODML index is not supported
class CV_EXPORTS AVIReadContainer
{
public:
    AVIReadContainer();

    void initStream(const std::string& filename);
    void initStream(std::shared_ptr<VideoInputStream> m_file_stream_);

    void close();
    //stores founded frames in m_frame_list which can be accessed via getFrames
    bool parseAvi(Codecs codec_) { return parseAviWithFrameList(m_frame_list, codec_); }
    //stores founded frames in in_frame_list. getFrames() would return empty list
    bool parseAvi(frame_list& in_frame_list, Codecs codec_) { return parseAviWithFrameList(in_frame_list, codec_); }
    size_t getFramesCount() { return m_frame_list.size(); }
    frame_list& getFrames() { return m_frame_list; }
    unsigned int getWidth() { return m_width; }
    unsigned int getHeight() { return m_height; }
    double getFps() { return m_fps; }
    std::vector<char> readFrame(frame_iterator it);
    bool parseRiff(frame_list &m_mjpeg_frames);

protected:

    bool parseAviWithFrameList(frame_list& in_frame_list, Codecs codec_);
    void skipJunk(RiffChunk& chunk);
    void skipJunk(RiffList& list);
    bool parseHdrlList(Codecs codec_);
    bool parseIndex(unsigned int index_size, frame_list& in_frame_list);
    bool parseMovi(frame_list& in_frame_list)
    {
        //not implemented
        CV_UNUSED(in_frame_list);
        // FIXIT: in_frame_list.empty();
        return true;
    }
    bool parseStrl(char stream_id, Codecs codec_);
    bool parseInfo()
    {
        //not implemented
        return true;
    }

    void printError(RiffList& list, unsigned int expected_fourcc);

    void printError(RiffChunk& chunk, unsigned int expected_fourcc);

    std::shared_ptr<VideoInputStream> m_file_stream;
    unsigned int   m_stream_id;
    unsigned long long int   m_movi_start;
    unsigned long long int    m_movi_end;
    frame_list m_frame_list;
    unsigned int   m_width;
    unsigned int   m_height;
    double     m_fps;
    bool       m_is_indx_present;
};

enum { COLORSPACE_GRAY=0, COLORSPACE_RGBA=1, COLORSPACE_BGR=2, COLORSPACE_YUV444P=3 };
enum StreamType { db, dc, pc, wb };
class BitStream;

// {xxdb|xxdc|xxpc|xxwb}
// xx - stream number: 00, 01, 02, ...
// db - uncompressed video frame
// dc - compressed video frame
// pc - palette change
// wb - audio frame


class CV_EXPORTS AVIWriteContainer
{
public:
    AVIWriteContainer();
    ~AVIWriteContainer();

    bool initContainer(const std::string& filename, double fps, cv::Size size, bool iscolor);
    void startWriteAVI(int stream_count);
    void writeStreamHeader(Codecs codec_);
    void startWriteChunk(uint32_t fourcc);
    void endWriteChunk();

    int getAVIIndex(int stream_number, StreamType strm_type);
    void writeIndex(int stream_number, StreamType strm_type);
    void finishWriteAVI();

    bool isOpenedStream() const;
    bool isEmptyFrameOffset() const { return frameOffset.empty(); }
    int getWidth() const { return width; }
    int getHeight() const { return height; }
    int getChannels() const { return channels; }
    size_t getMoviPointer() const { return moviPointer; }
    size_t getStreamPos() const;

    void pushFrameOffset(size_t elem) { frameOffset.push_back(elem); }
    void pushFrameSize(size_t elem) { frameSize.push_back(elem); }
    bool isEmptyFrameSize() const { return frameSize.empty(); }
    size_t atFrameSize(size_t i) const { return frameSize[i]; }
    size_t countFrameSize() const { return frameSize.size(); }
    void jputStreamShort(int val);
    void putStreamBytes(const uchar* buf, int count);
    void putStreamByte(int val);
    void jputStream(unsigned currval);
    void jflushStream(unsigned currval, int bitIdx);

private:
    std::shared_ptr<BitStream> strm;
    int outfps;
    int width, height, channels;
    size_t moviPointer;
    std::vector<size_t> frameOffset, frameSize, AVIChunkSizeIndex, frameNumIndexes;
};

}

#endif //CONTAINER_AVI_HPP
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

- **CV_EXPORTS**: A class/struct defined in this file
- **RiffChunk**: A class/struct defined in this file
- **BitStream**: A class/struct defined in this file
- **VideoInputStream**: A class/struct defined in this file
- **RiffList**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **CONTAINER_AVI_HPP()**: A function/method defined in this file
- **frame_list()**: A function/method defined in this file
- **__OPENCV_BUILD()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `deque`
- `vector`
- `string`
- `memory`
- `opencv2/core/types.hpp`

**Python Imports:**
- `outside`


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

