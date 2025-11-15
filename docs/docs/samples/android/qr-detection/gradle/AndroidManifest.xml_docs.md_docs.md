# Documentation for `docs/samples/android/qr-detection/gradle/AndroidManifest.xml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/qr-detection/gradle/AndroidManifest.xml_docs.md`
- **File Name**: `AndroidManifest.xml_docs.md`
- **File Size**: 2,292 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/qr-detection/gradle/AndroidManifest.xml_docs.md](../../../../../docs/samples/android/qr-detection/gradle/AndroidManifest.xml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/qr-detection/gradle` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/qr-detection/gradle/AndroidManifest.xml`

## File Metadata

- **Full Path**: `samples/android/qr-detection/gradle/AndroidManifest.xml`
- **File Name**: `AndroidManifest.xml`
- **File Size**: 1,206 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/qr-detection/gradle/AndroidManifest.xml](../../../../samples/android/qr-detection/gradle/AndroidManifest.xml)

## Purpose and Role

This file is located in the `samples/android/qr-detection/gradle` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="org.opencv.samples.qrdetection">

    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name">

        <activity
            android:exported="true"
            android:name=".QRdetectionActivity"
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

