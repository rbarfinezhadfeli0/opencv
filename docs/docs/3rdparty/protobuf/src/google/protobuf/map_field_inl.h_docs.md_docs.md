# Documentation for `docs/3rdparty/protobuf/src/google/protobuf/map_field_inl.h_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/protobuf/src/google/protobuf/map_field_inl.h_docs.md`
- **File Name**: `map_field_inl.h_docs.md`
- **File Size**: 17,840 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/protobuf/src/google/protobuf/map_field_inl.h_docs.md](../../../../../../docs/3rdparty/protobuf/src/google/protobuf/map_field_inl.h_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/protobuf/src/google/protobuf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/protobuf/src/google/protobuf/map_field_inl.h`

## File Metadata

- **Full Path**: `3rdparty/protobuf/src/google/protobuf/map_field_inl.h`
- **File Name**: `map_field_inl.h`
- **File Size**: 14,537 bytes
- **File Type**: .h
- **Link to Source**: [3rdparty/protobuf/src/google/protobuf/map_field_inl.h](../../../../../3rdparty/protobuf/src/google/protobuf/map_field_inl.h)

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

#ifndef GOOGLE_PROTOBUF_MAP_FIELD_INL_H__
#define GOOGLE_PROTOBUF_MAP_FIELD_INL_H__

#include <memory>

#include <google/protobuf/stubs/casts.h>
#include <google/protobuf/map.h>
#include <google/protobuf/map_field.h>
#include <google/protobuf/map_type_handler.h>

#ifdef SWIG
#error "You cannot SWIG proto headers"
#endif

namespace google {
namespace protobuf {
namespace internal {
// UnwrapMapKey template
template <typename T>
T UnwrapMapKey(const MapKey& map_key);
template <>
inline int32_t UnwrapMapKey<int32_t>(const MapKey& map_key) {
  return map_key.GetInt32Value();
}
template <>
inline uint32_t UnwrapMapKey<uint32_t>(const MapKey& map_key) {
  return map_key.GetUInt32Value();
}
template <>
inline int64_t UnwrapMapKey<int64_t>(const MapKey& map_key) {
  return map_key.GetInt64Value();
}
template <>
inline uint64_t UnwrapMapKey<uint64_t>(const MapKey& map_key) {
  return map_key.GetUInt64Value();
}
template <>
inline bool UnwrapMapKey<bool>(const MapKey& map_key) {
  return map_key.GetBoolValue();
}
template <>
inline std::string UnwrapMapKey<std::string>(const MapKey& map_key) {
  return map_key.GetStringValue();
}

// SetMapKey template
template <typename T>
inline void SetMapKey(MapKey* map_key, const T& value);
template <>
inline void SetMapKey<int32_t>(MapKey* map_key, const int32_t& value) {
  map_key->SetInt32Value(value);
}
template <>
inline void SetMapKey<uint32_t>(MapKey* map_key, const uint32_t& value) {
  map_key->SetUInt32Value(value);
}
template <>
inline void SetMapKey<int64_t>(MapKey* map_key, const int64_t& value) {
  map_key->SetInt64Value(value);
}
template <>
inline void SetMapKey<uint64_t>(MapKey* map_key, const uint64_t& value) {
  map_key->SetUInt64Value(value);
}
template <>
inline void SetMapKey<bool>(MapKey* map_key, const bool& value) {
  map_key->SetBoolValue(value);
}
template <>
inline void SetMapKey<std::string>(MapKey* map_key, const std::string& value) {
  map_key->SetStringValue(value);
}

// ------------------------TypeDefinedMapFieldBase---------------
template <typename Key, typename T>
typename Map<Key, T>::const_iterator&
TypeDefinedMapFieldBase<Key, T>::InternalGetIterator(
    const MapIterator* map_iter) const {
  return *reinterpret_cast<typename Map<Key, T>::const_iterator*>(
      map_iter->iter_);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::MapBegin(MapIterator* map_iter) const {
  InternalGetIterator(map_iter) = GetMap().begin();
  SetMapIteratorValue(map_iter);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::MapEnd(MapIterator* map_iter) const {
  InternalGetIterator(map_iter) = GetMap().end();
}

template <typename Key, typename T>
bool TypeDefinedMapFieldBase<Key, T>::EqualIterator(
    const MapIterator& a, const MapIterator& b) const {
  return InternalGetIterator(&a) == InternalGetIterator(&b);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::IncreaseIterator(
    MapIterator* map_iter) const {
  ++InternalGetIterator(map_iter);
  SetMapIteratorValue(map_iter);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::InitializeIterator(
    MapIterator* map_iter) const {
  map_iter->iter_ = new typename Map<Key, T>::const_iterator;
  GOOGLE_CHECK(map_iter->iter_ != nullptr);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::DeleteIterator(
    MapIterator* map_iter) const {
  delete reinterpret_cast<typename Map<Key, T>::const_iterator*>(
      map_iter->iter_);
}

template <typename Key, typename T>
void TypeDefinedMapFieldBase<Key, T>::CopyIterator(
    MapIterator* this_iter, const MapIterator& that_iter) const {
  InternalGetIterator(this_iter) = InternalGetIterator(&that_iter);
  this_iter->key_.SetType(that_iter.key_.type());
  // MapValueRef::type() fails when containing data is null. However, if
  // this_iter points to MapEnd, data can be null.
  this_iter->value_.SetType(
      static_cast<FieldDescriptor::CppType>(that_iter.value_.type_));
  SetMapIteratorValue(this_iter);
}

// ----------------------------------------------------------------------

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
int MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::size() const {
  MapFieldBase::SyncMapWithRepeatedField();
  return static_cast<int>(impl_.GetMap().size());
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::Clear() {
  if (this->MapFieldBase::repeated_field_ != nullptr) {
    RepeatedPtrField<EntryType>* repeated_field =
        reinterpret_cast<RepeatedPtrField<EntryType>*>(
            this->MapFieldBase::repeated_field_);
    repeated_field->Clear();
  }

  impl_.MutableMap()->clear();
  // Data in map and repeated field are both empty, but we can't set status
  // CLEAN. Because clear is a generated API, we cannot invalidate previous
  // reference to map.
  MapFieldBase::SetMapDirty();
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType,
              kValueFieldType>::SetMapIteratorValue(MapIterator* map_iter)
    const {
  const Map<Key, T>& map = impl_.GetMap();
  typename Map<Key, T>::const_iterator iter =
      TypeDefinedMapFieldBase<Key, T>::InternalGetIterator(map_iter);
  if (iter == map.end()) return;
  SetMapKey(&map_iter->key_, iter->first);
  map_iter->value_.SetValue(&iter->second);
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
bool MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::ContainsMapKey(
    const MapKey& map_key) const {
  const Map<Key, T>& map = impl_.GetMap();
  const Key& key = UnwrapMapKey<Key>(map_key);
  typename Map<Key, T>::const_iterator iter = map.find(key);
  return iter != map.end();
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
bool MapField<Derived, Key, T, kKeyFieldType,
              kValueFieldType>::InsertOrLookupMapValue(const MapKey& map_key,
                                                       MapValueRef* val) {
  // Always use mutable map because users may change the map value by
  // MapValueRef.
  Map<Key, T>* map = MutableMap();
  const Key& key = UnwrapMapKey<Key>(map_key);
  typename Map<Key, T>::iterator iter = map->find(key);
  if (map->end() == iter) {
    val->SetValue(&((*map)[key]));
    return true;
  }
  // Key is already in the map. Make sure (*map)[key] is not called.
  // [] may reorder the map and iterators.
  val->SetValue(&(iter->second));
  return false;
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
bool MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::LookupMapValue(
    const MapKey& map_key, MapValueConstRef* val) const {
  const Map<Key, T>& map = GetMap();
  const Key& key = UnwrapMapKey<Key>(map_key);
  typename Map<Key, T>::const_iterator iter = map.find(key);
  if (map.end() == iter) {
    return false;
  }
  // Key is already in the map. Make sure (*map)[key] is not called.
  // [] may reorder the map and iterators.
  val->SetValue(&(iter->second));
  return true;
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
bool MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::DeleteMapValue(
    const MapKey& map_key) {
  const Key& key = UnwrapMapKey<Key>(map_key);
  return MutableMap()->erase(key);
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::MergeFrom(
    const MapFieldBase& other) {
  MapFieldBase::SyncMapWithRepeatedField();
  const MapField& other_field = static_cast<const MapField&>(other);
  other_field.SyncMapWithRepeatedField();
  impl_.MergeFrom(other_field.impl_);
  MapFieldBase::SetMapDirty();
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::Swap(
    MapFieldBase* other) {
  MapFieldBase::Swap(other);
  MapField* other_field = down_cast<MapField*>(other);
  impl_.Swap(&other_field->impl_);
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType,
              kValueFieldType>::UnsafeShallowSwap(MapFieldBase* other) {
  InternalSwap(down_cast<MapField*>(other));
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType, kValueFieldType>::InternalSwap(
    MapField* other) {
  MapFieldBase::InternalSwap(other);
  impl_.InternalSwap(&other->impl_);
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType,
              kValueFieldType>::SyncRepeatedFieldWithMapNoLock() const {
  if (this->MapFieldBase::repeated_field_ == nullptr) {
    this->MapFieldBase::repeated_field_ =
        Arena::CreateMessage<RepeatedPtrField<Message> >(
            this->MapFieldBase::arena_);
  }
  const Map<Key, T>& map = impl_.GetMap();
  RepeatedPtrField<EntryType>* repeated_field =
      reinterpret_cast<RepeatedPtrField<EntryType>*>(
          this->MapFieldBase::repeated_field_);

  repeated_field->Clear();

  // The only way we can get at this point is through reflection and the
  // only way we can get the reflection object is by having called GetReflection
  // on the encompassing field. So that type must have existed and hence we
  // know that this MapEntry default_type has also already been constructed.
  // So it's safe to just call internal_default_instance().
  const Message* default_entry = Derived::internal_default_instance();
  for (typename Map<Key, T>::const_iterator it = map.begin(); it != map.end();
       ++it) {
    EntryType* new_entry =
        down_cast<EntryType*>(default_entry->New(this->MapFieldBase::arena_));
    repeated_field->AddAllocated(new_entry);
    (*new_entry->mutable_key()) = it->first;
    (*new_entry->mutable_value()) = it->second;
  }
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
void MapField<Derived, Key, T, kKeyFieldType,
              kValueFieldType>::SyncMapWithRepeatedFieldNoLock() const {
  Map<Key, T>* map = const_cast<MapField*>(this)->impl_.MutableMap();
  RepeatedPtrField<EntryType>* repeated_field =
      reinterpret_cast<RepeatedPtrField<EntryType>*>(
          this->MapFieldBase::repeated_field_);
  GOOGLE_CHECK(this->MapFieldBase::repeated_field_ != nullptr);
  map->clear();
  for (typename RepeatedPtrField<EntryType>::iterator it =
           repeated_field->begin();
       it != repeated_field->end(); ++it) {
    // Cast is needed because Map's api and internal storage is different when
    // value is enum. For enum, we cannot cast an int to enum. Thus, we have to
    // copy value. For other types, they have same exposed api type and internal
    // stored type. We should not introduce value copy for them. We achieve this
    // by casting to value for enum while casting to reference for other types.
    (*map)[it->key()] = static_cast<CastValueType>(it->value());
  }
}

template <typename Derived, typename Key, typename T,
          WireFormatLite::FieldType kKeyFieldType,
          WireFormatLite::FieldType kValueFieldType>
size_t MapField<Derived, Key, T, kKeyFieldType,
                kValueFieldType>::SpaceUsedExcludingSelfNoLock() const {
  size_t size = 0;
  if (this->MapFieldBase::repeated_field_ != nullptr) {
    size += this->MapFieldBase::repeated_field_->SpaceUsedExcludingSelfLong();
  }
  size += impl_.GetMap().SpaceUsedExcludingSelfLong();

  return size;
}
}  // namespace internal
}  // namespace protobuf
}  // namespace google

#endif  // GOOGLE_PROTOBUF_MAP_FIELD_INL_H__
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

### Functions and Methods

- **GOOGLE_PROTOBUF_MAP_FIELD_INL_H__()**: A function/method defined in this file
- **SWIG()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `google/protobuf/map_field.h`
- `memory`
- `google/protobuf/stubs/casts.h`
- `google/protobuf/map_type_handler.h`
- `google/protobuf/map.h`


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

