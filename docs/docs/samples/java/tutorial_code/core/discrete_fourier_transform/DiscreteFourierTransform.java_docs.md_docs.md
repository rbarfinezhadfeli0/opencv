# Documentation for `docs/samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java_docs.md`
- **File Name**: `DiscreteFourierTransform.java_docs.md`
- **File Size**: 7,734 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java_docs.md](../../../../../../docs/samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/core/discrete_fourier_transform` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java`
- **File Name**: `DiscreteFourierTransform.java`
- **File Size**: 4,318 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java](../../../../../samples/java/tutorial_code/core/discrete_fourier_transform/DiscreteFourierTransform.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/core/discrete_fourier_transform` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import org.opencv.core.*;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;

import java.util.List;
import java.util.*;

class DiscreteFourierTransformRun{
    private void help() {
        System.out.println("" +
                "This program demonstrated the use of the discrete Fourier transform (DFT). \n" +
                "The dft of an image is taken and it's power spectrum is displayed.\n" +
                "Usage:\n" +
                "./DiscreteFourierTransform [image_name -- default ../data/lena.jpg]");
    }

    public void run(String[] args){

        help();

        String filename = ((args.length > 0) ? args[0] : "../data/lena.jpg");

        Mat I = Imgcodecs.imread(filename, Imgcodecs.IMREAD_GRAYSCALE);
        if( I.empty() ) {
            System.out.println("Error opening image");
            System.exit(-1);
        }

        //! [expand]
        Mat padded = new Mat();                     //expand input image to optimal size
        int m = Core.getOptimalDFTSize( I.rows() );
        int n = Core.getOptimalDFTSize( I.cols() ); // on the border add zero values
        Core.copyMakeBorder(I, padded, 0, m - I.rows(), 0, n - I.cols(), Core.BORDER_CONSTANT, Scalar.all(0));
        //! [expand]

        //! [complex_and_real]
        List<Mat> planes = new ArrayList<Mat>();
        padded.convertTo(padded, CvType.CV_32F);
        planes.add(padded);
        planes.add(Mat.zeros(padded.size(), CvType.CV_32F));
        Mat complexI = new Mat();
        Core.merge(planes, complexI);         // Add to the expanded another plane with zeros
        //! [complex_and_real]

        //! [dft]
        Core.dft(complexI, complexI);         // this way the result may fit in the source matrix
        //! [dft]

        // compute the magnitude and switch to logarithmic scale
        // => log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))
        //! [magnitude]
        Core.split(complexI, planes);                               // planes.get(0) = Re(DFT(I)
                                                                    // planes.get(1) = Im(DFT(I))
        Core.magnitude(planes.get(0), planes.get(1), planes.get(0));// planes.get(0) = magnitude
        Mat magI = planes.get(0);
        //! [magnitude]

        //! [log]
        Mat matOfOnes = Mat.ones(magI.size(), magI.type());
        Core.add(matOfOnes, magI, magI);         // switch to logarithmic scale
        Core.log(magI, magI);
        //! [log]

        //! [crop_rearrange]
        // crop the spectrum, if it has an odd number of rows or columns
        magI = magI.submat(new Rect(0, 0, magI.cols() & -2, magI.rows() & -2));

        // rearrange the quadrants of Fourier image  so that the origin is at the image center
        int cx = magI.cols()/2;
        int cy = magI.rows()/2;

        Mat q0 = new Mat(magI, new Rect(0, 0, cx, cy));   // Top-Left - Create a ROI per quadrant
        Mat q1 = new Mat(magI, new Rect(cx, 0, cx, cy));  // Top-Right
        Mat q2 = new Mat(magI, new Rect(0, cy, cx, cy));  // Bottom-Left
        Mat q3 = new Mat(magI, new Rect(cx, cy, cx, cy)); // Bottom-Right

        Mat tmp = new Mat();               // swap quadrants (Top-Left with Bottom-Right)
        q0.copyTo(tmp);
        q3.copyTo(q0);
        tmp.copyTo(q3);

        q1.copyTo(tmp);                    // swap quadrant (Top-Right with Bottom-Left)
        q2.copyTo(q1);
        tmp.copyTo(q2);
        //! [crop_rearrange]

        magI.convertTo(magI, CvType.CV_8UC1);
        //! [normalize]
        Core.normalize(magI, magI, 0, 255, Core.NORM_MINMAX, CvType.CV_8UC1); // Transform the matrix with float values
                                                                            // into a viewable image form (float between
                                                                            // values 0 and 255).
        //! [normalize]

        HighGui.imshow("Input Image"       , I   );    // Show the result
        HighGui.imshow("Spectrum Magnitude", magI);
        HighGui.waitKey();

        System.exit(0);
    }
}


public class DiscreteFourierTransform {
    public static void main(String[] args) {
        // Load the native library.
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
        new DiscreteFourierTransformRun().run(args);
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

- **DiscreteFourierTransformRun**: A class/struct defined in this file
- **DiscreteFourierTransform**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.util.`
- `org.opencv.imgcodecs.Imgcodecs`
- `java.util.List`
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

