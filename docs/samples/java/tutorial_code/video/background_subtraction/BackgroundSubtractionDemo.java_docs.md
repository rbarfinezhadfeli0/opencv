# Documentation for `samples/java/tutorial_code/video/background_subtraction/BackgroundSubtractionDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/video/background_subtraction/BackgroundSubtractionDemo.java`
- **File Name**: `BackgroundSubtractionDemo.java`
- **File Size**: 2,548 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/video/background_subtraction/BackgroundSubtractionDemo.java](../../../../../samples/java/tutorial_code/video/background_subtraction/BackgroundSubtractionDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/video/background_subtraction` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.Point;
import org.opencv.core.Scalar;
import org.opencv.highgui.HighGui;
import org.opencv.imgproc.Imgproc;
import org.opencv.video.BackgroundSubtractor;
import org.opencv.video.Video;
import org.opencv.videoio.VideoCapture;
import org.opencv.videoio.Videoio;

class BackgroundSubtraction {
    public void run(String[] args) {
        String input = args.length > 0 ? args[0] : "../data/vtest.avi";
        boolean useMOG2 = args.length > 1 ? args[1] == "MOG2" : true;

        //! [create]
        BackgroundSubtractor backSub;
        if (useMOG2) {
            backSub = Video.createBackgroundSubtractorMOG2();
        } else {
            backSub = Video.createBackgroundSubtractorKNN();
        }
        //! [create]

        //! [capture]
        VideoCapture capture = new VideoCapture(input);
        if (!capture.isOpened()) {
            System.err.println("Unable to open: " + input);
            System.exit(0);
        }
        //! [capture]

        Mat frame = new Mat(), fgMask = new Mat();
        while (true) {
            capture.read(frame);
            if (frame.empty()) {
                break;
            }

            //! [apply]
            // update the background model
            backSub.apply(frame, fgMask);
            //! [apply]

            //! [display_frame_number]
            // get the frame number and write it on the current frame
            Imgproc.rectangle(frame, new Point(10, 2), new Point(100, 20), new Scalar(255, 255, 255), -1);
            String frameNumberString = String.format("%d", (int)capture.get(Videoio.CAP_PROP_POS_FRAMES));
            Imgproc.putText(frame, frameNumberString, new Point(15, 15), Core.FONT_HERSHEY_SIMPLEX, 0.5,
                    new Scalar(0, 0, 0));
            //! [display_frame_number]

            //! [show]
            // show the current frame and the fg masks
            HighGui.imshow("Frame", frame);
            HighGui.imshow("FG Mask", fgMask);
            //! [show]

            // get the input from the keyboard
            int keyboard = HighGui.waitKey(30);
            if (keyboard == 'q' || keyboard == 27) {
                break;
            }
        }

        HighGui.waitKey();
        System.exit(0);
    }
}

public class BackgroundSubtractionDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new BackgroundSubtraction().run(args);
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

- **BackgroundSubtractionDemo**: A class/struct defined in this file
- **BackgroundSubtraction**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.Scalar`
- `the`
- `org.opencv.videoio.Videoio`
- `org.opencv.core.Core`
- `org.opencv.video.Video`
- `org.opencv.video.BackgroundSubtractor`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.videoio.VideoCapture`
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

