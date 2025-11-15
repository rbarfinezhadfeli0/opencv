# Documentation for `modules/java/android_sdk/android_gradle_lib/res/values/attrs.xml`

## File Metadata

- **Full Path**: `modules/java/android_sdk/android_gradle_lib/res/values/attrs.xml`
- **File Name**: `attrs.xml`
- **File Size**: 382 bytes
- **File Type**: .xml
- **Link to Source**: [modules/java/android_sdk/android_gradle_lib/res/values/attrs.xml](../../../../../../modules/java/android_sdk/android_gradle_lib/res/values/attrs.xml)

## Purpose and Role

This file is located in the `modules/java/android_sdk/android_gradle_lib/res/values` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name = "CameraBridgeViewBase" >
       <attr name="show_fps" format="boolean"/>
       <attr name="camera_id" format="integer" >
          <enum name="any" value="-1" />
          <enum name="back" value="99" />
          <enum name="front" value="98" />
       </attr>
    </declare-styleable>
</resources>

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

