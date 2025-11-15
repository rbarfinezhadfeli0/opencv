# Documentation for `3rdparty/protobuf/src/google/protobuf/descriptor.cc`

## File Metadata

- **Full Path**: `3rdparty/protobuf/src/google/protobuf/descriptor.cc`
- **File Name**: `descriptor.cc`
- **File Size**: 308,712 bytes
- **File Type**: .cc
- **Link to Source**: [3rdparty/protobuf/src/google/protobuf/descriptor.cc](../../../../../3rdparty/protobuf/src/google/protobuf/descriptor.cc)

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

#include <google/protobuf/descriptor.h>

#include <algorithm>
#include <array>
#include <functional>
#include <limits>
#include <map>
#include <memory>
#include <set>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>

#include <google/protobuf/stubs/common.h>
#include <google/protobuf/stubs/logging.h>
#include <google/protobuf/stubs/stringprintf.h>
#include <google/protobuf/stubs/strutil.h>
#include <google/protobuf/any.h>
#include <google/protobuf/descriptor.pb.h>
#include <google/protobuf/stubs/once.h>
#include <google/protobuf/io/coded_stream.h>
#include <google/protobuf/io/tokenizer.h>
#include <google/protobuf/io/zero_copy_stream_impl.h>
#include <google/protobuf/descriptor_database.h>
#include <google/protobuf/dynamic_message.h>
#include <google/protobuf/generated_message_util.h>
#include <google/protobuf/text_format.h>
#include <google/protobuf/unknown_field_set.h>
#include <google/protobuf/wire_format.h>
#include <google/protobuf/stubs/casts.h>
#include <google/protobuf/stubs/substitute.h>
#include <google/protobuf/io/strtod.h>
#include <google/protobuf/stubs/map_util.h>
#include <google/protobuf/stubs/stl_util.h>
#include <google/protobuf/stubs/hash.h>

#undef PACKAGE  // autoheader #defines this.  :(


#include <google/protobuf/port_def.inc>

namespace google {
namespace protobuf {

class Symbol {
 public:
  enum Type {
    NULL_SYMBOL,
    MESSAGE,
    FIELD,
    ONEOF,
    ENUM,
    ENUM_VALUE,
    ENUM_VALUE_OTHER_PARENT,
    SERVICE,
    METHOD,
    PACKAGE,
    QUERY_KEY
  };

  Symbol() : ptr_(nullptr) {}

  // Every object we store derives from internal::SymbolBase, where we store the
  // symbol type enum.
  // Storing in the object can be done without using more space in most cases,
  // while storing it in the Symbol type would require 8 bytes.
#define DEFINE_MEMBERS(TYPE, TYPE_CONSTANT, FIELD)                             \
  explicit Symbol(TYPE* value) : ptr_(value) {                                 \
    value->symbol_type_ = TYPE_CONSTANT;                                       \
  }                                                                            \
  const TYPE* FIELD() const {                                                  \
    return type() == TYPE_CONSTANT ? static_cast<const TYPE*>(ptr_) : nullptr; \
  }

  DEFINE_MEMBERS(Descriptor, MESSAGE, descriptor)
  DEFINE_MEMBERS(FieldDescriptor, FIELD, field_descriptor)
  DEFINE_MEMBERS(OneofDescriptor, ONEOF, oneof_descriptor)
  DEFINE_MEMBERS(EnumDescriptor, ENUM, enum_descriptor)
  DEFINE_MEMBERS(ServiceDescriptor, SERVICE, service_descriptor)
  DEFINE_MEMBERS(MethodDescriptor, METHOD, method_descriptor)

  // We use a special node for FileDescriptor.
  // It is potentially added to the table with multiple different names, so we
  // need a separate place to put the name.
  struct Package : internal::SymbolBase {
    const std::string* name;
    const FileDescriptor* file;
  };
  DEFINE_MEMBERS(Package, PACKAGE, package_file_descriptor)

  // Enum values have two different parents.
  // We use two different identitied for the same object to determine the two
  // different insertions in the map.
  static Symbol EnumValue(EnumValueDescriptor* value, int n) {
    Symbol s;
    internal::SymbolBase* ptr;
    if (n == 0) {
      ptr = static_cast<internal::SymbolBaseN<0>*>(value);
      ptr->symbol_type_ = ENUM_VALUE;
    } else {
      ptr = static_cast<internal::SymbolBaseN<1>*>(value);
      ptr->symbol_type_ = ENUM_VALUE_OTHER_PARENT;
    }
    s.ptr_ = ptr;
    return s;
  }

  const EnumValueDescriptor* enum_value_descriptor() const {
    return type() == ENUM_VALUE
               ? static_cast<const EnumValueDescriptor*>(
                     static_cast<const internal::SymbolBaseN<0>*>(ptr_))
           : type() == ENUM_VALUE_OTHER_PARENT
               ? static_cast<const EnumValueDescriptor*>(
                     static_cast<const internal::SymbolBaseN<1>*>(ptr_))
               : nullptr;
  }

  // Not a real symbol.
  // Only used for heterogeneous lookups and never actually inserted in the
  // tables.
  struct QueryKey : internal::SymbolBase {
    StringPiece name;
    const void* parent;
    int field_number;
  };
  DEFINE_MEMBERS(QueryKey, QUERY_KEY, query_key);
#undef DEFINE_MEMBERS

  Type type() const {
    return ptr_ == nullptr ? NULL_SYMBOL
                           : static_cast<Type>(ptr_->symbol_type_);
  }
  bool IsNull() const { return type() == NULL_SYMBOL; }
  bool IsType() const { return type() == MESSAGE || type() == ENUM; }
  bool IsAggregate() const {
    return type() == MESSAGE || type() == PACKAGE || type() == ENUM ||
           type() == SERVICE;
  }

  const FileDescriptor* GetFile() const {
    switch (type()) {
      case MESSAGE:
        return descriptor()->file();
      case FIELD:
        return field_descriptor()->file();
      case ONEOF:
        return oneof_descriptor()->containing_type()->file();
      case ENUM:
        return enum_descriptor()->file();
      case ENUM_VALUE:
        return enum_value_descriptor()->type()->file();
      case SERVICE:
        return service_descriptor()->file();
      case METHOD:
        return method_descriptor()->service()->file();
      case PACKAGE:
        return package_file_descriptor()->file;
      default:
        return nullptr;
    }
  }

  StringPiece full_name() const {
    switch (type()) {
      case MESSAGE:
        return descriptor()->full_name();
      case FIELD:
        return field_descriptor()->full_name();
      case ONEOF:
        return oneof_descriptor()->full_name();
      case ENUM:
        return enum_descriptor()->full_name();
      case ENUM_VALUE:
        return enum_value_descriptor()->full_name();
      case SERVICE:
        return service_descriptor()->full_name();
      case METHOD:
        return method_descriptor()->full_name();
      case PACKAGE:
        return *package_file_descriptor()->name;
      case QUERY_KEY:
        return query_key()->name;
      default:
        GOOGLE_CHECK(false);
    }
    return "";
  }

  std::pair<const void*, StringPiece> parent_name_key() const {
    const auto or_file = [&](const void* p) { return p ? p : GetFile(); };
    switch (type()) {
      case MESSAGE:
        return {or_file(descriptor()->containing_type()), descriptor()->name()};
      case FIELD: {
        auto* field = field_descriptor();
        return {or_file(field->is_extension() ? field->extension_scope()
                                              : field->containing_type()),
                field->name()};
      }
      case ONEOF:
        return {oneof_descriptor()->containing_type(),
                oneof_descriptor()->name()};
      case ENUM:
        return {or_file(enum_descriptor()->containing_type()),
                enum_descriptor()->name()};
      case ENUM_VALUE:
        return {or_file(enum_value_descriptor()->type()->containing_type()),
                enum_value_descriptor()->name()};
      case ENUM_VALUE_OTHER_PARENT:
        return {enum_value_descriptor()->type(),
                enum_value_descriptor()->name()};
      case SERVICE:
        return {GetFile(), service_descriptor()->name()};
      case METHOD:
        return {method_descriptor()->service(), method_descriptor()->name()};
      case QUERY_KEY:
        return {query_key()->parent, query_key()->name};
      default:
        GOOGLE_CHECK(false);
    }
    return {};
  }

  std::pair<const void*, int> parent_number_key() const {
    switch (type()) {
      case FIELD:
        return {field_descriptor()->containing_type(),
                field_descriptor()->number()};
      case ENUM_VALUE:
        return {enum_value_descriptor()->type(),
                enum_value_descriptor()->number()};
      case QUERY_KEY:
        return {query_key()->parent, query_key()->field_number};
      default:
        GOOGLE_CHECK(false);
    }
    return {};
  }

 private:
  const internal::SymbolBase* ptr_;
};

const FieldDescriptor::CppType
    FieldDescriptor::kTypeToCppTypeMap[MAX_TYPE + 1] = {
        static_cast<CppType>(0),  // 0 is reserved for errors

        CPPTYPE_DOUBLE,   // TYPE_DOUBLE
        CPPTYPE_FLOAT,    // TYPE_FLOAT
        CPPTYPE_INT64,    // TYPE_INT64
        CPPTYPE_UINT64,   // TYPE_UINT64
        CPPTYPE_INT32,    // TYPE_INT32
        CPPTYPE_UINT64,   // TYPE_FIXED64
        CPPTYPE_UINT32,   // TYPE_FIXED32
        CPPTYPE_BOOL,     // TYPE_BOOL
        CPPTYPE_STRING,   // TYPE_STRING
        CPPTYPE_MESSAGE,  // TYPE_GROUP
        CPPTYPE_MESSAGE,  // TYPE_MESSAGE
        CPPTYPE_STRING,   // TYPE_BYTES
        CPPTYPE_UINT32,   // TYPE_UINT32
        CPPTYPE_ENUM,     // TYPE_ENUM
        CPPTYPE_INT32,    // TYPE_SFIXED32
        CPPTYPE_INT64,    // TYPE_SFIXED64
        CPPTYPE_INT32,    // TYPE_SINT32
        CPPTYPE_INT64,    // TYPE_SINT64
};

const char* const FieldDescriptor::kTypeToName[MAX_TYPE + 1] = {
    "ERROR",  // 0 is reserved for errors

    "double",    // TYPE_DOUBLE
    "float",     // TYPE_FLOAT
    "int64",     // TYPE_INT64
    "uint64",    // TYPE_UINT64
    "int32",     // TYPE_INT32
    "fixed64",   // TYPE_FIXED64
    "fixed32",   // TYPE_FIXED32
    "bool",      // TYPE_BOOL
    "string",    // TYPE_STRING
    "group",     // TYPE_GROUP
    "message",   // TYPE_MESSAGE
    "bytes",     // TYPE_BYTES
    "uint32",    // TYPE_UINT32
    "enum",      // TYPE_ENUM
    "sfixed32",  // TYPE_SFIXED32
    "sfixed64",  // TYPE_SFIXED64
    "sint32",    // TYPE_SINT32
    "sint64",    // TYPE_SINT64
};

const char* const FieldDescriptor::kCppTypeToName[MAX_CPPTYPE + 1] = {
    "ERROR",  // 0 is reserved for errors

    "int32",    // CPPTYPE_INT32
    "int64",    // CPPTYPE_INT64
    "uint32",   // CPPTYPE_UINT32
    "uint64",   // CPPTYPE_UINT64
    "double",   // CPPTYPE_DOUBLE
    "float",    // CPPTYPE_FLOAT
    "bool",     // CPPTYPE_BOOL
    "enum",     // CPPTYPE_ENUM
    "string",   // CPPTYPE_STRING
    "message",  // CPPTYPE_MESSAGE
};

const char* const FieldDescriptor::kLabelToName[MAX_LABEL + 1] = {
    "ERROR",  // 0 is reserved for errors

    "optional",  // LABEL_OPTIONAL
    "required",  // LABEL_REQUIRED
    "repeated",  // LABEL_REPEATED
};

const char* FileDescriptor::SyntaxName(FileDescriptor::Syntax syntax) {
  switch (syntax) {
    case SYNTAX_PROTO2:
      return "proto2";
    case SYNTAX_PROTO3:
      return "proto3";
    case SYNTAX_UNKNOWN:
      return "unknown";
  }
  GOOGLE_LOG(FATAL) << "can't reach here.";
  return nullptr;
}

static const char* const kNonLinkedWeakMessageReplacementName = "google.protobuf.Empty";

#if !defined(_MSC_VER) || (_MSC_VER >= 1900 && _MSC_VER < 1912)
const int FieldDescriptor::kMaxNumber;
const int FieldDescriptor::kFirstReservedNumber;
const int FieldDescriptor::kLastReservedNumber;
#endif

namespace {

// Note:  I distrust ctype.h due to locales.
char ToUpper(char ch) {
  return (ch >= 'a' && ch <= 'z') ? (ch - 'a' + 'A') : ch;
}

char ToLower(char ch) {
  return (ch >= 'A' && ch <= 'Z') ? (ch - 'A' + 'a') : ch;
}

std::string ToCamelCase(const std::string& input, bool lower_first) {
  bool capitalize_next = !lower_first;
  std::string result;
  result.reserve(input.size());

  for (char character : input) {
    if (character == '_') {
      capitalize_next = true;
    } else if (capitalize_next) {
      result.push_back(ToUpper(character));
      capitalize_next = false;
    } else {
      result.push_back(character);
    }
  }

  // Lower-case the first letter.
  if (lower_first && !result.empty()) {
    result[0] = ToLower(result[0]);
  }

  return result;
}

std::string ToJsonName(const std::string& input) {
  bool capitalize_next = false;
  std::string result;
  result.reserve(input.size());

  for (char character : input) {
    if (character == '_') {
      capitalize_next = true;
    } else if (capitalize_next) {
      result.push_back(ToUpper(character));
      capitalize_next = false;
    } else {
      result.push_back(character);
    }
  }

  return result;
}

std::string EnumValueToPascalCase(const std::string& input) {
  bool next_upper = true;
  std::string result;
  result.reserve(input.size());

  for (char character : input) {
    if (character == '_') {
      next_upper = true;
    } else {
      if (next_upper) {
        result.push_back(ToUpper(character));
      } else {
        result.push_back(ToLower(character));
      }
      next_upper = false;
    }
  }

  return result;
}

// Class to remove an enum prefix from enum values.
class PrefixRemover {
 public:
  PrefixRemover(StringPiece prefix) {
    // Strip underscores and lower-case the prefix.
    for (char character : prefix) {
      if (character != '_') {
        prefix_ += ascii_tolower(character);
      }
    }
  }

