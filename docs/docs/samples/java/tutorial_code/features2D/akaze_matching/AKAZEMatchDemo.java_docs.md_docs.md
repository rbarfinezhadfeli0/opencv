# Documentation for `docs/samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java_docs.md`
- **File Name**: `AKAZEMatchDemo.java_docs.md`
- **File Size**: 10,879 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java_docs.md](../../../../../../docs/samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/java/tutorial_code/features2D/akaze_matching` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java`

## File Metadata

- **Full Path**: `samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java`
- **File Name**: `AKAZEMatchDemo.java`
- **File Size**: 7,015 bytes
- **File Type**: .java
- **Link to Source**: [samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java](../../../../../samples/java/tutorial_code/features2D/akaze_matching/AKAZEMatchDemo.java)

## Purpose and Role

This file is located in the `samples/java/tutorial_code/features2D/akaze_matching` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.DMatch;
import org.opencv.core.KeyPoint;
import org.opencv.core.Mat;
import org.opencv.core.MatOfDMatch;
import org.opencv.core.MatOfKeyPoint;
import org.opencv.core.Scalar;
import org.opencv.features2d.AKAZE;
import org.opencv.features2d.DescriptorMatcher;
import org.opencv.features2d.Features2d;
import org.opencv.highgui.HighGui;
import org.opencv.imgcodecs.Imgcodecs;
import org.w3c.dom.Document;
import org.xml.sax.SAXException;

class AKAZEMatch {
    public void run(String[] args) {
        //! [load]
        String filename1 = args.length > 2 ? args[0] : "../data/graf1.png";
        String filename2 = args.length > 2 ? args[1] : "../data/graf3.png";
        String filename3 = args.length > 2 ? args[2] : "../data/H1to3p.xml";
        Mat img1 = Imgcodecs.imread(filename1, Imgcodecs.IMREAD_GRAYSCALE);
        Mat img2 = Imgcodecs.imread(filename2, Imgcodecs.IMREAD_GRAYSCALE);
        if (img1.empty() || img2.empty()) {
            System.err.println("Cannot read images!");
            System.exit(0);
        }

        File file = new File(filename3);
        DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder documentBuilder;
        Document document;
        Mat homography = new Mat(3, 3, CvType.CV_64F);
        double[] homographyData = new double[(int) (homography.total()*homography.channels())];
        try {
            documentBuilder = documentBuilderFactory.newDocumentBuilder();
            document = documentBuilder.parse(file);
            String homographyStr = document.getElementsByTagName("data").item(0).getTextContent();
            String[] splited = homographyStr.split("\\s+");
            int idx = 0;
            for (String s : splited) {
                if (!s.isEmpty()) {
                    homographyData[idx] = Double.parseDouble(s);
                    idx++;
                }
            }
        } catch (ParserConfigurationException e) {
            e.printStackTrace();
            System.exit(0);
        } catch (SAXException e) {
            e.printStackTrace();
            System.exit(0);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(0);
        }
        homography.put(0, 0, homographyData);
        //! [load]

        //! [AKAZE]
        AKAZE akaze = AKAZE.create();
        MatOfKeyPoint kpts1 = new MatOfKeyPoint(), kpts2 = new MatOfKeyPoint();
        Mat desc1 = new Mat(), desc2 = new Mat();
        akaze.detectAndCompute(img1, new Mat(), kpts1, desc1);
        akaze.detectAndCompute(img2, new Mat(), kpts2, desc2);
        //! [AKAZE]

        //! [2-nn matching]
        DescriptorMatcher matcher = DescriptorMatcher.create(DescriptorMatcher.BRUTEFORCE_HAMMING);
        List<MatOfDMatch> knnMatches = new ArrayList<>();
        matcher.knnMatch(desc1, desc2, knnMatches, 2);
        //! [2-nn matching]

        //! [ratio test filtering]
        float ratioThreshold = 0.8f; // Nearest neighbor matching ratio
        List<KeyPoint> listOfMatched1 = new ArrayList<>();
        List<KeyPoint> listOfMatched2 = new ArrayList<>();
        List<KeyPoint> listOfKeypoints1 = kpts1.toList();
        List<KeyPoint> listOfKeypoints2 = kpts2.toList();
        for (int i = 0; i < knnMatches.size(); i++) {
            DMatch[] matches = knnMatches.get(i).toArray();
            float dist1 = matches[0].distance;
            float dist2 = matches[1].distance;
            if (dist1 < ratioThreshold * dist2) {
                listOfMatched1.add(listOfKeypoints1.get(matches[0].queryIdx));
                listOfMatched2.add(listOfKeypoints2.get(matches[0].trainIdx));
            }
        }
        //! [ratio test filtering]

        //! [homography check]
        double inlierThreshold = 2.5; // Distance threshold to identify inliers with homography check
        List<KeyPoint> listOfInliers1 = new ArrayList<>();
        List<KeyPoint> listOfInliers2 = new ArrayList<>();
        List<DMatch> listOfGoodMatches = new ArrayList<>();
        for (int i = 0; i < listOfMatched1.size(); i++) {
            Mat col = new Mat(3, 1, CvType.CV_64F);
            double[] colData = new double[(int) (col.total() * col.channels())];
            colData[0] = listOfMatched1.get(i).pt.x;
            colData[1] = listOfMatched1.get(i).pt.y;
            colData[2] = 1.0;
            col.put(0, 0, colData);

            Mat colRes = new Mat();
            Core.gemm(homography, col, 1.0, new Mat(), 0.0, colRes);
            colRes.get(0, 0, colData);
            Core.multiply(colRes, new Scalar(1.0 / colData[2]), col);
            col.get(0, 0, colData);

            double dist = Math.sqrt(Math.pow(colData[0] - listOfMatched2.get(i).pt.x, 2) +
                    Math.pow(colData[1] - listOfMatched2.get(i).pt.y, 2));

            if (dist < inlierThreshold) {
                listOfGoodMatches.add(new DMatch(listOfInliers1.size(), listOfInliers2.size(), 0));
                listOfInliers1.add(listOfMatched1.get(i));
                listOfInliers2.add(listOfMatched2.get(i));
            }
        }
        //! [homography check]

        //! [draw final matches]
        Mat res = new Mat();
        MatOfKeyPoint inliers1 = new MatOfKeyPoint(listOfInliers1.toArray(new KeyPoint[listOfInliers1.size()]));
        MatOfKeyPoint inliers2 = new MatOfKeyPoint(listOfInliers2.toArray(new KeyPoint[listOfInliers2.size()]));
        MatOfDMatch goodMatches = new MatOfDMatch(listOfGoodMatches.toArray(new DMatch[listOfGoodMatches.size()]));
        Features2d.drawMatches(img1, inliers1, img2, inliers2, goodMatches, res);
        Imgcodecs.imwrite("akaze_result.png", res);

        double inlierRatio = listOfInliers1.size() / (double) listOfMatched1.size();
        System.out.println("A-KAZE Matching Results");
        System.out.println("*******************************");
        System.out.println("# Keypoints 1:                        \t" + listOfKeypoints1.size());
        System.out.println("# Keypoints 2:                        \t" + listOfKeypoints2.size());
        System.out.println("# Matches:                            \t" + listOfMatched1.size());
        System.out.println("# Inliers:                            \t" + listOfInliers1.size());
        System.out.println("# Inliers Ratio:                      \t" + inlierRatio);

        HighGui.imshow("result", res);
        HighGui.waitKey();
        //! [draw final matches]

        System.exit(0);
    }
}

public class AKAZEMatchDemo {
    public static void main(String[] args) {
        // Load the native OpenCV library
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

        new AKAZEMatch().run(args);
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

- **AKAZEMatch**: A class/struct defined in this file
- **AKAZEMatchDemo**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `org.opencv.features2d.AKAZE`
- `org.opencv.core.CvType`
- `java.io.IOException`
- `org.opencv.imgcodecs.Imgcodecs`
- `javax.xml.parsers.DocumentBuilderFactory`
- `org.opencv.core.Core`
- `org.opencv.highgui.HighGui`
- `org.opencv.core.DMatch`
- `java.util.List`
- `org.w3c.dom.Document`
- `org.opencv.features2d.Features2d`
- `javax.xml.parsers.DocumentBuilder`
- `java.util.ArrayList`
- `org.opencv.core.KeyPoint`
- `javax.xml.parsers.ParserConfigurationException`
- `java.io.File`
- `org.opencv.core.Scalar`
- `org.opencv.core.MatOfKeyPoint`
- `org.opencv.features2d.DescriptorMatcher`
- `org.opencv.core.MatOfDMatch`
- `org.xml.sax.SAXException`
- `org.opencv.core.Mat`


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

