# Documentation for `docs/samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java_docs.md`
- **File Name**: `PointPolygonTestDemo.java_docs.md`
- **File Size**: 7,277 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java_docs.md](../../../../../../docs/samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/ShapeDescriptors/point_polygon_test` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java`
- **File Name**: `PointPolygonTestDemo.java`
- **File Size**: 3,633 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java](../../../../../samples/java/tutorial_code/ShapeDescriptors/point_polygon_test/PointPolygonTestDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/ShapeDescriptors/point_polygon_test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.ArrayList;
import java.util.List;

import org.opencv.core.Core;
import org.opencv.core.Core.MinMaxLocResult;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.MatOfPoint;
import org.opencv.core.MatOfPoint2f;
import org.opencv.core.Point;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.highgui.HighGui;
import org.opencv.imgproc.Imgproc;

class PointPolygonTest {
    public void run() {
        /// Create an image
        int r = 100;
        Mat src = Mat.zeros(new Size(4 * r, 4 * r), CvType.CV_8U);

        /// Create a sequence of points to make a contour
        List<Point> vert = new ArrayList<>(6);
        vert.add(new Point(3 * r / 2, 1.34 * r));
        vert.add(new Point(1 * r, 2 * r));
        vert.add(new Point(3 * r / 2, 2.866 * r));
        vert.add(new Point(5 * r / 2, 2.866 * r));
        vert.add(new Point(3 * r, 2 * r));
        vert.add(new Point(5 * r / 2, 1.34 * r));

        /// Draw it in src
        for (int i = 0; i < 6; i++) {
            Imgproc.line(src, vert.get(i), vert.get((i + 1) % 6), new Scalar(255), 3);
        }

        /// Get the contours
        List<MatOfPoint> contours = new ArrayList<>();
        Mat hierarchy = new Mat();
        Imgproc.findContours(src, contours, hierarchy, Imgproc.RETR_TREE, Imgproc.CHAIN_APPROX_SIMPLE);

        /// Calculate the distances to the contour
        Mat rawDist = new Mat(src.size(), CvType.CV_32F);
        float[] rawDistData = new float[(int) (rawDist.total() * rawDist.channels())];
        for (int i = 0; i < src.rows(); i++) {
            for (int j = 0; j < src.cols(); j++) {
                rawDistData[i * src.cols() + j] = (float) Imgproc
                        .pointPolygonTest(new MatOfPoint2f(contours.get(0).toArray()), new Point(j, i), true);
            }
        }
        rawDist.put(0, 0, rawDistData);

        MinMaxLocResult res = Core.minMaxLoc(rawDist);
        double minVal = Math.abs(res.minVal);
        double maxVal = Math.abs(res.maxVal);

        /// Depicting the distances graphically
        Mat drawing = Mat.zeros(src.size(), CvType.CV_8UC3);
        byte[] drawingData = new byte[(int) (drawing.total() * drawing.channels())];
        for (int i = 0; i < src.rows(); i++) {
            for (int j = 0; j < src.cols(); j++) {
                if (rawDistData[i * src.cols() + j] < 0) {
                    drawingData[(i * src.cols() + j) * 3] =
                            (byte) (255 - Math.abs(rawDistData[i * src.cols() + j]) * 255 / minVal);
                } else if (rawDistData[i * src.cols() + j] > 0) {
                    drawingData[(i * src.cols() + j) * 3 + 2] =
                            (byte) (255 - rawDistData[i * src.cols() + j] * 255 / maxVal);
                } else {
                    drawingData[(i * src.cols() + j) * 3] = (byte) 255;
                    drawingData[(i * src.cols() + j) * 3 + 1] = (byte) 255;
                    drawingData[(i * src.cols() + j) * 3 + 2] = (byte) 255;
                }
            }
        }
        drawing.put(0, 0, drawingData);
        Imgproc.circle(drawing, res.maxLoc, (int)res.maxVal, new Scalar(255, 255, 255), 2, 8, 0);

        /// Show your results
        HighGui.imshow("Source", src);
        HighGui.imshow("Distance and inscribed circle", drawing);

        HighGui.waitKey();
        System.exit(0);
    }
}

public class PointPolygonTestDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new PointPolygonTest().run();
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

- **PointPolygonTestDemo**: A class/struct defined in this file
- **PointPolygonTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `org.opencv.core.CvType`
- `org.opencv.core.Core.MinMaxLocResult`
- `org.opencv.core.MatOfPoint`
- `java.util.List`
- `org.opencv.imgproc.Imgproc`
- `java.util.ArrayList`
- `org.opencv.core.MatOfPoint2f`
- `org.opencv.core.Size`
- `org.opencv.core.Mat`
- `org.opencv.core.Core`
- `org.opencv.highgui.HighGui`
- `org.opencv.core.Point`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

