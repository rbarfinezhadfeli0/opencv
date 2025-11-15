# Documentation for `docs/platforms/linux/riscv64-andes-gcc.toolchain.cmake_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/linux/riscv64-andes-gcc.toolchain.cmake_docs.md`
- **File Name**: `riscv64-andes-gcc.toolchain.cmake_docs.md`
- **File Size**: 1,797 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/linux/riscv64-andes-gcc.toolchain.cmake_docs.md](../../../docs/platforms/linux/riscv64-andes-gcc.toolchain.cmake_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/linux` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/linux/riscv64-andes-gcc.toolchain.cmake`

## File Metadata

- **Full Path**: `platforms/linux/riscv64-andes-gcc.toolchain.cmake`
- **File Name**: `riscv64-andes-gcc.toolchain.cmake`
- **File Size**: 747 bytes
- **File Type**: .cmake
- **Link to Source**: [platforms/linux/riscv64-andes-gcc.toolchain.cmake](../../platforms/linux/riscv64-andes-gcc.toolchain.cmake)

## Purpose and Role

This file is located in the `platforms/linux` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR riscv64)

message(STATUS "RISCV: $ENV{RISCV}")
message(STATUS "RISCV_GCC_INSTALL_ROOT: $ENV{RISCV_GCC_INSTALL_ROOT}")

set(RISCV_GCC_INSTALL_ROOT $ENV{RISCV} CACHE PATH "Path to GCC for RISC-V cross compiler installation directory")

set(CMAKE_C_COMPILER  ${RISCV_GCC_INSTALL_ROOT}/bin/riscv64-linux-gcc)
set(CMAKE_CXX_COMPILER ${RISCV_GCC_INSTALL_ROOT}/bin/riscv64-linux-g++)

# fix toolchain macro
# enable rvp

set(CMAKE_C_FLAGS_INIT "-march=rv64gc -mext-dsp -D__ANDES=1")
set(CMAKE_CXX_FLAGS_INIT "-march=rv64gc -mext-dsp -D__ANDES=1")

# fix segment address

set(CMAKE_EXE_LINKER_FLAGS_INIT "-Wl,-Ttext-segment=0x50000")
set(CMAKE_SHARED_LINKER_FLAGS_INIT "-Wl,-Ttext-segment=0x50000")

```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

