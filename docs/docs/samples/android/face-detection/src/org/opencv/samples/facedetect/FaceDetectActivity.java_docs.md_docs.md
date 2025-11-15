# Documentation for `docs/samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java_docs.md`
- **File Name**: `FaceDetectActivity.java_docs.md`
- **File Size**: 11,584 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java_docs.md](../../../../../../../../../docs/samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/face-detection/src/org/opencv/samples/facedetect` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java`

## File Metadata

- **Full Path**: `samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java`
- **File Name**: `FaceDetectActivity.java`
- **File Size**: 7,470 bytes
- **File Type**: .java
- **Link to Source**: [samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java](../../../../../../../../samples/android/face-detection/src/org/opencv/samples/facedetect/FaceDetectActivity.java)

## Purpose and Role

This file is located in the `samples/android/face-detection/src/org/opencv/samples/facedetect` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.samples.facedetect;

import java.lang.Math;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Collections;
import java.util.List;

import org.opencv.android.CameraActivity;
import org.opencv.android.CameraBridgeViewBase.CvCameraViewFrame;
import org.opencv.android.OpenCVLoader;
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.core.MatOfByte;
import org.opencv.core.Point;
import org.opencv.core.Rect;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.android.CameraBridgeViewBase;
import org.opencv.android.CameraBridgeViewBase.CvCameraViewListener2;
import org.opencv.objdetect.FaceDetectorYN;
import org.opencv.imgproc.Imgproc;

import android.app.Activity;
import android.content.Context;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.WindowManager;
import android.widget.Toast;

public class FaceDetectActivity extends CameraActivity implements CvCameraViewListener2 {

    private static final String    TAG  = "OCVSample::Activity";

    private static final Scalar    BOX_COLOR         = new Scalar(0, 255, 0);
    private static final Scalar    RIGHT_EYE_COLOR   = new Scalar(255, 0, 0);
    private static final Scalar    LEFT_EYE_COLOR    = new Scalar(0, 0, 255);
    private static final Scalar    NOSE_TIP_COLOR    = new Scalar(0, 255, 0);
    private static final Scalar    MOUTH_RIGHT_COLOR = new Scalar(255, 0, 255);
    private static final Scalar    MOUTH_LEFT_COLOR  = new Scalar(0, 255, 255);

    private Mat                    mRgba;
    private Mat                    mBgr;
    private Mat                    mBgrScaled;
    private Size                   mInputSize = null;
    private float                  mScale = 2.f;
    private MatOfByte              mModelBuffer;
    private MatOfByte              mConfigBuffer;
    private FaceDetectorYN         mFaceDetector;
    private Mat                    mFaces;

    private CameraBridgeViewBase   mOpenCvCameraView;

    public FaceDetectActivity() {
        Log.i(TAG, "Instantiated new " + this.getClass());
    }

    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        Log.i(TAG, "called onCreate");
        super.onCreate(savedInstanceState);

        if (OpenCVLoader.initLocal()) {
            Log.i(TAG, "OpenCV loaded successfully");
        } else {
            Log.e(TAG, "OpenCV initialization failed!");
            (Toast.makeText(this, "OpenCV initialization failed!", Toast.LENGTH_LONG)).show();
            return;
        }

        byte[] buffer;
        try {
            // load cascade file from application resources
            InputStream is = getResources().openRawResource(R.raw.face_detection_yunet_2023mar);

            int size = is.available();
            buffer = new byte[size];
            int bytesRead = is.read(buffer);
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
            Log.e(TAG, "Failed to ONNX model from resources! Exception thrown: " + e);
            (Toast.makeText(this, "Failed to ONNX model from resources!", Toast.LENGTH_LONG)).show();
            return;
        }

        mModelBuffer = new MatOfByte(buffer);
        mConfigBuffer = new MatOfByte();

        mFaceDetector = FaceDetectorYN.create("onnx", mModelBuffer, mConfigBuffer, new Size(320, 320));
        if (mFaceDetector == null) {
            Log.e(TAG, "Failed to create FaceDetectorYN!");
            (Toast.makeText(this, "Failed to create FaceDetectorYN!", Toast.LENGTH_LONG)).show();
            return;
        } else
            Log.i(TAG, "FaceDetectorYN initialized successfully!");


        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

        setContentView(R.layout.face_detect_surface_view);

        mOpenCvCameraView = (CameraBridgeViewBase) findViewById(R.id.fd_activity_surface_view);
        mOpenCvCameraView.setVisibility(CameraBridgeViewBase.VISIBLE);
        mOpenCvCameraView.setCvCameraViewListener(this);
    }

    @Override
    public void onPause()
    {
        super.onPause();
        if (mOpenCvCameraView != null)
            mOpenCvCameraView.disableView();
    }

    @Override
    public void onResume()
    {
        super.onResume();
        if (mOpenCvCameraView != null)
            mOpenCvCameraView.enableView();
    }

    @Override
    protected List<? extends CameraBridgeViewBase> getCameraViewList() {
        return Collections.singletonList(mOpenCvCameraView);
    }

    public void onDestroy() {
        super.onDestroy();
        mOpenCvCameraView.disableView();
    }

    public void onCameraViewStarted(int width, int height) {
        mRgba = new Mat();
        mBgr = new Mat();
        mBgrScaled = new Mat();
        mFaces = new Mat();
    }

    public void onCameraViewStopped() {
        mRgba.release();
        mBgr.release();
        mBgrScaled.release();
        mFaces.release();
    }

    public void visualize(Mat rgba, Mat faces) {

        int thickness = 2;
        float[] faceData = new float[faces.cols() * faces.channels()];

        for (int i = 0; i < faces.rows(); i++)
        {
            faces.get(i, 0, faceData);

            Log.d(TAG, "Detected face (" + faceData[0] + ", " + faceData[1] + ", " +
                                           faceData[2] + ", " + faceData[3] + ")");

            // Draw bounding box
            Imgproc.rectangle(rgba, new Rect(Math.round(mScale*faceData[0]), Math.round(mScale*faceData[1]),
                                             Math.round(mScale*faceData[2]), Math.round(mScale*faceData[3])),
                              BOX_COLOR, thickness);
            // Draw landmarks
            Imgproc.circle(rgba, new Point(Math.round(mScale*faceData[4]), Math.round(mScale*faceData[5])),
                           2, RIGHT_EYE_COLOR, thickness);
            Imgproc.circle(rgba, new Point(Math.round(mScale*faceData[6]), Math.round(mScale*faceData[7])),
                           2, LEFT_EYE_COLOR, thickness);
            Imgproc.circle(rgba, new Point(Math.round(mScale*faceData[8]), Math.round(mScale*faceData[9])),
                           2, NOSE_TIP_COLOR, thickness);
            Imgproc.circle(rgba, new Point(Math.round(mScale*faceData[10]), Math.round(mScale*faceData[11])),
                           2, MOUTH_RIGHT_COLOR, thickness);
            Imgproc.circle(rgba, new Point(Math.round(mScale*faceData[12]), Math.round(mScale*faceData[13])),
                           2, MOUTH_LEFT_COLOR, thickness);
        }
    }

    public Mat onCameraFrame(CvCameraViewFrame inputFrame) {

        mRgba = inputFrame.rgba();

        Size inputSize = new Size(Math.round(mRgba.cols()/mScale), Math.round(mRgba.rows()/mScale));
        if (mInputSize == null || !mInputSize.equals(inputSize)) {
            mInputSize = inputSize;
            mFaceDetector.setInputSize(mInputSize);
        }

        Imgproc.cvtColor(mRgba, mBgr, Imgproc.COLOR_RGBA2BGR);
        Imgproc.resize(mBgr, mBgrScaled, mInputSize);

        if (mFaceDetector != null) {
            int status = mFaceDetector.detect(mBgrScaled, mFaces);
            Log.d(TAG, "Detector returned status " + status);
            visualize(mRgba, mFaces);
        }

        return mRgba;
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

- **FaceDetectActivity**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.io.InputStream`
- `resources`
- `java.io.IOException`
- `android.content.Context`
- `org.opencv.android.OpenCVLoader`
- `org.opencv.imgproc.Imgproc`
- `org.opencv.core.Size`
- `org.opencv.objdetect.FaceDetectorYN`
- `org.opencv.core.Core`
- `java.io.FileOutputStream`
- `java.util.Collections`
- `org.opencv.android.CameraBridgeViewBase.CvCameraViewFrame`
- `java.util.List`
- `application`
- `org.opencv.core.Rect`
- `org.opencv.core.MatOfByte`
- `android.view.MenuItem`
- `android.os.Bundle`
- `android.util.Log`
- `java.lang.Math`
- `org.opencv.android.CameraActivity`
- `org.opencv.android.CameraBridgeViewBase`
- `org.opencv.core.Point`
- `java.io.File`
- `org.opencv.core.Scalar`
- `android.widget.Toast`
- `android.app.Activity`
- `android.view.WindowManager`
- `android.view.Menu`
- `org.opencv.android.CameraBridgeViewBase.CvCameraViewListener2`
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

