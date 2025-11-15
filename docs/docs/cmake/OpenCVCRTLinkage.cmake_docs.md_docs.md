# Documentation for `docs/cmake/OpenCVCRTLinkage.cmake_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/OpenCVCRTLinkage.cmake_docs.md`
- **File Name**: `OpenCVCRTLinkage.cmake_docs.md`
- **File Size**: 5,206 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/OpenCVCRTLinkage.cmake_docs.md](../../docs/cmake/OpenCVCRTLinkage.cmake_docs.md)

## Purpose and Role

This file is located in the `docs/cmake` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/OpenCVCRTLinkage.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVCRTLinkage.cmake`
- **File Name**: `OpenCVCRTLinkage.cmake`
- **File Size**: 4,262 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVCRTLinkage.cmake](../cmake/OpenCVCRTLinkage.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# Use statically or dynamically linked CRT?

if(NOT MSVC)
  message(FATAL_ERROR "CRT options are available only for MSVC")
endif()

# Removing LNK4075 warnings for debug WinRT builds
# "LNK4075: ignoring '/INCREMENTAL' due to '/OPT:ICF' specification"
# "LNK4075: ignoring '/INCREMENTAL' due to '/OPT:REF' specification"
if(MSVC AND WINRT)
  # Optional verification checks since we don't know existing contents of variables below
  string(REPLACE "/OPT:ICF " "/OPT:NOICF " CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/OPT:REF " "/OPT:NOREF " CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL:YES " "/INCREMENTAL:NO " CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL " "/INCREMENTAL:NO " CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG}")

  string(REPLACE "/OPT:ICF " "/OPT:NOICF " CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/OPT:REF " "/OPT:NOREF" CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL:YES " "/INCREMENTAL:NO " CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL " "/INCREMENTAL:NO " CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS_DEBUG}")

  string(REPLACE "/OPT:ICF " "/OPT:NOICF " CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/OPT:REF " "/OPT:NOREF " CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL:YES " "/INCREMENTAL:NO " CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS_DEBUG}")
  string(REPLACE "/INCREMENTAL " "/INCREMENTAL:NO " CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS_DEBUG}")

  # Mandatory
  set(CMAKE_MODULE_LINKER_FLAGS_DEBUG  "${CMAKE_MODULE_LINKER_FLAGS_DEBUG} /INCREMENTAL:NO /OPT:NOREF /OPT:NOICF")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG     "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /INCREMENTAL:NO /OPT:NOREF /OPT:NOICF")
  set(CMAKE_SHARED_LINKER_FLAGS_DEBUG  "${CMAKE_SHARED_LINKER_FLAGS_DEBUG} /INCREMENTAL:NO /OPT:NOREF /OPT:NOICF")
endif()

# Ignore warning: This object file does not define any previously undefined public symbols, ...
set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} /IGNORE:4221")

if(POLICY CMP0091)
  cmake_policy(GET CMP0091 MSVC_RUNTIME_SET_BY_ABSTRACTION)
endif()

if(NOT BUILD_SHARED_LIBS AND BUILD_WITH_STATIC_CRT)
  if(MSVC_RUNTIME_SET_BY_ABSTRACTION STREQUAL "NEW")
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
  else()
    foreach(flag_var
            CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
            CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
      if(${flag_var} MATCHES "/MD")
        string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
      endif()
      if(${flag_var} MATCHES "/MDd")
        string(REGEX REPLACE "/MDd" "/MTd" ${flag_var} "${${flag_var}}")
      endif()
    endforeach(flag_var)
  endif()

  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /NODEFAULTLIB:atlthunk.lib")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:libcmt.lib /NODEFAULTLIB:libcpmt.lib /NODEFAULTLIB:msvcrt.lib")
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /NODEFAULTLIB:libcmtd.lib /NODEFAULTLIB:libcpmtd.lib /NODEFAULTLIB:msvcrtd.lib")
else()
  if(NOT MSVC_RUNTIME_SET_BY_ABSTRACTION STREQUAL "NEW")
    foreach(flag_var
            CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
            CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
      if(${flag_var} MATCHES "/MT")
        string(REGEX REPLACE "/MT" "/MD" ${flag_var} "${${flag_var}}")
      endif()
      if(${flag_var} MATCHES "/MTd")
        string(REGEX REPLACE "/MTd" "/MDd" ${flag_var} "${${flag_var}}")
      endif()
    endforeach(flag_var)
  endif()
endif()

```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

