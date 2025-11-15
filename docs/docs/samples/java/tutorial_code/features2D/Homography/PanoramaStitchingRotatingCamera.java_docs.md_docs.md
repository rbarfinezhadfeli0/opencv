# Documentation for `docs/samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java_docs.md`
- **File Name**: `PanoramaStitchingRotatingCamera.java_docs.md`
- **File Size**: 6,967 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java_docs.md](../../../../../../docs/samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java`
- **File Name**: `PanoramaStitchingRotatingCamera.java`
- **File Size**: 3,486 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java](../../../../../samples/java/tutorial_code/features2D/Homography/PanoramaStitchingRotatingCamera.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.ArrayList;
import java.util.List;

import org.opencv.core.*;
import org.opencv.core.Range;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;


class PanoramaStitchingRotatingCameraRun {
    void basicPanoramaStitching (String[] args) {
        String img1path = args[0], img2path = args[1];
        Mat img1 = new Mat(), img2 = new Mat();
        img1 = Imgcodecs.imread(img1path);
        img2 = Imgcodecs.imread(img2path);

        //! [camera-pose-from-Blender-at-location-1]
        Mat c1Mo = new Mat( 4, 4, CvType.CV_64FC1 );
        c1Mo.put(0 ,0 ,0.9659258723258972, 0.2588190734386444, 0.0, 1.5529145002365112,
                 0.08852133899927139, -0.3303661346435547, -0.9396926164627075, -0.10281121730804443,
                 -0.24321036040782928, 0.9076734185218811, -0.342020183801651, 6.130080699920654,
                 0, 0, 0, 1 );
        //! [camera-pose-from-Blender-at-location-1]

        //! [camera-pose-from-Blender-at-location-2]
        Mat c2Mo = new Mat( 4, 4, CvType.CV_64FC1 );
        c2Mo.put(0, 0, 0.9659258723258972, -0.2588190734386444, 0.0, -1.5529145002365112,
                 -0.08852133899927139, -0.3303661346435547, -0.9396926164627075, -0.10281121730804443,
                 0.24321036040782928, 0.9076734185218811, -0.342020183801651, 6.130080699920654,
                 0, 0, 0, 1);
        //! [camera-pose-from-Blender-at-location-2]

        //! [camera-intrinsics-from-Blender]
        Mat cameraMatrix = new Mat(3, 3, CvType.CV_64FC1);
        cameraMatrix.put(0, 0, 700.0, 0.0, 320.0, 0.0, 700.0, 240.0, 0, 0, 1 );
        //! [camera-intrinsics-from-Blender]

        //! [extract-rotation]
        Range rowRange = new Range(0,3);
        Range colRange = new Range(0,3);
        //! [extract-rotation]

        //! [compute-rotation-displacement]
        //c1Mo * oMc2
        Mat R1 = new  Mat(c1Mo, rowRange, colRange);
        Mat R2 = new Mat(c2Mo, rowRange, colRange);
        Mat R_2to1 = new Mat();
        Core.gemm(R1, R2.t(), 1, new Mat(), 0, R_2to1 );
        //! [compute-rotation-displacement]

        //! [compute-homography]
        Mat tmp = new Mat(), H = new Mat();
        Core.gemm(cameraMatrix, R_2to1, 1, new Mat(), 0, tmp);
        Core.gemm(tmp, cameraMatrix.inv(), 1, new Mat(), 0, H);
        Scalar s = new Scalar(H.get(2, 2)[0]);
        Core.divide(H, s, H);
        System.out.println(H.dump());
        //! [compute-homography]

        //! [stitch]
        Mat img_stitch = new Mat();
        Imgproc.warpPerspective(img2, img_stitch, H, new Size(img2.cols()*2, img2.rows()) );
        Mat half = new Mat();
        half =  new Mat(img_stitch, new Rect(0, 0, img1.cols(), img1.rows()));
        img1.copyTo(half);
        //! [stitch]

        Mat img_compare = new Mat();
        Mat img_space = Mat.zeros(new Size(50, img1.rows()), CvType.CV_8UC3);
        List<Mat>list = new ArrayList<>();
        list.add(img1);
        list.add(img_space);
        list.add(img2);
        Core.hconcat(list, img_compare);

        HighGui.imshow("Compare Images", img_compare);
        HighGui.imshow("Panorama Stitching", img_stitch);
        HighGui.waitKey(0);
        System.exit(0);
    }
}

public class PanoramaStitchingRotatingCamera {
    public static void main(String[] args) {
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
        new PanoramaStitchingRotatingCameraRun().basicPanoramaStitching(args);
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

- **PanoramaStitchingRotatingCameraRun**: A class/struct defined in this file
- **PanoramaStitchingRotatingCamera**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.core.Range`
- `java.util.List`
- `org.opencv.imgproc.Imgproc`
- `java.util.ArrayList`
- `org.opencv.core.`
- `org.opencv.highgui.HighGui`


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

