# Documentation for `platforms/ios/cmake/Toolchains/xcodebuild_wrapper.in`

## File Metadata

- **Full Path**: `platforms/ios/cmake/Toolchains/xcodebuild_wrapper.in`
- **File Name**: `xcodebuild_wrapper.in`
- **File Size**: 465 bytes
- **File Type**: .in
- **Link to Source**: [platforms/ios/cmake/Toolchains/xcodebuild_wrapper.in](../../../../platforms/ios/cmake/Toolchains/xcodebuild_wrapper.in)

## Purpose and Role

This file is located in the `platforms/ios/cmake/Toolchains` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/sh

# Force 'Debug' configuration
# Details: https://github.com/opencv/opencv/issues/13856
if [[ "$@" =~ "-project CMAKE_TRY_COMPILE.xcodeproj" && -z "${OPENCV_SKIP_XCODEBUILD_FORCE_TRYCOMPILE_DEBUG}" ]]; then
  ARGS=()
  for ((i=1; i<=$#; i++))
  do
    arg=${!i}
    ARGS+=("$arg")
    if [[ "$arg" == "-configuration" ]]; then
      ARGS+=("Debug")
      i=$(($i+1))
    fi
  done
  set -- "${ARGS[@]}"
fi

@CMAKE_MAKE_PROGRAM@ @XCODEBUILD_EXTRA_ARGS@ $*

```

## General Information

This file is part of the OpenCV repository infrastructure.

