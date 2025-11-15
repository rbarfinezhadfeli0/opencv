# Documentation for `apps/interactive-calibration/defaultConfig.xml`

## File Metadata

- **Full Path**: `apps/interactive-calibration/defaultConfig.xml`
- **File Name**: `defaultConfig.xml`
- **File Size**: 497 bytes
- **File Type**: .xml
- **Link to Source**: [apps/interactive-calibration/defaultConfig.xml](../../apps/interactive-calibration/defaultConfig.xml)

## Purpose and Role

This file is located in the `apps/interactive-calibration` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0"?>
<opencv_storage>
<charuco_dict>0</charuco_dict>
<charuco_square_length>200</charuco_square_length>
<charuco_marker_size>100</charuco_marker_size>
<calibration_step>1</calibration_step>
<max_frames_num>30</max_frames_num>
<min_frames_num>10</min_frames_num>
<solver_eps>1e-7</solver_eps>
<solver_max_iters>30</solver_max_iters>
<fast_solver>0</fast_solver>
<frame_filter_conv_param>0.1</frame_filter_conv_param>
<camera_resolution>800 600</camera_resolution>
</opencv_storage>
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