  // Tries to remove the enum prefix from this enum value.
  // If this is not possible, returns the input verbatim.
  std::string MaybeRemove(StringPiece str) {
    // We can't just lowercase and strip str and look for a prefix.
    // We need to properly recognize the difference between:
    //
    //   enum Foo {
    //     FOO_BAR_BAZ = 0;
    //     FOO_BARBAZ = 1;
    //   }
    //
    // This is acceptable (though perhaps not advisable) because even when
    // we PascalCase, these two will still be distinct (BarBaz vs. Barbaz).
    size_t i, j;

    // Skip past prefix_ in str if we can.
    for (i = 0, j = 0; i < str.size() && j < prefix_.size(); i++) {
      if (str[i] == '_') {
        continue;
      }

      if (ascii_tolower(str[i]) != prefix_[j++]) {
        return std::string(str);
      }
    }

    // If we didn't make it through the prefix, we've failed to strip the
    // prefix.
    if (j < prefix_.size()) {
      return std::string(str);
    }

    // Skip underscores between prefix and further characters.
    while (i < str.size() && str[i] == '_') {
      i++;
    }

    // Enum label can't be the empty string.
    if (i == str.size()) {
      return std::string(str);
    }

    // We successfully stripped the prefix.
    str.remove_prefix(i);
    return std::string(str);
  }

 private:
  std::string prefix_;
};

// A DescriptorPool contains a bunch of hash-maps to implement the
// various Find*By*() methods.  Since hashtable lookups are O(1), it's
// most efficient to construct a fixed set of large hash-maps used by
// all objects in the pool rather than construct one or more small
// hash-maps for each object.
//
// The keys to these hash-maps are (parent, name) or (parent, number) pairs.

typedef std::pair<const void*, StringPiece> PointerStringPair;

typedef std::pair<const Descriptor*, int> DescriptorIntPair;

#define HASH_MAP std::unordered_map
#define HASH_SET std::unordered_set
#define HASH_FXN hash

template <typename PairType>
struct PointerIntegerPairHash {
  size_t operator()(const PairType& p) const {
    static const size_t prime1 = 16777499;
    static const size_t prime2 = 16777619;
    return reinterpret_cast<size_t>(p.first) * prime1 ^
           static_cast<size_t>(p.second) * prime2;
  }

#ifdef _MSC_VER
  // Used only by MSVC and platforms where hash_map is not available.
  static const size_t bucket_size = 4;
  static const size_t min_buckets = 8;
#endif
  inline bool operator()(const PairType& a, const PairType& b) const {
    return a < b;
  }
};

struct PointerStringPairHash {
  size_t operator()(const PointerStringPair& p) const {
    static const size_t prime = 16777619;
    hash<StringPiece> string_hash;
    return reinterpret_cast<size_t>(p.first) * prime ^
           static_cast<size_t>(string_hash(p.second));
  }

#ifdef _MSC_VER
  // Used only by MSVC and platforms where hash_map is not available.
  static const size_t bucket_size = 4;
  static const size_t min_buckets = 8;
#endif
  inline bool operator()(const PointerStringPair& a,
                         const PointerStringPair& b) const {
    return a < b;
  }
};


const Symbol kNullSymbol;

struct SymbolByFullNameHash {
  size_t operator()(Symbol s) const {
    return HASH_FXN<StringPiece>{}(s.full_name());
  }
};
struct SymbolByFullNameEq {
  bool operator()(Symbol a, Symbol b) const {
    return a.full_name() == b.full_name();
  }
};
using SymbolsByNameSet =
    HASH_SET<Symbol, SymbolByFullNameHash, SymbolByFullNameEq>;

struct SymbolByParentHash {
  size_t operator()(Symbol s) const {
    return PointerStringPairHash{}(s.parent_name_key());
  }
};
struct SymbolByParentEq {
  bool operator()(Symbol a, Symbol b) const {
    return a.parent_name_key() == b.parent_name_key();
  }
};
using SymbolsByParentSet =
    HASH_SET<Symbol, SymbolByParentHash, SymbolByParentEq>;

typedef HASH_MAP<StringPiece, const FileDescriptor*,
                 HASH_FXN<StringPiece>>
    FilesByNameMap;

typedef HASH_MAP<PointerStringPair, const FieldDescriptor*,
                 PointerStringPairHash>
    FieldsByNameMap;

struct FieldsByNumberHash {
  size_t operator()(Symbol s) const {
    return PointerIntegerPairHash<std::pair<const void*, int>>{}(
        s.parent_number_key());
  }
};
struct FieldsByNumberEq {
  bool operator()(Symbol a, Symbol b) const {
    return a.parent_number_key() == b.parent_number_key();
  }
};
using FieldsByNumberSet =
    HASH_SET<Symbol, FieldsByNumberHash, FieldsByNumberEq>;
using EnumValuesByNumberSet = FieldsByNumberSet;

// This is a map rather than a hash-map, since we use it to iterate
// through all the extensions that extend a given Descriptor, and an
// ordered data structure that implements lower_bound is convenient
// for that.
typedef std::map<DescriptorIntPair, const FieldDescriptor*>
    ExtensionsGroupedByDescriptorMap;
typedef HASH_MAP<std::string, const SourceCodeInfo_Location*>
    LocationsByPathMap;

std::set<std::string>* NewAllowedProto3Extendee() {
  auto allowed_proto3_extendees = new std::set<std::string>;
  const char* kOptionNames[] = {
      "FileOptions",      "MessageOptions", "FieldOptions",  "EnumOptions",
      "EnumValueOptions", "ServiceOptions", "MethodOptions", "OneofOptions"};
  for (const char* option_name : kOptionNames) {
    // descriptor.proto has a different package name in opensource. We allow
    // both so the opensource protocol compiler can also compile internal
    // proto3 files with custom options. See: b/27567912
    allowed_proto3_extendees->insert(std::string("google.protobuf.") +
                                     option_name);
    // Split the word to trick the opensource processing scripts so they
    // will keep the original package name.
    allowed_proto3_extendees->insert(std::string("proto") + "2." + option_name);
  }
  return allowed_proto3_extendees;
}

// Checks whether the extendee type is allowed in proto3.
// Only extensions to descriptor options are allowed. We use name comparison
// instead of comparing the descriptor directly because the extensions may be
// defined in a different pool.
bool AllowedExtendeeInProto3(const std::string& name) {
  static auto allowed_proto3_extendees =
      internal::OnShutdownDelete(NewAllowedProto3Extendee());
  return allowed_proto3_extendees->find(name) !=
         allowed_proto3_extendees->end();
}

// This bump allocator arena is optimized for the use case of this file. It is
// mostly optimized for memory usage, since these objects are expected to live
// for the entirety of the program.
//
// Some differences from other arenas:
//  - It has a fixed number of non-trivial types it can hold. This allows
//    tracking the allocations with a single byte. In contrast, google::protobuf::Arena
//    uses 16 bytes per non-trivial object created.
//  - It has some extra metadata for rollbacks. This is necessary for
//    implementing the API below. This metadata is flushed at the end and would
//    not cause persistent memory usage.
//  - It tries to squeeze every byte of out the blocks. If an allocation is too
//    large for the current block we move the block to a secondary area where we
//    can still use it for smaller objects. This complicates rollback logic but
//    makes it much more memory efficient.
//
//  The allocation strategy is as follows:
//   - Memory is allocated from the front, with a forced 8 byte alignment.
//   - Metadata is allocated from the back, one byte per element.
//   - The metadata encodes one of two things:
//     * For types we want to track, the index into KnownTypes.
//     * For raw memory blocks, the size of the block (in 8 byte increments
//       to allow for a larger limit).
//   - When the raw data is too large to represent in the metadata byte, we
//     allocate this memory separately in the heap and store an OutOfLineAlloc
//     object instead. These come from large array allocations and alike.
//
//  Blocks are kept in 3 areas:
//   - `current_` is the one we are currently allocating from. When we need to
//     allocate a block that doesn't fit there, we make a new block and move the
//     old `current_` to one of the areas below.
//   - Blocks that have no more usable space left (ie less than 9 bytes) are
//     stored in `full_blocks_`.
//   - Blocks that have some usable space are categorized in
//     `small_size_blocks_` depending on how much space they have left.
//     See `kSmallSizes` to see which sizes we track.
//
class TableArena {
 public:
  // Allocate a block on `n` bytes, with no destructor information saved.
  void* AllocateMemory(uint32_t n) {
    uint32_t tag = SizeToRawTag(n) + kFirstRawTag;
    if (tag > 255) {
      // We can't fit the size, use an OutOfLineAlloc.
      return Create<OutOfLineAlloc>(OutOfLineAlloc{::operator new(n), n})->ptr;
    }

    return AllocRawInternal(n, static_cast<Tag>(tag));
  }

  // Allocate and construct an element of type `T` as if by
  // `T(std::forward<Args>(args...))`.
  // The object is registered for destruction, if its destructor is not trivial.
  template <typename T, typename... Args>
  T* Create(Args&&... args) {
    static_assert(alignof(T) <= 8, "");
    return ::new (AllocRawInternal(sizeof(T), TypeTag<T>(KnownTypes{})))
        T(std::forward<Args>(args)...);
  }

  TableArena() {}

  TableArena(const TableArena&) = delete;
  TableArena& operator=(const TableArena&) = delete;

  ~TableArena() {
    // Uncomment this to debug usage statistics of the arena blocks.
    // PrintUsageInfo();

    for (Block* list : GetLists()) {
      while (list != nullptr) {
        Block* b = list;
        list = list->next;
        b->VisitBlock(DestroyVisitor{});
        b->Destroy();
      }
    }
  }


  // This function exists for debugging only.
  // It can be called from the destructor to dump some info in the tests to
  // inspect the usage of the arena.
  void PrintUsageInfo() const {
    const auto print_histogram = [](Block* b, int size) {
      std::map<uint32_t, uint32_t> unused_space_count;
      int count = 0;
      for (; b != nullptr; b = b->next) {
        ++unused_space_count[b->space_left()];
        ++count;
      }
      if (size > 0) {
        fprintf(stderr, "  Blocks `At least %d`", size);
      } else {
        fprintf(stderr, "  Blocks `full`");
      }
      fprintf(stderr, ": %d blocks.\n", count);
      for (auto p : unused_space_count) {
        fprintf(stderr, "    space=%4u, count=%3u\n", p.first, p.second);
      }
    };

    fprintf(stderr, "TableArena unused space histogram:\n");
    fprintf(stderr, "  Current: %u\n",
            current_ != nullptr ? current_->space_left() : 0);
    print_histogram(full_blocks_, 0);
    for (size_t i = 0; i < kSmallSizes.size(); ++i) {
      print_histogram(small_size_blocks_[i], kSmallSizes[i]);
    }
  }

  // Current allocation count.
  // This can be used for checkpointing.
  size_t num_allocations() const { return num_allocations_; }

