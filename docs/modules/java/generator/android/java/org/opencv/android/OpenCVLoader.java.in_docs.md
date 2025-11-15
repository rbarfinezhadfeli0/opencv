# Documentation for `modules/java/generator/android/java/org/opencv/android/OpenCVLoader.java.in`

## File Metadata

- **Full Path**: `modules/java/generator/android/java/org/opencv/android/OpenCVLoader.java.in`
- **File Name**: `OpenCVLoader.java.in`
- **File Size**: 1,276 bytes
- **File Type**: .in
- **Link to Source**: [modules/java/generator/android/java/org/opencv/android/OpenCVLoader.java.in](../../../../../../../../modules/java/generator/android/java/org/opencv/android/OpenCVLoader.java.in)

## Purpose and Role

This file is located in the `modules/java/generator/android/java/org/opencv/android` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
package org.opencv.android;

import android.content.Context;

/**
 * Helper class provides common initialization methods for OpenCV library.
 */
public class OpenCVLoader
{
    /**
     * Current OpenCV Library version
     */
    public static final String OPENCV_VERSION = "@OPENCV_VERSION_MAJOR@.@OPENCV_VERSION_MINOR@.@OPENCV_VERSION_PATCH@";


    /**
     * Synonym for initLocal. Deprecated.
     */
    @Deprecated
    public static boolean initDebug()
    {
        return StaticHelper.initOpenCV(false);
    }

    /**
     * Loads and initializes OpenCV library from current application package. Roughly, it's an analog of system.loadLibrary("opencv_java").
     * @return Returns true is initialization of OpenCV was successful.
     */
    public static boolean initLocal()
    {
        return StaticHelper.initOpenCV(false);
    }

    /**
     * Loads and initializes OpenCV library from current application package. Roughly, it's an analog of system.loadLibrary("opencv_java").
     * @param InitCuda load and initialize CUDA runtime libraries.
     * @return Returns true is initialization of OpenCV was successful.
     */
    @Deprecated
    public static boolean initDebug(boolean InitCuda)
    {
        return StaticHelper.initOpenCV(InitCuda);
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

