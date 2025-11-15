# Documentation for `docs/samples/android/face-detection/gradle/AndroidManifest.xml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/face-detection/gradle/AndroidManifest.xml_docs.md`
- **File Name**: `AndroidManifest.xml_docs.md`
- **File Size**: 2,528 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/face-detection/gradle/AndroidManifest.xml_docs.md](../../../../../docs/samples/android/face-detection/gradle/AndroidManifest.xml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/face-detection/gradle` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/face-detection/gradle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/face-detection/gradle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,432 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/face-detection/gradle/AndroidManifest.xml](../../../../samples/android/face-detection/gradle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/face-detection/gradle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="org.opencv.samples.facedetect"
>

    <application
        android:label="@string/app_name"
        android:icon="@drawable/icon">

        <activity
                  android:exported="true"
                  android:name="FaceDetectActivity"
                  android:label="@string/app_name"
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

