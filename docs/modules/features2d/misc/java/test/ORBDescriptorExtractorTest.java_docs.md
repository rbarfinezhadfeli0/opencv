# Documentation for `modules/features2d/misc/java/test/ORBDescriptorExtractorTest.java`

## File Metadata

- **Full Path**: `modules/features2d/misc/java/test/ORBDescriptorExtractorTest.java`
- **File Name**: `ORBDescriptorExtractorTest.java`
- **File Size**: 4,600 bytes
- **File Type**: .java
- **Link to Source**: [modules/features2d/misc/java/test/ORBDescriptorExtractorTest.java](../../../../../modules/features2d/misc/java/test/ORBDescriptorExtractorTest.java)

## Purpose and Role

This file is located in the `modules/features2d/misc/java/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.test.features2d;

import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.MatOfKeyPoint;
import org.opencv.core.Point;
import org.opencv.core.Scalar;
import org.opencv.core.KeyPoint;
import org.opencv.features2d.ORB;
import org.opencv.test.OpenCVTestCase;
import org.opencv.test.OpenCVTestRunner;
import org.opencv.imgproc.Imgproc;

public class ORBDescriptorExtractorTest extends OpenCVTestCase {

    ORB extractor;
    int matSize;

    public static void assertDescriptorsClose(Mat expected, Mat actual, int allowedDistance) {
        double distance = Core.norm(expected, actual, Core.NORM_HAMMING);
        assertTrue("expected:<" + allowedDistance + "> but was:<" + distance + ">", distance <= allowedDistance);
    }

    private Mat getTestImg() {
        Mat cross = new Mat(matSize, matSize, CvType.CV_8U, new Scalar(255));
        Imgproc.line(cross, new Point(20, matSize / 2), new Point(matSize - 21, matSize / 2), new Scalar(100), 2);
        Imgproc.line(cross, new Point(matSize / 2, 20), new Point(matSize / 2, matSize - 21), new Scalar(100), 2);

        return cross;
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        extractor = ORB.create();
        matSize = 100;
    }

    public void testComputeListOfMatListOfListOfKeyPointListOfMat() {
        fail("Not yet implemented");
    }

    public void testComputeMatListOfKeyPointMat() {
        KeyPoint point = new KeyPoint(55.775577545166016f, 44.224422454833984f, 16, 9.754629f, 8617.863f, 1, -1);
        MatOfKeyPoint keypoints = new MatOfKeyPoint(point);
        Mat img = getTestImg();
        Mat descriptors = new Mat();

        extractor.compute(img, keypoints, descriptors);

        Mat truth = new Mat(1, 32, CvType.CV_8UC1) {
            {
                put(0, 0,
                        6, 74, 6, 129, 2, 130, 56, 0, 44, 132, 66, 165, 172, 6, 3, 72, 102, 61, 171, 214, 0, 144, 65, 232, 4, 32, 138, 131, 4, 21, 37, 217);
            }
        };
        assertDescriptorsClose(truth, descriptors, 1);
    }

    public void testCreate() {
        assertNotNull(extractor);
    }

    public void testDescriptorSize() {
        assertEquals(32, extractor.descriptorSize());
    }

    public void testDescriptorType() {
        assertEquals(CvType.CV_8U, extractor.descriptorType());
    }

    public void testEmpty() {
//        assertFalse(extractor.empty());
        fail("Not yet implemented"); // ORB does not override empty() method
    }

    public void testReadYml() {
        KeyPoint point = new KeyPoint(55.775577545166016f, 44.224422454833984f, 16, 9.754629f, 8617.863f, 1, -1);
        MatOfKeyPoint keypoints = new MatOfKeyPoint(point);
        Mat img = getTestImg();
        Mat descriptors = new Mat();

        String filename = OpenCVTestRunner.getTempFileName("yml");
        writeFile(filename, "%YAML:1.0\n---\nnfeatures: 500\nscaleFactor: 1.1\nnlevels: 3\nedgeThreshold: 31\nfirstLevel: 0\nwta_k: 2\nscoreType: 0\npatchSize: 31\nfastThreshold: 20\n");
        extractor.read(filename);

        assertEquals(500, extractor.getMaxFeatures());
        assertEquals(1.1, extractor.getScaleFactor());
        assertEquals(3, extractor.getNLevels());
        assertEquals(31, extractor.getEdgeThreshold());
        assertEquals(0, extractor.getFirstLevel());
        assertEquals(2, extractor.getWTA_K());
        assertEquals(0, extractor.getScoreType());
        assertEquals(31, extractor.getPatchSize());
        assertEquals(20, extractor.getFastThreshold());

        extractor.compute(img, keypoints, descriptors);

        Mat truth = new Mat(1, 32, CvType.CV_8UC1) {
            {
                put(0, 0,
                        6, 10, 22, 5, 2, 130, 56, 0, 44, 164, 66, 165, 140, 6, 1, 72, 38, 61, 163, 210, 0, 208, 1, 104, 4, 32, 74, 131, 0, 37, 37, 67);
            }
        };
        assertDescriptorsClose(truth, descriptors, 1);
    }

    public void testWriteYml() {
        String filename = OpenCVTestRunner.getTempFileName("yml");

        extractor.write(filename);

        String truth = "%YAML:1.0\n---\nname: \"Feature2D.ORB\"\nnfeatures: 500\nscaleFactor: 1.2000000476837158\nnlevels: 8\nedgeThreshold: 31\nfirstLevel: 0\nwta_k: 2\nscoreType: 0\npatchSize: 31\nfastThreshold: 20\n";
//        String truth = "%YAML:1.0\n---\n";
        String actual = readFile(filename);
        actual = actual.replaceAll("e\\+000", "e+00"); // NOTE: workaround for different platforms double representation
        assertEquals(truth, actual);
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

- **ORBDescriptorExtractorTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `org.opencv.core.CvType`
- `org.opencv.features2d.ORB`
- `org.opencv.test.OpenCVTestCase`
- `org.opencv.core.MatOfKeyPoint`
- `org.opencv.core.Core`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.KeyPoint`
- `org.opencv.core.Mat`
- `org.opencv.core.Point`
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

