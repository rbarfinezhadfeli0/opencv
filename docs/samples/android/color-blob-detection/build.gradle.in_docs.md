# Documentation for `samples/android/color-blob-detection/build.gradle.in`

## File Metadata

- **Full Path**: `samples/android/color-blob-detection/build.gradle.in`
- **File Name**: `build.gradle.in`
- **File Size**: 1,220 bytes
- **File Type**: .in
- **Link to Source**: [samples/android/color-blob-detection/build.gradle.in](../../../samples/android/color-blob-detection/build.gradle.in)

## Purpose and Role

This file is located in the `samples/android/color-blob-detection` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
apply plugin: 'com.android.application'

android {
    namespace 'org.opencv.samples.colorblobdetect'
    compileSdkVersion @ANDROID_COMPILE_SDK_VERSION@
    defaultConfig {
        applicationId "org.opencv.samples.colorblobdetect"
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
        println 'Using OpenCV from from SDK'
        implementation project(':opencv')
    } else if (gradle.opencv_source == "maven_local" || gradle.opencv_source == "maven_central") {
        println 'Using OpenCV from Maven repo'
        implementation 'org.opencv:opencv:@OPENCV_VERSION_PLAIN@'
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

