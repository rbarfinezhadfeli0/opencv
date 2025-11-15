# Documentation for `samples/android/tutorial-2-mixedprocessing/gradle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/tutorial-2-mixedprocessing/gradle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,486 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/tutorial-2-mixedprocessing/gradle/AndroidManifest.xml](../../../../samples/android/tutorial-2-mixedprocessing/gradle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/tutorial-2-mixedprocessing/gradle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="org.opencv.samples.tutorial2"
>

    <application
        android:label="@string/app_name"
        android:icon="@drawable/icon">

        <activity
                  android:exported="true"
                  android:name="Tutorial2Activity"
                  android:label="@string/app_name"
                  android:screenOrientation="landscape"
                  android:configChanges="keyboardHidden|orientation">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <supports-screens android:resizeable="true"
                      android:smallScreens="true"
                      android:normalScreens="true"
                      android:largeScreens="true"
                      android:anyDensity="true" />

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

