# Documentation for `docs/samples/install/linux_install_b.sh_docs.md`

## File Metadata

- **Full Path**: `docs/samples/install/linux_install_b.sh_docs.md`
- **File Name**: `linux_install_b.sh_docs.md`
- **File Size**: 1,194 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/install/linux_install_b.sh_docs.md](../../../docs/samples/install/linux_install_b.sh_docs.md)

## Purpose and Role

This file is located in the `docs/samples/install` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/install/linux_install_b.sh`

## File Metadata

- **Full Path**: `samples/install/linux_install_b.sh`
- **File Name**: `linux_install_b.sh`
- **File Size**: 630 bytes
- **File Type**: .sh
- **Link to Source**: [samples/install/linux_install_b.sh](../../samples/install/linux_install_b.sh)

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

# [clang]
sudo apt install -y clang
# [clang]

# [ninja]
sudo apt install -y ninja-build
# [ninja]

# [cmake]
sudo apt install -y cmake
# [cmake]

# [git]
sudo apt install -y git
# [git]

# [download]
git clone https://github.com/opencv/opencv.git
git -C opencv checkout 4.x
# [download]

# [prepare]
mkdir -p build && cd build
# [prepare]

# [configure]
cmake -GNinja ../opencv
# [configure]

# [build]
ninja
# [build]

# [install]
sudo ninja install
# [install]

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

