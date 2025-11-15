# Documentation for `modules/java/test/android_test/src/org/opencv/test/android/UtilsTest.java`

## File Metadata

- **Full Path**: `modules/java/test/android_test/src/org/opencv/test/android/UtilsTest.java`
- **File Name**: `UtilsTest.java`
- **File Size**: 8,674 bytes
- **File Type**: .java
- **Link to Source**: [modules/java/test/android_test/src/org/opencv/test/android/UtilsTest.java](../../../../../../../../../modules/java/test/android_test/src/org/opencv/test/android/UtilsTest.java)

## Purpose and Role

This file is located in the `modules/java/test/android_test/src/org/opencv/test/android` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.test.android;

import org.opencv.android.Utils;
import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.Scalar;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;
import org.opencv.test.OpenCVTestCase;
import org.opencv.test.OpenCVTestRunner;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Color;
import android.util.Log;

public class UtilsTest extends OpenCVTestCase {

    private int[] testImgWH = new int[]{64, 48};
    private byte[] testImgBgColor = new byte[]{1, 2, 3};
    private int[] testImgRect = new int[] {15, 17, 25, 37};
    private byte[] testImgRectColor = new byte[]{45, 15, 67};

    private Mat createTestBGRImg() {
        Mat img = new Mat(testImgWH[1], testImgWH[0], CvType.CV_8UC3,
                          new Scalar(testImgBgColor[2], testImgBgColor[1], testImgBgColor[0]));
        byte[] color = new byte[]{testImgRectColor[2], testImgRectColor[1], testImgRectColor[0]};

        for (int i = testImgRect[1]; i < testImgRect[3]; i++) {
            for (int j = testImgRect[0]; j < testImgRect[2]; j++) {
                img.put(i, j, color);
            }
        }
        return img;
    }

    private Bitmap createTestBitmap() {
        Bitmap img = Bitmap.createBitmap(testImgWH[0], testImgWH[1], Bitmap.Config.ARGB_8888);
        img.eraseColor(Color.argb(255, testImgBgColor[0], testImgBgColor[1] ,testImgBgColor[2]));

        for (int i = testImgRect[1]; i < testImgRect[3]; i++) {
            for (int j = testImgRect[0]; j < testImgRect[2]; j++) {
                img.setPixel(j, i, Color.argb(
                        255, testImgRectColor[0], testImgRectColor[1], testImgRectColor[2]));
            }
        }
        return img;
    }

    public void testMatBitmapConversion() {
        Mat mat = new Mat();
        Imgproc.cvtColor(createTestBGRImg(), mat, Imgproc.COLOR_BGR2RGBA);
        Bitmap bmp = createTestBitmap();

        Bitmap convertedBmp = Bitmap.createBitmap(
                Bitmap.createBitmap(testImgWH[0], testImgWH[1], Bitmap.Config.ARGB_8888));
        Utils.matToBitmap(mat, convertedBmp);
        assertTrue(bmp.sameAs(convertedBmp));

        Mat convertedMat = new Mat();
        Utils.bitmapToMat(bmp, convertedMat);
        Mat diff = new Mat();
        Core.absdiff(mat, convertedMat, diff);
        Scalar channelsDiff = Core.sumElems(diff);
        assertEquals(0.0, channelsDiff.val[0]);
        assertEquals(0.0, channelsDiff.val[1]);
        assertEquals(0.0, channelsDiff.val[2]);
        assertEquals(0.0, channelsDiff.val[3]);
    }


    public void testBitmapToMat() {
        BitmapFactory.Options opt16 = new BitmapFactory.Options();
        opt16.inPreferredConfig = Bitmap.Config.RGB_565;
        Bitmap bmp16 = BitmapFactory.decodeFile(OpenCVTestRunner.LENA_PATH, opt16);
        Mat m16 = new Mat();
        Utils.bitmapToMat(bmp16, m16);
        assertTrue(m16.rows() == 512 && m16.cols() == 512 && m16.type() == CvType.CV_8UC4);

        /*BitmapFactory.Options opt32 = new BitmapFactory.Options();
        opt32.inPreferredConfig = Bitmap.Config.ARGB_8888;
        Bitmap bmp32 = BitmapFactory.decodeFile(OpenCVTestRunner.LENA_PATH, opt32);*/
        Bitmap bmp32 = bmp16.copy(Bitmap.Config.ARGB_8888, false);
        Mat m32 = new Mat();
        Utils.bitmapToMat(bmp32, m32);

        assertTrue(m16.rows() == m32.rows() && m16.cols() == m32.cols() && m16.type() == m32.type());

        double maxDiff = Core.norm(m16, m32, Core.NORM_INF);
        Log.d("Bmp->Mat", "bmp16->Mat vs bmp32->Mat diff = " + maxDiff);

        assertTrue(maxDiff <= 8 /* 8 == 2^8 / 2^5 */);
    }

    public void testExportResourceContextInt() {
        fail("Not yet implemented");
    }

    public void testExportResourceContextIntString() {
        fail("Not yet implemented");
    }

    public void testLoadResourceContextInt() {
        fail("Not yet implemented");
    }

    public void testLoadResourceContextIntInt() {
        fail("Not yet implemented");
    }

