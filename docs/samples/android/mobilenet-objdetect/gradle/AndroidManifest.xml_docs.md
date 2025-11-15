# Documentation for `samples/android/mobilenet-objdetect/gradle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/mobilenet-objdetect/gradle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,251 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/mobilenet-objdetect/gradle/AndroidManifest.xml](../../../../samples/android/mobilenet-objdetect/gradle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/mobilenet-objdetect/gradle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="org.opencv.samples.opencv_mobilenet">

    <application
        android:label="@string/app_name"
        android:icon="@drawable/icon">
        <!-- //! [mobilenet_tutorial] -->
        <activity
                  android:exported="true"
                  android:name=".MainActivity"
                  android:screenOrientation="landscape">  <!--Screen orientation-->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <!--Allow to use a camera-->
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-feature android:name="android.hardware.camera" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.front" android:required="false"/>
    <uses-feature android:name="android.hardware.camera.front.autofocus" android:required="false"/>

</manifest>
<!-- //! [mobilenet_tutorial] -->

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

