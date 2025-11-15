# Documentation for `modules/core/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/core/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 845 bytes
- **File Type**: .json
- **Link to Source**: [modules/core/misc/js/gen_dict.json](../../../../modules/core/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/core/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": [
            "absdiff", "add", "addWeighted", "bitwise_and", "bitwise_not", "bitwise_or", "bitwise_xor", "cartToPolar",
            "compare", "convertScaleAbs", "copyMakeBorder", "countNonZero", "determinant", "dft", "divide", "eigen",
            "exp", "flip", "getOptimalDFTSize","gemm", "hconcat", "inRange", "invert", "kmeans", "log", "magnitude",
            "max", "mean", "meanStdDev", "merge", "min", "minMaxLoc", "mixChannels", "multiply", "norm", "normalize",
            "perspectiveTransform", "polarToCart", "pow", "randn", "randu", "reduce", "repeat", "rotate", "setIdentity", "setRNGSeed",
            "solve", "solvePoly", "split", "sqrt", "subtract", "trace", "transform", "transpose", "vconcat",
            "setLogLevel", "getLogLevel", "LUT"
        ],
        "Algorithm": []
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

