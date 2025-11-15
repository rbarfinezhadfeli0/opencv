# Documentation for `modules/videoio/misc/plugin_gstreamer/build.sh`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_gstreamer/build.sh`
- **File Name**: `build.sh`
- **File Size**: 211 bytes
- **File Type**: .sh
- **Link to Source**: [modules/videoio/misc/plugin_gstreamer/build.sh](../../../../modules/videoio/misc/plugin_gstreamer/build.sh)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_gstreamer` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash

set -e

cmake -GNinja \
    -DOPENCV_PLUGIN_NAME=opencv_videoio_gstreamer \
    -DOPENCV_PLUGIN_DESTINATION=$1 \
    -DCMAKE_BUILD_TYPE=$2 \
    /opencv/modules/videoio/misc/plugin_gstreamer

ninja

```

## General Information

This file is part of the OpenCV repository infrastructure.

