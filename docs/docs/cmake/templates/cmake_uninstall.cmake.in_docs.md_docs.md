# Documentation for `docs/cmake/templates/cmake_uninstall.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/cmake_uninstall.cmake.in_docs.md`
- **File Name**: `cmake_uninstall.cmake.in_docs.md`
- **File Size**: 1,747 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/cmake_uninstall.cmake.in_docs.md](../../../docs/cmake/templates/cmake_uninstall.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/cmake_uninstall.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/cmake_uninstall.cmake.in`
- **File Name**: `cmake_uninstall.cmake.in`
- **File Size**: 1,151 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/cmake_uninstall.cmake.in](../../cmake/templates/cmake_uninstall.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# -----------------------------------------------
# File that provides "make uninstall" target
#  We use the file 'install_manifest.txt'
#
# Details: https://gitlab.kitware.com/cmake/community/-/wikis/FAQ#can-i-do-make-uninstall-with-cmake
# -----------------------------------------------

if(NOT EXISTS "@CMAKE_BINARY_DIR@/install_manifest.txt")
  message(FATAL_ERROR "Cannot find install manifest: \"@CMAKE_BINARY_DIR@/install_manifest.txt\"")
endif()

file(READ "@CMAKE_BINARY_DIR@/install_manifest.txt" files)
string(REGEX REPLACE "\n" ";" files "${files}")
foreach(file ${files})
  message(STATUS "Uninstalling $ENV{DESTDIR}${file}")
  if(IS_SYMLINK "$ENV{DESTDIR}${file}" OR EXISTS "$ENV{DESTDIR}${file}")
    exec_program(
        "@CMAKE_COMMAND@" ARGS "-E remove \"$ENV{DESTDIR}${file}\""
        OUTPUT_VARIABLE rm_out
        RETURN_VALUE rm_retval
    )
    if(NOT "${rm_retval}" STREQUAL 0)
      message(FATAL_ERROR "Problem when removing $ENV{DESTDIR}${file}")
    endif()
  else(IS_SYMLINK "$ENV{DESTDIR}${file}" OR EXISTS "$ENV{DESTDIR}${file}")
    message(STATUS "File $ENV{DESTDIR}${file} does not exist.")
  endif()
endforeach()

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

