# Documentation for `modules/objdetect/misc/java/test/QRCodeDetectorTest.java`

## File Metadata

- **Full Path**: `modules/objdetect/misc/java/test/QRCodeDetectorTest.java`
- **File Name**: `QRCodeDetectorTest.java`
- **File Size**: 3,363 bytes
- **File Type**: .java
- **Link to Source**: [modules/objdetect/misc/java/test/QRCodeDetectorTest.java](../../../../../modules/objdetect/misc/java/test/QRCodeDetectorTest.java)

## Purpose and Role

This file is located in the `modules/objdetect/misc/java/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.test.objdetect;

import java.util.List;
import org.opencv.core.Mat;
import org.opencv.core.Size;
import org.opencv.objdetect.QRCodeDetector;
import org.opencv.objdetect.QRCodeEncoder;
import org.opencv.objdetect.QRCodeEncoder_Params;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;
import org.opencv.test.OpenCVTestCase;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;

public class QRCodeDetectorTest extends OpenCVTestCase {

    private final static String ENV_OPENCV_TEST_DATA_PATH = "OPENCV_TEST_DATA_PATH";
    private String testDataPath;

    @Override
    protected void setUp() throws Exception {
        super.setUp();

        // relys on https://developer.android.com/reference/java/lang/System
        isTestCaseEnabled = System.getProperties().getProperty("java.vm.name") != "Dalvik";

        if (isTestCaseEnabled) {
            testDataPath = System.getenv(ENV_OPENCV_TEST_DATA_PATH);
            if (testDataPath == null)
                throw new Exception(ENV_OPENCV_TEST_DATA_PATH + " has to be defined!");
        }
    }

    public void testDetectAndDecode() {
        Mat img = Imgcodecs.imread(testDataPath + "/cv/qrcode/link_ocv.jpg");
        assertFalse(img.empty());
        QRCodeDetector detector = new QRCodeDetector();
        assertNotNull(detector);
        String output = detector.detectAndDecode(img);
        assertEquals(output, "https://opencv.org/");
    }

    public void testDetectAndDecodeMulti() {
        Mat img = Imgcodecs.imread(testDataPath + "/cv/qrcode/multiple/6_qrcodes.png");
        assertFalse(img.empty());
        QRCodeDetector detector = new QRCodeDetector();
        assertNotNull(detector);
        List < String > output = new ArrayList< String >();
        boolean result = detector.detectAndDecodeMulti(img, output);
        assertTrue(result);
        assertEquals(output.size(), 6);
        List < String > expectedResults = Arrays.asList("SKIP", "EXTRA", "TWO STEPS FORWARD", "STEP BACK", "QUESTION", "STEP FORWARD");
        assertEquals(new HashSet<String>(output), new HashSet<String>(expectedResults));
    }

    public void testKanji() {
        byte[] inp = new byte[]{(byte)0x82, (byte)0xb1, (byte)0x82, (byte)0xf1, (byte)0x82, (byte)0xc9, (byte)0x82,
                                (byte)0xbf, (byte)0x82, (byte)0xcd, (byte)0x90, (byte)0xa2, (byte)0x8a, (byte)0x45};
        QRCodeEncoder_Params params = new QRCodeEncoder_Params();
        params.set_mode(QRCodeEncoder.MODE_KANJI);
        QRCodeEncoder encoder = QRCodeEncoder.create(params);

        Mat qrcode = new Mat();
        encoder.encode(inp, qrcode);
        Imgproc.resize(qrcode, qrcode, new Size(0, 0), 2, 2, Imgproc.INTER_NEAREST);

        QRCodeDetector detector = new QRCodeDetector();
        byte[] output = detector.detectAndDecodeBytes(qrcode);
        assertEquals(detector.getEncoding(), QRCodeEncoder.ECI_SHIFT_JIS);
        assertArrayEquals(inp, output);

        List < byte[] > outputs = new ArrayList< byte[] >();
        assertTrue(detector.detectAndDecodeBytesMulti(qrcode, outputs));
        assertEquals(detector.getEncoding(0), QRCodeEncoder.ECI_SHIFT_JIS);
        assertArrayEquals(inp, outputs.get(0));
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

- **QRCodeDetectorTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.objdetect.QRCodeEncoder_Params`
- `java.util.HashSet`
- `org.opencv.test.OpenCVTestCase`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.objdetect.QRCodeEncoder`
- `java.util.List`
- `java.util.Arrays`
- `org.opencv.imgproc.Imgproc`
- `java.util.ArrayList`
- `java.io.UnsupportedEncodingException`
- `java.nio.charset.Charset`
- `org.opencv.core.Size`
- `org.opencv.objdetect.QRCodeDetector`
- `org.opencv.core.Mat`


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

