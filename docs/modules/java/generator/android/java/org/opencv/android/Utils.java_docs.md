# Documentation for `modules/java/generator/android/java/org/opencv/android/Utils.java`

## File Metadata

- **Full Path**: `modules/java/generator/android/java/org/opencv/android/Utils.java`
- **File Name**: `Utils.java`
- **File Size**: 5,818 bytes
- **File Type**: .java
- **Link to Source**: [modules/java/generator/android/java/org/opencv/android/Utils.java](../../../../../../../../modules/java/generator/android/java/org/opencv/android/Utils.java)

## Purpose and Role

This file is located in the `modules/java/generator/android/java/org/opencv/android` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.android;

import android.content.Context;
import android.graphics.Bitmap;

import org.opencv.core.CvException;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.imgcodecs.Imgcodecs;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;

public class Utils {

    public static String exportResource(Context context, int resourceId) {
        return exportResource(context, resourceId, "OpenCV_data");
    }

    public static String exportResource(Context context, int resourceId, String dirname) {
        String fullname = context.getResources().getString(resourceId);
        String resName = fullname.substring(fullname.lastIndexOf("/") + 1);
        try {
            InputStream is = context.getResources().openRawResource(resourceId);
            File resDir = context.getDir(dirname, Context.MODE_PRIVATE);
            File resFile = new File(resDir, resName);

            FileOutputStream os = new FileOutputStream(resFile);

            byte[] buffer = new byte[4096];
            int bytesRead;
            while ((bytesRead = is.read(buffer)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
            is.close();
            os.close();

            return resFile.getAbsolutePath();
        } catch (IOException e) {
            e.printStackTrace();
            throw new CvException("Failed to export resource " + resName
                    + ". Exception thrown: " + e);
        }
    }

    public static Mat loadResource(Context context, int resourceId) throws IOException
    {
        return loadResource(context, resourceId, -1);
    }

    public static Mat loadResource(Context context, int resourceId, int flags) throws IOException
    {
        InputStream is = context.getResources().openRawResource(resourceId);
        ByteArrayOutputStream os = new ByteArrayOutputStream(is.available());

        byte[] buffer = new byte[4096];
        int bytesRead;
        while ((bytesRead = is.read(buffer)) != -1) {
            os.write(buffer, 0, bytesRead);
        }
        is.close();

        Mat encoded = new Mat(1, os.size(), CvType.CV_8U);
        encoded.put(0, 0, os.toByteArray());
        os.close();

        Mat decoded = Imgcodecs.imdecode(encoded, flags);
        encoded.release();

        return decoded;
    }

    /**
     * Converts Android Bitmap to OpenCV Mat.
     * <p>
     * This function converts an Android Bitmap image to the OpenCV Mat.
     * <br>'ARGB_8888' and 'RGB_565' input Bitmap formats are supported.
     * <br>The output Mat is always created of the same size as the input Bitmap and of the 'CV_8UC4' type,
     * it keeps the image in RGBA format.
     * <br>This function throws an exception if the conversion fails.
     * @param bmp is a valid input Bitmap object of the type 'ARGB_8888' or 'RGB_565'.
     * @param mat is a valid output Mat object, it will be reallocated if needed, so it may be empty.
     * @param unPremultiplyAlpha is a flag, that determines, whether the bitmap needs to be converted from alpha premultiplied format (like Android keeps 'ARGB_8888' ones) to regular one; this flag is ignored for 'RGB_565' bitmaps.
     */
    public static void bitmapToMat(Bitmap bmp, Mat mat, boolean unPremultiplyAlpha) {
        if (bmp == null)
            throw new IllegalArgumentException("bmp == null");
        if (mat == null)
            throw new IllegalArgumentException("mat == null");
        nBitmapToMat2(bmp, mat.nativeObj, unPremultiplyAlpha);
    }

    /**
     * Short form of the bitmapToMat(bmp, mat, unPremultiplyAlpha=false).
     * @param bmp is a valid input Bitmap object of the type 'ARGB_8888' or 'RGB_565'.
     * @param mat is a valid output Mat object, it will be reallocated if needed, so Mat may be empty.
     */
    public static void bitmapToMat(Bitmap bmp, Mat mat) {
        bitmapToMat(bmp, mat, false);
    }


    /**
     * Converts OpenCV Mat to Android Bitmap.
     * <p>
     * <br>This function converts an image in the OpenCV Mat representation to the Android Bitmap.
     * <br>The input Mat object has to be of the types 'CV_8UC1' (gray-scale), 'CV_8UC3' (RGB) or 'CV_8UC4' (RGBA).
     * <br>The output Bitmap object has to be of the same size as the input Mat and of the types 'ARGB_8888' or 'RGB_565'.
     * <br>This function throws an exception if the conversion fails.
     *
     * @param mat is a valid input Mat object of types 'CV_8UC1', 'CV_8UC3' or 'CV_8UC4'.
     * @param bmp is a valid Bitmap object of the same size as the Mat and of type 'ARGB_8888' or 'RGB_565'.
     * @param premultiplyAlpha is a flag, that determines, whether the Mat needs to be converted to alpha premultiplied format (like Android keeps 'ARGB_8888' bitmaps); the flag is ignored for 'RGB_565' bitmaps.
     */
    public static void matToBitmap(Mat mat, Bitmap bmp, boolean premultiplyAlpha) {
        if (mat == null)
            throw new IllegalArgumentException("mat == null");
        if (bmp == null)
            throw new IllegalArgumentException("bmp == null");
        nMatToBitmap2(mat.nativeObj, bmp, premultiplyAlpha);
    }

    /**
     * Short form of the <b>matToBitmap(mat, bmp, premultiplyAlpha=false)</b>
     * @param mat is a valid input Mat object of the types 'CV_8UC1', 'CV_8UC3' or 'CV_8UC4'.
     * @param bmp is a valid Bitmap object of the same size as the Mat and of type 'ARGB_8888' or 'RGB_565'.
     */
    public static void matToBitmap(Mat mat, Bitmap bmp) {
        matToBitmap(mat, bmp, false);
    }


    private static native void nBitmapToMat2(Bitmap b, long m_addr, boolean unPremultiplyAlpha);

    private static native void nMatToBitmap2(long m_addr, Bitmap b, boolean premultiplyAlpha);
}
```

## High-Level Overview

This is a Java file that provides Java bindings or Android support for OpenCV.

**Key Characteristics:**
- Implements Java API for OpenCV functionality
- May use JNI to interface with native code
- Follows Java coding conventions
- Part of the OpenCV Java/Android SDK


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **Utils**: A class/struct defined in this file

### Functions and Methods

- **converts()**: A function/method defined in this file
- **throws()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `android.graphics.Bitmap`
- `org.opencv.core.CvException`
- `java.io.File`
- `org.opencv.core.CvType`
- `java.io.IOException`
- `java.io.InputStream`
- `android.content.Context`
- `org.opencv.imgcodecs.Imgcodecs`
- `java.io.ByteArrayOutputStream`
- `alpha`
- `org.opencv.core.Mat`
- `java.io.FileOutputStream`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

