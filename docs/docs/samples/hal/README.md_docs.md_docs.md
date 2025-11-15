# Documentation for `docs/samples/hal/README.md_docs.md`

## File Metadata

- **Full Path**: `docs/samples/hal/README.md_docs.md`
- **File Name**: `README.md_docs.md`
- **File Size**: 1,871 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/hal/README.md_docs.md](../../../docs/samples/hal/README.md_docs.md)

## Purpose and Role

This file is located in the `docs/samples/hal` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/hal/README.md`

## File Metadata

- **Full Path**: `samples/hal/README.md`
- **File Name**: `README.md`
- **File Size**: 1,326 bytes
- **File Type**: .md
- **Link to Source**: [samples/hal/README.md](../../samples/hal/README.md)

## Purpose and Role

This file is located in the `samples/hal` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

Custom HAL samples
==================

Samples in this folder are intended to demonstrate functionality replacement mechanism in the OpenCV library.

The __c_hal__ is the example of pure C replacement library with all functions returning error. It can be used to verify error handling in the function switching code.

The __slow_hal__ contains naive C++ implementations of the element-wise logical array operations (and, or, xor, not) making them twice slower than the default.

Build custom HAL replacement library
------------------------------------

1. Create folder for build (for example `<home-dir>/my-hal-build`)
2. Go to the created folder and run cmake: `cmake <opencv-src>/samples/hal/slow_hal`
3. Run make

After build you will find static library in the build folder: `libslow_hal.a`

Build OpenCV with HAL replacement
---------------------------------

1. Create folder for build (for example `<home-dir>/my-opencv-build`)
2. Go to the created folder and run cmake:
    ```
    cmake \
        -DOpenCV_HAL_DIR="<home-dir>/my-hal-build/" \
        <opencv-src>
    ```
3. Run make (or `make opencv_perf_core` to build the demonstration test executable only)
4. After build you can run the tests and verify that some functions works slower:
    ```
    ./bin/opencv_perf_core --gtest_filter=*bitwise_and*
    ```


## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