    public void testMatToBitmap() {
        Mat imgBGR = Imgcodecs.imread( OpenCVTestRunner.LENA_PATH );
        assertTrue(imgBGR != null && !imgBGR.empty() && imgBGR.channels() == 3);

        Mat m16 = new Mat(imgBGR.rows(), imgBGR.cols(), CvType.CV_8UC4);
        Mat m32 = new Mat(imgBGR.rows(), imgBGR.cols(), CvType.CV_8UC4);

        Bitmap bmp16 = Bitmap.createBitmap(imgBGR.cols(), imgBGR.rows(), Bitmap.Config.RGB_565);
        Bitmap bmp32 = Bitmap.createBitmap(imgBGR.cols(), imgBGR.rows(), Bitmap.Config.ARGB_8888);

        double maxDiff;
        Scalar s0 = new Scalar(0);
        Scalar s255 = Scalar.all(255);


        // RGBA
        Mat imgRGBA = new Mat();
        Imgproc.cvtColor(imgBGR, imgRGBA, Imgproc.COLOR_BGR2RGBA);
        assertFalse(imgRGBA.empty() && imgRGBA.channels() == 4);

        bmp16.eraseColor(Color.BLACK); m16.setTo(s0);
        Utils.matToBitmap(imgRGBA, bmp16); Utils.bitmapToMat(bmp16, m16);
        maxDiff = Core.norm(imgRGBA, m16, Core.NORM_INF);
        Log.d("RGBA->bmp16->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff <= 8 /* 8 == 2^8 / 2^5 */);

        bmp32.eraseColor(Color.WHITE); m32.setTo(s255);
        Utils.matToBitmap(imgRGBA, bmp32); Utils.bitmapToMat(bmp32, m32);
        maxDiff = Core.norm(imgRGBA, m32, Core.NORM_INF);
        Log.d("RGBA->bmp32->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff == 0);


        // RGB
        Mat imgRGB = new Mat();
        Imgproc.cvtColor(imgBGR, imgRGB, Imgproc.COLOR_BGR2RGB);
        assertFalse(imgRGB.empty() && imgRGB.channels() == 3);

        bmp16.eraseColor(Color.BLACK); m16.setTo(s0);
        Utils.matToBitmap(imgRGB, bmp16); Utils.bitmapToMat(bmp16, m16);
        maxDiff = Core.norm(imgRGBA, m16, Core.NORM_INF);
        Log.d("RGB->bmp16->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff <= 8 /* 8 == 2^8 / 2^5 */);

        bmp32.eraseColor(Color.WHITE); m32.setTo(s255);
        Utils.matToBitmap(imgRGB, bmp32); Utils.bitmapToMat(bmp32, m32);
        maxDiff = Core.norm(imgRGBA, m32, Core.NORM_INF);
        Log.d("RGB->bmp32->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff == 0);


        // Gray
        Mat imgGray = new Mat();
        Imgproc.cvtColor(imgBGR, imgGray, Imgproc.COLOR_BGR2GRAY);
        assertFalse(imgGray.empty() && imgGray.channels() == 1);
        Mat tmp = new Mat();

        bmp16.eraseColor(Color.BLACK); m16.setTo(s0);
        Utils.matToBitmap(imgGray, bmp16); Utils.bitmapToMat(bmp16, m16);
        Core.extractChannel(m16, tmp, 0);
        maxDiff = Core.norm(imgGray, tmp, Core.NORM_INF);
        Log.d("Gray->bmp16->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff <= 8 /* 8 == 2^8 / 2^5 */);

        bmp32.eraseColor(Color.WHITE); m32.setTo(s255);
        Utils.matToBitmap(imgGray, bmp32); Utils.bitmapToMat(bmp32, m32);
        tmp.setTo(s0);
        Core.extractChannel(m32, tmp, 0);
        maxDiff = Core.norm(imgGray, tmp, Core.NORM_INF);
        Log.d("Gray->bmp32->RGBA", "maxDiff = " + maxDiff);
        assertTrue(maxDiff == 0);

    }

    public void testAlphaPremultiplication() {
        final int size = 256;
        Bitmap bmp = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        Mat mOrig  = new Mat(size, size, CvType.CV_8UC4);
        Mat mUnPre = new Mat(size, size, CvType.CV_8UC4);
        for(int y=0; y<size; y++) {
            int a = y;
            for(int x=0; x<size; x++) {
                int color = Color.argb(a, 0, x, y);
                bmp.setPixel(x, y, color);
                mOrig.put(y, x, Color.red(color), Color.green(color), Color.blue(color), Color.alpha(color));
                int colorUnPre = bmp.getPixel(x, y);
                mUnPre.put(y, x, Color.red(colorUnPre), Color.green(colorUnPre), Color.blue(colorUnPre), Color.alpha(colorUnPre));
            }
        }

        // Bitmap -> Mat
        Mat m1 = new Mat();
        Mat m2 = new Mat();

        Utils.bitmapToMat(bmp, m1, false);
        Imgproc.cvtColor(mOrig, m2, Imgproc.COLOR_RGBA2mRGBA);
        assertMatEqual(m1, m2, 1.1);

        Utils.bitmapToMat(bmp, m1, true);
        assertMatEqual(m1, mUnPre, 1.1);

        // Mat -> Bitmap
        Bitmap bmp1 = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);

        Utils.matToBitmap(mOrig, bmp1, true);
        Utils.bitmapToMat(bmp1, m1, true);
        //assertMatEqual(m1, mUnPre, 1.1);
        Mat diff = new Mat();
        Core.absdiff(m1, mUnPre, diff);
        int numDiff = Core.countNonZero(diff.reshape(1));
        assertTrue(numDiff < size * 4);
    }

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

- **UtilsTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `android.graphics.Bitmap`
- `org.opencv.core.Scalar`
- `org.opencv.core.CvType`
- `android.graphics.BitmapFactory`
- `org.opencv.android.Utils`
- `org.opencv.test.OpenCVTestCase`
- `org.opencv.imgcodecs.Imgcodecs`
- `android.util.Log`
- `org.opencv.core.Core`
- `org.opencv.imgproc.Imgproc`
- `android.graphics.Color`
- `org.opencv.core.Mat`
- `org.opencv.test.OpenCVTestRunner`


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

