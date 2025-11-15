# Documentation for `doc/js_tutorials/js_assets/js_style_transfer_model_info.json`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_style_transfer_model_info.json`
- **File Name**: `js_style_transfer_model_info.json`
- **File Size**: 2,737 bytes
- **File Type**: .json
- **Link to Source**: [doc/js_tutorials/js_assets/js_style_transfer_model_info.json](../../../doc/js_tutorials/js_assets/js_style_transfer_model_info.json)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "torch": [
        {
            "model": "candy.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/candy.t7"
        },
        {
            "model": "composition_vii.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//eccv16/composition_vii.t7"
        },
        {
            "model": "feathers.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/feathers.t7"
        },
        {
            "model": "la_muse.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/la_muse.t7"
        },
        {
            "model": "mosaic.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/mosaic.t7"
        },
        {
            "model": "starry_night.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//eccv16/starry_night.t7"
        },
        {
            "model": "the_scream.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/the_scream.t7"
        },
        {
            "model": "the_wave.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//eccv16/the_wave.t7"
        },
        {
            "model": "udnie.t7",
            "inputSize": "224, 224",
            "mean": "104, 117, 123",
            "std": "1",
            "swapRB": "false",
            "modelUrl": "https://cs.stanford.edu/people/jcjohns/fast-neural-style/models//instance_norm/udnie.t7"
        }
    ]
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

