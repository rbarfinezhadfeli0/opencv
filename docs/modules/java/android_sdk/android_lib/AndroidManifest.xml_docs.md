# Documentation for `modules/java/android_sdk/android_lib/AndroidManifest.xml`

## File Metadata

- **Full Path**: `modules/java/android_sdk/android_lib/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 222 bytes
- **File Type**: .xml
- **Link to Source**: [modules/java/android_sdk/android_lib/AndroidManifest.xml](../../../../modules/java/android_sdk/android_lib/AndroidManifest.xml)

## Purpose and Role

This file is located in the `modules/java/android_sdk/android_lib` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="org.opencv">

    <uses-sdk android:minSdkVersion="8" android:targetSdkVersion="21" />
</manifest>

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

