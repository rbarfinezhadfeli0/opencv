# Documentation for `docs/samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java_docs.md`
- **File Name**: `BasicLinearTransformsDemo.java_docs.md`
- **File Size**: 6,788 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java_docs.md](../../../../../../docs/samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java`
- **File Name**: `BasicLinearTransformsDemo.java`
- **File Size**: 3,301 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java](../../../../../samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image/BasicLinearTransformsDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/ImgProc/changing_contrast_brightness_image` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.Scanner;

import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;

class BasicLinearTransforms {
    private byte saturate(double val) {
        int iVal = (int) Math.round(val);
        iVal = iVal > 255 ? 255 : (iVal < 0 ? 0 : iVal);
        return (byte) iVal;
    }

    public void run(String[] args) {
        /// Read image given by user
        //! [basic-linear-transform-load]
        String imagePath = args.length > 0 ? args[0] : "../data/lena.jpg";
        Mat image = Imgcodecs.imread(imagePath);
        if (image.empty()) {
            System.out.println("Empty image: " + imagePath);
            System.exit(0);
        }
        //! [basic-linear-transform-load]

        //! [basic-linear-transform-output]
        Mat newImage = Mat.zeros(image.size(), image.type());
        //! [basic-linear-transform-output]

        //! [basic-linear-transform-parameters]
        double alpha = 1.0; /*< Simple contrast control */
        int beta = 0;       /*< Simple brightness control */

        /// Initialize values
        System.out.println(" Basic Linear Transforms ");
        System.out.println("-------------------------");
        try (Scanner scanner = new Scanner(System.in)) {
            System.out.print("* Enter the alpha value [1.0-3.0]: ");
            alpha = scanner.nextDouble();
            System.out.print("* Enter the beta value [0-100]: ");
            beta = scanner.nextInt();
        }
        //! [basic-linear-transform-parameters]

        /// Do the operation newImage(i,j) = alpha*image(i,j) + beta
        /// Instead of these 'for' loops we could have used simply:
        /// image.convertTo(newImage, -1, alpha, beta);
        /// but we wanted to show you how to access the pixels :)
        //! [basic-linear-transform-operation]
        byte[] imageData = new byte[(int) (image.total()*image.channels())];
        image.get(0, 0, imageData);
        byte[] newImageData = new byte[(int) (newImage.total()*newImage.channels())];
        for (int y = 0; y < image.rows(); y++) {
            for (int x = 0; x < image.cols(); x++) {
                for (int c = 0; c < image.channels(); c++) {
                    double pixelValue = imageData[(y * image.cols() + x) * image.channels() + c];
                    /// Java byte range is [-128, 127]
                    pixelValue = pixelValue < 0 ? pixelValue + 256 : pixelValue;
                    newImageData[(y * image.cols() + x) * image.channels() + c]
                            = saturate(alpha * pixelValue + beta);
                }
            }
        }
        newImage.put(0, 0, newImageData);
        //! [basic-linear-transform-operation]

        //! [basic-linear-transform-display]
        /// Show stuff
        HighGui.imshow("Original Image", image);
        HighGui.imshow("New Image", newImage);

        /// Wait until user press some key
        HighGui.waitKey();
        //! [basic-linear-transform-display]
        System.exit(0);
    }
}

public class BasicLinearTransformsDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new BasicLinearTransforms().run(args);
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

- **BasicLinearTransforms**: A class/struct defined in this file
- **BasicLinearTransformsDemo**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.core.Core`
- `java.util.Scanner`
- `org.opencv.core.Mat`
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