  // Rollback the latest allocations until we reach back to `checkpoint`
  // num_allocations.
  void RollbackTo(size_t checkpoint) {
    while (num_allocations_ > checkpoint) {
      GOOGLE_DCHECK(!rollback_info_.empty());
      auto& info = rollback_info_.back();
      Block* b = info.block;

      VisitAlloc(b->data(), &b->start_offset, &b->end_offset, DestroyVisitor{},
                 KnownTypes{});
      if (--info.count == 0) {
        rollback_info_.pop_back();
      }
      --num_allocations_;
    }

    // Reconstruct the lists and destroy empty blocks.
    auto lists = GetLists();
    current_ = full_blocks_ = nullptr;
    small_size_blocks_.fill(nullptr);

    for (Block* list : lists) {
      while (list != nullptr) {
        Block* b = list;
        list = list->next;

        if (b->start_offset == 0) {
          // This is empty, free it.
          b->Destroy();
        } else {
          RelocateToUsedList(b);
        }
      }
    }
  }

  // Clear all rollback information. Reduces memory usage.
  // Trying to rollback past num_allocations() is now impossible.
  void ClearRollbackData() {
    rollback_info_.clear();
    rollback_info_.shrink_to_fit();
  }

 private:
  static constexpr size_t RoundUp(size_t n) { return (n + 7) & ~7; }

  using Tag = unsigned char;

  void* AllocRawInternal(uint32_t size, Tag tag) {
    GOOGLE_DCHECK_GT(size, 0);
    size = RoundUp(size);

    Block* to_relocate = nullptr;
    Block* to_use = nullptr;

    for (size_t i = 0; i < kSmallSizes.size(); ++i) {
      if (small_size_blocks_[i] != nullptr && size <= kSmallSizes[i]) {
        to_use = to_relocate = PopBlock(small_size_blocks_[i]);
        break;
      }
    }

    if (to_relocate != nullptr) {
      // We found one in the loop.
    } else if (current_ != nullptr && size + 1 <= current_->space_left()) {
      to_use = current_;
    } else {
      // No space left anywhere, make a new block.
      to_relocate = current_;
      // For now we hardcode the size to one page. Note that the maximum we can
      // allocate in the block according to the limits of Tag is less than 2k,
      // so this can fit anything that Tag can represent.
      constexpr size_t kBlockSize = 4096;
      to_use = current_ = ::new (::operator new(kBlockSize)) Block(kBlockSize);
      GOOGLE_DCHECK_GE(current_->space_left(), size + 1);
    }

    ++num_allocations_;
    if (!rollback_info_.empty() && rollback_info_.back().block == to_use) {
      ++rollback_info_.back().count;
    } else {
      rollback_info_.push_back({to_use, 1});
    }

    void* p = to_use->Allocate(size, tag);
    if (to_relocate != nullptr) {
      RelocateToUsedList(to_relocate);
    }
    return p;
  }

  static void OperatorDelete(void* p, size_t s) {
#if defined(__GXX_DELETE_WITH_SIZE__) || defined(__cpp_sized_deallocation)
    ::operator delete(p, s);
#else
    ::operator delete(p);
#endif
  }

  struct OutOfLineAlloc {
    void* ptr;
    uint32_t size;
  };

  template <typename... T>
  struct TypeList {
    static constexpr Tag kSize = static_cast<Tag>(sizeof...(T));
  };

  template <typename T, typename Visitor>
  static void RunVisitor(char* p, uint16_t* start, Visitor visit) {
    *start -= RoundUp(sizeof(T));
    visit(reinterpret_cast<T*>(p + *start));
  }

  // Visit the allocation at the passed location.
  // It updates start/end to be after the visited object.
  // This allows visiting a whole block by calling the function in a loop.
  template <typename Visitor, typename... T>
  static void VisitAlloc(char* p, uint16_t* start, uint16_t* end, Visitor visit,
                         TypeList<T...>) {
    const Tag tag = static_cast<Tag>(p[*end]);
    if (tag >= kFirstRawTag) {
      // Raw memory. Skip it.
      *start -= TagToSize(tag);
    } else {
      using F = void (*)(char*, uint16_t*, Visitor);
      static constexpr F kFuncs[] = {&RunVisitor<T, Visitor>...};
      kFuncs[tag](p, start, visit);
    }
    ++*end;
  }

  template <typename U, typename... Ts>
  static constexpr Tag TypeTag(TypeList<U, Ts...>) {
    return 0;
  }

  template <
      typename U, typename T, typename... Ts,
      typename = typename std::enable_if<!std::is_same<U, T>::value>::type>
  static constexpr Tag TypeTag(TypeList<T, Ts...>) {
    return 1 + TypeTag<U>(TypeList<Ts...>{});
  }

  template <typename U>
  static constexpr Tag TypeTag(TypeList<>) {
    static_assert(std::is_trivially_destructible<U>::value, "");
    return SizeToRawTag(sizeof(U));
  }

  using KnownTypes =
      TypeList<OutOfLineAlloc, std::string,
               // For name arrays
               std::array<std::string, 2>, std::array<std::string, 3>,
               std::array<std::string, 4>, std::array<std::string, 5>,
               FileDescriptorTables, SourceCodeInfo, FileOptions,
               MessageOptions, FieldOptions, ExtensionRangeOptions,
               OneofOptions, EnumOptions, EnumValueOptions, ServiceOptions,
               MethodOptions>;
  static constexpr Tag kFirstRawTag = KnownTypes::kSize;


  struct DestroyVisitor {
    template <typename T>
    void operator()(T* p) {
      p->~T();
    }
    void operator()(OutOfLineAlloc* p) { OperatorDelete(p->ptr, p->size); }
  };

  static uint32_t SizeToRawTag(size_t n) { return (RoundUp(n) / 8) - 1; }

  static uint32_t TagToSize(Tag tag) {
    GOOGLE_DCHECK_GE(tag, kFirstRawTag);
    return static_cast<uint32_t>(tag - kFirstRawTag + 1) * 8;
  }

  struct Block {
    uint16_t start_offset;
    uint16_t end_offset;
    uint16_t capacity;
    Block* next;

    // `allocated_size` is the total size of the memory block allocated.
    // The `Block` structure is constructed at the start and the rest of the
    // memory is used as the payload of the `Block`.
    explicit Block(uint32_t allocated_size) {
      start_offset = 0;
      end_offset = capacity =
          reinterpret_cast<char*>(this) + allocated_size - data();
      next = nullptr;
    }

    char* data() {
      return reinterpret_cast<char*>(this) + RoundUp(sizeof(Block));
    }

    uint32_t memory_used() {
      return data() + capacity - reinterpret_cast<char*>(this);
    }
    uint32_t space_left() const { return end_offset - start_offset; }

    void* Allocate(uint32_t n, Tag tag) {
      GOOGLE_DCHECK_LE(n + 1, space_left());
      void* p = data() + start_offset;
      start_offset += n;
      data()[--end_offset] = tag;
      return p;
    }

    void Destroy() { OperatorDelete(this, memory_used()); }

    void PrependTo(Block*& list) {
      next = list;
      list = this;
    }

    template <typename Visitor>
    void VisitBlock(Visitor visit) {
      for (uint16_t s = start_offset, e = end_offset; s != 0;) {
        VisitAlloc(data(), &s, &e, visit, KnownTypes{});
      }
    }
  };

  Block* PopBlock(Block*& list) {
    Block* res = list;
    list = list->next;
    return res;
  }

  void RelocateToUsedList(Block* to_relocate) {
    if (current_ == nullptr) {
      current_ = to_relocate;
      current_->next = nullptr;
      return;
    } else if (current_->space_left() < to_relocate->space_left()) {
      std::swap(current_, to_relocate);
      current_->next = nullptr;
    }

    for (int i = kSmallSizes.size(); --i >= 0;) {
      if (to_relocate->space_left() >= 1 + kSmallSizes[i]) {
        to_relocate->PrependTo(small_size_blocks_[i]);
        return;
      }
    }

    to_relocate->PrependTo(full_blocks_);
  }

  static constexpr std::array<uint8_t, 6> kSmallSizes = {
      {// Sizes for pointer arrays.
       8, 16, 24, 32,
       // Sizes for string arrays (for descriptor names).
       // The most common array sizes are 2 and 3.
       2 * sizeof(std::string), 3 * sizeof(std::string)}};

  // Helper function to iterate all lists.
  std::array<Block*, 2 + kSmallSizes.size()> GetLists() const {
    std::array<Block*, 2 + kSmallSizes.size()> res;
    res[0] = current_;
    res[1] = full_blocks_;
    std::copy(small_size_blocks_.begin(), small_size_blocks_.end(), &res[2]);
    return res;
  }

  Block* current_ = nullptr;
  std::array<Block*, kSmallSizes.size()> small_size_blocks_ = {{}};
  Block* full_blocks_ = nullptr;

  size_t num_allocations_ = 0;
  struct RollbackInfo {
    Block* block;
    size_t count;
  };
  std::vector<RollbackInfo> rollback_info_;
};

constexpr std::array<uint8_t, 6> TableArena::kSmallSizes;

}  // anonymous namespace

// ===================================================================
// DescriptorPool::Tables

class DescriptorPool::Tables {
 public:
  Tables();
  ~Tables();

  // Record the current state of the tables to the stack of checkpoints.
  // Each call to AddCheckpoint() must be paired with exactly one call to either
  // ClearLastCheckpoint() or RollbackToLastCheckpoint().
  //
  // This is used when building files, since some kinds of validation errors
  // cannot be detected until the file's descriptors have already been added to
  // the tables.
  //
  // This supports recursive checkpoints, since building a file may trigger
  // recursive building of other files. Note that recursive checkpoints are not
  // normally necessary; explicit dependencies are built prior to checkpointing.
  // So although we recursively build transitive imports, there is at most one
  // checkpoint in the stack during dependency building.
  //
  // Recursive checkpoints only arise during cross-linking of the descriptors.
  // Symbol references must be resolved, via DescriptorBuilder::FindSymbol and
  // friends. If the pending file references an unknown symbol
  // (e.g., it is not defined in the pending file's explicit dependencies), and
  // the pool is using a fallback database, and that database contains a file
  // defining that symbol, and that file has not yet been built by the pool,
  // the pool builds the file during cross-linking, leading to another
  // checkpoint.
  void AddCheckpoint();

  // Mark the last checkpoint as having cleared successfully, removing it from
  // the stack. If the stack is empty, all pending symbols will be committed.
  //
  // Note that this does not guarantee that the symbols added since the last
  // checkpoint won't be rolled back: if a checkpoint gets rolled back,
  // everything past that point gets rolled back, including symbols added after
  // checkpoints that were pushed onto the stack after it and marked as cleared.
  void ClearLastCheckpoint();

  // Roll back the Tables to the state of the checkpoint at the top of the
  // stack, removing everything that was added after that point.
  void RollbackToLastCheckpoint();

  // The stack of files which are currently being built.  Used to detect
  // cyclic dependencies when loading files from a DescriptorDatabase.  Not
  // used when fallback_database_ == nullptr.
  std::vector<std::string> pending_files_;

  // A set of files which we have tried to load from the fallback database
  // and encountered errors.  We will not attempt to load them again during
  // execution of the current public API call, but for compatibility with
  // legacy clients, this is cleared at the beginning of each public API call.
  // Not used when fallback_database_ == nullptr.
  HASH_SET<std::string> known_bad_files_;

  // A set of symbols which we have tried to load from the fallback database
  // and encountered errors. We will not attempt to load them again during
  // execution of the current public API call, but for compatibility with
  // legacy clients, this is cleared at the beginning of each public API call.
  HASH_SET<std::string> known_bad_symbols_;

  // The set of descriptors for which we've already loaded the full
  // set of extensions numbers from fallback_database_.
  HASH_SET<const Descriptor*> extensions_loaded_from_db_;

  // Maps type name to Descriptor::WellKnownType.  This is logically global
  // and const, but we make it a member here to simplify its construction and
  // destruction.  This only has 20-ish entries and is one per DescriptorPool,
  // so the overhead is small.
  HASH_MAP<std::string, Descriptor::WellKnownType> well_known_types_;

  // -----------------------------------------------------------------
  // Finding items.

  // Find symbols.  This returns a null Symbol (symbol.IsNull() is true)
  // if not found.
  inline Symbol FindSymbol(StringPiece key) const;

  // This implements the body of DescriptorPool::Find*ByName().  It should
  // really be a private method of DescriptorPool, but that would require
  // declaring Symbol in descriptor.h, which would drag all kinds of other
  // stuff into the header.  Yay C++.
  Symbol FindByNameHelper(const DescriptorPool* pool, StringPiece name);

  // These return nullptr if not found.
  inline const FileDescriptor* FindFile(StringPiece key) const;
  inline const FieldDescriptor* FindExtension(const Descriptor* extendee,
                                              int number) const;
  inline void FindAllExtensions(const Descriptor* extendee,
                                std::vector<const FieldDescriptor*>* out) const;

  // -----------------------------------------------------------------
  // Adding items.

  // These add items to the corresponding tables.  They return false if
  // the key already exists in the table.  For AddSymbol(), the string passed
  // in must be one that was constructed using AllocateString(), as it will
  // be used as a key in the symbols_by_name_ map without copying.
  bool AddSymbol(const std::string& full_name, Symbol symbol);
  bool AddFile(const FileDescriptor* file);
  bool AddExtension(const FieldDescriptor* field);

