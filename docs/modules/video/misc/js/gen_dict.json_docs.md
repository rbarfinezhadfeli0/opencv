# Documentation for `modules/video/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/video/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 466 bytes
- **File Type**: .json
- **Link to Source**: [modules/video/misc/js/gen_dict.json](../../../../modules/video/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/video/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": [
            "CamShift",
            "calcOpticalFlowFarneback",
            "calcOpticalFlowPyrLK",
            "createBackgroundSubtractorMOG2",
            "findTransformECC",
            "meanShift"
        ],
        "BackgroundSubtractorMOG2": ["BackgroundSubtractorMOG2", "apply"],
        "BackgroundSubtractor": ["apply", "getBackgroundImage"],
        "TrackerMIL": ["create"],
        "TrackerMIL_Params": []
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

