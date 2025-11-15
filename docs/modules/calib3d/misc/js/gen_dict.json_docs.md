# Documentation for `modules/calib3d/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/calib3d/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 553 bytes
- **File Type**: .json
- **Link to Source**: [modules/calib3d/misc/js/gen_dict.json](../../../../modules/calib3d/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/calib3d/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": [
            "findHomography",
            "calibrateCameraExtended",
            "drawFrameAxes",
            "estimateAffine2D",
            "getDefaultNewCameraMatrix",
            "initUndistortRectifyMap",
            "Rodrigues",
            "solvePnP",
            "solvePnPRansac",
            "solvePnPRefineLM",
            "projectPoints",
            "undistort",
            "fisheye_initUndistortRectifyMap",
            "fisheye_projectPoints"
        ],
        "UsacParams": ["UsacParams"]
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

