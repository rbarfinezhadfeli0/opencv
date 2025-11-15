# Documentation for `modules/objdetect/misc/java/test/ArucoTest.java`

## File Metadata

- **Full Path**: `modules/objdetect/misc/java/test/ArucoTest.java`
- **File Name**: `ArucoTest.java`
- **File Size**: 4,417 bytes
- **File Type**: .java
- **Link to Source**: [modules/objdetect/misc/java/test/ArucoTest.java](../../../../../modules/objdetect/misc/java/test/ArucoTest.java)

## Purpose and Role

This file is located in the `modules/objdetect/misc/java/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.test.aruco;

import java.util.ArrayList;
import java.util.List;

import org.opencv.test.OpenCVTestCase;
import org.junit.Assert;
import org.opencv.core.Scalar;
import org.opencv.core.Mat;
import org.opencv.core.MatOfInt;
import org.opencv.core.Size;
import org.opencv.core.CvType;
import org.opencv.objdetect.*;


public class ArucoTest extends OpenCVTestCase {

    public void testGenerateBoards() {
        Dictionary dictionary = Objdetect.getPredefinedDictionary(Objdetect.DICT_4X4_50);

        Mat point1 = new Mat(4, 3, CvType.CV_32FC1);
        int row = 0, col = 0;
        double squareLength = 40.;
        point1.put(row, col, 0, 0, 0,
                   0, squareLength, 0,
                   squareLength, squareLength, 0,
                   0, squareLength, 0);
        List<Mat>objPoints = new ArrayList<Mat>();
        objPoints.add(point1);

        Mat ids = new Mat(1, 1, CvType.CV_32SC1);
        ids.put(row, col, 0);

        Board board = new Board(objPoints, dictionary, ids);

        Mat image = new Mat();
        board.generateImage(new Size(80, 80), image, 2);

        assertTrue(image.total() > 0);
    }

    public void testArucoIssue3133() {
        byte[][] marker = {{0,1,1},{1,1,1},{0,1,1}};
        Dictionary dictionary = Objdetect.extendDictionary(1, 3);
        dictionary.set_maxCorrectionBits(0);
        Mat markerBits = new Mat(3, 3, CvType.CV_8UC1);
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                markerBits.put(i, j, marker[i][j]);
            }
        }

        Mat markerCompressed = Dictionary.getByteListFromBits(markerBits);
        assertMatNotEqual(markerCompressed, dictionary.get_bytesList());

        dictionary.set_bytesList(markerCompressed);
        assertMatEqual(markerCompressed, dictionary.get_bytesList());
    }

    public void testArucoDetector() {
        Dictionary dictionary = Objdetect.getPredefinedDictionary(0);
        DetectorParameters detectorParameters = new DetectorParameters();
        ArucoDetector detector = new ArucoDetector(dictionary, detectorParameters);

        Mat markerImage = new Mat();
        int id = 1, offset = 5, size = 40;
        Objdetect.generateImageMarker(dictionary, id, size, markerImage, detectorParameters.get_markerBorderBits());

        Mat image = new Mat(markerImage.rows() + 2*offset, markerImage.cols() + 2*offset,
                            CvType.CV_8UC1, new Scalar(255));
        Mat m = image.submat(offset, size+offset, offset, size+offset);
        markerImage.copyTo(m);

        List<Mat> corners = new ArrayList();
        Mat ids = new Mat();
        detector.detectMarkers(image, corners, ids);

        assertEquals(1, corners.size());
        Mat res = corners.get(0);
        assertArrayEquals(new double[]{offset, offset}, res.get(0, 0), 0.0);
        assertArrayEquals(new double[]{size + offset - 1, offset}, res.get(0, 1), 0.0);
        assertArrayEquals(new double[]{size + offset - 1, size + offset - 1}, res.get(0, 2), 0.0);
        assertArrayEquals(new double[]{offset, size + offset - 1}, res.get(0, 3), 0.0);
    }

    public void testCharucoDetector() {
        Dictionary dictionary = Objdetect.getPredefinedDictionary(0);
        int boardSizeX = 3, boardSizeY = 3;
        CharucoBoard board = new CharucoBoard(new Size(boardSizeX, boardSizeY), 1.f, 0.8f, dictionary);
        CharucoDetector charucoDetector = new CharucoDetector(board);

        int cellSize = 80;
        Mat boardImage = new Mat();
        board.generateImage(new Size(cellSize*boardSizeX, cellSize*boardSizeY), boardImage);

        assertTrue(boardImage.total() > 0);

        Mat charucoCorners = new Mat();
        Mat charucoIds = new Mat();
        charucoDetector.detectBoard(boardImage, charucoCorners, charucoIds);

        assertEquals(4, charucoIds.total());
        int[] intCharucoIds = (new MatOfInt(charucoIds)).toArray();
        Assert.assertArrayEquals(new int[]{0, 1, 2, 3}, intCharucoIds);

        double eps = 0.2;
        assertArrayEquals(new double[]{cellSize, cellSize}, charucoCorners.get(0, 0), eps);
        assertArrayEquals(new double[]{2*cellSize, cellSize}, charucoCorners.get(1, 0), eps);
        assertArrayEquals(new double[]{cellSize, 2*cellSize}, charucoCorners.get(2, 0), eps);
        assertArrayEquals(new double[]{2*cellSize, 2*cellSize}, charucoCorners.get(3, 0), eps);
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

- **ArucoTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `org.opencv.core.CvType`
- `org.junit.Assert`
- `org.opencv.test.OpenCVTestCase`
- `java.util.List`
- `java.util.ArrayList`
- `org.opencv.objdetect.`
- `org.opencv.core.Size`
- `org.opencv.core.MatOfInt`
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

