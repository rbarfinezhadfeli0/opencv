# Documentation for `docs/samples/android/qr-detection/build.gradle.in_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/qr-detection/build.gradle.in_docs.md`
- **File Name**: `build.gradle.in_docs.md`
- **File Size**: 1,739 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/qr-detection/build.gradle.in_docs.md](../../../../docs/samples/android/qr-detection/build.gradle.in_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/qr-detection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/qr-detection/build.gradle.in`

## File Metadata

- **Full Path**: `samples/android/qr-detection/build.gradle.in`
- **File Name**: `build.gradle.in`
- **File Size**: 1,120 bytes
- **File Type**: .in
- **Link to Source**: [samples/android/qr-detection/build.gradle.in](../../../samples/android/qr-detection/build.gradle.in)

## Purpose and Role

This file is located in the `samples/android/qr-detection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
apply plugin: 'com.android.application'

android {
    namespace 'org.opencv.samples.qrdetection'
    compileSdkVersion @ANDROID_COMPILE_SDK_VERSION@
    defaultConfig {
        applicationId "org.opencv.samples.qrdetection"
        minSdkVersion @ANDROID_MIN_SDK_VERSION@
        targetSdkVersion @ANDROID_TARGET_SDK_VERSION@
        versionCode 301
        versionName "3.01"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets {
        main {
            java.srcDirs = @ANDROID_SAMPLE_JAVA_PATH@
            res.srcDirs = @ANDROID_SAMPLE_RES_PATH@
            manifest.srcFile '@ANDROID_SAMPLE_MANIFEST_PATH@'
        }
    }
}

dependencies {
    //implementation fileTree(dir: 'libs', include: ['*.jar'])
    if (gradle.opencv_source == "sdk_path") {
        implementation project(':opencv')
    } else if (gradle.opencv_source == "maven_local" || gradle.opencv_source == "maven_central") {
        implementation 'org.opencv:opencv:@OPENCV_VERSION_PLAIN@'
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

