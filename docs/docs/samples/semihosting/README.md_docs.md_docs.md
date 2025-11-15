# Documentation for `docs/samples/semihosting/README.md_docs.md`

## File Metadata

- **Full Path**: `docs/samples/semihosting/README.md_docs.md`
- **File Name**: `README.md_docs.md`
- **File Size**: 1,534 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/semihosting/README.md_docs.md](../../../docs/samples/semihosting/README.md_docs.md)

## Purpose and Role

This file is located in the `docs/samples/semihosting` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/semihosting/README.md`

## File Metadata

- **Full Path**: `samples/semihosting/README.md`
- **File Name**: `README.md`
- **File Size**: 951 bytes
- **File Type**: .md
- **Link to Source**: [samples/semihosting/README.md](../../samples/semihosting/README.md)

## Purpose and Role

This file is located in the `samples/semihosting` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Arm semihosting

This folder contain a toolchain file and a couple of examples for
building OpenCV based applications that can run in an [Arm
semihosting](https://developer.arm.com/documentation/100863/latest)
setup.

OpenCV can be compiled to target a semihosting platform as follows:

```
cmake ../opencv/ \
    -DCMAKE_TOOLCHAIN_FILE=../opencv/platforms/semihosting/aarch64-semihosting.toolchain.cmake \
    -DSEMIHOSTING_TOOLCHAIN_PATH=/path/to/baremetal-toolchain/bin/ \
    -DBUILD_EXAMPLES=ON -GNinja
```

A barematel toolchain for targeting aarch64 semihosting can be found
[here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads),
under `aarch64-none-elf`.

The code of the examples in the `norm` and `histogram` folders can be
executed with qemu in Linux userspace:

```
    qemu-aarch64 ./bin/example_semihosting_histogram
    qemu-aarch64 ./bin/example_semihosting_norm
```


## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

