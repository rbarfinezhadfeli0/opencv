# Documentation for `samples/java/tutorial_code/ImgProc/erosion_dilatation/MorphologyDemo1.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/ImgProc/erosion_dilatation/MorphologyDemo1.java`
- **File Name**: `MorphologyDemo1.java`
- **File Size**: 5,697 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/ImgProc/erosion_dilatation/MorphologyDemo1.java](../../../../../samples/java/tutorial_code/ImgProc/erosion_dilatation/MorphologyDemo1.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/ImgProc/erosion_dilatation` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.Image;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

import javax.swing.BoxLayout;
import javax.swing.ImageIcon;
import javax.swing.JComboBox;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JSlider;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.Point;
import org.opencv.core.Size;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

public class MorphologyDemo1 {
    private static final String[] ELEMENT_TYPE = { "Rectangle", "Cross", "Ellipse", "Diamond" };
    private static final String[] MORPH_OP = { "Erosion", "Dilatation" };
    private static final int MAX_KERNEL_SIZE = 21;
    private Mat matImgSrc;
    private Mat matImgDst = new Mat();
    private int elementType = Imgproc.MORPH_RECT;
    private int kernelSize = 0;
    private boolean doErosion = true;
    private JFrame frame;
    private JLabel imgLabel;

    //! [constructor]
    public MorphologyDemo1(String[] args) {
        String imagePath = args.length > 0 ? args[0] : "../data/LinuxLogo.jpg";
        matImgSrc = Imgcodecs.imread(imagePath);
        if (matImgSrc.empty()) {
            System.out.println("Empty image: " + imagePath);
            System.exit(0);
        }

        // Create and set up the window.
        frame = new JFrame("Erosion and dilatation demo");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Set up the content pane.
        Image img = HighGui.toBufferedImage(matImgSrc);
        addComponentsToPane(frame.getContentPane(), img);
        // Use the content pane's default BorderLayout. No need for
        // setLayout(new BorderLayout());
        // Display the window.
        frame.pack();
        frame.setVisible(true);
    }
    //! [constructor]

    //! [components]
    private void addComponentsToPane(Container pane, Image img) {
        if (!(pane.getLayout() instanceof BorderLayout)) {
            pane.add(new JLabel("Container doesn't use BorderLayout!"));
            return;
        }

        JPanel sliderPanel = new JPanel();
        sliderPanel.setLayout(new BoxLayout(sliderPanel, BoxLayout.PAGE_AXIS));

        JComboBox<String> elementTypeBox = new JComboBox<>(ELEMENT_TYPE);
        elementTypeBox.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                @SuppressWarnings("unchecked")
                JComboBox<String> cb = (JComboBox<String>)e.getSource();
                if (cb.getSelectedIndex() == 0) {
                    elementType = Imgproc.MORPH_RECT;
                } else if (cb.getSelectedIndex() == 1) {
                    elementType = Imgproc.MORPH_CROSS;
                } else if (cb.getSelectedIndex() == 2) {
                    elementType = Imgproc.MORPH_ELLIPSE;
                } else if (cb.getSelectedIndex() == 3) {
                    elementType = Imgproc.MORPH_DIAMOND;
                }
                update();
            }
        });
        sliderPanel.add(elementTypeBox);

        sliderPanel.add(new JLabel("Kernel size: 2n + 1"));
        JSlider slider = new JSlider(0, MAX_KERNEL_SIZE, 0);
        slider.setMajorTickSpacing(5);
        slider.setMinorTickSpacing(5);
        slider.setPaintTicks(true);
        slider.setPaintLabels(true);
        slider.addChangeListener(new ChangeListener() {
            @Override
            public void stateChanged(ChangeEvent e) {
                JSlider source = (JSlider) e.getSource();
                kernelSize = source.getValue();
                update();
            }
        });
        sliderPanel.add(slider);

        JComboBox<String> morphOpBox = new JComboBox<>(MORPH_OP);
        morphOpBox.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                @SuppressWarnings("unchecked")
                JComboBox<String> cb = (JComboBox<String>)e.getSource();
                doErosion = cb.getSelectedIndex() == 0;
                update();
            }
        });
        sliderPanel.add(morphOpBox);

        pane.add(sliderPanel, BorderLayout.PAGE_START);
        imgLabel = new JLabel(new ImageIcon(img));
        pane.add(imgLabel, BorderLayout.CENTER);
    }
    //! [components]

    //! [update]
    private void update() {
        //! [kernel]
        Mat element = Imgproc.getStructuringElement(elementType, new Size(2 * kernelSize + 1, 2 * kernelSize + 1),
                new Point(kernelSize, kernelSize));
        //! [kernel]

        if (doErosion) {
            //! [erosion]
            Imgproc.erode(matImgSrc, matImgDst, element);
            //! [erosion]
        } else {
            //! [dilation]
            Imgproc.dilate(matImgSrc, matImgDst, element);
            //! [dilation]
        }
        Image img = HighGui.toBufferedImage(matImgDst);
        imgLabel.setIcon(new ImageIcon(img));
        frame.repaint();
    }
    //! [update]

    //! [main]
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        // Schedule a job for the event dispatch thread:
        // creating and showing this application's GUI.
        javax.swing.SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new MorphologyDemo1(args);
            }
        });
    }
    //! [main]
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

- **MorphologyDemo1**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.awt.BorderLayout`
- `org.opencv.imgcodecs.Imgcodecs`
- `java.awt.event.ActionEvent`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.Size`
- `org.opencv.highgui.HighGui`
- `javax.swing.JSlider`
- `org.opencv.core.Core`
- `java.awt.Container`
- `java.awt.Image`
- `javax.swing.JPanel`
- `javax.swing.JComboBox`
- `javax.swing.BoxLayout`
- `javax.swing.event.ChangeEvent`
- `javax.swing.JFrame`
- `javax.swing.JLabel`
- `javax.swing.event.ChangeListener`
- `org.opencv.core.Point`
- `java.awt.event.ActionListener`
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

