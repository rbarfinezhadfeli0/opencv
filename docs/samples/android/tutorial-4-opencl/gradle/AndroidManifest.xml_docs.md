# Documentation for `samples/android/tutorial-4-opencl/gradle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/tutorial-4-opencl/gradle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,103 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/tutorial-4-opencl/gradle/AndroidManifest.xml](../../../../samples/android/tutorial-4-opencl/gradle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/tutorial-4-opencl/gradle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="org.opencv.samples.tutorial4"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-feature android:glEsVersion="0x00020000" android:required="true"/>
    <uses-feature android:name="android.hardware.camera"/>
    <uses-feature android:name="android.hardware.camera2" android:required="false"/>
    <uses-permission android:name="android.permission.CAMERA"/>

    <application
        android:allowBackup="true"
        android:icon="@drawable/icon">
        <activity
            android:exported="true"
            android:name=".Tutorial4Activity"
            android:label="@string/app_name"
            android:screenOrientation="landscape"
            android:configChanges="keyboardHidden|orientation">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

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

