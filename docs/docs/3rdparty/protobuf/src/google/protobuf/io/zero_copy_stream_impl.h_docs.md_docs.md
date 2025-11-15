# Documentation for `docs/3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h_docs.md`
- **File Name**: `zero_copy_stream_impl.h_docs.md`
- **File Size**: 16,305 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h_docs.md](../../../../../../../docs/3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/protobuf/src/google/protobuf/io` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h`

## File Metadata

- **Full Path**: `3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h`
- **File Name**: `zero_copy_stream_impl.h`
- **File Size**: 12,752 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h](../../../../../../3rdparty/protobuf/src/google/protobuf/io/zero_copy_stream_impl.h)

## Purpose and Role

This file is located in the `3rdparty/protobuf/src/google/protobuf/io` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// Protocol Buffers - Google's data interchange format
// Copyright 2008 Google Inc.  All rights reserved.
// https://developers.google.com/protocol-buffers/
//
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions are
// met:
//
//     * Redistributions of source code must retain the above copyright
// notice, this list of conditions and the following disclaimer.
//     * Redistributions in binary form must reproduce the above
// copyright notice, this list of conditions and the following disclaimer
// in the documentation and/or other materials provided with the
// distribution.
//     * Neither the name of Google Inc. nor the names of its
// contributors may be used to endorse or promote products derived from
// this software without specific prior written permission.
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

// Author: kenton@google.com (Kenton Varda)
//  Based on original Protocol Buffers design by
//  Sanjay Ghemawat, Jeff Dean, and others.
//
// This file contains common implementations of the interfaces defined in
// zero_copy_stream.h which are only included in the full (non-lite)
// protobuf library.  These implementations include Unix file descriptors
// and C++ iostreams.  See also:  zero_copy_stream_impl_lite.h

#ifndef GOOGLE_PROTOBUF_IO_ZERO_COPY_STREAM_IMPL_H__
#define GOOGLE_PROTOBUF_IO_ZERO_COPY_STREAM_IMPL_H__


#include <iosfwd>
#include <string>

#include <google/protobuf/stubs/common.h>
#include <google/protobuf/io/zero_copy_stream.h>
#include <google/protobuf/io/zero_copy_stream_impl_lite.h>

#include <google/protobuf/port_def.inc>

namespace google {
namespace protobuf {
namespace io {

// ===================================================================

// A ZeroCopyInputStream which reads from a file descriptor.
//
// FileInputStream is preferred over using an ifstream with IstreamInputStream.
// The latter will introduce an extra layer of buffering, harming performance.
// Also, it's conceivable that FileInputStream could someday be enhanced
// to use zero-copy file descriptors on OSs which support them.
class PROTOBUF_EXPORT FileInputStream : public ZeroCopyInputStream {
 public:
  // Creates a stream that reads from the given Unix file descriptor.
  // If a block_size is given, it specifies the number of bytes that
  // should be read and returned with each call to Next().  Otherwise,
  // a reasonable default is used.
  explicit FileInputStream(int file_descriptor, int block_size = -1);

  // Flushes any buffers and closes the underlying file.  Returns false if
  // an error occurs during the process; use GetErrno() to examine the error.
  // Even if an error occurs, the file descriptor is closed when this returns.
  bool Close();

  // By default, the file descriptor is not closed when the stream is
  // destroyed.  Call SetCloseOnDelete(true) to change that.  WARNING:
  // This leaves no way for the caller to detect if close() fails.  If
  // detecting close() errors is important to you, you should arrange
  // to close the descriptor yourself.
  void SetCloseOnDelete(bool value) { copying_input_.SetCloseOnDelete(value); }

  // If an I/O error has occurred on this file descriptor, this is the
  // errno from that error.  Otherwise, this is zero.  Once an error
  // occurs, the stream is broken and all subsequent operations will
  // fail.
  int GetErrno() const { return copying_input_.GetErrno(); }

  // implements ZeroCopyInputStream ----------------------------------
  bool Next(const void** data, int* size) override;
  void BackUp(int count) override;
  bool Skip(int count) override;
  int64_t ByteCount() const override;