  // -----------------------------------------------------------------
  // Allocating memory.

  // Allocate an object which will be reclaimed when the pool is
  // destroyed.  Note that the object's destructor will never be called,
  // so its fields must be plain old data (primitive data types and
  // pointers).  All of the descriptor types are such objects.
  template <typename Type>
  Type* Allocate();

  // Allocate an array of objects which will be reclaimed when the
  // pool in destroyed.  Again, destructors are never called.
  template <typename Type>
  Type* AllocateArray(int count);

  // Allocate a string which will be destroyed when the pool is destroyed.
  // The string is initialized to the given value for convenience.
  const std::string* AllocateString(StringPiece value);

  // Copy the input into a NUL terminated string whose lifetime is managed by
  // the pool.
  const char* Strdup(StringPiece value);

  // Allocates an array of strings which will be destroyed when the pool is
  // destroyed. The array is initialized with the input values.
  template <typename... In>
  const std::string* AllocateStringArray(In&&... values);

  struct FieldNamesResult {
    std::string* array;
    int lowercase_index;
    int camelcase_index;
    int json_index;
  };
  // Allocate all 5 names of the field:
  // name, full name, lowercase, camelcase and json.
  // This function will dedup the strings when possible.
  // The resulting array contains `name` at index 0, `full_name` at index 1 and
  // the other 3 indices are specified in the result.
  FieldNamesResult AllocateFieldNames(const std::string& name,
                                      const std::string& scope,
                                      const std::string* opt_json_name);

  // Create an object that will be deleted when the pool is destroyed.
  // The object is value initialized, and its destructor will be called if
  // non-trivial.
  template <typename Type>
  Type* Create();

  // Allocate a protocol message object.  Some older versions of GCC have
  // trouble understanding explicit template instantiations in some cases, so
  // in those cases we have to pass a dummy pointer of the right type as the
  // parameter instead of specifying the type explicitly.
  template <typename Type>
  Type* AllocateMessage(Type* dummy = nullptr);

  // Allocate a FileDescriptorTables object.
  FileDescriptorTables* AllocateFileTables();

 private:
  // All other memory allocated in the pool.  Must be first as other objects can
  // point into these.
  TableArena arena_;

  SymbolsByNameSet symbols_by_name_;
  FilesByNameMap files_by_name_;
  ExtensionsGroupedByDescriptorMap extensions_;

  struct CheckPoint {
    explicit CheckPoint(const Tables* tables)
        : arena_before_checkpoint(tables->arena_.num_allocations()),
          pending_symbols_before_checkpoint(
              tables->symbols_after_checkpoint_.size()),
          pending_files_before_checkpoint(
              tables->files_after_checkpoint_.size()),
          pending_extensions_before_checkpoint(
              tables->extensions_after_checkpoint_.size()) {}
    int arena_before_checkpoint;
    int pending_symbols_before_checkpoint;
    int pending_files_before_checkpoint;
    int pending_extensions_before_checkpoint;
  };
  std::vector<CheckPoint> checkpoints_;
  std::vector<const char*> symbols_after_checkpoint_;
  std::vector<const char*> files_after_checkpoint_;
  std::vector<DescriptorIntPair> extensions_after_checkpoint_;

  // Allocate some bytes which will be reclaimed when the pool is
  // destroyed.
  void* AllocateBytes(int size);
};

// Contains tables specific to a particular file.  These tables are not
// modified once the file has been constructed, so they need not be
// protected by a mutex.  This makes operations that depend only on the
// contents of a single file -- e.g. Descriptor::FindFieldByName() --
// lock-free.
//
// For historical reasons, the definitions of the methods of
// FileDescriptorTables and DescriptorPool::Tables are interleaved below.
// These used to be a single class.
class FileDescriptorTables {
 public:
  FileDescriptorTables();
  ~FileDescriptorTables();

  // Empty table, used with placeholder files.
  inline static const FileDescriptorTables& GetEmptyInstance();

  // -----------------------------------------------------------------
  // Finding items.

  // Returns a null Symbol (symbol.IsNull() is true) if not found.
  inline Symbol FindNestedSymbol(const void* parent,
                                 StringPiece name) const;

  // These return nullptr if not found.
  inline const FieldDescriptor* FindFieldByNumber(const Descriptor* parent,
                                                  int number) const;
  inline const FieldDescriptor* FindFieldByLowercaseName(
      const void* parent, StringPiece lowercase_name) const;
  inline const FieldDescriptor* FindFieldByCamelcaseName(
      const void* parent, StringPiece camelcase_name) const;
  inline const EnumValueDescriptor* FindEnumValueByNumber(
      const EnumDescriptor* parent, int number) const;
  // This creates a new EnumValueDescriptor if not found, in a thread-safe way.
  inline const EnumValueDescriptor* FindEnumValueByNumberCreatingIfUnknown(
      const EnumDescriptor* parent, int number) const;

  // -----------------------------------------------------------------
  // Adding items.

  // These add items to the corresponding tables.  They return false if
  // the key already exists in the table.  For AddAliasUnderParent(), the
  // string passed in must be one that was constructed using AllocateString(),
  // as it will be used as a key in the symbols_by_parent_ map without copying.
  bool AddAliasUnderParent(const void* parent, const std::string& name,
                           Symbol symbol);
  bool AddFieldByNumber(FieldDescriptor* field);
  bool AddEnumValueByNumber(EnumValueDescriptor* value);

  // Adds the field to the lowercase_name and camelcase_name maps.  Never
  // fails because we allow duplicates; the first field by the name wins.
  void AddFieldByStylizedNames(const FieldDescriptor* field);

  // Populates p->first->locations_by_path_ from p->second.
  // Unusual signature dictated by internal::call_once.
  static void BuildLocationsByPath(
      std::pair<const FileDescriptorTables*, const SourceCodeInfo*>* p);

  // Returns the location denoted by the specified path through info,
  // or nullptr if not found.
  // The value of info must be that of the corresponding FileDescriptor.
  // (Conceptually a pure function, but stateful as an optimisation.)
  const SourceCodeInfo_Location* GetSourceLocation(
      const std::vector<int>& path, const SourceCodeInfo* info) const;

  // Must be called after BuildFileImpl(), even if the build failed and
  // we are going to roll back to the last checkpoint.
  void FinalizeTables();

 private:
  const void* FindParentForFieldsByMap(const FieldDescriptor* field) const;
  static void FieldsByLowercaseNamesLazyInitStatic(
      const FileDescriptorTables* tables);
  void FieldsByLowercaseNamesLazyInitInternal() const;
  static void FieldsByCamelcaseNamesLazyInitStatic(
      const FileDescriptorTables* tables);
  void FieldsByCamelcaseNamesLazyInitInternal() const;

  SymbolsByParentSet symbols_by_parent_;
  mutable FieldsByNameMap fields_by_lowercase_name_;
  std::unique_ptr<FieldsByNameMap> fields_by_lowercase_name_tmp_;
  mutable internal::once_flag fields_by_lowercase_name_once_;
  mutable FieldsByNameMap fields_by_camelcase_name_;
  std::unique_ptr<FieldsByNameMap> fields_by_camelcase_name_tmp_;
  mutable internal::once_flag fields_by_camelcase_name_once_;
  FieldsByNumberSet fields_by_number_;  // Not including extensions.
  EnumValuesByNumberSet enum_values_by_number_;
  mutable EnumValuesByNumberSet unknown_enum_values_by_number_
      PROTOBUF_GUARDED_BY(unknown_enum_values_mu_);

  // Populated on first request to save space, hence constness games.
  mutable internal::once_flag locations_by_path_once_;
  mutable LocationsByPathMap locations_by_path_;

