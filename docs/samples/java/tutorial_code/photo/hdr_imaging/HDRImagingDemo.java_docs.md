# Documentation for `samples/java/tutorial_code/photo/hdr_imaging/HDRImagingDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/photo/hdr_imaging/HDRImagingDemo.java`
- **File Name**: `HDRImagingDemo.java`
- **File Size**: 3,414 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/photo/hdr_imaging/HDRImagingDemo.java](../../../../../samples/java/tutorial_code/photo/hdr_imaging/HDRImagingDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/photo/hdr_imaging` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.Scalar;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.photo.CalibrateDebevec;
import org.opencv.photo.MergeDebevec;
import org.opencv.photo.MergeMertens;
import org.opencv.photo.Photo;
import org.opencv.photo.Tonemap;

class HDRImaging {
    public void loadExposureSeq(String path, List<Mat> images, List<Float> times) {
        path += "/";

        List<String> lines;
        try {
            lines = Files.readAllLines(Paths.get(path + "list.txt"));

            for (String line : lines) {
                String[] splitStr = line.split("\\s+");
                if (splitStr.length == 2) {
                    String name = splitStr[0];
                    Mat img = Imgcodecs.imread(path + name);
                    images.add(img);
                    float val = Float.parseFloat(splitStr[1]);
                    times.add(1/ val);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void run(String[] args) {
        String path = args.length > 0 ? args[0] : "";
        if (path.isEmpty()) {
            System.out.println("Path is empty. Use the directory that contains images and exposure times.");
            System.exit(0);
        }

        //! [Load images and exposure times]
        List<Mat> images = new ArrayList<>();
        List<Float> times = new ArrayList<>();
        loadExposureSeq(path, images, times);
        //! [Load images and exposure times]

        //! [Estimate camera response]
        Mat response = new Mat();
        CalibrateDebevec calibrate = Photo.createCalibrateDebevec();
        Mat matTimes = new Mat(times.size(), 1, CvType.CV_32F);
        float[] arrayTimes = new float[(int) (matTimes.total()*matTimes.channels())];
        for (int i = 0; i < times.size(); i++) {
            arrayTimes[i] = times.get(i);
        }
        matTimes.put(0, 0, arrayTimes);
        calibrate.process(images, response, matTimes);
        //! [Estimate camera response]

        //! [Make HDR image]
        Mat hdr = new Mat();
        MergeDebevec mergeDebevec = Photo.createMergeDebevec();
        mergeDebevec.process(images, hdr, matTimes);
        //! [Make HDR image]

        //! [Tonemap HDR image]
        Mat ldr = new Mat();
        Tonemap tonemap = Photo.createTonemap(2.2f);
        tonemap.process(hdr, ldr);
        //! [Tonemap HDR image]

        //! [Perform exposure fusion]
        Mat fusion = new Mat();
        MergeMertens mergeMertens = Photo.createMergeMertens();
        mergeMertens.process(images, fusion);
        //! [Perform exposure fusion]

        //! [Write results]
        Core.multiply(fusion, new Scalar(255,255,255), fusion);
        Core.multiply(ldr, new Scalar(255,255,255), ldr);
        Imgcodecs.imwrite("fusion.png", fusion);
        Imgcodecs.imwrite("ldr.png", ldr);
        Imgcodecs.imwrite("hdr.hdr", hdr);
        //! [Write results]

        System.exit(0);
    }
}

public class HDRImagingDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new HDRImaging().run(args);
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

- **HDRImaging**: A class/struct defined in this file
- **HDRImagingDemo**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `java.nio.file.Paths`
- `java.io.IOException`
- `org.opencv.core.CvType`
- `org.opencv.photo.Photo`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.photo.CalibrateDebevec`
- `org.opencv.photo.MergeDebevec`
- `org.opencv.core.Core`
- `java.util.List`
- `java.util.ArrayList`
- `org.opencv.photo.Tonemap`
- `org.opencv.photo.MergeMertens`
- `org.opencv.core.Mat`
- `java.nio.file.Files`


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

