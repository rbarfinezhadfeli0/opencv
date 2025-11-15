# Documentation for `modules/highgui/misc/plugins/plugin_gtk/build.sh`

## File Metadata

- **Full Path**: `modules/highgui/misc/plugins/plugin_gtk/build.sh`
- **File Name**: `build.sh`
- **File Size**: 419 bytes
- **File Type**: .sh
- **Link to Source**: [modules/highgui/misc/plugins/plugin_gtk/build.sh](../../../../../modules/highgui/misc/plugins/plugin_gtk/build.sh)

## Purpose and Role

This file is located in the `modules/highgui/misc/plugins/plugin_gtk` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash

set -e

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"

OPENCV_PLUGIN_DESTINATION=$1
OPENCV_PLUGIN_NAME=opencv_highgui_$2
CMAKE_BUILD_TYPE=${3:-Release}

shift 3 || true

set -x
cmake -GNinja \
    -DOPENCV_PLUGIN_NAME=${OPENCV_PLUGIN_NAME} \
    -DOPENCV_PLUGIN_DESTINATION=${OPENCV_PLUGIN_DESTINATION} \
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} \
    "$@" \
    $DIR

ninja -v

```

## General Information

This file is part of the OpenCV repository infrastructure.

