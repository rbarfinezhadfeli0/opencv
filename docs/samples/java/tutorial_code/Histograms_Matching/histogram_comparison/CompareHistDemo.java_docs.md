# Documentation for `samples/java/tutorial_code/Histograms_Matching/histogram_comparison/CompareHistDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/Histograms_Matching/histogram_comparison/CompareHistDemo.java`
- **File Name**: `CompareHistDemo.java`
- **File Size**: 4,175 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/Histograms_Matching/histogram_comparison/CompareHistDemo.java](../../../../../samples/java/tutorial_code/Histograms_Matching/histogram_comparison/CompareHistDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/Histograms_Matching/histogram_comparison` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.Arrays;
import java.util.List;

import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfFloat;
import org.opencv.core.MatOfInt;
import org.opencv.core.Range;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class CompareHist {
    public void run(String[] args) {
        //! [Load three images with different environment settings]
        if (args.length != 3) {
            System.err.println("You must supply 3 arguments that correspond to the paths to 3 images.");
            System.exit(0);
        }
        Mat srcBase = Imgcodecs.imread(args[0]);
        Mat srcTest1 = Imgcodecs.imread(args[1]);
        Mat srcTest2 = Imgcodecs.imread(args[2]);
        if (srcBase.empty() || srcTest1.empty() || srcTest2.empty()) {
            System.err.println("Cannot read the images");
            System.exit(0);
        }
        //! [Load three images with different environment settings]

        //! [Convert to HSV]
        Mat hsvBase = new Mat(), hsvTest1 = new Mat(), hsvTest2 = new Mat();
        Imgproc.cvtColor( srcBase, hsvBase, Imgproc.COLOR_BGR2HSV );
        Imgproc.cvtColor( srcTest1, hsvTest1, Imgproc.COLOR_BGR2HSV );
        Imgproc.cvtColor( srcTest2, hsvTest2, Imgproc.COLOR_BGR2HSV );
        //! [Convert to HSV]

        //! [Convert to HSV half]
        Mat hsvHalfDown = hsvBase.submat( new Range( hsvBase.rows()/2, hsvBase.rows() - 1 ), new Range( 0, hsvBase.cols() - 1 ) );
        //! [Convert to HSV half]

        //! [Using 50 bins for hue and 60 for saturation]
        int hBins = 50, sBins = 60;
        int[] histSize = { hBins, sBins };

        // hue varies from 0 to 179, saturation from 0 to 255
        float[] ranges = { 0, 180, 0, 256 };

        // Use the 0-th and 1-st channels
        int[] channels = { 0, 1 };
        //! [Using 50 bins for hue and 60 for saturation]

        //! [Calculate the histograms for the HSV images]
        Mat histBase = new Mat(), histHalfDown = new Mat(), histTest1 = new Mat(), histTest2 = new Mat();

        List<Mat> hsvBaseList = Arrays.asList(hsvBase);
        Imgproc.calcHist(hsvBaseList, new MatOfInt(channels), new Mat(), histBase, new MatOfInt(histSize), new MatOfFloat(ranges), false);
        Core.normalize(histBase, histBase, 1, 0, Core.NORM_L1);

        List<Mat> hsvHalfDownList = Arrays.asList(hsvHalfDown);
        Imgproc.calcHist(hsvHalfDownList, new MatOfInt(channels), new Mat(), histHalfDown, new MatOfInt(histSize), new MatOfFloat(ranges), false);
        Core.normalize(histHalfDown, histHalfDown, 1, 0, Core.NORM_L1);

        List<Mat> hsvTest1List = Arrays.asList(hsvTest1);
        Imgproc.calcHist(hsvTest1List, new MatOfInt(channels), new Mat(), histTest1, new MatOfInt(histSize), new MatOfFloat(ranges), false);
        Core.normalize(histTest1, histTest1, 1, 0, Core.NORM_L1);

        List<Mat> hsvTest2List = Arrays.asList(hsvTest2);
        Imgproc.calcHist(hsvTest2List, new MatOfInt(channels), new Mat(), histTest2, new MatOfInt(histSize), new MatOfFloat(ranges), false);
        Core.normalize(histTest2, histTest2, 1, 0, Core.NORM_L1);
        //! [Calculate the histograms for the HSV images]

        //! [Apply the histogram comparison methods]
        for( int compareMethod = 0; compareMethod < 6; compareMethod++ ) {
            double baseBase = Imgproc.compareHist( histBase, histBase, compareMethod );
            double baseHalf = Imgproc.compareHist( histBase, histHalfDown, compareMethod );
            double baseTest1 = Imgproc.compareHist( histBase, histTest1, compareMethod );
            double baseTest2 = Imgproc.compareHist( histBase, histTest2, compareMethod );

            System.out.println("Method " + compareMethod + " Perfect, Base-Half, Base-Test(1), Base-Test(2) : " + baseBase + " / " + baseHalf
                    + " / " + baseTest1 + " / " + baseTest2);
        }
        //! [Apply the histogram comparison methods]
    }
}

public class CompareHistDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new CompareHist().run(args);
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

- **CompareHist**: A class/struct defined in this file
- **CompareHistDemo**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.core.Range`
- `org.opencv.core.Core`
- `java.util.List`
- `java.util.Arrays`
- `org.opencv.imgproc.Imgproc`
- `0`
- `org.opencv.core.MatOfInt`
- `org.opencv.core.Mat`
- `org.opencv.core.MatOfFloat`


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

