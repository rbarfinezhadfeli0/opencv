# Documentation for `samples/install/linux_quick_install_contrib.sh`

## File Metadata

- **Full Path**: `samples/install/linux_quick_install_contrib.sh`
- **File Name**: `linux_quick_install_contrib.sh`
- **File Size**: 700 bytes
- **File Type**: .sh
- **Link to Source**: [samples/install/linux_quick_install_contrib.sh](../../samples/install/linux_quick_install_contrib.sh)

## Purpose and Role

This file is located in the `samples/install` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#!/bin/bash
# This file contains documentation snippets for Linux installation tutorial
if [ "$1" = "--check" ] ; then
sudo()
{
    command $@
}
fi

# [body]
# Install minimal prerequisites (Ubuntu 18.04 as reference)
sudo apt update && sudo apt install -y cmake g++ wget unzip

# Download and unpack sources
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.x.zip
unzip opencv.zip
unzip opencv_contrib.zip

# Create build directory and switch into it
mkdir -p build && cd build

# Configure
cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-4.x/modules ../opencv-4.x

# Build
cmake --build .
# [body]

```

## General Information

This file is part of the OpenCV repository infrastructure.

