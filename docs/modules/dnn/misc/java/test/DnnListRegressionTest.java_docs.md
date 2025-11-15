# Documentation for `modules/dnn/misc/java/test/DnnListRegressionTest.java`

## File Metadata

- **Full Path**: `modules/dnn/misc/java/test/DnnListRegressionTest.java`
- **File Name**: `DnnListRegressionTest.java`
- **File Size**: 4,989 bytes
- **File Type**: .java
- **Link to Source**: [modules/dnn/misc/java/test/DnnListRegressionTest.java](../../../../../modules/dnn/misc/java/test/DnnListRegressionTest.java)

## Purpose and Role

This file is located in the `modules/dnn/misc/java/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.test.dnn;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfInt;
import org.opencv.core.MatOfFloat;
import org.opencv.core.MatOfByte;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.dnn.DictValue;
import org.opencv.dnn.Dnn;
import org.opencv.dnn.Layer;
import org.opencv.dnn.Net;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;
import org.opencv.test.OpenCVTestCase;

/*
*  regression test for #12324,
*    testing various java.util.List invocations,
*    which use the LIST_GET macro
*/

public class DnnListRegressionTest extends OpenCVTestCase {

    private final static String ENV_OPENCV_DNN_TEST_DATA_PATH = "OPENCV_DNN_TEST_DATA_PATH";

    private final static String ENV_OPENCV_TEST_DATA_PATH = "OPENCV_TEST_DATA_PATH";

    String modelFileName = "";
    String sourceImageFile = "";

    Net net;

    @Override
    protected void setUp() throws Exception {
        super.setUp();

        String envDnnTestDataPath = System.getenv(ENV_OPENCV_DNN_TEST_DATA_PATH);

        if(envDnnTestDataPath == null){
            isTestCaseEnabled = false;
            return;
        }

        File dnnTestDataPath = new File(envDnnTestDataPath);
        modelFileName =  new File(dnnTestDataPath, "dnn/tensorflow_inception_graph.pb").toString();

        String envTestDataPath = System.getenv(ENV_OPENCV_TEST_DATA_PATH);

        if(envTestDataPath == null) throw new Exception(ENV_OPENCV_TEST_DATA_PATH + " has to be defined!");

        File testDataPath = new File(envTestDataPath);

        File f = new File(testDataPath, "dnn/grace_hopper_227.png");
        sourceImageFile = f.toString();
        if(!f.exists()) throw new Exception("Test image is missing: " + sourceImageFile);

        net = Dnn.readNetFromTensorflow(modelFileName);

        Mat image = Imgcodecs.imread(sourceImageFile);
        assertNotNull("Loading image from file failed!", image);

        Mat inputBlob = Dnn.blobFromImage(image, 1.0, new Size(224, 224), new Scalar(0), true, true);
        assertNotNull("Converting image to blob failed!", inputBlob);

        net.setInput(inputBlob, "input");
    }

    public void testSetInputsNames() {
        List<String> inputs = new ArrayList();
        inputs.add("input");
        try {
            net.setInputsNames(inputs);
        } catch(Exception e) {
            fail("Net setInputsNames failed: " + e.getMessage());
        }
    }

    public void testForward() {
        List<Mat> outs = new ArrayList();
        List<String> outNames = new ArrayList();
        outNames.add("softmax2");
        try {
            net.forward(outs,outNames);
        } catch(Exception e) {
            fail("Net forward failed: " + e.getMessage());
        }
    }

    public void testGetMemoryConsumption() {
        int layerId = 1;
        List<MatOfInt> netInputShapes = new ArrayList();
        netInputShapes.add(new MatOfInt(1, 3, 224, 224));
        long[] weights=null;
        long[] blobs=null;
        try {
            net.getMemoryConsumption(layerId, netInputShapes, weights, blobs);
        } catch(Exception e) {
            fail("Net getMemoryConsumption failed: " + e.getMessage());
        }
    }

    public void testGetFLOPS() {
        int layerId = 1;
        List<MatOfInt> netInputShapes = new ArrayList();
        netInputShapes.add(new MatOfInt(1, 3, 224, 224));
        try {
            net.getFLOPS(layerId, netInputShapes);
        } catch(Exception e) {
            fail("Net getFLOPS failed: " + e.getMessage());
        }
    }

    public void testGetLayersShapes() {
        List<MatOfInt> netInputShapes = new ArrayList();
        netInputShapes.add(new MatOfInt(1, 3, 224, 224));

        MatOfInt layersIds = new MatOfInt();
        List<List<MatOfInt>> inLayersShapes = new ArrayList();
        List<List<MatOfInt>> outLayersShapes = new ArrayList();
        try {
            net.getLayersShapes(netInputShapes, layersIds, inLayersShapes, outLayersShapes);

            assertEquals(layersIds.total(), inLayersShapes.size());
            assertEquals(layersIds.total(), outLayersShapes.size());

            // Layer ID for "conv2d0_pre_relu/conv"
            int layerId = 1;

            MatOfInt expectedInShape = new MatOfInt(1, 3, 224, 224);
            MatOfInt expectedOutShape = new MatOfInt(1, 64, 112, 112);

            // Test inLayersShapes
            MatOfInt actualInShape = inLayersShapes.get(layerId).get(0);
            assertMatEqual(expectedInShape, actualInShape);

            // Test outLayersShapes
            MatOfInt actualOutShape = outLayersShapes.get(layerId).get(0);
            assertMatEqual(expectedOutShape, actualOutShape);

        } catch(Exception e) {
            fail("Net getLayersShapes failed: " + e.getMessage());
        }
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

- **DnnListRegressionTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.io.IOException`
- `org.opencv.imgcodecs.Imgcodecs`
- `org.opencv.imgproc.Imgproc`
- `file`
- `org.opencv.core.Size`
- `org.opencv.core.MatOfInt`
- `org.opencv.core.Core`
- `org.opencv.core.MatOfFloat`
- `org.opencv.dnn.Layer`
- `org.opencv.test.OpenCVTestCase`
- `java.util.List`
- `org.opencv.core.MatOfByte`
- `org.opencv.dnn.Net`
- `org.opencv.dnn.Dnn`
- `java.util.ArrayList`
- `java.io.File`
- `org.opencv.core.Scalar`
- `java.io.FileInputStream`
- `org.opencv.dnn.DictValue`
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