  // Mutex to protect the unknown-enum-value map due to dynamic
  // EnumValueDescriptor creation on unknown values.
  mutable internal::WrappedMutex unknown_enum_values_mu_;
};

DescriptorPool::Tables::Tables() {
  well_known_types_.insert({
      {"google.protobuf.DoubleValue", Descriptor::WELLKNOWNTYPE_DOUBLEVALUE},
      {"google.protobuf.FloatValue", Descriptor::WELLKNOWNTYPE_FLOATVALUE},
      {"google.protobuf.Int64Value", Descriptor::WELLKNOWNTYPE_INT64VALUE},
      {"google.protobuf.UInt64Value", Descriptor::WELLKNOWNTYPE_UINT64VALUE},
      {"google.protobuf.Int32Value", Descriptor::WELLKNOWNTYPE_INT32VALUE},
      {"google.protobuf.UInt32Value", Descriptor::WELLKNOWNTYPE_UINT32VALUE},
      {"google.protobuf.StringValue", Descriptor::WELLKNOWNTYPE_STRINGVALUE},
      {"google.protobuf.BytesValue", Descriptor::WELLKNOWNTYPE_BYTESVALUE},
      {"google.protobuf.BoolValue", Descriptor::WELLKNOWNTYPE_BOOLVALUE},
      {"google.protobuf.Any", Descriptor::WELLKNOWNTYPE_ANY},
      {"google.protobuf.FieldMask", Descriptor::WELLKNOWNTYPE_FIELDMASK},
      {"google.protobuf.Duration", Descriptor::WELLKNOWNTYPE_DURATION},
      {"google.protobuf.Timestamp", Descriptor::WELLKNOWNTYPE_TIMESTAMP},
      {"google.protobuf.Value", Descriptor::WELLKNOWNTYPE_VALUE},
      {"google.protobuf.ListValue", Descriptor::WELLKNOWNTYPE_LISTVALUE},
      {"google.protobuf.Struct", Descriptor::WELLKNOWNTYPE_STRUCT},
  });
}

DescriptorPool::Tables::~Tables() { GOOGLE_DCHECK(checkpoints_.empty()); }

FileDescriptorTables::FileDescriptorTables()
    : fields_by_lowercase_name_tmp_(new FieldsByNameMap()),
      fields_by_camelcase_name_tmp_(new FieldsByNameMap()) {}

FileDescriptorTables::~FileDescriptorTables() {}

inline const FileDescriptorTables& FileDescriptorTables::GetEmptyInstance() {
  static auto file_descriptor_tables =
      internal::OnShutdownDelete(new FileDescriptorTables());
  return *file_descriptor_tables;
}

void DescriptorPool::Tables::AddCheckpoint() {
  checkpoints_.push_back(CheckPoint(this));
}

void DescriptorPool::Tables::ClearLastCheckpoint() {
  GOOGLE_DCHECK(!checkpoints_.empty());
  checkpoints_.pop_back();
  if (checkpoints_.empty()) {
    // All checkpoints have been cleared: we can now commit all of the pending
    // data.
    symbols_after_checkpoint_.clear();
    files_after_checkpoint_.clear();
    extensions_after_checkpoint_.clear();
    arena_.ClearRollbackData();
  }
}

void DescriptorPool::Tables::RollbackToLastCheckpoint() {
  GOOGLE_DCHECK(!checkpoints_.empty());
  const CheckPoint& checkpoint = checkpoints_.back();

  for (size_t i = checkpoint.pending_symbols_before_checkpoint;
       i < symbols_after_checkpoint_.size(); i++) {
    Symbol::QueryKey name;
    name.name = symbols_after_checkpoint_[i];
    symbols_by_name_.erase(Symbol(&name));
  }
  for (size_t i = checkpoint.pending_files_before_checkpoint;
       i < files_after_checkpoint_.size(); i++) {
    files_by_name_.erase(files_after_checkpoint_[i]);
  }
  for (size_t i = checkpoint.pending_extensions_before_checkpoint;
       i < extensions_after_checkpoint_.size(); i++) {
    extensions_.erase(extensions_after_checkpoint_[i]);
  }

  symbols_after_checkpoint_.resize(
      checkpoint.pending_symbols_before_checkpoint);
  files_after_checkpoint_.resize(checkpoint.pending_files_before_checkpoint);
  extensions_after_checkpoint_.resize(
      checkpoint.pending_extensions_before_checkpoint);

  arena_.RollbackTo(checkpoint.arena_before_checkpoint);
  checkpoints_.pop_back();
}

// -------------------------------------------------------------------

inline Symbol DescriptorPool::Tables::FindSymbol(StringPiece key) const {
  Symbol::QueryKey name;
  name.name = key;
  auto it = symbols_by_name_.find(Symbol(&name));
  return it == symbols_by_name_.end() ? kNullSymbol : *it;
}

inline Symbol FileDescriptorTables::FindNestedSymbol(
    const void* parent, StringPiece name) const {
  Symbol::QueryKey query;
  query.name = name;
  query.parent = parent;
  auto it = symbols_by_parent_.find(Symbol(&query));
  return it == symbols_by_parent_.end() ? kNullSymbol : *it;
}

Symbol DescriptorPool::Tables::FindByNameHelper(const DescriptorPool* pool,
                                                StringPiece name) {
  if (pool->mutex_ != nullptr) {
    // Fast path: the Symbol is already cached.  This is just a hash lookup.
    ReaderMutexLock lock(pool->mutex_);
    if (known_bad_symbols_.empty() && known_bad_files_.empty()) {
      Symbol result = FindSymbol(name);
      if (!result.IsNull()) return result;
    }
  }
  MutexLockMaybe lock(pool->mutex_);
  if (pool->fallback_database_ != nullptr) {
    known_bad_symbols_.clear();
    known_bad_files_.clear();
  }
  Symbol result = FindSymbol(name);

  if (result.IsNull() && pool->underlay_ != nullptr) {
    // Symbol not found; check the underlay.
    result = pool->underlay_->tables_->FindByNameHelper(pool->underlay_, name);
  }

  if (result.IsNull()) {
    // Symbol still not found, so check fallback database.
    if (pool->TryFindSymbolInFallbackDatabase(name)) {
      result = FindSymbol(name);
    }
  }

  return result;
}

inline const FileDescriptor* DescriptorPool::Tables::FindFile(
    StringPiece key) const {
  return FindPtrOrNull(files_by_name_, key);
}

inline const FieldDescriptor* FileDescriptorTables::FindFieldByNumber(
    const Descriptor* parent, int number) const {
  // If `number` is within the sequential range, just index into the parent
  // without doing a table lookup.
  if (parent != nullptr &&  //
      1 <= number && number <= parent->sequential_field_limit_) {
    return parent->field(number - 1);
  }

  Symbol::QueryKey query;
  query.parent = parent;
  query.field_number = number;

  auto it = fields_by_number_.find(Symbol(&query));
  return it == fields_by_number_.end() ? nullptr : it->field_descriptor();
}

const void* FileDescriptorTables::FindParentForFieldsByMap(
    const FieldDescriptor* field) const {
  if (field->is_extension()) {
    if (field->extension_scope() == nullptr) {
      return field->file();
    } else {
      return field->extension_scope();
    }
  } else {
    return field->containing_type();
  }
}

void FileDescriptorTables::FieldsByLowercaseNamesLazyInitStatic(
    const FileDescriptorTables* tables) {
  tables->FieldsByLowercaseNamesLazyInitInternal();
}

void FileDescriptorTables::FieldsByLowercaseNamesLazyInitInternal() const {
  for (Symbol symbol : symbols_by_parent_) {
    const FieldDescriptor* field = symbol.field_descriptor();
    if (!field) continue;
    PointerStringPair lowercase_key(FindParentForFieldsByMap(field),
                                    field->lowercase_name().c_str());
    InsertIfNotPresent(&fields_by_lowercase_name_, lowercase_key, field);
  }
}

inline const FieldDescriptor* FileDescriptorTables::FindFieldByLowercaseName(
    const void* parent, StringPiece lowercase_name) const {
  internal::call_once(
      fields_by_lowercase_name_once_,
      &FileDescriptorTables::FieldsByLowercaseNamesLazyInitStatic, this);
  return FindPtrOrNull(fields_by_lowercase_name_,
                            PointerStringPair(parent, lowercase_name));
}

void FileDescriptorTables::FieldsByCamelcaseNamesLazyInitStatic(
    const FileDescriptorTables* tables) {
  tables->FieldsByCamelcaseNamesLazyInitInternal();
}

void FileDescriptorTables::FieldsByCamelcaseNamesLazyInitInternal() const {
  for (Symbol symbol : symbols_by_parent_) {
    const FieldDescriptor* field = symbol.field_descriptor();
    if (!field) continue;
    PointerStringPair camelcase_key(FindParentForFieldsByMap(field),
                                    field->camelcase_name().c_str());
    InsertIfNotPresent(&fields_by_camelcase_name_, camelcase_key, field);
  }
}

inline const FieldDescriptor* FileDescriptorTables::FindFieldByCamelcaseName(
    const void* parent, StringPiece camelcase_name) const {
  internal::call_once(
      fields_by_camelcase_name_once_,
      FileDescriptorTables::FieldsByCamelcaseNamesLazyInitStatic, this);
  return FindPtrOrNull(fields_by_camelcase_name_,
                            PointerStringPair(parent, camelcase_name));
}

inline const EnumValueDescriptor* FileDescriptorTables::FindEnumValueByNumber(
    const EnumDescriptor* parent, int number) const {
  // If `number` is within the sequential range, just index into the parent
  // without doing a table lookup.
  const int base = parent->value(0)->number();
  if (base <= number &&
      number <= static_cast<int64_t>(base) + parent->sequential_value_limit_) {
    return parent->value(number - base);
  }

  Symbol::QueryKey query;
  query.parent = parent;
  query.field_number = number;

  auto it = enum_values_by_number_.find(Symbol(&query));
  return it == enum_values_by_number_.end() ? nullptr
                                            : it->enum_value_descriptor();
}

inline const EnumValueDescriptor*
FileDescriptorTables::FindEnumValueByNumberCreatingIfUnknown(
    const EnumDescriptor* parent, int number) const {
  // First try, with map of compiled-in values.
  {
    const auto* value = FindEnumValueByNumber(parent, number);
    if (value != nullptr) {
      return value;
    }
  }

  Symbol::QueryKey query;
  query.parent = parent;
  query.field_number = number;

  // Second try, with reader lock held on unknown enum values: common case.
  {
    ReaderMutexLock l(&unknown_enum_values_mu_);
    auto it = unknown_enum_values_by_number_.find(Symbol(&query));
    if (it != unknown_enum_values_by_number_.end() &&
        it->enum_value_descriptor() != nullptr) {
      return it->enum_value_descriptor();
    }
  }
  // If not found, try again with writer lock held, and create new descriptor if
  // necessary.
  {
    WriterMutexLock l(&unknown_enum_values_mu_);
    auto it = unknown_enum_values_by_number_.find(Symbol(&query));
    if (it != unknown_enum_values_by_number_.end() &&
        it->enum_value_descriptor() != nullptr) {
      return it->enum_value_descriptor();
    }

    // Create an EnumValueDescriptor dynamically. We don't insert it into the
    // EnumDescriptor (it's not a part of the enum as originally defined), but
    // we do insert it into the table so that we can return the same pointer
    // later.
    std::string enum_value_name = StringPrintf("UNKNOWN_ENUM_VALUE_%s_%d",
                                               parent->name().c_str(), number);
    auto* pool = DescriptorPool::generated_pool();
    auto* tables = const_cast<DescriptorPool::Tables*>(pool->tables_.get());
    EnumValueDescriptor* result;
    {
      // Must lock the pool because we will do allocations in the shared arena.
      MutexLockMaybe l2(pool->mutex_);
      result = tables->Allocate<EnumValueDescriptor>();
      result->all_names_ = tables->AllocateStringArray(
          enum_value_name,
          StrCat(parent->full_name(), ".", enum_value_name));
    }
    result->number_ = number;
    result->type_ = parent;
    result->options_ = &EnumValueOptions::default_instance();
    unknown_enum_values_by_number_.insert(Symbol::EnumValue(result, 0));
    return result;
  }
}

inline const FieldDescriptor* DescriptorPool::Tables::FindExtension(
    const Descriptor* extendee, int number) const {
  return FindPtrOrNull(extensions_, std::make_pair(extendee, number));
}

inline void DescriptorPool::Tables::FindAllExtensions(
    const Descriptor* extendee,
    std::vector<const FieldDescriptor*>* out) const {
  ExtensionsGroupedByDescriptorMap::const_iterator it =
      extensions_.lower_bound(std::make_pair(extendee, 0));
  for (; it != extensions_.end() && it->first.first == extendee; ++it) {
    out->push_back(it->second);
  }
}

// -------------------------------------------------------------------

bool DescriptorPool::Tables::AddSymbol(const std::string& full_name,
                                       Symbol symbol) {
  GOOGLE_DCHECK_EQ(full_name, symbol.full_name());
  if (symbols_by_name_.insert(symbol).second) {
    symbols_after_checkpoint_.push_back(full_name.c_str());
    return true;
  } else {
    return false;
  }
}

bool FileDescriptorTables::AddAliasUnderParent(const void* parent,
                                               const std::string& name,
                                               Symbol symbol) {
  GOOGLE_DCHECK_EQ(name, symbol.parent_name_key().second);
  GOOGLE_DCHECK_EQ(parent, symbol.parent_name_key().first);
  return symbols_by_parent_.insert(symbol).second;
}

bool DescriptorPool::Tables::AddFile(const FileDescriptor* file) {
  if (InsertIfNotPresent(&files_by_name_, file->name(), file)) {
    files_after_checkpoint_.push_back(file->name().c_str());
    return true;
  } else {
    return false;
  }
}

void FileDescriptorTables::FinalizeTables() {
  // Clean up the temporary maps used by AddFieldByStylizedNames().
  fields_by_lowercase_name_tmp_ = nullptr;
  fields_by_camelcase_name_tmp_ = nullptr;
}

void FileDescriptorTables::AddFieldByStylizedNames(
    const FieldDescriptor* field) {
  const void* parent = FindParentForFieldsByMap(field);

  // We want fields_by_{lower,camel}case_name_ to be lazily built, but
  // cross-link order determines which entry will be present in the case of a
  // conflict. So we use the temporary maps that get destroyed after
  // BuildFileImpl() to detect the conflicts, and only store the conflicts in
  // the map that will persist. We will then lazily populate the rest of the
  // entries from fields_by_number_.

  PointerStringPair lowercase_key(parent, field->lowercase_name().c_str());
  if (!InsertIfNotPresent(fields_by_lowercase_name_tmp_.get(),
                               lowercase_key, field)) {
    InsertIfNotPresent(
        &fields_by_lowercase_name_, lowercase_key,
        FindPtrOrNull(*fields_by_lowercase_name_tmp_, lowercase_key));
  }

  PointerStringPair camelcase_key(parent, field->camelcase_name().c_str());
  if (!InsertIfNotPresent(fields_by_camelcase_name_tmp_.get(),
                               camelcase_key, field)) {
    InsertIfNotPresent(
        &fields_by_camelcase_name_, camelcase_key,
        FindPtrOrNull(*fields_by_camelcase_name_tmp_, camelcase_key));
  }
}

bool FileDescriptorTables::AddFieldByNumber(FieldDescriptor* field) {
  // Skip fields that are at the start of the sequence.
  if (field->containing_type() != nullptr && field->number() >= 1 &&
      field->number() <= field->containing_type()->sequential_field_limit_) {
    if (field->is_extension()) {
      // Conflicts with the field that already exists in the sequential range.
      return false;
    }
    // Only return true if the field at that index matches. Otherwise it
    // conflicts with the existing field in the sequential range.
    return field->containing_type()->field(field->number() - 1) == field;
  }

  return fields_by_number_.insert(Symbol(field)).second;
}

bool FileDescriptorTables::AddEnumValueByNumber(EnumValueDescriptor* value) {
  // Skip values that are at the start of the sequence.
  const int base = value->type()->value(0)->number();
  if (base <= value->number() &&
      value->number() <=
          static_cast<int64_t>(base) + value->type()->sequential_value_limit_)
    return true;
  return enum_values_by_number_.insert(Symbol::EnumValue(value, 0)).second;
}

bool DescriptorPool::Tables::AddExtension(const FieldDescriptor* field) {
  DescriptorIntPair key(field->containing_type(), field->number());
  if (InsertIfNotPresent(&extensions_, key, field)) {
    extensions_after_checkpoint_.push_back(key);
    return true;
  } else {
    return false;
  }
}

// -------------------------------------------------------------------

template <typename Type>
Type* DescriptorPool::Tables::Allocate() {
  return reinterpret_cast<Type*>(AllocateBytes(sizeof(Type)));
}

template <typename Type>
Type* DescriptorPool::Tables::AllocateArray(int count) {
  return reinterpret_cast<Type*>(AllocateBytes(sizeof(Type) * count));
}

const std::string* DescriptorPool::Tables::AllocateString(
    StringPiece value) {
  return arena_.Create<std::string>(value);
}

const char* DescriptorPool::Tables::Strdup(StringPiece value) {
  char* p = AllocateArray<char>(static_cast<int>(value.size() + 1));
  memcpy(p, value.data(), value.size());
  p[value.size()] = 0;
  return p;
}

template <typename... In>
const std::string* DescriptorPool::Tables::AllocateStringArray(In&&... values) {
  auto& array = *arena_.Create<std::array<std::string, sizeof...(In)>>();
  array = {{std::string(std::forward<In>(values))...}};
  return array.data();
}

DescriptorPool::Tables::FieldNamesResult
DescriptorPool::Tables::AllocateFieldNames(const std::string& name,
                                           const std::string& scope,
                                           const std::string* opt_json_name) {
  std::string lowercase_name = name;
  LowerString(&lowercase_name);

  std::string camelcase_name = ToCamelCase(name, /* lower_first = */ true);
  std::string json_name;
  if (opt_json_name != nullptr) {
    json_name = *opt_json_name;
  } else {
    json_name = ToJsonName(name);
  }

  const bool lower_eq_name = lowercase_name == name;
  const bool camel_eq_name = camelcase_name == name;
  const bool json_eq_name = json_name == name;
  const bool json_eq_camel = json_name == camelcase_name;

  const int total_count = 2 + (lower_eq_name ? 0 : 1) +
                          (camel_eq_name ? 0 : 1) +
                          (json_eq_name || json_eq_camel ? 0 : 1);
  FieldNamesResult result{nullptr, 0, 0, 0};
  // We use std::array to allow handling of the destruction of the strings.
  switch (total_count) {
    case 2:
      result.array = arena_.Create<std::array<std::string, 2>>()->data();
      break;
    case 3:
      result.array = arena_.Create<std::array<std::string, 3>>()->data();
      break;
    case 4:
      result.array = arena_.Create<std::array<std::string, 4>>()->data();
      break;
    case 5:
      result.array = arena_.Create<std::array<std::string, 5>>()->data();
      break;
  }

  result.array[0] = name;
  if (scope.empty()) {
    result.array[1] = name;
  } else {
    result.array[1] = StrCat(scope, ".", name);
  }
  int index = 2;
  if (lower_eq_name) {
    result.lowercase_index = 0;
  } else {
    result.lowercase_index = index;
    result.array[index++] = std::move(lowercase_name);
  }

  if (camel_eq_name) {
    result.camelcase_index = 0;
  } else {
    result.camelcase_index = index;
    result.array[index++] = std::move(camelcase_name);
  }

  if (json_eq_name) {
    result.json_index = 0;
  } else if (json_eq_camel) {
    result.json_index = result.camelcase_index;
  } else {
    result.json_index = index;
    result.array[index] = std::move(json_name);
  }

  return result;
}

template <typename Type>
Type* DescriptorPool::Tables::Create() {
  return arena_.Create<Type>();
}

template <typename Type>
Type* DescriptorPool::Tables::AllocateMessage(Type* /* dummy */) {
  return arena_.Create<Type>();
}

FileDescriptorTables* DescriptorPool::Tables::AllocateFileTables() {
  return arena_.Create<FileDescriptorTables>();
}

void* DescriptorPool::Tables::AllocateBytes(int size) {
  if (size == 0) return nullptr;
  return arena_.AllocateMemory(size);
}

void FileDescriptorTables::BuildLocationsByPath(
    std::pair<const FileDescriptorTables*, const SourceCodeInfo*>* p) {
  for (int i = 0, len = p->second->location_size(); i < len; ++i) {
    const SourceCodeInfo_Location* loc = &p->second->location().Get(i);
    p->first->locations_by_path_[Join(loc->path(), ",")] = loc;
  }
}

const SourceCodeInfo_Location* FileDescriptorTables::GetSourceLocation(
    const std::vector<int>& path, const SourceCodeInfo* info) const {
  std::pair<const FileDescriptorTables*, const SourceCodeInfo*> p(
      std::make_pair(this, info));
  internal::call_once(locations_by_path_once_,
                      FileDescriptorTables::BuildLocationsByPath, &p);
  return FindPtrOrNull(locations_by_path_, Join(path, ","));
}

// ===================================================================
// DescriptorPool

DescriptorPool::ErrorCollector::~ErrorCollector() {}

DescriptorPool::DescriptorPool()
    : mutex_(nullptr),
      fallback_database_(nullptr),
      default_error_collector_(nullptr),
      underlay_(nullptr),
      tables_(new Tables),
      enforce_dependencies_(true),
      lazily_build_dependencies_(false),
      allow_unknown_(false),
      enforce_weak_(false),
      disallow_enforce_utf8_(false) {}

DescriptorPool::DescriptorPool(DescriptorDatabase* fallback_database,
                               ErrorCollector* error_collector)
    : mutex_(new internal::WrappedMutex),
      fallback_database_(fallback_database),
      default_error_collector_(error_collector),
      underlay_(nullptr),
      tables_(new Tables),
      enforce_dependencies_(true),
      lazily_build_dependencies_(false),
      allow_unknown_(false),
      enforce_weak_(false),
      disallow_enforce_utf8_(false) {}

DescriptorPool::DescriptorPool(const DescriptorPool* underlay)
    : mutex_(nullptr),
      fallback_database_(nullptr),
      default_error_collector_(nullptr),
      underlay_(underlay),
      tables_(new Tables),
      enforce_dependencies_(true),
      lazily_build_dependencies_(false),
      allow_unknown_(false),
      enforce_weak_(false),
      disallow_enforce_utf8_(false) {}

DescriptorPool::~DescriptorPool() {
  if (mutex_ != nullptr) delete mutex_;
}

// DescriptorPool::BuildFile() defined later.
// DescriptorPool::BuildFileCollectingErrors() defined later.

void DescriptorPool::InternalDontEnforceDependencies() {
  enforce_dependencies_ = false;
}

void DescriptorPool::AddUnusedImportTrackFile(ConstStringParam file_name,
                                              bool is_error) {
  unused_import_track_files_[std::string(file_name)] = is_error;
}

void DescriptorPool::ClearUnusedImportTrackFiles() {
  unused_import_track_files_.clear();
}

bool DescriptorPool::InternalIsFileLoaded(ConstStringParam filename) const {
  MutexLockMaybe lock(mutex_);
  return tables_->FindFile(filename) != nullptr;
}

// generated_pool ====================================================

namespace {


EncodedDescriptorDatabase* GeneratedDatabase() {
  static auto generated_database =
      internal::OnShutdownDelete(new EncodedDescriptorDatabase());
  return generated_database;
}

DescriptorPool* NewGeneratedPool() {
  auto generated_pool = new DescriptorPool(GeneratedDatabase());
  generated_pool->InternalSetLazilyBuildDependencies();
  return generated_pool;
}

}  // anonymous namespace

DescriptorDatabase* DescriptorPool::internal_generated_database() {
  return GeneratedDatabase();
}

DescriptorPool* DescriptorPool::internal_generated_pool() {
  static DescriptorPool* generated_pool =
      internal::OnShutdownDelete(NewGeneratedPool());
  return generated_pool;
}

const DescriptorPool* DescriptorPool::generated_pool() {
  const DescriptorPool* pool = internal_generated_pool();
  // Ensure that descriptor.proto has been registered in the generated pool.
  DescriptorProto::descriptor();
  return pool;
}


void DescriptorPool::InternalAddGeneratedFile(
    const void* encoded_file_descriptor, int size) {
  // So, this function is called in the process of initializing the
  // descriptors for generated proto classes.  Each generated .pb.cc file
  // has an internal procedure called AddDescriptors() which is called at
  // process startup, and that function calls this one in order to register
  // the raw bytes of the FileDescriptorProto representing the file.
  //
  // We do not actually construct the descriptor objects right away.  We just
  // hang on to the bytes until they are actually needed.  We actually construct
  // the descriptor the first time one of the following things happens:
  // * Someone calls a method like descriptor(), GetDescriptor(), or
  //   GetReflection() on the generated types, which requires returning the
  //   descriptor or an object based on it.
  // * Someone looks up the descriptor in DescriptorPool::generated_pool().
  //
  // Once one of these happens, the DescriptorPool actually parses the
  // FileDescriptorProto and generates a FileDescriptor (and all its children)
  // based on it.
  //
  // Note that FileDescriptorProto is itself a generated protocol message.
  // Therefore, when we parse one, we have to be very careful to avoid using
  // any descriptor-based operations, since this might cause infinite recursion
  // or deadlock.
  GOOGLE_CHECK(GeneratedDatabase()->Add(encoded_file_descriptor, size));
}


// Find*By* methods ==================================================

// TODO(kenton):  There's a lot of repeated code here, but I'm not sure if
//   there's any good way to factor it out.  Think about this some time when
//   there's nothing more important to do (read: never).

const FileDescriptor* DescriptorPool::FindFileByName(
    ConstStringParam name) const {
  MutexLockMaybe lock(mutex_);
  if (fallback_database_ != nullptr) {
    tables_->known_bad_symbols_.clear();
    tables_->known_bad_files_.clear();
  }
  const FileDescriptor* result = tables_->FindFile(name);
  if (result != nullptr) return result;
  if (underlay_ != nullptr) {
    result = underlay_->FindFileByName(name);
    if (result != nullptr) return result;
  }
  if (TryFindFileInFallbackDatabase(name)) {
    result = tables_->FindFile(name);
    if (result != nullptr) return result;
  }
  return nullptr;
}

const FileDescriptor* DescriptorPool::FindFileContainingSymbol(
    ConstStringParam symbol_name) const {
  MutexLockMaybe lock(mutex_);
  if (fallback_database_ != nullptr) {
    tables_->known_bad_symbols_.clear();
    tables_->known_bad_files_.clear();
  }
  Symbol result = tables_->FindSymbol(symbol_name);
  if (!result.IsNull()) return result.GetFile();
  if (underlay_ != nullptr) {
    const FileDescriptor* file_result =
        underlay_->FindFileContainingSymbol(symbol_name);
    if (file_result != nullptr) return file_result;
  }
  if (TryFindSymbolInFallbackDatabase(symbol_name)) {
    result = tables_->FindSymbol(symbol_name);
    if (!result.IsNull()) return result.GetFile();
  }
  return nullptr;
}

const Descriptor* DescriptorPool::FindMessageTypeByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).descriptor();
}

