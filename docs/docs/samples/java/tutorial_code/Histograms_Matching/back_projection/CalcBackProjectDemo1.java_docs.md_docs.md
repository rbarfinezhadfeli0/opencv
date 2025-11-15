# Documentation for `docs/samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java_docs.md`
- **File Name**: `CalcBackProjectDemo1.java_docs.md`
- **File Size**: 9,827 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java_docs.md](../../../../../../docs/samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/Histograms_Matching/back_projection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java`
- **File Name**: `CalcBackProjectDemo1.java`
- **File Size**: 5,937 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java](../../../../../samples/java/tutorial_code/Histograms_Matching/back_projection/CalcBackProjectDemo1.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/Histograms_Matching/back_projection` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.Image;
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
import org.opencv.core.Scalar;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class CalcBackProject1 {
    private Mat hue;
    private Mat histImg = new Mat();
    private JFrame frame;
    private JLabel imgLabel;
    private JLabel backprojLabel;
    private JLabel histImgLabel;
    private static final int MAX_SLIDER = 180;
    private int bins = 25;

    public CalcBackProject1(String[] args) {
        //! [Read the image]
        if (args.length != 1) {
            System.err.println("You must supply one argument that corresponds to the path to the image.");
            System.exit(0);
        }

        Mat src = Imgcodecs.imread(args[0]);
        if (src.empty()) {
            System.err.println("Empty image: " + args[0]);
            System.exit(0);
        }
        //! [Read the image]

        //! [Transform it to HSV]
        Mat hsv = new Mat();
        Imgproc.cvtColor(src, hsv, Imgproc.COLOR_BGR2HSV);
        //! [Transform it to HSV]

        //! [Use only the Hue value]
        hue = new Mat(hsv.size(), hsv.depth());
        Core.mixChannels(Arrays.asList(hsv), Arrays.asList(hue), new MatOfInt(0, 0));
        //! [Use only the Hue value]

        // Create and set up the window.
        frame = new JFrame("Back Projection 1 demo");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Set up the content pane.
        Image img = HighGui.toBufferedImage(src);
        addComponentsToPane(frame.getContentPane(), img);
        //! [Show the image]
        // Use the content pane's default BorderLayout. No need for
        // setLayout(new BorderLayout());
        // Display the window.
        frame.pack();
        frame.setVisible(true);
        //! [Show the image]
    }

    private void addComponentsToPane(Container pane, Image img) {
        if (!(pane.getLayout() instanceof BorderLayout)) {
            pane.add(new JLabel("Container doesn't use BorderLayout!"));
            return;
        }

        //! [Create Trackbar to enter the number of bins]
        JPanel sliderPanel = new JPanel();
        sliderPanel.setLayout(new BoxLayout(sliderPanel, BoxLayout.PAGE_AXIS));

        sliderPanel.add(new JLabel("* Hue  bins: "));
        JSlider slider = new JSlider(0, MAX_SLIDER, bins);
        slider.setMajorTickSpacing(25);
        slider.setMinorTickSpacing(5);
        slider.setPaintTicks(true);
        slider.setPaintLabels(true);
        slider.addChangeListener(new ChangeListener() {
            @Override
            public void stateChanged(ChangeEvent e) {
                JSlider source = (JSlider) e.getSource();
                bins = source.getValue();
                update();
            }
        });
        sliderPanel.add(slider);
        pane.add(sliderPanel, BorderLayout.PAGE_START);
        //! [Create Trackbar to enter the number of bins]

        JPanel imgPanel = new JPanel();
        imgLabel = new JLabel(new ImageIcon(img));
        imgPanel.add(imgLabel);

        backprojLabel = new JLabel();
        imgPanel.add(backprojLabel);

        histImgLabel = new JLabel();
        imgPanel.add(histImgLabel);
        pane.add(imgPanel, BorderLayout.CENTER);
    }

    private void update() {
        //! [initialize]
        int histSize = Math.max(bins, 2);
        float[] hueRange = {0, 180};
        //! [initialize]

        //! [Get the Histogram and normalize it]
        Mat hist = new Mat();
        List<Mat> hueList = Arrays.asList(hue);
        Imgproc.calcHist(hueList, new MatOfInt(0), new Mat(), hist, new MatOfInt(histSize), new MatOfFloat(hueRange), false);
        Core.normalize(hist, hist, 0, 255, Core.NORM_MINMAX);
        //! [Get the Histogram and normalize it]

        //! [Get Backprojection]
        Mat backproj = new Mat();
        Imgproc.calcBackProject(hueList, new MatOfInt(0), hist, backproj, new MatOfFloat(hueRange), 1);
        //! [Get Backprojection]

        //! [Draw the backproj]
        Image backprojImg = HighGui.toBufferedImage(backproj);
        backprojLabel.setIcon(new ImageIcon(backprojImg));
        //! [Draw the backproj]

        //! [Draw the histogram]
        int w = 400, h = 400;
        int binW = (int) Math.round((double) w / histSize);
        histImg = Mat.zeros(h, w, CvType.CV_8UC3);

        float[] histData = new float[(int) (hist.total() * hist.channels())];
        hist.get(0, 0, histData);
        for (int i = 0; i < bins; i++) {
            Imgproc.rectangle(histImg, new Point(i * binW, h),
                    new Point((i + 1) * binW, h - Math.round(histData[i] * h / 255.0)), new Scalar(0, 0, 255), Imgproc.FILLED);
        }
        Image histImage = HighGui.toBufferedImage(histImg);
        histImgLabel.setIcon(new ImageIcon(histImage));
        //! [Draw the histogram]

        frame.repaint();
        frame.pack();
    }
}

public class CalcBackProjectDemo1 {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        // Schedule a job for the event dispatch thread:
        // creating and showing this application's GUI.
        javax.swing.SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new CalcBackProject1(args);
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

- **CalcBackProjectDemo1**: A class/struct defined in this file
- **CalcBackProject1**: A class/struct defined in this file


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
- `javax.swing.event.ChangeEvent`
- `javax.swing.JFrame`
- `javax.swing.JLabel`
- `javax.swing.event.ChangeListener`
- `org.opencv.core.Point`
- `org.opencv.core.Scalar`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

