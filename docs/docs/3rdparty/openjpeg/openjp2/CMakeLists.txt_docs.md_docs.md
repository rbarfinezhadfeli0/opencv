# Documentation for `docs/3rdparty/openjpeg/openjp2/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 4,126 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/CMakeLists.txt_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/CMakeLists.txt`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 3,118 bytes
- **File Type**: .txt
- **Link to Source**: [3rdparty/openjpeg/openjp2/CMakeLists.txt](../../../3rdparty/openjpeg/openjp2/CMakeLists.txt)

## Purpose and Role

This file is located in the `3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# Defines the source code for the library
set(OPENJPEG_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/thread.c
  ${CMAKE_CURRENT_SOURCE_DIR}/bio.c
  ${CMAKE_CURRENT_SOURCE_DIR}/cio.c
  ${CMAKE_CURRENT_SOURCE_DIR}/dwt.c
  ${CMAKE_CURRENT_SOURCE_DIR}/event.c
  ${CMAKE_CURRENT_SOURCE_DIR}/ht_dec.c
  ${CMAKE_CURRENT_SOURCE_DIR}/image.c
  ${CMAKE_CURRENT_SOURCE_DIR}/invert.c
  ${CMAKE_CURRENT_SOURCE_DIR}/j2k.c
  ${CMAKE_CURRENT_SOURCE_DIR}/jp2.c
  ${CMAKE_CURRENT_SOURCE_DIR}/mct.c
  ${CMAKE_CURRENT_SOURCE_DIR}/mqc.c
  ${CMAKE_CURRENT_SOURCE_DIR}/openjpeg.c
  ${CMAKE_CURRENT_SOURCE_DIR}/opj_clock.c
  ${CMAKE_CURRENT_SOURCE_DIR}/pi.c
  ${CMAKE_CURRENT_SOURCE_DIR}/t1.c
  ${CMAKE_CURRENT_SOURCE_DIR}/t2.c
  ${CMAKE_CURRENT_SOURCE_DIR}/tcd.c
  ${CMAKE_CURRENT_SOURCE_DIR}/tgt.c
  ${CMAKE_CURRENT_SOURCE_DIR}/function_list.c
  ${CMAKE_CURRENT_SOURCE_DIR}/opj_malloc.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sparse_array.c
)

option(OPJ_DISABLE_TPSOT_FIX "Disable TPsot==TNsot fix. See https://github.com/uclouvain/openjpeg/issues/254." OFF)
if(OPJ_DISABLE_TPSOT_FIX)
  add_definitions(-DOPJ_DISABLE_TPSOT_FIX)
endif()

# Special case for old i586-mingw32msvc-gcc cross compiler
# if(NOT WIN32 AND CMAKE_COMPILER_IS_GNUCC AND CMAKE_C_COMPILER MATCHES ".*mingw32msvc.*" )
#   set(WIN32 YES)
# endif()

ocv_warnings_disable(CMAKE_C_FLAGS
    -Wundef -Wstrict-prototypes -Wcast-function-type
    -Wshadow   # v2.4.0: GCC
    -Wunused-function   # v2.4.0: Clang
)

ocv_warnings_disable(CMAKE_C_FLAGS /wd4819) # vs2019 Win64

add_library(${OPENJPEG_LIBRARY_NAME} STATIC ${OPENJPEG_SRCS})

target_compile_definitions(${OPENJPEG_LIBRARY_NAME} PUBLIC OPJ_STATIC)

ocv_include_directories("${CMAKE_CURRENT_LIST_DIR}" "${CMAKE_CURRENT_BINARY_DIR}")

if(UNIX)
  target_link_libraries(${OPENJPEG_LIBRARY_NAME} PRIVATE m)
endif()

set_target_properties(${OPENJPEG_LIBRARY_NAME}
  PROPERTIES
    ${OPENJPEG_LIBRARY_PROPERTIES}
)

#################################################################################
# threading configuration
#################################################################################

option(OPJ_USE_THREAD "Build with thread/mutex support " ON)
if(NOT OPJ_USE_THREAD)
  add_definitions(-DMUTEX_stub)
endif()

find_package(Threads QUIET)

if(OPJ_USE_THREAD AND WIN32 AND NOT Threads_FOUND )
  add_definitions(-DMUTEX_win32)
  set(Threads_FOUND YES)
endif()

if(OPJ_USE_THREAD AND Threads_FOUND AND CMAKE_USE_WIN32_THREADS_INIT)
  add_definitions(-DMUTEX_win32)
endif()

if(OPJ_USE_THREAD AND Threads_FOUND AND CMAKE_USE_PTHREADS_INIT )
  add_definitions(-DMUTEX_pthread)
endif()

if(OPJ_USE_THREAD AND NOT Threads_FOUND)
  message(STATUS "No thread library found and thread/mutex support is required by OPJ_USE_THREAD option")
  set(OCV_CAN_BUILD_OPENJPEG FALSE PARENT_SCOPE)
endif()

if(OPJ_USE_THREAD AND Threads_FOUND AND CMAKE_USE_PTHREADS_INIT)
  target_link_libraries(${OPENJPEG_LIBRARY_NAME} PRIVATE ${CMAKE_THREAD_LIBS_INIT})
endif()

if(NOT BUILD_SHARED_LIBS)
  ocv_install_target(${OPENJPEG_LIBRARY_NAME}
    EXPORT OpenCVModules
    ARCHIVE DESTINATION ${OPENCV_3P_LIB_INSTALL_PATH} COMPONENT dev
  )
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