const FieldDescriptor* DescriptorPool::FindFieldByName(
    ConstStringParam name) const {
  if (const FieldDescriptor* field =
          tables_->FindByNameHelper(this, name).field_descriptor()) {
    if (!field->is_extension()) {
      return field;
    }
  }
  return nullptr;
}

const FieldDescriptor* DescriptorPool::FindExtensionByName(
    ConstStringParam name) const {
  if (const FieldDescriptor* field =
          tables_->FindByNameHelper(this, name).field_descriptor()) {
    if (field->is_extension()) {
      return field;
    }
  }
  return nullptr;
}

const OneofDescriptor* DescriptorPool::FindOneofByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).oneof_descriptor();
}

const EnumDescriptor* DescriptorPool::FindEnumTypeByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).enum_descriptor();
}

const EnumValueDescriptor* DescriptorPool::FindEnumValueByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).enum_value_descriptor();
}

const ServiceDescriptor* DescriptorPool::FindServiceByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).service_descriptor();
}

const MethodDescriptor* DescriptorPool::FindMethodByName(
    ConstStringParam name) const {
  return tables_->FindByNameHelper(this, name).method_descriptor();
}

const FieldDescriptor* DescriptorPool::FindExtensionByNumber(
    const Descriptor* extendee, int number) const {
  if (extendee->extension_range_count() == 0) return nullptr;
  // A faster path to reduce lock contention in finding extensions, assuming
  // most extensions will be cache hit.
  if (mutex_ != nullptr) {
    ReaderMutexLock lock(mutex_);
    const FieldDescriptor* result = tables_->FindExtension(extendee, number);
    if (result != nullptr) {
      return result;
    }
  }
  MutexLockMaybe lock(mutex_);
  if (fallback_database_ != nullptr) {
    tables_->known_bad_symbols_.clear();
    tables_->known_bad_files_.clear();
  }
  const FieldDescriptor* result = tables_->FindExtension(extendee, number);
  if (result != nullptr) {
    return result;
  }
  if (underlay_ != nullptr) {
    result = underlay_->FindExtensionByNumber(extendee, number);
    if (result != nullptr) return result;
  }
  if (TryFindExtensionInFallbackDatabase(extendee, number)) {
    result = tables_->FindExtension(extendee, number);
    if (result != nullptr) {
      return result;
    }
  }
  return nullptr;
}

const FieldDescriptor* DescriptorPool::InternalFindExtensionByNumberNoLock(
    const Descriptor* extendee, int number) const {
  if (extendee->extension_range_count() == 0) return nullptr;

  const FieldDescriptor* result = tables_->FindExtension(extendee, number);
  if (result != nullptr) {
    return result;
  }

  if (underlay_ != nullptr) {
    result = underlay_->InternalFindExtensionByNumberNoLock(extendee, number);
    if (result != nullptr) return result;
  }

  return nullptr;
}

const FieldDescriptor* DescriptorPool::FindExtensionByPrintableName(
    const Descriptor* extendee, ConstStringParam printable_name) const {
  if (extendee->extension_range_count() == 0) return nullptr;
  const FieldDescriptor* result = FindExtensionByName(printable_name);
  if (result != nullptr && result->containing_type() == extendee) {
    return result;
  }
  if (extendee->options().message_set_wire_format()) {
    // MessageSet extensions may be identified by type name.
    const Descriptor* type = FindMessageTypeByName(printable_name);
    if (type != nullptr) {
      // Look for a matching extension in the foreign type's scope.
      const int type_extension_count = type->extension_count();
      for (int i = 0; i < type_extension_count; i++) {
        const FieldDescriptor* extension = type->extension(i);
        if (extension->containing_type() == extendee &&
            extension->type() == FieldDescriptor::TYPE_MESSAGE &&
            extension->is_optional() && extension->message_type() == type) {
          // Found it.
          return extension;
        }
      }
    }
  }
  return nullptr;
}

