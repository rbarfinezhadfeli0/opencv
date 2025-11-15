# Documentation for `docs/samples/install/linux_install_a.sh_docs.md`

## File Metadata

- **Full Path**: `docs/samples/install/linux_install_a.sh_docs.md`
- **File Name**: `linux_install_a.sh_docs.md`
- **File Size**: 1,328 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/install/linux_install_a.sh_docs.md](../../../docs/samples/install/linux_install_a.sh_docs.md)

## Purpose and Role

This file is located in the `docs/samples/install` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/install/linux_install_a.sh`

## File Metadata

- **Full Path**: `samples/install/linux_install_a.sh`
- **File Name**: `linux_install_a.sh`
- **File Size**: 764 bytes
- **File Type**: .sh
- **Link to Source**: [samples/install/linux_install_a.sh](../../samples/install/linux_install_a.sh)

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

sudo apt update

# [gcc]
sudo apt install -y g++
# [gcc]

# [make]
sudo apt install -y make
# [make]

# [cmake]
sudo apt install -y cmake
# [cmake]

# [wget]
sudo apt install -y wget unzip
# [wget]

# [download]
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.zip
unzip opencv.zip
mv opencv-4.x opencv
# [download]

# [prepare]
mkdir -p build && cd build
# [prepare]

# [configure]
cmake ../opencv
# [configure]

# [build]
make -j4
# [build]

# [check]
ls bin
ls lib
# [check]

# [check cmake]
ls OpenCVConfig*.cmake
ls OpenCVModules.cmake
# [check cmake]

# [install]
sudo make install
# [install]

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