 private:
  class PROTOBUF_EXPORT CopyingFileInputStream : public CopyingInputStream {
   public:
    CopyingFileInputStream(int file_descriptor);
    ~CopyingFileInputStream() override;

    bool Close();
    void SetCloseOnDelete(bool value) { close_on_delete_ = value; }
    int GetErrno() const { return errno_; }

    // implements CopyingInputStream ---------------------------------
    int Read(void* buffer, int size) override;
    int Skip(int count) override;

   private:
    // The file descriptor.
    const int file_;
    bool close_on_delete_;
    bool is_closed_;

    // The errno of the I/O error, if one has occurred.  Otherwise, zero.
    int errno_;

    // Did we try to seek once and fail?  If so, we assume this file descriptor
    // doesn't support seeking and won't try again.
    bool previous_seek_failed_;

    GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(CopyingFileInputStream);
  };

  CopyingFileInputStream copying_input_;
  CopyingInputStreamAdaptor impl_;

  GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(FileInputStream);
};

// ===================================================================

// A ZeroCopyOutputStream which writes to a file descriptor.
//
// FileOutputStream is preferred over using an ofstream with
// OstreamOutputStream.  The latter will introduce an extra layer of buffering,
// harming performance.  Also, it's conceivable that FileOutputStream could
// someday be enhanced to use zero-copy file descriptors on OSs which
// support them.
class PROTOBUF_EXPORT FileOutputStream : public CopyingOutputStreamAdaptor {
 public:
  // Creates a stream that writes to the given Unix file descriptor.
  // If a block_size is given, it specifies the size of the buffers
  // that should be returned by Next().  Otherwise, a reasonable default
  // is used.
  explicit FileOutputStream(int file_descriptor, int block_size = -1);

  ~FileOutputStream() override;

  // Flushes any buffers and closes the underlying file.  Returns false if
  // an error occurs during the process; use GetErrno() to examine the error.
  // Even if an error occurs, the file descriptor is closed when this returns.
  bool Close();

  // By default, the file descriptor is not closed when the stream is
  // destroyed.  Call SetCloseOnDelete(true) to change that.  WARNING:
  // This leaves no way for the caller to detect if close() fails.  If
  // detecting close() errors is important to you, you should arrange
  // to close the descriptor yourself.
  void SetCloseOnDelete(bool value) { copying_output_.SetCloseOnDelete(value); }

  // If an I/O error has occurred on this file descriptor, this is the
  // errno from that error.  Otherwise, this is zero.  Once an error
  // occurs, the stream is broken and all subsequent operations will
  // fail.
  int GetErrno() const { return copying_output_.GetErrno(); }

 private:
  class PROTOBUF_EXPORT CopyingFileOutputStream : public CopyingOutputStream {
   public:
    CopyingFileOutputStream(int file_descriptor);
    ~CopyingFileOutputStream() override;

    bool Close();
    void SetCloseOnDelete(bool value) { close_on_delete_ = value; }
    int GetErrno() const { return errno_; }

    // implements CopyingOutputStream --------------------------------
    bool Write(const void* buffer, int size) override;

   private:
    // The file descriptor.
    const int file_;
    bool close_on_delete_;
    bool is_closed_;

    // The errno of the I/O error, if one has occurred.  Otherwise, zero.
    int errno_;

    GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(CopyingFileOutputStream);
  };

  CopyingFileOutputStream copying_output_;

  GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(FileOutputStream);
};

// ===================================================================

// A ZeroCopyInputStream which reads from a C++ istream.
//
// Note that for reading files (or anything represented by a file descriptor),
// FileInputStream is more efficient.
class PROTOBUF_EXPORT IstreamInputStream : public ZeroCopyInputStream {
 public:
  // Creates a stream that reads from the given C++ istream.
  // If a block_size is given, it specifies the number of bytes that
  // should be read and returned with each call to Next().  Otherwise,
  // a reasonable default is used.
  explicit IstreamInputStream(std::istream* stream, int block_size = -1);

  // implements ZeroCopyInputStream ----------------------------------
  bool Next(const void** data, int* size) override;
  void BackUp(int count) override;
  bool Skip(int count) override;
  int64_t ByteCount() const override;

