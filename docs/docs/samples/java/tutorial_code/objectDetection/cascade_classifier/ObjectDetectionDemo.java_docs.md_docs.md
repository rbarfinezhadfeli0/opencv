# Documentation for `docs/samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java_docs.md`
- **File Name**: `ObjectDetectionDemo.java_docs.md`
- **File Size**: 7,189 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java_docs.md](../../../../../../docs/samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/objectDetection/cascade_classifier` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java`
- **File Name**: `ObjectDetectionDemo.java`
- **File Size**: 3,579 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java](../../../../../samples/java/tutorial_code/objectDetection/cascade_classifier/ObjectDetectionDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/objectDetection/cascade_classifier` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.util.List;

import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfRect;
import org.opencv.core.Point;
import org.opencv.core.Rect;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.highgui.HighGui;
import org.opencv.imgproc.Imgproc;
import org.opencv.objdetect.CascadeClassifier;
import org.opencv.videoio.VideoCapture;

class ObjectDetection {
    public void detectAndDisplay(Mat frame, CascadeClassifier faceCascade, CascadeClassifier eyesCascade) {
        Mat frameGray = new Mat();
        Imgproc.cvtColor(frame, frameGray, Imgproc.COLOR_BGR2GRAY);
        Imgproc.equalizeHist(frameGray, frameGray);

        // -- Detect faces
        MatOfRect faces = new MatOfRect();
        faceCascade.detectMultiScale(frameGray, faces);

        List<Rect> listOfFaces = faces.toList();
        for (Rect face : listOfFaces) {
            Point center = new Point(face.x + face.width / 2, face.y + face.height / 2);
            Imgproc.ellipse(frame, center, new Size(face.width / 2, face.height / 2), 0, 0, 360,
                    new Scalar(255, 0, 255));

            Mat faceROI = frameGray.submat(face);

            // -- In each face, detect eyes
            MatOfRect eyes = new MatOfRect();
            eyesCascade.detectMultiScale(faceROI, eyes);

            List<Rect> listOfEyes = eyes.toList();
            for (Rect eye : listOfEyes) {
                Point eyeCenter = new Point(face.x + eye.x + eye.width / 2, face.y + eye.y + eye.height / 2);
                int radius = (int) Math.round((eye.width + eye.height) * 0.25);
                Imgproc.circle(frame, eyeCenter, radius, new Scalar(255, 0, 0), 4);
            }
        }

        //-- Show what you got
        HighGui.imshow("Capture - Face detection", frame );
    }

    public void run(String[] args) {
        String filenameFaceCascade = args.length > 2 ? args[0] : "../../data/haarcascades/haarcascade_frontalface_alt.xml";
        String filenameEyesCascade = args.length > 2 ? args[1] : "../../data/haarcascades/haarcascade_eye_tree_eyeglasses.xml";
        int cameraDevice = args.length > 2 ? Integer.parseInt(args[2]) : 0;

        CascadeClassifier faceCascade = new CascadeClassifier();
        CascadeClassifier eyesCascade = new CascadeClassifier();

        if (!faceCascade.load(filenameFaceCascade)) {
            System.err.println("--(!)Error loading face cascade: " + filenameFaceCascade);
            System.exit(0);
        }
        if (!eyesCascade.load(filenameEyesCascade)) {
            System.err.println("--(!)Error loading eyes cascade: " + filenameEyesCascade);
            System.exit(0);
        }

        VideoCapture capture = new VideoCapture(cameraDevice);
        if (!capture.isOpened()) {
            System.err.println("--(!)Error opening video capture");
            System.exit(0);
        }

        Mat frame = new Mat();
        while (capture.read(frame)) {
            if (frame.empty()) {
                System.err.println("--(!) No captured frame -- Break!");
                break;
            }

            //-- 3. Apply the classifier to the frame
            detectAndDisplay(frame, faceCascade, eyesCascade);

            if (HighGui.waitKey(10) == 27) {
                break;// escape
            }
        }

        System.exit(0);
    }
}

public class ObjectDetectionDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new ObjectDetection().run(args);
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

- **ObjectDetectionDemo**: A class/struct defined in this file
- **ObjectDetection**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `org.opencv.core.Core`
- `org.opencv.core.MatOfRect`
- `java.util.List`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.objdetect.CascadeClassifier`
- `org.opencv.core.Rect`
- `org.opencv.videoio.VideoCapture`
- `org.opencv.core.Size`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

