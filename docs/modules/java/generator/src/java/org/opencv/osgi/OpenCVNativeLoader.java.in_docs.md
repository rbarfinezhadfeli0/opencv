# Documentation for `modules/java/generator/src/java/org/opencv/osgi/OpenCVNativeLoader.java.in`

## File Metadata

- **Full Path**: `modules/java/generator/src/java/org/opencv/osgi/OpenCVNativeLoader.java.in`
- **File Name**: `OpenCVNativeLoader.java.in`
- **File Size**: 640 bytes
- **File Type**: .in
- **Link to Source**: [modules/java/generator/src/java/org/opencv/osgi/OpenCVNativeLoader.java.in](../../../../../../../../modules/java/generator/src/java/org/opencv/osgi/OpenCVNativeLoader.java.in)

## Purpose and Role

This file is located in the `modules/java/generator/src/java/org/opencv/osgi` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
package org.opencv.osgi;

import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * This class is intended to provide a convenient way to load OpenCV's native
 * library from the Java bundle. If Blueprint is enabled in the OSGi container
 * this class will be instantiated automatically and the init() method called
 * loading the native library.
 */
public class OpenCVNativeLoader implements OpenCVInterface {

    public void init() {
        System.loadLibrary("opencv_java@OPENCV_JAVA_LIB_NAME_SUFFIX@");
        Logger.getLogger("org.opencv.osgi").log(Level.INFO, "Successfully loaded OpenCV native library.");
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

