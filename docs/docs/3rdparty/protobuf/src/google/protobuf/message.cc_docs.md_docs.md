# Documentation for `docs/3rdparty/protobuf/src/google/protobuf/message.cc_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/protobuf/src/google/protobuf/message.cc_docs.md`
- **File Name**: `message.cc_docs.md`
- **File Size**: 17,822 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/protobuf/src/google/protobuf/message.cc_docs.md](../../../../../../docs/3rdparty/protobuf/src/google/protobuf/message.cc_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/protobuf/src/google/protobuf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/protobuf/src/google/protobuf/message.cc`

## File Metadata

- **Full Path**: `3rdparty/protobuf/src/google/protobuf/message.cc`
- **File Name**: `message.cc`
- **File Size**: 13,588 bytes
- **File Type**: .cc
- **Link to Source**: [3rdparty/protobuf/src/google/protobuf/message.cc](../../../../../3rdparty/protobuf/src/google/protobuf/message.cc)

## Purpose and Role

This file is located in the `3rdparty/protobuf/src/google/protobuf` directory and serves as part of the OpenCV library infrastructure.

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

#include <google/protobuf/message.h>

#include <iostream>
#include <stack>
#include <unordered_map>

#include <google/protobuf/stubs/casts.h>
#include <google/protobuf/stubs/logging.h>
#include <google/protobuf/stubs/common.h>
#include <google/protobuf/descriptor.pb.h>
#include <google/protobuf/parse_context.h>
#include <google/protobuf/reflection_internal.h>
#include <google/protobuf/io/coded_stream.h>
#include <google/protobuf/io/zero_copy_stream_impl.h>
#include <google/protobuf/descriptor.h>
#include <google/protobuf/generated_message_reflection.h>
#include <google/protobuf/generated_message_util.h>
#include <google/protobuf/map_field.h>
#include <google/protobuf/map_field_inl.h>
#include <google/protobuf/reflection_ops.h>
#include <google/protobuf/unknown_field_set.h>
#include <google/protobuf/wire_format.h>
#include <google/protobuf/wire_format_lite.h>
#include <google/protobuf/stubs/strutil.h>
#include <google/protobuf/stubs/map_util.h>
#include <google/protobuf/stubs/stl_util.h>
#include <google/protobuf/stubs/hash.h>

#include <google/protobuf/port_def.inc>

namespace google {
namespace protobuf {

namespace internal {

// TODO(gerbens) make this factorized better. This should not have to hop
// to reflection. Currently uses GeneratedMessageReflection and thus is
// defined in generated_message_reflection.cc
void RegisterFileLevelMetadata(const DescriptorTable* descriptor_table);

}  // namespace internal

using internal::ReflectionOps;
using internal::WireFormat;
using internal::WireFormatLite;

void Message::MergeFrom(const Message& from) {
  auto* class_to = GetClassData();
  auto* class_from = from.GetClassData();
  auto* merge_to_from = class_to ? class_to->merge_to_from : nullptr;
  if (class_to == nullptr || class_to != class_from) {
    merge_to_from = [](Message* to, const Message& from) {
      ReflectionOps::Merge(from, to);
    };
  }
  merge_to_from(this, from);
}

void Message::CheckTypeAndMergeFrom(const MessageLite& other) {
  MergeFrom(*down_cast<const Message*>(&other));
}

void Message::CopyFrom(const Message& from) {
  if (&from == this) return;

  auto* class_to = GetClassData();
  auto* class_from = from.GetClassData();
  auto* copy_to_from = class_to ? class_to->copy_to_from : nullptr;

  if (class_to == nullptr || class_to != class_from) {
    const Descriptor* descriptor = GetDescriptor();
    GOOGLE_CHECK_EQ(from.GetDescriptor(), descriptor)
        << ": Tried to copy from a message with a different type. "
           "to: "
        << descriptor->full_name()
        << ", "
           "from: "
        << from.GetDescriptor()->full_name();
    copy_to_from = [](Message* to, const Message& from) {
      ReflectionOps::Copy(from, to);
    };
  }
  copy_to_from(this, from);
}

void Message::CopyWithSizeCheck(Message* to, const Message& from) {
#ifndef NDEBUG
  size_t from_size = from.ByteSizeLong();
#endif
  to->Clear();
#ifndef NDEBUG
  GOOGLE_CHECK_EQ(from_size, from.ByteSizeLong())
      << "Source of CopyFrom changed when clearing target.  Either "
         "source is a nested message in target (not allowed), or "
         "another thread is modifying the source.";
#endif
  to->GetClassData()->merge_to_from(to, from);
}

std::string Message::GetTypeName() const {
  return GetDescriptor()->full_name();
}

void Message::Clear() { ReflectionOps::Clear(this); }

bool Message::IsInitialized() const {
  return ReflectionOps::IsInitialized(*this);
}

void Message::FindInitializationErrors(std::vector<std::string>* errors) const {
  return ReflectionOps::FindInitializationErrors(*this, "", errors);
}

std::string Message::InitializationErrorString() const {
  std::vector<std::string> errors;
  FindInitializationErrors(&errors);
  return Join(errors, ", ");
}

void Message::CheckInitialized() const {
  GOOGLE_CHECK(IsInitialized()) << "Message of type \"" << GetDescriptor()->full_name()
                         << "\" is missing required fields: "
                         << InitializationErrorString();
}

void Message::DiscardUnknownFields() {
  return ReflectionOps::DiscardUnknownFields(this);
}

const char* Message::_InternalParse(const char* ptr,
                                    internal::ParseContext* ctx) {
  return WireFormat::_InternalParse(this, ptr, ctx);
}

uint8_t* Message::_InternalSerialize(uint8_t* target,
                                     io::EpsCopyOutputStream* stream) const {
  return WireFormat::_InternalSerialize(*this, target, stream);
}

size_t Message::ByteSizeLong() const {
  size_t size = WireFormat::ByteSize(*this);
  SetCachedSize(internal::ToCachedSize(size));
  return size;
}

void Message::SetCachedSize(int /* size */) const {
  GOOGLE_LOG(FATAL) << "Message class \"" << GetDescriptor()->full_name()
             << "\" implements neither SetCachedSize() nor ByteSize().  "
                "Must implement one or the other.";
}

size_t Message::ComputeUnknownFieldsSize(
    size_t total_size, internal::CachedSize* cached_size) const {
  total_size += WireFormat::ComputeUnknownFieldsSize(
      _internal_metadata_.unknown_fields<UnknownFieldSet>(
          UnknownFieldSet::default_instance));
  cached_size->Set(internal::ToCachedSize(total_size));
  return total_size;
}

size_t Message::MaybeComputeUnknownFieldsSize(
    size_t total_size, internal::CachedSize* cached_size) const {
  if (PROTOBUF_PREDICT_FALSE(_internal_metadata_.have_unknown_fields())) {
    return ComputeUnknownFieldsSize(total_size, cached_size);
  }
  cached_size->Set(internal::ToCachedSize(total_size));
  return total_size;
}

size_t Message::SpaceUsedLong() const {
  return GetReflection()->SpaceUsedLong(*this);
}

uint64_t Message::GetInvariantPerBuild(uint64_t salt) {
  return salt;
}

// =============================================================================
// MessageFactory

MessageFactory::~MessageFactory() {}

namespace {


#define HASH_MAP std::unordered_map
#define STR_HASH_FXN hash<::google::protobuf::StringPiece>


class GeneratedMessageFactory final : public MessageFactory {
 public:
  static GeneratedMessageFactory* singleton();

  void RegisterFile(const google::protobuf::internal::DescriptorTable* table);
  void RegisterType(const Descriptor* descriptor, const Message* prototype);

  // implements MessageFactory ---------------------------------------
  const Message* GetPrototype(const Descriptor* type) override;

 private:
  // Only written at static init time, so does not require locking.
  HASH_MAP<StringPiece, const google::protobuf::internal::DescriptorTable*,
           STR_HASH_FXN>
      file_map_;

  internal::WrappedMutex mutex_;
  // Initialized lazily, so requires locking.
  std::unordered_map<const Descriptor*, const Message*> type_map_;
};

GeneratedMessageFactory* GeneratedMessageFactory::singleton() {
  static auto instance =
      internal::OnShutdownDelete(new GeneratedMessageFactory);
  return instance;
}

void GeneratedMessageFactory::RegisterFile(
    const google::protobuf::internal::DescriptorTable* table) {
  if (!InsertIfNotPresent(&file_map_, table->filename, table)) {
    GOOGLE_LOG(FATAL) << "File is already registered: " << table->filename;
  }
}

void GeneratedMessageFactory::RegisterType(const Descriptor* descriptor,
                                           const Message* prototype) {
  GOOGLE_DCHECK_EQ(descriptor->file()->pool(), DescriptorPool::generated_pool())
      << "Tried to register a non-generated type with the generated "
         "type registry.";

  // This should only be called as a result of calling a file registration
  // function during GetPrototype(), in which case we already have locked
  // the mutex.
  mutex_.AssertHeld();
  if (!InsertIfNotPresent(&type_map_, descriptor, prototype)) {
    GOOGLE_LOG(DFATAL) << "Type is already registered: " << descriptor->full_name();
  }
}


const Message* GeneratedMessageFactory::GetPrototype(const Descriptor* type) {
  {
    ReaderMutexLock lock(&mutex_);
    const Message* result = FindPtrOrNull(type_map_, type);
    if (result != nullptr) return result;
  }

  // If the type is not in the generated pool, then we can't possibly handle
  // it.
  if (type->file()->pool() != DescriptorPool::generated_pool()) return nullptr;

  // Apparently the file hasn't been registered yet.  Let's do that now.
  const internal::DescriptorTable* registration_data =
      FindPtrOrNull(file_map_, type->file()->name().c_str());
  if (registration_data == nullptr) {
    GOOGLE_LOG(DFATAL) << "File appears to be in generated pool but wasn't "
                   "registered: "
                << type->file()->name();
    return nullptr;
  }

  WriterMutexLock lock(&mutex_);

  // Check if another thread preempted us.
  const Message* result = FindPtrOrNull(type_map_, type);
  if (result == nullptr) {
    // Nope.  OK, register everything.
    internal::RegisterFileLevelMetadata(registration_data);
    // Should be here now.
    result = FindPtrOrNull(type_map_, type);
  }

  if (result == nullptr) {
    GOOGLE_LOG(DFATAL) << "Type appears to be in generated pool but wasn't "
                << "registered: " << type->full_name();
  }

  return result;
}

}  // namespace

MessageFactory* MessageFactory::generated_factory() {
  return GeneratedMessageFactory::singleton();
}

void MessageFactory::InternalRegisterGeneratedFile(
    const google::protobuf::internal::DescriptorTable* table) {
  GeneratedMessageFactory::singleton()->RegisterFile(table);
}

void MessageFactory::InternalRegisterGeneratedMessage(
    const Descriptor* descriptor, const Message* prototype) {
  GeneratedMessageFactory::singleton()->RegisterType(descriptor, prototype);
}


namespace {
template <typename T>
T* GetSingleton() {
  static T singleton;
  return &singleton;
}
}  // namespace

const internal::RepeatedFieldAccessor* Reflection::RepeatedFieldAccessor(
    const FieldDescriptor* field) const {
  GOOGLE_CHECK(field->is_repeated());
  switch (field->cpp_type()) {
#define HANDLE_PRIMITIVE_TYPE(TYPE, type) \
  case FieldDescriptor::CPPTYPE_##TYPE:   \
    return GetSingleton<internal::RepeatedFieldPrimitiveAccessor<type> >();
    HANDLE_PRIMITIVE_TYPE(INT32, int32_t)
    HANDLE_PRIMITIVE_TYPE(UINT32, uint32_t)
    HANDLE_PRIMITIVE_TYPE(INT64, int64_t)
    HANDLE_PRIMITIVE_TYPE(UINT64, uint64_t)
    HANDLE_PRIMITIVE_TYPE(FLOAT, float)
    HANDLE_PRIMITIVE_TYPE(DOUBLE, double)
    HANDLE_PRIMITIVE_TYPE(BOOL, bool)
    HANDLE_PRIMITIVE_TYPE(ENUM, int32_t)
#undef HANDLE_PRIMITIVE_TYPE
    case FieldDescriptor::CPPTYPE_STRING:
      switch (field->options().ctype()) {
        default:
        case FieldOptions::STRING:
          return GetSingleton<internal::RepeatedPtrFieldStringAccessor>();
      }
      break;
    case FieldDescriptor::CPPTYPE_MESSAGE:
      if (field->is_map()) {
        return GetSingleton<internal::MapFieldAccessor>();
      } else {
        return GetSingleton<internal::RepeatedPtrFieldMessageAccessor>();
      }
  }
  GOOGLE_LOG(FATAL) << "Should not reach here.";
  return nullptr;
}

namespace internal {
template <>
#if defined(_MSC_VER) && (_MSC_VER >= 1800)
// Note: force noinline to workaround MSVC compiler bug with /Zc:inline, issue
// #240
PROTOBUF_NOINLINE
#endif
    Message*
    GenericTypeHandler<Message>::NewFromPrototype(const Message* prototype,
                                                  Arena* arena) {
  return prototype->New(arena);
}
template <>
#if defined(_MSC_VER) && (_MSC_VER >= 1800)
// Note: force noinline to workaround MSVC compiler bug with /Zc:inline, issue
// #240
PROTOBUF_NOINLINE
#endif
    Arena*
    GenericTypeHandler<Message>::GetOwningArena(Message* value) {
  return value->GetOwningArena();
}
}  // namespace internal

}  // namespace protobuf
}  // namespace google

#include <google/protobuf/port_undef.inc>
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

- **GeneratedMessageFactory**: A class/struct defined in this file

### Functions and Methods

- **HANDLE_PRIMITIVE_TYPE()**: A function/method defined in this file
- **NDEBUG()**: A function/method defined in this file
- **during()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `google/protobuf/io/zero_copy_stream_impl.h`
- `google/protobuf/map_field.h`
- `google/protobuf/map_field_inl.h`
- `google/protobuf/io/coded_stream.h`
- `iostream`
- `google/protobuf/stubs/casts.h`
- `google/protobuf/parse_context.h`
- `google/protobuf/reflection_ops.h`
- `google/protobuf/descriptor.pb.h`
- `google/protobuf/generated_message_reflection.h`
- `google/protobuf/message.h`
- `google/protobuf/wire_format_lite.h`
- `google/protobuf/stubs/strutil.h`
- `google/protobuf/reflection_internal.h`
- `google/protobuf/stubs/common.h`
- `google/protobuf/stubs/logging.h`
- `google/protobuf/unknown_field_set.h`
- `google/protobuf/stubs/hash.h`
- `unordered_map`
- `google/protobuf/stubs/stl_util.h`
- `google/protobuf/generated_message_util.h`
- `stack`
- `google/protobuf/descriptor.h`
- `google/protobuf/port_def.inc`
- `google/protobuf/stubs/map_util.h`
- `google/protobuf/wire_format.h`
- `google/protobuf/port_undef.inc`

**Python Imports:**
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

