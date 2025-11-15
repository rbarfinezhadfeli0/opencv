# Documentation for `cmake/templates/xcode-launch-cxx.in`

## File Metadata

- **Full Path**: `cmake/templates/xcode-launch-cxx.in`
- **File Name**: `xcode-launch-cxx.in`
- **File Size**: 323 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/xcode-launch-cxx.in](../../cmake/templates/xcode-launch-cxx.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/sh
# https://crascit.com/2016/04/09/using-ccache-with-cmake/

# Xcode generator doesn't include the compiler as the
# first argument, Ninja and Makefiles do. Handle both cases.
if [[ "$1" = "${CMAKE_CXX_COMPILER}" ]] ; then
    shift
fi

export CCACHE_CPP2=true
exec "${CCACHE_PROGRAM}" "${CMAKE_CXX_COMPILER}" "$@"

```

## General Information

This file is part of the OpenCV repository infrastructure.

