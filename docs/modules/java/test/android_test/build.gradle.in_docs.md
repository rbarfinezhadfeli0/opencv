# Documentation for `modules/java/test/android_test/build.gradle.in`

## File Metadata

- **Full Path**: `modules/java/test/android_test/build.gradle.in`
- **File Name**: `build.gradle.in`
- **File Size**: 645 bytes
- **File Type**: .in
- **Link to Source**: [modules/java/test/android_test/build.gradle.in](../../../../modules/java/test/android_test/build.gradle.in)

## Purpose and Role

This file is located in the `modules/java/test/android_test` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {

    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:@ANDROID_GRADLE_PLUGIN_VERSION@'
        classpath 'org.jetbrains.kotlin:kotlin-gradle-plugin:@KOTLIN_PLUGIN_VERSION@'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

