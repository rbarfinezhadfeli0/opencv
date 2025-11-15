# Documentation for `modules/dnn/src/webnn/README.md`

## File Metadata

- **Full Path**: `modules/dnn/src/webnn/README.md`
- **File Name**: `README.md`
- **File Size**: 652 bytes
- **File Type**: .md
- **Link to Source**: [modules/dnn/src/webnn/README.md](../../../../modules/dnn/src/webnn/README.md)

## Purpose and Role

This file is located in the `modules/dnn/src/webnn` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

## Build Instructions

### Build WebNN-native and set the environment variable

Refer to [WebNN's build instructions](https://github.com/webmachinelearning/webnn-native) to complete the build of WebNN-native.

Set environment variable `WEBNN_NATIVE_DIR` to enable native DNN_BACKEND_WEBNN build: `export WEBNN_NATIVE_DIR=${PATH_TO_WebNN}`. Please let `WEBNN_NATIVE_DIR` points the output directory of webnn-native build (e.g. webnn-native/out/Release).

### Test native DNN_BACKEND_WEBNN backend
Add -DWITH_WEBNN=ON to the cmake command to build the WebNN module such as:
`cmake -DWITH_WEBNN=ON ../opencv` (according to the @ref tutorial_linux_install)

## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

