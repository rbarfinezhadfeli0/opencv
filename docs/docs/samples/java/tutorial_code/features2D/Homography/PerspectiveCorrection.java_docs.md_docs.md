# Documentation for `docs/samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java_docs.md`
- **File Name**: `PerspectiveCorrection.java_docs.md`
- **File Size**: 6,487 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java_docs.md](../../../../../../docs/samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java`
- **File Name**: `PerspectiveCorrection.java`
- **File Size**: 3,050 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java](../../../../../samples/java/tutorial_code/features2D/Homography/PerspectiveCorrection.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/features2D/Homography` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

import org.opencv.core.*;
import org.opencv.calib3d.Calib3d;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;


class PerspectiveCorrectionRun {
    void perspectiveCorrection (String[] args) {
        String img1Path = args[0], img2Path = args[1];
        Mat img1 = Imgcodecs.imread(img1Path);
        Mat img2 = Imgcodecs.imread(img2Path);

        //! [find-corners]
        MatOfPoint2f corners1 = new MatOfPoint2f(), corners2 = new MatOfPoint2f();
        boolean found1 = Calib3d.findChessboardCorners(img1, new Size(9, 6), corners1 );
        boolean found2 = Calib3d.findChessboardCorners(img2, new Size(9, 6), corners2 );
        //! [find-corners]

        if (!found1 || !found2) {
            System.out.println("Error, cannot find the chessboard corners in both images.");
            System.exit(-1);
        }

        //! [estimate-homography]
        Mat H = new Mat();
        H = Calib3d.findHomography(corners1, corners2);
        System.out.println(H.dump());
        //! [estimate-homography]

        //! [warp-chessboard]
        Mat img1_warp = new Mat();
        Imgproc.warpPerspective(img1, img1_warp, H, img1.size());
        //! [warp-chessboard]

        Mat img_draw_warp = new Mat();
        List<Mat> list1 = new ArrayList<>(), list2 = new ArrayList<>() ;
        list1.add(img2);
        list1.add(img1_warp);
        Core.hconcat(list1, img_draw_warp);
        HighGui.imshow("Desired chessboard view / Warped source chessboard view", img_draw_warp);

        //! [compute-transformed-corners]
        Mat img_draw_matches = new Mat();
        list2.add(img1);
        list2.add(img2);
        Core.hconcat(list2, img_draw_matches);
        Point []corners1Arr = corners1.toArray();

        for (int i = 0 ; i < corners1Arr.length; i++) {
            Mat pt1 = new Mat(3, 1, CvType.CV_64FC1), pt2 = new Mat();
            pt1.put(0, 0, corners1Arr[i].x, corners1Arr[i].y, 1 );

            Core.gemm(H, pt1, 1, new Mat(), 0, pt2);
            double[] data = pt2.get(2, 0);
            Core.divide(pt2, new Scalar(data[0]), pt2);

            double[] data1 =pt2.get(0, 0);
            double[] data2 = pt2.get(1, 0);
            Point end = new Point((int)(img1.cols()+ data1[0]), (int)data2[0]);
            Imgproc.line(img_draw_matches, corners1Arr[i], end, RandomColor(), 2);
        }

        HighGui.imshow("Draw matches", img_draw_matches);
        HighGui.waitKey(0);
        //! [compute-transformed-corners]

        System.exit(0);
    }

    Scalar RandomColor () {
        Random rng = new Random();
        int r = rng.nextInt(256);
        int g = rng.nextInt(256);
        int b = rng.nextInt(256);
        return new Scalar(r, g, b);
    }
}

public class PerspectiveCorrection {
    public static void main (String[] args) {
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
        new PerspectiveCorrectionRun().perspectiveCorrection(args);
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

- **PerspectiveCorrection**: A class/struct defined in this file
- **PerspectiveCorrectionRun**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.imgcodecs.Imgcodecs`
- `java.util.Random`
- `java.util.List`
- `org.opencv.imgproc.Imgproc`
- `java.util.ArrayList`
- `org.opencv.calib3d.Calib3d`
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

