# Documentation for `modules/java/test/android_test/tests_module/build.gradle.in`

## File Metadata

- **Full Path**: `modules/java/test/android_test/tests_module/build.gradle.in`
- **File Name**: `build.gradle.in`
- **File Size**: 1,005 bytes
- **File Type**: .in
- **Link to Source**: [modules/java/test/android_test/tests_module/build.gradle.in](../../../../../modules/java/test/android_test/tests_module/build.gradle.in)

## Purpose and Role

This file is located in the `modules/java/test/android_test/tests_module` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
    namespace 'org.opencv.tests'
    compileSdkVersion @ANDROID_COMPILE_SDK_VERSION@
    defaultConfig {
        applicationId "org.opencv.tests"
        minSdkVersion @ANDROID_MIN_SDK_VERSION@
        targetSdkVersion @ANDROID_TARGET_SDK_VERSION@
        versionCode 301
        versionName "3.01"

        testInstrumentationRunner "org.opencv.test.OpenCVTestRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets {
        androidTest {
            java.srcDirs = [@ANDROID_TESTS_SRC_DIRS@]
        }
        main {
            manifest.srcFile 'AndroidManifest.xml'
            res.srcDirs = [@ANDROID_TESTS_RES_DIR@]
        }
    }
}

dependencies {
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
    implementation project(':opencv')
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

