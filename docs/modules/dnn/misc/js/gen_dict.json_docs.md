# Documentation for `modules/dnn/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/dnn/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 376 bytes
- **File Type**: .json
- **Link to Source**: [modules/dnn/misc/js/gen_dict.json](../../../../modules/dnn/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/dnn/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "dnn_Net": ["setInput", "forward", "setPreferableBackend","getUnconnectedOutLayersNames"],
        "": ["readNetFromCaffe", "readNetFromTensorflow", "readNetFromTorch", "readNetFromDarknet",
            "readNetFromONNX", "readNetFromTFLite", "readNet", "blobFromImage"]
    },
    "namespace_prefix_override":
    {
        "dnn": ""
    }
}

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

