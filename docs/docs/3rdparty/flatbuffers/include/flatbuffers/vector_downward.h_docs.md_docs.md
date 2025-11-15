# Documentation for `docs/3rdparty/flatbuffers/include/flatbuffers/vector_downward.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/flatbuffers/include/flatbuffers/vector_downward.h_docs.md`
- **File Name**: `vector_downward.h_docs.md`
- **File Size**: 12,071 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/flatbuffers/include/flatbuffers/vector_downward.h_docs.md](../../../../../docs/3rdparty/flatbuffers/include/flatbuffers/vector_downward.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/flatbuffers/include/flatbuffers` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/flatbuffers/include/flatbuffers/vector_downward.h`

## File Metadata

- **Full Path**: `3rdparty/flatbuffers/include/flatbuffers/vector_downward.h`
- **File Name**: `vector_downward.h`
- **File Size**: 8,661 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/flatbuffers/include/flatbuffers/vector_downward.h](../../../../3rdparty/flatbuffers/include/flatbuffers/vector_downward.h)

## Purpose and Role

This file is located in the `3rdparty/flatbuffers/include/flatbuffers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * Copyright 2021 Google Inc. All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

#ifndef FLATBUFFERS_VECTOR_DOWNWARD_H_
#define FLATBUFFERS_VECTOR_DOWNWARD_H_

#include <cstdint>

#include <algorithm>

#include "flatbuffers/base.h"
#include "flatbuffers/default_allocator.h"
#include "flatbuffers/detached_buffer.h"

namespace flatbuffers {

// This is a minimal replication of std::vector<uint8_t> functionality,
// except growing from higher to lower addresses. i.e. push_back() inserts data
// in the lowest address in the vector.
// Since this vector leaves the lower part unused, we support a "scratch-pad"
// that can be stored there for temporary data, to share the allocated space.
// Essentially, this supports 2 std::vectors in a single buffer.
template<typename SizeT = uoffset_t> class vector_downward {
 public:
  explicit vector_downward(size_t initial_size, Allocator *allocator,
                           bool own_allocator, size_t buffer_minalign,
                           const SizeT max_size = FLATBUFFERS_MAX_BUFFER_SIZE)
      : allocator_(allocator),
        own_allocator_(own_allocator),
        initial_size_(initial_size),
        max_size_(max_size),
        buffer_minalign_(buffer_minalign),
        reserved_(0),
        size_(0),
        buf_(nullptr),
        cur_(nullptr),
        scratch_(nullptr) {}

  vector_downward(vector_downward &&other) noexcept
      // clang-format on
      : allocator_(other.allocator_),
        own_allocator_(other.own_allocator_),
        initial_size_(other.initial_size_),
        max_size_(other.max_size_),
        buffer_minalign_(other.buffer_minalign_),
        reserved_(other.reserved_),
        size_(other.size_),
        buf_(other.buf_),
        cur_(other.cur_),
        scratch_(other.scratch_) {
    // No change in other.allocator_
    // No change in other.initial_size_
    // No change in other.buffer_minalign_
    other.own_allocator_ = false;
    other.reserved_ = 0;
    other.buf_ = nullptr;
    other.cur_ = nullptr;
    other.scratch_ = nullptr;
  }

  vector_downward &operator=(vector_downward &&other) noexcept {
    // Move construct a temporary and swap idiom
    vector_downward temp(std::move(other));
    swap(temp);
    return *this;
  }

  ~vector_downward() {
    clear_buffer();
    clear_allocator();
  }

  void reset() {
    clear_buffer();
    clear();
  }

  void clear() {
    if (buf_) {
      cur_ = buf_ + reserved_;
    } else {
      reserved_ = 0;
      cur_ = nullptr;
    }
    size_ = 0;
    clear_scratch();
  }

  void clear_scratch() { scratch_ = buf_; }

  void clear_allocator() {
    if (own_allocator_ && allocator_) { delete allocator_; }
    allocator_ = nullptr;
    own_allocator_ = false;
  }

  void clear_buffer() {
    if (buf_) Deallocate(allocator_, buf_, reserved_);
    buf_ = nullptr;
  }

  // Relinquish the pointer to the caller.
  uint8_t *release_raw(size_t &allocated_bytes, size_t &offset) {
    auto *buf = buf_;
    allocated_bytes = reserved_;
    offset = vector_downward::offset();

    // release_raw only relinquishes the buffer ownership.
    // Does not deallocate or reset the allocator. Destructor will do that.
    buf_ = nullptr;
    clear();
    return buf;
  }

  // Relinquish the pointer to the caller.
  DetachedBuffer release() {
    // allocator ownership (if any) is transferred to DetachedBuffer.
    DetachedBuffer fb(allocator_, own_allocator_, buf_, reserved_, cur_,
                      size());
    if (own_allocator_) {
      allocator_ = nullptr;
      own_allocator_ = false;
    }
    buf_ = nullptr;
    clear();
    return fb;
  }

  size_t ensure_space(size_t len) {
    FLATBUFFERS_ASSERT(cur_ >= scratch_ && scratch_ >= buf_);
    // If the length is larger than the unused part of the buffer, we need to
    // grow.
    if (len > unused_buffer_size()) { reallocate(len); }
    FLATBUFFERS_ASSERT(size() < max_size_);
    return len;
  }

  inline uint8_t *make_space(size_t len) {
    if (len) {
      ensure_space(len);
      cur_ -= len;
      size_ += static_cast<SizeT>(len);
    }
    return cur_;
  }

  // Returns nullptr if using the DefaultAllocator.
  Allocator *get_custom_allocator() { return allocator_; }

  // The current offset into the buffer.
  size_t offset() const { return cur_ - buf_; }

  // The total size of the vector (both the buffer and scratch parts).
  inline SizeT size() const { return size_; }

  // The size of the buffer part of the vector that is currently unused.
  SizeT unused_buffer_size() const { return static_cast<SizeT>(cur_ - scratch_); }

  // The size of the scratch part of the vector.
  SizeT scratch_size() const { return static_cast<SizeT>(scratch_ - buf_); }

  size_t capacity() const { return reserved_; }

  uint8_t *data() const {
    FLATBUFFERS_ASSERT(cur_);
    return cur_;
  }

  uint8_t *scratch_data() const {
    FLATBUFFERS_ASSERT(buf_);
    return buf_;
  }

  uint8_t *scratch_end() const {
    FLATBUFFERS_ASSERT(scratch_);
    return scratch_;
  }

  uint8_t *data_at(size_t offset) const { return buf_ + reserved_ - offset; }

  void push(const uint8_t *bytes, size_t num) {
    if (num > 0) { memcpy(make_space(num), bytes, num); }
  }

  // Specialized version of push() that avoids memcpy call for small data.
  template<typename T> void push_small(const T &little_endian_t) {
    make_space(sizeof(T));
    *reinterpret_cast<T *>(cur_) = little_endian_t;
  }

  template<typename T> void scratch_push_small(const T &t) {
    ensure_space(sizeof(T));
    *reinterpret_cast<T *>(scratch_) = t;
    scratch_ += sizeof(T);
  }

  // fill() is most frequently called with small byte counts (<= 4),
  // which is why we're using loops rather than calling memset.
  void fill(size_t zero_pad_bytes) {
    make_space(zero_pad_bytes);
    for (size_t i = 0; i < zero_pad_bytes; i++) cur_[i] = 0;
  }

  // Version for when we know the size is larger.
  // Precondition: zero_pad_bytes > 0
  void fill_big(size_t zero_pad_bytes) {
    memset(make_space(zero_pad_bytes), 0, zero_pad_bytes);
  }

  void pop(size_t bytes_to_remove) {
    cur_ += bytes_to_remove;
    size_ -= static_cast<SizeT>(bytes_to_remove);
  }

  void scratch_pop(size_t bytes_to_remove) { scratch_ -= bytes_to_remove; }

  void swap(vector_downward &other) {
    using std::swap;
    swap(allocator_, other.allocator_);
    swap(own_allocator_, other.own_allocator_);
    swap(initial_size_, other.initial_size_);
    swap(buffer_minalign_, other.buffer_minalign_);
    swap(reserved_, other.reserved_);
    swap(size_, other.size_);
    swap(max_size_, other.max_size_);
    swap(buf_, other.buf_);
    swap(cur_, other.cur_);
    swap(scratch_, other.scratch_);
  }

  void swap_allocator(vector_downward &other) {
    using std::swap;
    swap(allocator_, other.allocator_);
    swap(own_allocator_, other.own_allocator_);
  }

 private:
  // You shouldn't really be copying instances of this class.
  FLATBUFFERS_DELETE_FUNC(vector_downward(const vector_downward &));
  FLATBUFFERS_DELETE_FUNC(vector_downward &operator=(const vector_downward &));

  Allocator *allocator_;
  bool own_allocator_;
  size_t initial_size_;

  // The maximum size the vector can be.
  SizeT max_size_;
  size_t buffer_minalign_;
  size_t reserved_;
  SizeT size_;
  uint8_t *buf_;
  uint8_t *cur_;  // Points at location between empty (below) and used (above).
  uint8_t *scratch_;  // Points to the end of the scratchpad in use.

  void reallocate(size_t len) {
    auto old_reserved = reserved_;
    auto old_size = size();
    auto old_scratch_size = scratch_size();
    reserved_ +=
        (std::max)(len, old_reserved ? old_reserved / 2 : initial_size_);
    reserved_ = (reserved_ + buffer_minalign_ - 1) & ~(buffer_minalign_ - 1);
    if (buf_) {
      buf_ = ReallocateDownward(allocator_, buf_, old_reserved, reserved_,
                                old_size, old_scratch_size);
    } else {
      buf_ = Allocate(allocator_, reserved_);
    }
    cur_ = buf_ + reserved_ - old_size;
    scratch_ = buf_ + old_scratch_size;
  }
};

}  // namespace flatbuffers

#endif  // FLATBUFFERS_VECTOR_DOWNWARD_H_
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

- **a**: A class/struct defined in this file
- **vector_downward**: A class/struct defined in this file

### Functions and Methods

- **FLATBUFFERS_VECTOR_DOWNWARD_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cstdint`
- `flatbuffers/default_allocator.h`
- `flatbuffers/detached_buffer.h`
- `flatbuffers/base.h`
- `algorithm`

**Python Imports:**
- `higher`


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

