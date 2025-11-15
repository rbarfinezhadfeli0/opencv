# Documentation for `modules/java/test/android_test/settings.gradle`

## File Metadata

- **Full Path**: `modules/java/test/android_test/settings.gradle`
- **File Name**: `settings.gradle`
- **File Size**: 147 bytes
- **File Type**: .gradle
- **Link to Source**: [modules/java/test/android_test/settings.gradle](../../../../modules/java/test/android_test/settings.gradle)

## Purpose and Role

This file is located in the `modules/java/test/android_test` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
rootProject.name = 'android_test'

include ':opencv'
project(':opencv').projectDir = new File('../opencv_android/opencv')

include ':tests_module'

```

## General Information

This file is part of the OpenCV repository infrastructure.

