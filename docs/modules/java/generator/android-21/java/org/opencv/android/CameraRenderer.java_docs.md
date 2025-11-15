# Documentation for `modules/java/generator/android-21/java/org/opencv/android/CameraRenderer.java`

## File Metadata

- **Full Path**: `modules/java/generator/android-21/java/org/opencv/android/CameraRenderer.java`
- **File Name**: `CameraRenderer.java`
- **File Size**: 6,735 bytes
- **File Type**: .java
- **Link to Source**: [modules/java/generator/android-21/java/org/opencv/android/CameraRenderer.java](../../../../../../../../modules/java/generator/android-21/java/org/opencv/android/CameraRenderer.java)

## Purpose and Role

This file is located in the `modules/java/generator/android-21/java/org/opencv/android` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.android;

import java.io.IOException;
import java.util.List;

import android.annotation.TargetApi;
import android.hardware.Camera;
import android.hardware.Camera.Size;
import android.os.Build;
import android.util.Log;

@TargetApi(15)
@SuppressWarnings("deprecation")
public class CameraRenderer extends CameraGLRendererBase {

    public static final String LOGTAG = "CameraRenderer";

    private Camera mCamera;
    private boolean mPreviewStarted = false;

    CameraRenderer(CameraGLSurfaceView view) {
        super(view);
    }

    @Override
    protected synchronized void closeCamera() {
        Log.i(LOGTAG, "closeCamera");
        if(mCamera != null) {
            mCamera.stopPreview();
            mPreviewStarted = false;
            mCamera.release();
            mCamera = null;
        }
    }

    @Override
    protected synchronized void openCamera(int id) {
        Log.i(LOGTAG, "openCamera");
        closeCamera();
        if (id == CameraBridgeViewBase.CAMERA_ID_ANY) {
            Log.d(LOGTAG, "Trying to open camera with old open()");
            try {
                mCamera = Camera.open();
            }
            catch (Exception e){
                Log.e(LOGTAG, "Camera is not available (in use or does not exist): " + e.getLocalizedMessage());
            }

            if(mCamera == null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
                boolean connected = false;
                for (int camIdx = 0; camIdx < Camera.getNumberOfCameras(); ++camIdx) {
                    Log.d(LOGTAG, "Trying to open camera with new open(" + camIdx + ")");
                    try {
                        mCamera = Camera.open(camIdx);
                        connected = true;
                    } catch (RuntimeException e) {
                        Log.e(LOGTAG, "Camera #" + camIdx + "failed to open: " + e.getLocalizedMessage());
                    }
                    if (connected) break;
                }
            }
        } else {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
                int localCameraIndex = mCameraIndex;
                if (mCameraIndex == CameraBridgeViewBase.CAMERA_ID_BACK) {
                    Log.i(LOGTAG, "Trying to open BACK camera");
                    Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
                    for (int camIdx = 0; camIdx < Camera.getNumberOfCameras(); ++camIdx) {
                        Camera.getCameraInfo( camIdx, cameraInfo );
                        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                            localCameraIndex = camIdx;
                            break;
                        }
                    }
                } else if (mCameraIndex == CameraBridgeViewBase.CAMERA_ID_FRONT) {
                    Log.i(LOGTAG, "Trying to open FRONT camera");
                    Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
                    for (int camIdx = 0; camIdx < Camera.getNumberOfCameras(); ++camIdx) {
                        Camera.getCameraInfo( camIdx, cameraInfo );
                        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                            localCameraIndex = camIdx;
                            break;
                        }
                    }
                }
                if (localCameraIndex == CameraBridgeViewBase.CAMERA_ID_BACK) {
                    Log.e(LOGTAG, "Back camera not found!");
                } else if (localCameraIndex == CameraBridgeViewBase.CAMERA_ID_FRONT) {
                    Log.e(LOGTAG, "Front camera not found!");
                } else {
                    Log.d(LOGTAG, "Trying to open camera with new open(" + localCameraIndex + ")");
                    try {
                        mCamera = Camera.open(localCameraIndex);
                    } catch (RuntimeException e) {
                        Log.e(LOGTAG, "Camera #" + localCameraIndex + "failed to open: " + e.getLocalizedMessage());
                    }
                }
            }
        }
        if(mCamera == null) {
            Log.e(LOGTAG, "Error: can't open camera");
            return;
        }
        Camera.Parameters params = mCamera.getParameters();
        List<String> FocusModes = params.getSupportedFocusModes();
        if (FocusModes != null && FocusModes.contains(Camera.Parameters.FOCUS_MODE_CONTINUOUS_VIDEO))
        {
            params.setFocusMode(Camera.Parameters.FOCUS_MODE_CONTINUOUS_VIDEO);
        }
        mCamera.setParameters(params);

        try {
            mCamera.setPreviewTexture(mSTexture);
        } catch (IOException ioe) {
            Log.e(LOGTAG, "setPreviewTexture() failed: " + ioe.getMessage());
        }
    }

    @Override
    public synchronized void setCameraPreviewSize(int width, int height) {
        Log.i(LOGTAG, "setCameraPreviewSize: "+width+"x"+height);
        if(mCamera == null) {
            Log.e(LOGTAG, "Camera isn't initialized!");
            return;
        }

        if(mMaxCameraWidth  > 0 && mMaxCameraWidth  < width)  width  = mMaxCameraWidth;
        if(mMaxCameraHeight > 0 && mMaxCameraHeight < height) height = mMaxCameraHeight;

        Camera.Parameters param = mCamera.getParameters();
        List<Size> psize = param.getSupportedPreviewSizes();
        int bestWidth = 0, bestHeight = 0;
        if (psize.size() > 0) {
            float aspect = (float)width / height;
            for (Size size : psize) {
                int w = size.width, h = size.height;
                Log.d(LOGTAG, "checking camera preview size: "+w+"x"+h);
                if ( w <= width && h <= height &&
                     w >= bestWidth && h >= bestHeight &&
                     Math.abs(aspect - (float)w/h) < 0.2 ) {
                    bestWidth = w;
                    bestHeight = h;
                }
            }
            if(bestWidth <= 0 || bestHeight <= 0) {
                bestWidth  = psize.get(0).width;
                bestHeight = psize.get(0).height;
                Log.e(LOGTAG, "Error: best size was not selected, using "+bestWidth+" x "+bestHeight);
            } else {
                Log.i(LOGTAG, "Selected best size: "+bestWidth+" x "+bestHeight);
            }

            if(mPreviewStarted) {
                mCamera.stopPreview();
                mPreviewStarted = false;
            }
            mCameraWidth  = bestWidth;
            mCameraHeight = bestHeight;
            param.setPreviewSize(bestWidth, bestHeight);
        }
        param.set("orientation", "landscape");
        mCamera.setParameters(param);
        mCamera.startPreview();
        mPreviewStarted = true;
    }
}```

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

- **CameraRenderer**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `java.io.IOException`
- `android.os.Build`
- `android.hardware.Camera.Size`
- `android.util.Log`
- `android.annotation.TargetApi`
- `java.util.List`
- `android.hardware.Camera`


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

