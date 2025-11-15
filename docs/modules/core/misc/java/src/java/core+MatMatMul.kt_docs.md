# Documentation for `modules/core/misc/java/src/java/core+MatMatMul.kt`

## File Metadata

- **Full Path**: `modules/core/misc/java/src/java/core+MatMatMul.kt`
- **File Name**: `core+MatMatMul.kt`
- **File Size**: 86 bytes
- **File Type**: .kt
- **Link to Source**: [modules/core/misc/java/src/java/core+MatMatMul.kt](../../../../../../modules/core/misc/java/src/java/core+MatMatMul.kt)

## Purpose and Role

This file is located in the `modules/core/misc/java/src/java` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
package org.opencv.core

operator fun Mat.times(other: Mat): Mat = this.matMul(other)

```

## General Information

This file is part of the OpenCV repository infrastructure.

