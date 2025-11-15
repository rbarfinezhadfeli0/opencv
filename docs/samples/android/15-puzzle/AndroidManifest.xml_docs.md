# Documentation for `samples/android/15-puzzle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/15-puzzle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,270 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/15-puzzle/AndroidManifest.xml](../../../samples/android/15-puzzle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/15-puzzle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="org.opencv.samples.puzzle15"
    android:versionCode="301"
    android:versionName="3.01" >

    <uses-sdk android:minSdkVersion="8"/>

    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name" >

        <activity
            android:name=".Puzzle15Activity"
            android:label="@string/app_name"
            android:screenOrientation="landscape"
            android:configChanges="keyboardHidden|orientation" >

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <uses-permission android:name="android.permission.CAMERA"/>

    <uses-feature android:name="android.hardware.camera" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.front" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.front.autofocus" android:required="false"/>

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