void DescriptorPool::FindAllExtensions(
    const Descriptor* extendee,
    std::vector<const FieldDescriptor*>* out) const {
  MutexLockMaybe lock(mutex_);
  if (fallback_database_ != nullptr) {
    tables_->known_bad_symbols_.clear();
    tables_->known_bad_files_.clear();
  }

  // Initialize tables_->extensions_ from the fallback database first
  // (but do this only once per descriptor).
  if (fallback_database_ != nullptr &&
      tables_->extensions_loaded_from_db_.count(extendee) == 0) {
    std::vector<int> numbers;
    if (fallback_database_->FindAllExtensionNumbers(extendee->full_name(),
                                                    &numbers)) {
      for (int number : numbers) {
        if (tables_->FindExtension(extendee, number) == nullptr) {
          TryFindExtensionInFallbackDatabase(extendee, number);
        }
      }
      tables_->extensions_loaded_from_db_.insert(extendee);
    }
  }

  tables_->FindAllExtensions(extendee, out);
  if (underlay_ != nullptr) {
    underlay_->FindAllExtensions(extendee, out);
  }
}


// -------------------------------------------------------------------

const FieldDescriptor* Descriptor::FindFieldByNumber(int key) const {
  const FieldDescriptor* result = file()->tables_->FindFieldByNumber(this, key);
  if (result == nullptr || result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const FieldDescriptor* Descriptor::FindFieldByLowercaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result =
      file()->tables_->FindFieldByLowercaseName(this, key);
  if (result == nullptr || result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const FieldDescriptor* Descriptor::FindFieldByCamelcaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result =
      file()->tables_->FindFieldByCamelcaseName(this, key);
  if (result == nullptr || result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const FieldDescriptor* Descriptor::FindFieldByName(ConstStringParam key) const {
  const FieldDescriptor* field =
      file()->tables_->FindNestedSymbol(this, key).field_descriptor();
  return field != nullptr && !field->is_extension() ? field : nullptr;
}

const OneofDescriptor* Descriptor::FindOneofByName(ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).oneof_descriptor();
}

const FieldDescriptor* Descriptor::FindExtensionByName(
    ConstStringParam key) const {
  const FieldDescriptor* field =
      file()->tables_->FindNestedSymbol(this, key).field_descriptor();
  return field != nullptr && field->is_extension() ? field : nullptr;
}

const FieldDescriptor* Descriptor::FindExtensionByLowercaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result =
      file()->tables_->FindFieldByLowercaseName(this, key);
  if (result == nullptr || !result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const FieldDescriptor* Descriptor::FindExtensionByCamelcaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result =
      file()->tables_->FindFieldByCamelcaseName(this, key);
  if (result == nullptr || !result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const Descriptor* Descriptor::FindNestedTypeByName(ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).descriptor();
}

const EnumDescriptor* Descriptor::FindEnumTypeByName(
    ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).enum_descriptor();
}

const EnumValueDescriptor* Descriptor::FindEnumValueByName(
    ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).enum_value_descriptor();
}

const FieldDescriptor* Descriptor::map_key() const {
  if (!options().map_entry()) return nullptr;
  GOOGLE_DCHECK_EQ(field_count(), 2);
  return field(0);
}

const FieldDescriptor* Descriptor::map_value() const {
  if (!options().map_entry()) return nullptr;
  GOOGLE_DCHECK_EQ(field_count(), 2);
  return field(1);
}

const EnumValueDescriptor* EnumDescriptor::FindValueByName(
    ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).enum_value_descriptor();
}

const EnumValueDescriptor* EnumDescriptor::FindValueByNumber(int key) const {
  return file()->tables_->FindEnumValueByNumber(this, key);
}

const EnumValueDescriptor* EnumDescriptor::FindValueByNumberCreatingIfUnknown(
    int key) const {
  return file()->tables_->FindEnumValueByNumberCreatingIfUnknown(this, key);
}

const MethodDescriptor* ServiceDescriptor::FindMethodByName(
    ConstStringParam key) const {
  return file()->tables_->FindNestedSymbol(this, key).method_descriptor();
}

const Descriptor* FileDescriptor::FindMessageTypeByName(
    ConstStringParam key) const {
  return tables_->FindNestedSymbol(this, key).descriptor();
}

const EnumDescriptor* FileDescriptor::FindEnumTypeByName(
    ConstStringParam key) const {
  return tables_->FindNestedSymbol(this, key).enum_descriptor();
}

const EnumValueDescriptor* FileDescriptor::FindEnumValueByName(
    ConstStringParam key) const {
  return tables_->FindNestedSymbol(this, key).enum_value_descriptor();
}

const ServiceDescriptor* FileDescriptor::FindServiceByName(
    ConstStringParam key) const {
  return tables_->FindNestedSymbol(this, key).service_descriptor();
}

const FieldDescriptor* FileDescriptor::FindExtensionByName(
    ConstStringParam key) const {
  const FieldDescriptor* field =
      tables_->FindNestedSymbol(this, key).field_descriptor();
  return field != nullptr && field->is_extension() ? field : nullptr;
}

const FieldDescriptor* FileDescriptor::FindExtensionByLowercaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result = tables_->FindFieldByLowercaseName(this, key);
  if (result == nullptr || !result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

const FieldDescriptor* FileDescriptor::FindExtensionByCamelcaseName(
    ConstStringParam key) const {
  const FieldDescriptor* result = tables_->FindFieldByCamelcaseName(this, key);
  if (result == nullptr || !result->is_extension()) {
    return nullptr;
  } else {
    return result;
  }
}

void Descriptor::ExtensionRange::CopyTo(
    DescriptorProto_ExtensionRange* proto) const {
  proto->set_start(this->start);
  proto->set_end(this->end);
  if (options_ != &ExtensionRangeOptions::default_instance()) {
    *proto->mutable_options() = *options_;
  }
}

const Descriptor::ExtensionRange*
Descriptor::FindExtensionRangeContainingNumber(int number) const {
  // Linear search should be fine because we don't expect a message to have
  // more than a couple extension ranges.
  for (int i = 0; i < extension_range_count(); i++) {
    if (number >= extension_range(i)->start &&
        number < extension_range(i)->end) {
      return extension_range(i);
    }
  }
  return nullptr;
}

const Descriptor::ReservedRange* Descriptor::FindReservedRangeContainingNumber(
    int number) const {
  // TODO(chrisn): Consider a non-linear search.
  for (int i = 0; i < reserved_range_count(); i++) {
    if (number >= reserved_range(i)->start && number < reserved_range(i)->end) {
      return reserved_range(i);
    }
  }
  return nullptr;
}

const EnumDescriptor::ReservedRange*
EnumDescriptor::FindReservedRangeContainingNumber(int number) const {
  // TODO(chrisn): Consider a non-linear search.
  for (int i = 0; i < reserved_range_count(); i++) {
    if (number >= reserved_range(i)->start &&
        number <= reserved_range(i)->end) {
      return reserved_range(i);
    }
  }
  return nullptr;
}

// -------------------------------------------------------------------

bool DescriptorPool::TryFindFileInFallbackDatabase(
    StringPiece name) const {
  if (fallback_database_ == nullptr) return false;

  auto name_string = std::string(name);
  if (tables_->known_bad_files_.count(name_string) > 0) return false;

  FileDescriptorProto file_proto;
  if (!fallback_database_->FindFileByName(name_string, &file_proto) ||
      BuildFileFromDatabase(file_proto) == nullptr) {
    tables_->known_bad_files_.insert(std::move(name_string));
    return false;
  }
  return true;
}

bool DescriptorPool::IsSubSymbolOfBuiltType(StringPiece name) const {
  auto prefix = std::string(name);
  for (;;) {
    std::string::size_type dot_pos = prefix.find_last_of('.');
    if (dot_pos == std::string::npos) {
      break;
    }
    prefix = prefix.substr(0, dot_pos);
    Symbol symbol = tables_->FindSymbol(prefix);
    // If the symbol type is anything other than PACKAGE, then its complete
    // definition is already known.
    if (!symbol.IsNull() && symbol.type() != Symbol::PACKAGE) {
      return true;
    }
  }
  if (underlay_ != nullptr) {
    // Check to see if any prefix of this symbol exists in the underlay.
    return underlay_->IsSubSymbolOfBuiltType(name);
  }
  return false;
}

bool DescriptorPool::TryFindSymbolInFallbackDatabase(
    StringPiece name) const {
  if (fallback_database_ == nullptr) return false;

  auto name_string = std::string(name);
  if (tables_->known_bad_symbols_.count(name_string) > 0) return false;

  FileDescriptorProto file_proto;
  if (  // We skip looking in the fallback database if the name is a sub-symbol
        // of any descriptor that already exists in the descriptor pool (except
        // for package descriptors).  This is valid because all symbols except
        // for packages are defined in a single file, so if the symbol exists
        // then we should already have its definition.
        //
        // The other reason to do this is to support "overriding" type
        // definitions by merging two databases that define the same type. (Yes,
        // people do this.)  The main difficulty with making this work is that
        // FindFileContainingSymbol() is allowed to return both false positives
        // (e.g., SimpleDescriptorDatabase, UpgradedDescriptorDatabase) and
        // false negatives (e.g. ProtoFileParser, SourceTreeDescriptorDatabase).
        // When two such databases are merged, looking up a non-existent
        // sub-symbol of a type that already exists in the descriptor pool can
        // result in an attempt to load multiple definitions of the same type.
        // The check below avoids this.
      IsSubSymbolOfBuiltType(name)

      // Look up file containing this symbol in fallback database.
      || !fallback_database_->FindFileContainingSymbol(name_string, &file_proto)

      // Check if we've already built this file. If so, it apparently doesn't
      // contain the symbol we're looking for.  Some DescriptorDatabases
      // return false positives.
      || tables_->FindFile(file_proto.name()) != nullptr

      // Build the file.
      || BuildFileFromDatabase(file_proto) == nullptr) {
    tables_->known_bad_symbols_.insert(std::move(name_string));
    return false;
  }

  return true;
}

bool DescriptorPool::TryFindExtensionInFallbackDatabase(
    const Descriptor* containing_type, int field_number) const {
  if (fallback_database_ == nullptr) return false;

  FileDescriptorProto file_proto;
  if (!fallback_database_->FindFileContainingExtension(
          containing_type->full_name(), field_number, &file_proto)) {
    return false;
  }

  if (tables_->FindFile(file_proto.name()) != nullptr) {
    // We've already loaded this file, and it apparently doesn't contain the
    // extension we're looking for.  Some DescriptorDatabases return false
    // positives.
    return false;
  }

  if (BuildFileFromDatabase(file_proto) == nullptr) {
    return false;
  }

  return true;
}

// ===================================================================

bool FieldDescriptor::is_map_message_type() const {
  return type_descriptor_.message_type->options().map_entry();
}

std::string FieldDescriptor::DefaultValueAsString(
    bool quote_string_type) const {
  GOOGLE_CHECK(has_default_value()) << "No default value";
  switch (cpp_type()) {
    case CPPTYPE_INT32:
      return StrCat(default_value_int32_t());
    case CPPTYPE_INT64:
      return StrCat(default_value_int64_t());
    case CPPTYPE_UINT32:
      return StrCat(default_value_uint32_t());
    case CPPTYPE_UINT64:
      return StrCat(default_value_uint64_t());
    case CPPTYPE_FLOAT:
      return SimpleFtoa(default_value_float());
    case CPPTYPE_DOUBLE:
      return SimpleDtoa(default_value_double());
    case CPPTYPE_BOOL:
      return default_value_bool() ? "true" : "false";
    case CPPTYPE_STRING:
      if (quote_string_type) {
        return "\"" + CEscape(default_value_string()) + "\"";
      } else {
        if (type() == TYPE_BYTES) {
          return CEscape(default_value_string());
        } else {
          return default_value_string();
        }
      }
    case CPPTYPE_ENUM:
      return default_value_enum()->name();
    case CPPTYPE_MESSAGE:
      GOOGLE_LOG(DFATAL) << "Messages can't have default values!";
      break;
  }
  GOOGLE_LOG(FATAL) << "Can't get here: failed to get default value as string";
  return "";
}

// CopyTo methods ====================================================

void FileDescriptor::CopyTo(FileDescriptorProto* proto) const {
  proto->set_name(name());
  if (!package().empty()) proto->set_package(package());
  // TODO(liujisi): Also populate when syntax="proto2".
  if (syntax() == SYNTAX_PROTO3) proto->set_syntax(SyntaxName(syntax()));

  for (int i = 0; i < dependency_count(); i++) {
    proto->add_dependency(dependency(i)->name());
  }

  for (int i = 0; i < public_dependency_count(); i++) {
    proto->add_public_dependency(public_dependencies_[i]);
  }

  for (int i = 0; i < weak_dependency_count(); i++) {
    proto->add_weak_dependency(weak_dependencies_[i]);
  }

  for (int i = 0; i < message_type_count(); i++) {
    message_type(i)->CopyTo(proto->add_message_type());
  }
  for (int i = 0; i < enum_type_count(); i++) {
    enum_type(i)->CopyTo(proto->add_enum_type());
  }
  for (int i = 0; i < service_count(); i++) {
    service(i)->CopyTo(proto->add_service());
  }
  for (int i = 0; i < extension_count(); i++) {
    extension(i)->CopyTo(proto->add_extension());
  }

  if (&options() != &FileOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void FileDescriptor::CopyJsonNameTo(FileDescriptorProto* proto) const {
  if (message_type_count() != proto->message_type_size() ||
      extension_count() != proto->extension_size()) {
    GOOGLE_LOG(ERROR) << "Cannot copy json_name to a proto of a different size.";
    return;
  }
  for (int i = 0; i < message_type_count(); i++) {
    message_type(i)->CopyJsonNameTo(proto->mutable_message_type(i));
  }
  for (int i = 0; i < extension_count(); i++) {
    extension(i)->CopyJsonNameTo(proto->mutable_extension(i));
  }
}

void FileDescriptor::CopySourceCodeInfoTo(FileDescriptorProto* proto) const {
  if (source_code_info_ &&
      source_code_info_ != &SourceCodeInfo::default_instance()) {
    proto->mutable_source_code_info()->CopyFrom(*source_code_info_);
  }
}

void Descriptor::CopyTo(DescriptorProto* proto) const {
  proto->set_name(name());

  for (int i = 0; i < field_count(); i++) {
    field(i)->CopyTo(proto->add_field());
  }
  for (int i = 0; i < oneof_decl_count(); i++) {
    oneof_decl(i)->CopyTo(proto->add_oneof_decl());
  }
  for (int i = 0; i < nested_type_count(); i++) {
    nested_type(i)->CopyTo(proto->add_nested_type());
  }
  for (int i = 0; i < enum_type_count(); i++) {
    enum_type(i)->CopyTo(proto->add_enum_type());
  }
  for (int i = 0; i < extension_range_count(); i++) {
    extension_range(i)->CopyTo(proto->add_extension_range());
  }
  for (int i = 0; i < extension_count(); i++) {
    extension(i)->CopyTo(proto->add_extension());
  }
  for (int i = 0; i < reserved_range_count(); i++) {
    DescriptorProto::ReservedRange* range = proto->add_reserved_range();
    range->set_start(reserved_range(i)->start);
    range->set_end(reserved_range(i)->end);
  }
  for (int i = 0; i < reserved_name_count(); i++) {
    proto->add_reserved_name(reserved_name(i));
  }

  if (&options() != &MessageOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void Descriptor::CopyJsonNameTo(DescriptorProto* proto) const {
  if (field_count() != proto->field_size() ||
      nested_type_count() != proto->nested_type_size() ||
      extension_count() != proto->extension_size()) {
    GOOGLE_LOG(ERROR) << "Cannot copy json_name to a proto of a different size.";
    return;
  }
  for (int i = 0; i < field_count(); i++) {
    field(i)->CopyJsonNameTo(proto->mutable_field(i));
  }
  for (int i = 0; i < nested_type_count(); i++) {
    nested_type(i)->CopyJsonNameTo(proto->mutable_nested_type(i));
  }
  for (int i = 0; i < extension_count(); i++) {
    extension(i)->CopyJsonNameTo(proto->mutable_extension(i));
  }
}

void FieldDescriptor::CopyTo(FieldDescriptorProto* proto) const {
  proto->set_name(name());
  proto->set_number(number());
  if (has_json_name_) {
    proto->set_json_name(json_name());
  }
  if (proto3_optional_) {
    proto->set_proto3_optional(true);
  }
  // Some compilers do not allow static_cast directly between two enum types,
  // so we must cast to int first.
  proto->set_label(static_cast<FieldDescriptorProto::Label>(
      implicit_cast<int>(label())));
  proto->set_type(static_cast<FieldDescriptorProto::Type>(
      implicit_cast<int>(type())));

  if (is_extension()) {
    if (!containing_type()->is_unqualified_placeholder_) {
      proto->set_extendee(".");
    }
    proto->mutable_extendee()->append(containing_type()->full_name());
  }

  if (cpp_type() == CPPTYPE_MESSAGE) {
    if (message_type()->is_placeholder_) {
      // We don't actually know if the type is a message type.  It could be
      // an enum.
      proto->clear_type();
    }

    if (!message_type()->is_unqualified_placeholder_) {
      proto->set_type_name(".");
    }
    proto->mutable_type_name()->append(message_type()->full_name());
  } else if (cpp_type() == CPPTYPE_ENUM) {
    if (!enum_type()->is_unqualified_placeholder_) {
      proto->set_type_name(".");
    }
    proto->mutable_type_name()->append(enum_type()->full_name());
  }

  if (has_default_value()) {
    proto->set_default_value(DefaultValueAsString(false));
  }

  if (containing_oneof() != nullptr && !is_extension()) {
    proto->set_oneof_index(containing_oneof()->index());
  }

  if (&options() != &FieldOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void FieldDescriptor::CopyJsonNameTo(FieldDescriptorProto* proto) const {
  proto->set_json_name(json_name());
}

void OneofDescriptor::CopyTo(OneofDescriptorProto* proto) const {
  proto->set_name(name());
  if (&options() != &OneofOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void EnumDescriptor::CopyTo(EnumDescriptorProto* proto) const {
  proto->set_name(name());

  for (int i = 0; i < value_count(); i++) {
    value(i)->CopyTo(proto->add_value());
  }
  for (int i = 0; i < reserved_range_count(); i++) {
    EnumDescriptorProto::EnumReservedRange* range = proto->add_reserved_range();
    range->set_start(reserved_range(i)->start);
    range->set_end(reserved_range(i)->end);
  }
  for (int i = 0; i < reserved_name_count(); i++) {
    proto->add_reserved_name(reserved_name(i));
  }

  if (&options() != &EnumOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void EnumValueDescriptor::CopyTo(EnumValueDescriptorProto* proto) const {
  proto->set_name(name());
  proto->set_number(number());

  if (&options() != &EnumValueOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void ServiceDescriptor::CopyTo(ServiceDescriptorProto* proto) const {
  proto->set_name(name());

  for (int i = 0; i < method_count(); i++) {
    method(i)->CopyTo(proto->add_method());
  }

  if (&options() != &ServiceOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }
}

void MethodDescriptor::CopyTo(MethodDescriptorProto* proto) const {
  proto->set_name(name());

  if (!input_type()->is_unqualified_placeholder_) {
    proto->set_input_type(".");
  }
  proto->mutable_input_type()->append(input_type()->full_name());

  if (!output_type()->is_unqualified_placeholder_) {
    proto->set_output_type(".");
  }
  proto->mutable_output_type()->append(output_type()->full_name());

  if (&options() != &MethodOptions::default_instance()) {
    proto->mutable_options()->CopyFrom(options());
  }

  if (client_streaming_) {
    proto->set_client_streaming(true);
  }
  if (server_streaming_) {
    proto->set_server_streaming(true);
  }
}

// DebugString methods ===============================================

namespace {

bool RetrieveOptionsAssumingRightPool(
    int depth, const Message& options,
    std::vector<std::string>* option_entries) {
  option_entries->clear();
  const Reflection* reflection = options.GetReflection();
  std::vector<const FieldDescriptor*> fields;
  reflection->ListFields(options, &fields);
  for (const FieldDescriptor* field : fields) {
    int count = 1;
    bool repeated = false;
    if (field->is_repeated()) {
      count = reflection->FieldSize(options, field);
      repeated = true;
    }
    for (int j = 0; j < count; j++) {
      std::string fieldval;
      if (field->cpp_type() == FieldDescriptor::CPPTYPE_MESSAGE) {
        std::string tmp;
        TextFormat::Printer printer;
        printer.SetExpandAny(true);
        printer.SetInitialIndentLevel(depth + 1);
        printer.PrintFieldValueToString(options, field, repeated ? j : -1,
                                        &tmp);
        fieldval.append("{\n");
        fieldval.append(tmp);
        fieldval.append(depth * 2, ' ');
        fieldval.append("}");
      } else {
        TextFormat::PrintFieldValueToString(options, field, repeated ? j : -1,
                                            &fieldval);
      }
      std::string name;
      if (field->is_extension()) {
        name = "(." + field->full_name() + ")";
      } else {
        name = field->name();
      }
      option_entries->push_back(name + " = " + fieldval);
    }
  }
  return !option_entries->empty();
... [Content truncated - file is very large]
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

- **PrefixRemover**: A class/struct defined in this file
- **DestroyVisitor**: A class/struct defined in this file
- **one**: A class/struct defined in this file
- **FieldNamesResult**: A class/struct defined in this file
- **SymbolByParentEq**: A class/struct defined in this file
- **Package**: A class/struct defined in this file
- **FieldsByNumberEq**: A class/struct defined in this file
- **TableArena**: A class/struct defined in this file
- **RollbackInfo**: A class/struct defined in this file
- **Symbol**: A class/struct defined in this file
- **Block**: A class/struct defined in this file
- **OutOfLineAlloc**: A class/struct defined in this file
- **SymbolByFullNameHash**: A class/struct defined in this file
- **TypeList**: A class/struct defined in this file
- **DescriptorBuilder**: A class/struct defined in this file
- **CheckPoint**: A class/struct defined in this file
- **DescriptorT**: A class/struct defined in this file
- **AggregateOptionFinder**: A class/struct defined in this file
- **QueryKey**: A class/struct defined in this file
- **FieldsByNumberHash**: A class/struct defined in this file
- **a**: A class/struct defined in this file
- **OptionsToInterpret**: A class/struct defined in this file
- **DescriptorPool**: A class/struct defined in this file
- **SourceLocationCommentPrinter**: A class/struct defined in this file
- **the**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **SymbolByParentHash**: A class/struct defined in this file
- **an**: A class/struct defined in this file
- **PointerStringPairHash**: A class/struct defined in this file
- **AggregateErrorCollector**: A class/struct defined in this file
- **SymbolByFullNameEq**: A class/struct defined in this file
- **FileDescriptorTables**: A class/struct defined in this file
- **to**: A class/struct defined in this file
- **PointerIntegerPairHash**: A class/struct defined in this file
- **OptionInterpreter**: A class/struct defined in this file

### Functions and Methods

- **is()**: A function/method defined in this file
- **PROTOBUF_INTERNAL_IGNORE_FIELD_NAME_ERRORS_()**: A function/method defined in this file
- **in()**: A function/method defined in this file
- **PACKAGE()**: A function/method defined in this file
- **std()**: A function/method defined in this file
- **to()**: A function/method defined in this file
- **HASH_MAP()**: A function/method defined in this file
- **will()**: A function/method defined in this file
- **calls()**: A function/method defined in this file
- **VALIDATE_OPTIONS_FROM_ARRAY()**: A function/method defined in this file
- **that()**: A function/method defined in this file
- **BUILD_ARRAY()**: A function/method defined in this file
- **which()**: A function/method defined in this file
- **DEFINE_MEMBERS()**: A function/method defined in this file
- **_MSC_VER()**: A function/method defined in this file
- **exists()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `google/protobuf/io/zero_copy_stream_impl.h`
- `functional`
- `google/protobuf/dynamic_message.h`
- `google/protobuf/io/coded_stream.h`
- `google/protobuf/stubs/stringprintf.h`
- `google/protobuf/text_format.h`
- `google/protobuf/stubs/casts.h`
- `google/protobuf/descriptor.pb.h`
- `google/protobuf/any.h`
- `google/protobuf/descriptor_database.h`
- `google/protobuf/stubs/strutil.h`
- `array`
- `google/protobuf/io/tokenizer.h`
- `unordered_set`
- `google/protobuf/stubs/common.h`
- `google/protobuf/stubs/logging.h`
- `google/protobuf/unknown_field_set.h`
- `google/protobuf/stubs/hash.h`
- `unordered_map`
- `google/protobuf/stubs/stl_util.h`
- `google/protobuf/generated_message_util.h`
- `google/protobuf/io/strtod.h`
- `google/protobuf/stubs/once.h`
- `memory`
- `algorithm`
- `google/protobuf/descriptor.h`
- `google/protobuf/stubs/substitute.h`
- `google/protobuf/port_def.inc`
- `google/protobuf/stubs/map_util.h`
- `set`
- `google/protobuf/wire_format.h`
- `google/protobuf/port_undef.inc`
- `vector`
- `string`
- `limits`
- `map`

**Python Imports:**
- `one`
- `weak`
- `it.`
- `error`
- `other`
- `this`
- `public`
- `building`
- `is`
- `until`
- `enum`
- `p`
- `int32_t`
- `fields_by_number_.`
- `field`
- `a`
- `large`
- `the`
- `building.`
- `internal`
- `fallback_database_.`
- `unused_dependency.`
- `map`


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

