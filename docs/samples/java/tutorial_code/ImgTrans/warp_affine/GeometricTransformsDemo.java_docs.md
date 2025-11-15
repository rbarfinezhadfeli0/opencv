# Documentation for `samples/java/tutorial_code/ImgTrans/warp_affine/GeometricTransformsDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/ImgTrans/warp_affine/GeometricTransformsDemo.java`
- **File Name**: `GeometricTransformsDemo.java`
- **File Size**: 2,964 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/ImgTrans/warp_affine/GeometricTransformsDemo.java](../../../../../samples/java/tutorial_code/ImgTrans/warp_affine/GeometricTransformsDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/ImgTrans/warp_affine` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfPoint2f;
import org.opencv.core.Point;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class GeometricTransforms {
    public void run(String[] args) {
        //! [Load the image]
        String filename = args.length > 0 ? args[0] : "../data/lena.jpg";
        Mat src = Imgcodecs.imread(filename);
        if (src.empty()) {
            System.err.println("Cannot read image: " + filename);
            System.exit(0);
        }
        //! [Load the image]

        //! [Set your 3 points to calculate the  Affine Transform]
        Point[] srcTri = new Point[3];
        srcTri[0] = new Point( 0, 0 );
        srcTri[1] = new Point( src.cols() - 1, 0 );
        srcTri[2] = new Point( 0, src.rows() - 1 );

        Point[] dstTri = new Point[3];
        dstTri[0] = new Point( 0, src.rows()*0.33 );
        dstTri[1] = new Point( src.cols()*0.85, src.rows()*0.25 );
        dstTri[2] = new Point( src.cols()*0.15, src.rows()*0.7 );
        //! [Set your 3 points to calculate the  Affine Transform]

        //! [Get the Affine Transform]
        Mat warpMat = Imgproc.getAffineTransform( new MatOfPoint2f(srcTri), new MatOfPoint2f(dstTri) );
        //! [Get the Affine Transform]

        //! [Apply the Affine Transform just found to the src image]
        Mat warpDst = Mat.zeros( src.rows(), src.cols(), src.type() );

        Imgproc.warpAffine( src, warpDst, warpMat, warpDst.size() );
        //! [Apply the Affine Transform just found to the src image]

        /** Rotating the image after Warp */

        //! [Compute a rotation matrix with respect to the center of the image]
        Point center = new Point(warpDst.cols() / 2, warpDst.rows() / 2);
        double angle = -50.0;
        double scale = 0.6;
        //! [Compute a rotation matrix with respect to the center of the image]

        //! [Get the rotation matrix with the specifications above]
        Mat rotMat = Imgproc.getRotationMatrix2D( center, angle, scale );
        //! [Get the rotation matrix with the specifications above]

        //! [Rotate the warped image]
        Mat warpRotateDst = new Mat();
        Imgproc.warpAffine( warpDst, warpRotateDst, rotMat, warpDst.size() );
        //! [Rotate the warped image]

        //! [Show what you got]
        HighGui.imshow( "Source image", src );
        HighGui.imshow( "Warp", warpDst );
        HighGui.imshow( "Warp + Rotate", warpRotateDst );
        //! [Show what you got]

        //! [Wait until user exits the program]
        HighGui.waitKey(0);
        //! [Wait until user exits the program]

        System.exit(0);
    }
}

public class GeometricTransformsDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new GeometricTransforms().run(args);
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

- **GeometricTransforms**: A class/struct defined in this file
- **GeometricTransformsDemo**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.core.Core`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.MatOfPoint2f`
- `org.opencv.core.Mat`
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

