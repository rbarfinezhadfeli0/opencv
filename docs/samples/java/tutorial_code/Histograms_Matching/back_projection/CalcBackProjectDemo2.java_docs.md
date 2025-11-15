# Documentation for `samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo2.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo2.java`
- **File Name**: `CalcBackProjectDemo2.java`
- **File Size**: 6,417 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo2.java](../../../../../samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo2.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/Histograms_Matching/back_projection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.Image;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.util.Arrays;
import java.util.List;

import javax.swing.BoxLayout;
import javax.swing.ImageIcon;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JSlider;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.MatOfFloat;
import org.opencv.core.MatOfInt;
import org.opencv.core.Point;
import org.opencv.core.Range;
import org.opencv.core.Rect;
import org.opencv.core.Scalar;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class CalcBackProject2 {
    private Mat src;
    private Mat hsv = new Mat();
    private Mat mask = new Mat();
    private JFrame frame;
    private JLabel imgLabel;
    private JLabel backprojLabel;
    private JLabel maskImgLabel;
    private static final int MAX_SLIDER = 255;
    private int low = 20;
    private int up = 20;

    public CalcBackProject2(String[] args) {
        /// Read the image
        if (args.length != 1) {
            System.err.println("You must supply one argument that corresponds to the path to the image.");
            System.exit(0);
        }

        src = Imgcodecs.imread(args[0]);
        if (src.empty()) {
            System.err.println("Empty image: " + args[0]);
            System.exit(0);
        }

        /// Transform it to HSV
        Imgproc.cvtColor(src, hsv, Imgproc.COLOR_BGR2HSV);

        // Create and set up the window.
        frame = new JFrame("Back Projection 2 demo");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Set up the content pane.
        Image img = HighGui.toBufferedImage(src);
        addComponentsToPane(frame.getContentPane(), img);
        // Use the content pane's default BorderLayout. No need for
        // setLayout(new BorderLayout());
        // Display the window.
        frame.pack();
        frame.setVisible(true);
    }

    private void addComponentsToPane(Container pane, Image img) {
        if (!(pane.getLayout() instanceof BorderLayout)) {
            pane.add(new JLabel("Container doesn't use BorderLayout!"));
            return;
        }

        /// Set Trackbars for floodfill thresholds
        JPanel sliderPanel = new JPanel();
        sliderPanel.setLayout(new BoxLayout(sliderPanel, BoxLayout.PAGE_AXIS));

        sliderPanel.add(new JLabel("Low thresh"));
        JSlider slider = new JSlider(0, MAX_SLIDER, low);
        slider.setMajorTickSpacing(20);
        slider.setMinorTickSpacing(10);
        slider.setPaintTicks(true);
        slider.setPaintLabels(true);
        slider.addChangeListener(new ChangeListener() {
            @Override
            public void stateChanged(ChangeEvent e) {
                JSlider source = (JSlider) e.getSource();
                low = source.getValue();
            }
        });
        sliderPanel.add(slider);
        pane.add(sliderPanel, BorderLayout.PAGE_START);

        sliderPanel.add(new JLabel("High thresh"));
        slider = new JSlider(0, MAX_SLIDER, up);
        slider.setMajorTickSpacing(20);
        slider.setMinorTickSpacing(10);
        slider.setPaintTicks(true);
        slider.setPaintLabels(true);
        slider.addChangeListener(new ChangeListener() {
            @Override
            public void stateChanged(ChangeEvent e) {
                JSlider source = (JSlider) e.getSource();
                up = source.getValue();
            }
        });
        sliderPanel.add(slider);
        pane.add(sliderPanel, BorderLayout.PAGE_START);

        JPanel imgPanel = new JPanel();
        imgLabel = new JLabel(new ImageIcon(img));
        /// Set a Mouse Callback
        imgLabel.addMouseListener(new MouseAdapter() {
            @Override
            public void mousePressed(MouseEvent e) {
                update(e.getX(), e.getY());
            }
        });
        imgPanel.add(imgLabel);

        maskImgLabel = new JLabel();
        imgPanel.add(maskImgLabel);

        backprojLabel = new JLabel();
        imgPanel.add(backprojLabel);

        pane.add(imgPanel, BorderLayout.CENTER);
    }

    private void update(int x, int y) {
        // Fill and get the mask
        Point seed = new Point(x, y);

        int newMaskVal = 255;
        Scalar newVal = new Scalar(120, 120, 120);

        int connectivity = 8;
        int flags = connectivity + (newMaskVal << 8) + Imgproc.FLOODFILL_FIXED_RANGE + Imgproc.FLOODFILL_MASK_ONLY;

        Mat mask2 = Mat.zeros(src.rows() + 2, src.cols() + 2, CvType.CV_8U);
        Imgproc.floodFill(src, mask2, seed, newVal, new Rect(), new Scalar(low, low, low), new Scalar(up, up, up), flags);
        mask = mask2.submat(new Range(1, mask2.rows() - 1), new Range(1, mask2.cols() - 1));

        Image maskImg = HighGui.toBufferedImage(mask);
        maskImgLabel.setIcon(new ImageIcon(maskImg));

        int hBins = 30, sBins = 32;
        int[] histSize = { hBins, sBins };
        float[] ranges = { 0, 180, 0, 256 };
        int[] channels = { 0, 1 };

        /// Get the Histogram and normalize it
        Mat hist = new Mat();
        List<Mat> hsvList = Arrays.asList(hsv);
        Imgproc.calcHist(hsvList, new MatOfInt(channels), mask, hist, new MatOfInt(histSize), new MatOfFloat(ranges), false );

        Core.normalize(hist, hist, 0, 255, Core.NORM_MINMAX);

        /// Get Backprojection
        Mat backproj = new Mat();
        Imgproc.calcBackProject(hsvList, new MatOfInt(channels), hist, backproj, new MatOfFloat(ranges), 1);

        Image backprojImg = HighGui.toBufferedImage(backproj);
        backprojLabel.setIcon(new ImageIcon(backprojImg));

        frame.repaint();
        frame.pack();
    }
}

public class CalcBackProjectDemo2 {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        // Schedule a job for the event dispatch thread:
        // creating and showing this application's GUI.
        javax.swing.SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new CalcBackProject2(args);
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

- **CalcBackProject2**: A class/struct defined in this file
- **CalcBackProjectDemo2**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.core.CvType`
- `java.awt.BorderLayout`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.MatOfInt`
- `org.opencv.core.Core`
- `org.opencv.core.MatOfFloat`
- `javax.swing.JSlider`
- `java.awt.Container`
- `org.opencv.highgui.HighGui`
- `java.awt.Image`
- `javax.swing.JPanel`
- `javax.swing.BoxLayout`
- `java.util.List`
- `org.opencv.core.Rect`
- `javax.swing.event.ChangeEvent`
- `javax.swing.JFrame`
- `java.awt.event.MouseEvent`
- `javax.swing.JLabel`
- `javax.swing.event.ChangeListener`
- `org.opencv.core.Point`
- `java.awt.event.MouseAdapter`
- `org.opencv.core.Scalar`
- `org.opencv.core.Range`
- `java.util.Arrays`
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

