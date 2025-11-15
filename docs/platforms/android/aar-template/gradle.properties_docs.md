# Documentation for `platforms/android/aar-template/gradle.properties`

## File Metadata

- **Full Path**: `platforms/android/aar-template/gradle.properties`
- **File Name**: `gradle.properties`
- **File Size**: 1,267 bytes
- **File Type**: .properties
- **Link to Source**: [platforms/android/aar-template/gradle.properties](../../../platforms/android/aar-template/gradle.properties)

## Purpose and Role

This file is located in the `platforms/android/aar-template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Project-wide Gradle settings.
# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.
# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html
# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true
# AndroidX package structure to make it clearer which packages are bundled with the
# Android operating system, and which are packaged with your app's APK
# https://developer.android.com/topic/libraries/support-library/androidx-rn
android.useAndroidX=true
# Enables namespacing of each library's R class so that its R class includes only the
# resources declared in the library itself and none from the library's dependencies,
# thereby reducing the size of the R class for that library
android.nonTransitiveRClass=true
```

## General Information

This file is part of the OpenCV repository infrastructure.

