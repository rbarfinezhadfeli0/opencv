# Documentation for `modules/java/test/android_test/src/org/opencv/test/android/KotlinTest.kt`

## File Metadata

- **Full Path**: `modules/java/test/android_test/src/org/opencv/test/android/KotlinTest.kt`
- **File Name**: `KotlinTest.kt`
- **File Size**: 611 bytes
- **File Type**: .kt
- **Link to Source**: [modules/java/test/android_test/src/org/opencv/test/android/KotlinTest.kt](../../../../../../../../../modules/java/test/android_test/src/org/opencv/test/android/KotlinTest.kt)

## Purpose and Role

This file is located in the `modules/java/test/android_test/src/org/opencv/test/android` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
package org.opencv.test.android

import org.opencv.core.CvType
import org.opencv.core.Mat
import org.opencv.core.times
import org.opencv.test.OpenCVTestCase
import kotlin.math.abs

class KotlinTest : OpenCVTestCase() {
    fun testMatrixMultiplication() {
        val m1 = Mat.ones(2, 3, CvType.CV_32F)
        val m2 = Mat.ones(3, 2, CvType.CV_32F)

        val m3 = m1.matMul(m2)
        val m4 = m1 * m2

        val value1 = floatArrayOf(3f)
        m3.get(0, 1, value1)

        val value2 = floatArrayOf(5f)
        m4[0, 1, value2]

        assertGE(0.001, abs(value1[0] - value2[0]).toDouble())
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

