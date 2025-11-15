# Documentation for `modules/highgui/misc/java/src/java/highgui+HighGui.java`

## File Metadata

- **Full Path**: `modules/highgui/misc/java/src/java/highgui+HighGui.java`
- **File Name**: `highgui+HighGui.java`
- **File Size**: 5,771 bytes
- **File Type**: .java
- **Link to Source**: [modules/highgui/misc/java/src/java/highgui+HighGui.java](../../../../../../modules/highgui/misc/java/src/java/highgui+HighGui.java)

## Purpose and Role

This file is located in the `modules/highgui/misc/java/src/java` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.highgui;

import org.opencv.core.Mat;

import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.awt.image.BufferedImage;
import java.awt.image.DataBufferByte;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * This class was designed for use in Java applications
 * to recreate the OpenCV HighGui functionalities.
 */
public final class HighGui {

    // Constants for namedWindow
    public final static int WINDOW_NORMAL = ImageWindow.WINDOW_NORMAL;
    public final static int WINDOW_AUTOSIZE = ImageWindow.WINDOW_AUTOSIZE;

    // Control Variables
    public static int n_closed_windows = 0;
    public static int pressedKey = -1;
    public static CountDownLatch latch = new CountDownLatch(1);

    // Windows Map
    public static Map<String, ImageWindow> windows = new HashMap<String, ImageWindow>();

    public static void namedWindow(String winname) {
        namedWindow(winname, HighGui.WINDOW_AUTOSIZE);
    }

    public static void namedWindow(String winname, int flag) {
        ImageWindow newWin = new ImageWindow(winname, flag);
        if (windows.get(winname) == null) windows.put(winname, newWin);
    }

    public static void imshow(String winname, Mat img) {
        if (img.empty()) {
            System.err.println("Error: Empty image in imshow");
            System.exit(-1);
        } else {
            ImageWindow tmpWindow = windows.get(winname);
            if (tmpWindow == null) {
                ImageWindow newWin = new ImageWindow(winname, img);
                windows.put(winname, newWin);
            } else {
                tmpWindow.setMat(img);
            }
        }
    }

    public static Image toBufferedImage(Mat m) {
        int type = BufferedImage.TYPE_BYTE_GRAY;

        if (m.channels() > 1) {
            type = BufferedImage.TYPE_3BYTE_BGR;
        }
        BufferedImage image = new BufferedImage(m.cols(), m.rows(), type);
        final byte[] targetPixels = ((DataBufferByte) image.getRaster().getDataBuffer()).getData();
        m.get(0, 0, targetPixels);

        return image;
    }

    public static JFrame createJFrame(String title, int flag) {
        JFrame frame = new JFrame(title);

        frame.addWindowListener(new java.awt.event.WindowAdapter() {
            @Override
            public void windowClosing(java.awt.event.WindowEvent windowEvent) {
                n_closed_windows++;
                if (n_closed_windows == windows.size()) latch.countDown();
            }
        });

        frame.addKeyListener(new KeyListener() {
            @Override
            public void keyTyped(KeyEvent e) {
            }

            @Override
            public void keyReleased(KeyEvent e) {
            }

            @Override
            public void keyPressed(KeyEvent e) {
                pressedKey = e.getKeyCode();
                latch.countDown();
            }
        });

        if (flag == WINDOW_AUTOSIZE) frame.setResizable(false);

        return frame;
    }

    public static void waitKey(){
        waitKey(0);
    }

    public static int waitKey(int delay) {
        // Reset control values
        latch = new CountDownLatch(1);
        n_closed_windows = 0;
        pressedKey = -1;

        // If there are no windows to be shown return
        if (windows.isEmpty()) {
            System.err.println("Error: waitKey must be used after an imshow");
            System.exit(-1);
        }

        // Remove the unused windows
        Iterator<Map.Entry<String,
                ImageWindow>> iter = windows.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<String,
                    ImageWindow> entry = iter.next();
            ImageWindow win = entry.getValue();
            if (win.alreadyUsed) {
                iter.remove();
                win.frame.dispose();
            }
        }

        // (if) Create (else) Update frame
        for (ImageWindow win : windows.values()) {

            if (win.img != null) {

                ImageIcon icon = new ImageIcon(toBufferedImage(win.img));

                if (win.lbl == null) {
                    JFrame frame = createJFrame(win.name, win.flag);
                    JLabel lbl = new JLabel(icon);
                    win.setFrameLabelVisible(frame, lbl);
                } else {
                    win.lbl.setIcon(icon);
                }
            } else {
                System.err.println("Error: no imshow associated with" + " namedWindow: \"" + win.name + "\"");
                System.exit(-1);
            }
        }

        try {
            if (delay == 0) {
                latch.await();
            } else {
                latch.await(delay, TimeUnit.MILLISECONDS);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // Set all windows as already used
        for (ImageWindow win : windows.values())
            win.alreadyUsed = true;

        return pressedKey;
    }

    public static void destroyWindow(String winname) {
        ImageWindow tmpWin = windows.get(winname);
        if (tmpWin != null) windows.remove(winname);
    }

    public static void destroyAllWindows() {
        windows.clear();
    }

    public static void resizeWindow(String winname, int width, int height) {
        ImageWindow tmpWin = windows.get(winname);
        if (tmpWin != null) tmpWin.setNewDimension(width, height);
    }

    public static void moveWindow(String winname, int x, int y) {
        ImageWindow tmpWin = windows.get(winname);
        if (tmpWin != null) tmpWin.setNewPosition(x, y);
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

- **was**: A class/struct defined in this file
- **HighGui**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.util.concurrent.CountDownLatch`
- `java.awt.event.KeyListener`
- `javax.swing.`
- `java.awt.image.BufferedImage`
- `java.util.Iterator`
- `java.awt.event.KeyEvent`
- `java.util.concurrent.TimeUnit`
- `java.awt.image.DataBufferByte`
- `java.awt.`
- `org.opencv.core.Mat`
- `java.util.HashMap`
- `java.util.Map`


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

