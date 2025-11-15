# Documentation for `modules/videoio/misc/plugin_ffmpeg/build-ubuntu.sh`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_ffmpeg/build-ubuntu.sh`
- **File Name**: `build-ubuntu.sh`
- **File Size**: 214 bytes
- **File Type**: .sh
- **Link to Source**: [modules/videoio/misc/plugin_ffmpeg/build-ubuntu.sh](../../../../modules/videoio/misc/plugin_ffmpeg/build-ubuntu.sh)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_ffmpeg` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash

set -e

cmake -GNinja \
    -DOPENCV_PLUGIN_NAME=opencv_videoio_ffmpeg_ubuntu_$2 \
    -DOPENCV_PLUGIN_DESTINATION=$1 \
    -DCMAKE_BUILD_TYPE=$3 \
    /opencv/modules/videoio/misc/plugin_ffmpeg
ninja

```

## General Information

This file is part of the OpenCV repository infrastructure.