 private:
  class PROTOBUF_EXPORT CopyingIstreamInputStream : public CopyingInputStream {
   public:
    CopyingIstreamInputStream(std::istream* input);
    ~CopyingIstreamInputStream() override;

    // implements CopyingInputStream ---------------------------------
    int Read(void* buffer, int size) override;
    // (We use the default implementation of Skip().)

   private:
    // The stream.
    std::istream* input_;

    GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(CopyingIstreamInputStream);
  };

  CopyingIstreamInputStream copying_input_;
  CopyingInputStreamAdaptor impl_;

  GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(IstreamInputStream);
};

// ===================================================================

// A ZeroCopyOutputStream which writes to a C++ ostream.
//
// Note that for writing files (or anything represented by a file descriptor),
// FileOutputStream is more efficient.
class PROTOBUF_EXPORT OstreamOutputStream : public ZeroCopyOutputStream {
 public:
  // Creates a stream that writes to the given C++ ostream.
  // If a block_size is given, it specifies the size of the buffers
  // that should be returned by Next().  Otherwise, a reasonable default
  // is used.
  explicit OstreamOutputStream(std::ostream* stream, int block_size = -1);
  ~OstreamOutputStream() override;

  // implements ZeroCopyOutputStream ---------------------------------
  bool Next(void** data, int* size) override;
  void BackUp(int count) override;
  int64_t ByteCount() const override;

 private:
  class PROTOBUF_EXPORT CopyingOstreamOutputStream
      : public CopyingOutputStream {
   public:
    CopyingOstreamOutputStream(std::ostream* output);
    ~CopyingOstreamOutputStream() override;

    // implements CopyingOutputStream --------------------------------
    bool Write(const void* buffer, int size) override;

   private:
    // The stream.
    std::ostream* output_;

    GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(CopyingOstreamOutputStream);
  };

  CopyingOstreamOutputStream copying_output_;
  CopyingOutputStreamAdaptor impl_;

  GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(OstreamOutputStream);
};

// ===================================================================

// A ZeroCopyInputStream which reads from several other streams in sequence.
// ConcatenatingInputStream is unable to distinguish between end-of-stream
// and read errors in the underlying streams, so it assumes any errors mean
// end-of-stream.  So, if the underlying streams fail for any other reason,
// ConcatenatingInputStream may do odd things.  It is suggested that you do
// not use ConcatenatingInputStream on streams that might produce read errors
// other than end-of-stream.
class PROTOBUF_EXPORT ConcatenatingInputStream : public ZeroCopyInputStream {
 public:
  // All streams passed in as well as the array itself must remain valid
  // until the ConcatenatingInputStream is destroyed.
  ConcatenatingInputStream(ZeroCopyInputStream* const streams[], int count);
  ~ConcatenatingInputStream() override = default;

  // implements ZeroCopyInputStream ----------------------------------
  bool Next(const void** data, int* size) override;
  void BackUp(int count) override;
  bool Skip(int count) override;
  int64_t ByteCount() const override;


 private:
  // As streams are retired, streams_ is incremented and count_ is
  // decremented.
  ZeroCopyInputStream* const* streams_;
  int stream_count_;
  int64_t bytes_retired_;  // Bytes read from previous streams.

  GOOGLE_DISALLOW_EVIL_CONSTRUCTORS(ConcatenatingInputStream);
};

// ===================================================================

}  // namespace io
}  // namespace protobuf
}  // namespace google

#include <google/protobuf/port_undef.inc>

#endif  // GOOGLE_PROTOBUF_IO_ZERO_COPY_STREAM_IMPL_H__
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

- **PROTOBUF_EXPORT**: A class/struct defined in this file

### Functions and Methods

- **GOOGLE_PROTOBUF_IO_ZERO_COPY_STREAM_IMPL_H__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iosfwd`
- `google/protobuf/io/zero_copy_stream.h`
- `google/protobuf/io/zero_copy_stream_impl_lite.h`
- `google/protobuf/port_undef.inc`
- `string`
- `google/protobuf/stubs/common.h`
- `google/protobuf/port_def.inc`

**Python Imports:**
- `the`
- `previous`
- `several`
- `a`
- `that`


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

