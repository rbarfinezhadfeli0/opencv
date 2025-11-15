# Documentation for `3rdparty/zlib-ng/cmake/detect-coverage.cmake`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/cmake/detect-coverage.cmake`
- **File Name**: `detect-coverage.cmake`
- **File Size**: 1,805 bytes
- **File Type**: .cmake
- **Link to Source**: [3rdparty/zlib-ng/cmake/detect-coverage.cmake](../../../3rdparty/zlib-ng/cmake/detect-coverage.cmake)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# detect-coverage.cmake -- Detect supported compiler coverage flags
# Licensed under the Zlib license, see LICENSE.md for details

macro(add_code_coverage)
    # Check for -coverage flag support for Clang/GCC
    if(CMAKE_VERSION VERSION_LESS 3.14)
        set(CMAKE_REQUIRED_LIBRARIES -lgcov)
    else()
        set(CMAKE_REQUIRED_LINK_OPTIONS -coverage)
    endif()
    check_c_compiler_flag(-coverage HAVE_COVERAGE)
    set(CMAKE_REQUIRED_LIBRARIES)
    set(CMAKE_REQUIRED_LINK_OPTIONS)

    if(HAVE_COVERAGE)
        add_compile_options(-coverage)
        add_link_options(-coverage)
        message(STATUS "Code coverage enabled using: -coverage")
    else()
        # Some versions of GCC don't support -coverage shorthand
        if(CMAKE_VERSION VERSION_LESS 3.14)
            set(CMAKE_REQUIRED_LIBRARIES -lgcov)
        else()
            set(CMAKE_REQUIRED_LINK_OPTIONS -lgcov -fprofile-arcs)
        endif()
        check_c_compiler_flag("-ftest-coverage -fprofile-arcs -fprofile-values" HAVE_TEST_COVERAGE)
        set(CMAKE_REQUIRED_LIBRARIES)
        set(CMAKE_REQUIRED_LINK_OPTIONS)

        if(HAVE_TEST_COVERAGE)
            add_compile_options(-ftest-coverage -fprofile-arcs -fprofile-values)
            add_link_options(-lgcov -fprofile-arcs)
            message(STATUS "Code coverage enabled using: -ftest-coverage")
        else()
            message(WARNING "Compiler does not support code coverage")
            set(WITH_CODE_COVERAGE OFF)
        endif()
    endif()

    # Set optimization level to zero for code coverage builds
    if (WITH_CODE_COVERAGE)
        # Use CMake compiler flag variables due to add_compile_options failure on Windows GCC
        set(CMAKE_C_FLAGS "-O0 ${CMAKE_C_FLAGS}")
        set(CMAKE_CXX_FLAGS "-O0 ${CMAKE_CXX_FLAGS}")
    endif()
endmacro()

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

