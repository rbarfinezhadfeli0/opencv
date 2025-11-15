# Documentation for `samples/java/tutorial_code/ImgProc/tutorial_template_matching/MatchTemplateDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/ImgProc/tutorial_template_matching/MatchTemplateDemo.java`
- **File Name**: `MatchTemplateDemo.java`
- **File Size**: 5,930 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/ImgProc/tutorial_template_matching/MatchTemplateDemo.java](../../../../../samples/java/tutorial_code/ImgProc/tutorial_template_matching/MatchTemplateDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/ImgProc/tutorial_template_matching` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.awt.GridLayout;
import java.awt.Image;
import java.util.Hashtable;

import javax.swing.ImageIcon;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JSlider;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.Point;
import org.opencv.core.Scalar;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

class MatchTemplateDemoRun implements ChangeListener {

    //! [declare]
    /// Global Variables
    Boolean use_mask = false;
    Mat img = new Mat(), templ = new Mat();
    Mat mask = new Mat();

    int match_method;

    JLabel imgDisplay = new JLabel(), resultDisplay = new JLabel();
    //! [declare]

    public void run(String[] args) {
        if (args.length < 2) {
            System.out.println("Not enough parameters");
            System.out.println("Program arguments:\n<image_name> <template_name> [<mask_name>]");
            System.exit(-1);
        }

        //! [load_image]
        /// Load image and template
        img = Imgcodecs.imread(args[0], Imgcodecs.IMREAD_COLOR);
        templ = Imgcodecs.imread(args[1], Imgcodecs.IMREAD_COLOR);
        //! [load_image]

        if (args.length > 2) {
            use_mask = true;
            mask = Imgcodecs.imread(args[2], Imgcodecs.IMREAD_COLOR);
        }

        if (img.empty() || templ.empty() || (use_mask && mask.empty())) {
            System.out.println("Can't read one of the images");
            System.exit(-1);
        }

        matchingMethod();
        createJFrame();
    }

    private void matchingMethod() {
        Mat result = new Mat();

        //! [copy_source]
        /// Source image to display
        Mat img_display = new Mat();
        img.copyTo(img_display);
        //! [copy_source]

        //! [create_result_matrix]
        /// Create the result matrix
        int result_cols = img.cols() - templ.cols() + 1;
        int result_rows = img.rows() - templ.rows() + 1;

        result.create(result_rows, result_cols, CvType.CV_32FC1);
        //! [create_result_matrix]

        //! [match_template]
        /// Do the Matching and Normalize
        Boolean method_accepts_mask = (Imgproc.TM_SQDIFF == match_method || match_method == Imgproc.TM_CCORR_NORMED);
        if (use_mask && method_accepts_mask) {
            Imgproc.matchTemplate(img, templ, result, match_method, mask);
        } else {
            Imgproc.matchTemplate(img, templ, result, match_method);
        }
        //! [match_template]

        //! [normalize]
        Core.normalize(result, result, 0, 1, Core.NORM_MINMAX, -1, new Mat());
        //! [normalize]

        //! [best_match]
        /// Localizing the best match with minMaxLoc
        Point matchLoc;

        Core.MinMaxLocResult mmr = Core.minMaxLoc(result);
        //! [best_match]

        //! [match_loc]
        /// For SQDIFF and SQDIFF_NORMED, the best matches are lower values.
        /// For all the other methods, the higher the better
        if (match_method == Imgproc.TM_SQDIFF || match_method == Imgproc.TM_SQDIFF_NORMED) {
            matchLoc = mmr.minLoc;
        } else {
            matchLoc = mmr.maxLoc;
        }
        //! [match_loc]

        //! [imshow]
        /// Show me what you got
        Imgproc.rectangle(img_display, matchLoc, new Point(matchLoc.x + templ.cols(), matchLoc.y + templ.rows()),
                new Scalar(0, 0, 0), 2, 8, 0);
        Imgproc.rectangle(result, matchLoc, new Point(matchLoc.x + templ.cols(), matchLoc.y + templ.rows()),
                new Scalar(0, 0, 0), 2, 8, 0);

        Image tmpImg = HighGui.toBufferedImage(img_display);
        ImageIcon icon = new ImageIcon(tmpImg);
        imgDisplay.setIcon(icon);

        result.convertTo(result, CvType.CV_8UC1, 255.0);
        tmpImg = HighGui.toBufferedImage(result);
        icon = new ImageIcon(tmpImg);
        resultDisplay.setIcon(icon);
        //! [imshow]
    }

    @Override
    public void stateChanged(ChangeEvent e) {
        JSlider source = (JSlider) e.getSource();
        if (!source.getValueIsAdjusting()) {
            match_method = source.getValue();
            matchingMethod();
        }
    }

    private void createJFrame() {
        String title = "Source image; Control; Result image";
        JFrame frame = new JFrame(title);
        frame.setLayout(new GridLayout(2, 2));
        frame.add(imgDisplay);

        //! [create_trackbar]
        int min = 0, max = 5;
        JSlider slider = new JSlider(JSlider.VERTICAL, min, max, match_method);
        //! [create_trackbar]

        slider.setPaintTicks(true);
        slider.setPaintLabels(true);

        // Set the spacing for the minor tick mark
        slider.setMinorTickSpacing(1);

        // Customizing the labels
        Hashtable<Integer, JLabel> labelTable = new Hashtable<>();
        labelTable.put(new Integer(0), new JLabel("0 - SQDIFF"));
        labelTable.put(new Integer(1), new JLabel("1 - SQDIFF NORMED"));
        labelTable.put(new Integer(2), new JLabel("2 - TM CCORR"));
        labelTable.put(new Integer(3), new JLabel("3 - TM CCORR NORMED"));
        labelTable.put(new Integer(4), new JLabel("4 - TM COEFF"));
        labelTable.put(new Integer(5), new JLabel("5 - TM COEFF NORMED : (Method)"));
        slider.setLabelTable(labelTable);

        slider.addChangeListener(this);

        frame.add(slider);

        frame.add(resultDisplay);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.pack();
        frame.setVisible(true);
    }
}

public class MatchTemplateDemo {
    public static void main(String[] args) {
        // load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        // run code
        new MatchTemplateDemoRun().run(args);
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

- **MatchTemplateDemo**: A class/struct defined in this file
- **MatchTemplateDemoRun**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.awt.Image`
- `org.opencv.core.Scalar`
- `org.opencv.core.CvType`
- `org.opencv.highgui.HighGui`
- `java.util.Hashtable`
- `org.opencv.core.Point`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.Mat`
- `java.awt.GridLayout`
- `javax.swing.JSlider`
- `javax.swing.JLabel`
- `javax.swing.event.ChangeEvent`
- `javax.swing.event.ChangeListener`
- `org.opencv.core.Core`
- `javax.swing.ImageIcon`
- `javax.swing.JFrame`


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

