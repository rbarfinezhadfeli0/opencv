# Documentation for `docs/platforms/android/aar-template/settings.gradle_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/android/aar-template/settings.gradle_docs.md`
- **File Name**: `settings.gradle_docs.md`
- **File Size**: 957 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/android/aar-template/settings.gradle_docs.md](../../../../docs/platforms/android/aar-template/settings.gradle_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/android/aar-template` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/android/aar-template/settings.gradle`

## File Metadata

- **Full Path**: `platforms/android/aar-template/settings.gradle`
- **File Name**: `settings.gradle`
- **File Size**: 326 bytes
- **File Type**: .gradle
- **Link to Source**: [platforms/android/aar-template/settings.gradle](../../../platforms/android/aar-template/settings.gradle)

## Purpose and Role

This file is located in the `platforms/android/aar-template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "OpenCV"
include ':OpenCV'

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

