# Documentation for `docs/samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java_docs.md`
- **File Name**: `GoodFeaturesToTrackDemo.java_docs.md`
- **File Size**: 8,548 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java_docs.md](../../../../../../docs/samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/TrackingMotion/good_features_to_track` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java`
- **File Name**: `GoodFeaturesToTrackDemo.java`
- **File Size**: 4,702 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java](../../../../../samples/java/tutorial_code/TrackingMotion/good_features_to_track/GoodFeaturesToTrackDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/TrackingMotion/good_features_to_track` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.Image;
import java.util.Random;

import javax.swing.BoxLayout;
import javax.swing.ImageIcon;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JSlider;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfPoint;
import org.opencv.core.Point;
import org.opencv.core.Scalar;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class GoodFeaturesToTrack {
    private Mat src = new Mat();
    private Mat srcGray = new Mat();
    private JFrame frame;
    private JLabel imgLabel;
    private static final int MAX_THRESHOLD = 100;
    private int maxCorners = 23;
    private Random rng = new Random(12345);

    public GoodFeaturesToTrack(String[] args) {
        /// Load source image and convert it to gray
        String filename = args.length > 0 ? args[0] : "../data/pic3.png";
        src = Imgcodecs.imread(filename);
        if (src.empty()) {
            System.err.println("Cannot read image: " + filename);
            System.exit(0);
        }

        Imgproc.cvtColor(src, srcGray, Imgproc.COLOR_BGR2GRAY);

        // Create and set up the window.
        frame = new JFrame("Shi-Tomasi corner detector demo");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Set up the content pane.
        Image img = HighGui.toBufferedImage(src);
        addComponentsToPane(frame.getContentPane(), img);
        // Use the content pane's default BorderLayout. No need for
        // setLayout(new BorderLayout());
        // Display the window.
        frame.pack();
        frame.setVisible(true);
        update();
    }

    private void addComponentsToPane(Container pane, Image img) {
        if (!(pane.getLayout() instanceof BorderLayout)) {
            pane.add(new JLabel("Container doesn't use BorderLayout!"));
            return;
        }

        JPanel sliderPanel = new JPanel();
        sliderPanel.setLayout(new BoxLayout(sliderPanel, BoxLayout.PAGE_AXIS));

        sliderPanel.add(new JLabel("Max  corners:"));
        JSlider slider = new JSlider(0, MAX_THRESHOLD, maxCorners);
        slider.setMajorTickSpacing(20);
        slider.setMinorTickSpacing(10);
        slider.setPaintTicks(true);
        slider.setPaintLabels(true);
        slider.addChangeListener(new ChangeListener() {
            @Override
            public void stateChanged(ChangeEvent e) {
                JSlider source = (JSlider) e.getSource();
                maxCorners = source.getValue();
                update();
            }
        });
        sliderPanel.add(slider);
        pane.add(sliderPanel, BorderLayout.PAGE_START);

        imgLabel = new JLabel(new ImageIcon(img));
        pane.add(imgLabel, BorderLayout.CENTER);
    }

    private void update() {
        /// Parameters for Shi-Tomasi algorithm
        maxCorners = Math.max(maxCorners, 1);
        MatOfPoint corners = new MatOfPoint();
        double qualityLevel = 0.01;
        double minDistance = 10;
        int blockSize = 3, gradientSize = 3;
        boolean useHarrisDetector = false;
        double k = 0.04;

        /// Copy the source image
        Mat copy = src.clone();

        /// Apply corner detection
        Imgproc.goodFeaturesToTrack(srcGray, corners, maxCorners, qualityLevel, minDistance, new Mat(),
                blockSize, gradientSize, useHarrisDetector, k);

        /// Draw corners detected
        System.out.println("** Number of corners detected: " + corners.rows());
        int[] cornersData = new int[(int) (corners.total() * corners.channels())];
        corners.get(0, 0, cornersData);
        int radius = 4;
        for (int i = 0; i < corners.rows(); i++) {
            Imgproc.circle(copy, new Point(cornersData[i * 2], cornersData[i * 2 + 1]), radius,
                    new Scalar(rng.nextInt(256), rng.nextInt(256), rng.nextInt(256)), Imgproc.FILLED);
        }

        imgLabel.setIcon(new ImageIcon(HighGui.toBufferedImage(copy)));
        frame.repaint();
    }
}

public class GoodFeaturesToTrackDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        // Schedule a job for the event dispatch thread:
        // creating and showing this application's GUI.
        javax.swing.SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new GoodFeaturesToTrack(args);
            }
        });
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

- **GoodFeaturesToTrackDemo**: A class/struct defined in this file
- **GoodFeaturesToTrack**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.awt.BorderLayout`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.highgui.HighGui`
- `javax.swing.JSlider`
- `org.opencv.core.Core`
- `java.awt.Container`
- `java.awt.Image`
- `javax.swing.JPanel`
- `javax.swing.BoxLayout`
- `javax.swing.event.ChangeEvent`
- `javax.swing.JFrame`
- `java.util.Random`
- `javax.swing.JLabel`
- `javax.swing.event.ChangeListener`
- `org.opencv.core.Point`
- `org.opencv.core.Scalar`
- `org.opencv.core.MatOfPoint`
- `org.opencv.core.Mat`
- `javax.swing.ImageIcon`


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

