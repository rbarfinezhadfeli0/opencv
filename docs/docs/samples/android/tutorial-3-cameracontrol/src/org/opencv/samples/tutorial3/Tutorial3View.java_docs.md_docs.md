# Documentation for `docs/samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java_docs.md`

## File Metadata

- **Full Path**: `docs/samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java_docs.md`
- **File Name**: `Tutorial3View.java_docs.md`
- **File Size**: 8,329 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java_docs.md](../../../../../../../../../docs/samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java_docs.md)

## Purpose and Role

This file is located in the `docs/samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java`

## File Metadata

- **Full Path**: `samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java`
- **File Name**: `Tutorial3View.java`
- **File Size**: 4,486 bytes
- **File Type**: .java
- **Link to Source**: [samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java](../../../../../../../../samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3/Tutorial3View.java)

## Purpose and Role

This file is located in the `samples/android/tutorial-3-cameracontrol/src/org/opencv/samples/tutorial3` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.samples.tutorial3;

import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.List;
import java.util.Objects;

import org.opencv.android.JavaCameraView;

import android.content.ContentResolver;
import android.content.ContentValues;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.hardware.Camera;
import android.hardware.Camera.PictureCallback;
import android.hardware.Camera.Size;
import android.net.Uri;
import android.os.Build;
import android.os.Environment;
import android.provider.MediaStore;
import android.util.AttributeSet;
import android.util.Log;

public class Tutorial3View extends JavaCameraView implements PictureCallback {

    private static final String TAG = "Sample::Tutorial3View";
    private String mPictureFileName;

    public Tutorial3View(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public List<String> getEffectList() {
        return mCamera.getParameters().getSupportedColorEffects();
    }

    public boolean isEffectSupported() {
        return (mCamera.getParameters().getColorEffect() != null);
    }

    public String getEffect() {
        return mCamera.getParameters().getColorEffect();
    }

    public void setEffect(String effect) {
        Camera.Parameters params = mCamera.getParameters();
        params.setColorEffect(effect);
        mCamera.setParameters(params);
    }

    public List<Size> getResolutionList() {
        return mCamera.getParameters().getSupportedPreviewSizes();
    }

    public void setResolution(Size resolution) {
        disconnectCamera();
        mMaxHeight = resolution.height;
        mMaxWidth = resolution.width;
        connectCamera(getWidth(), getHeight());
    }

    public Size getResolution() {
        return mCamera.getParameters().getPreviewSize();
    }

    public void takePicture(final String fileName) {
        Log.i(TAG, "Taking picture");
        this.mPictureFileName = fileName;
        // Postview and jpeg are sent in the same buffers if the queue is not empty when performing a capture.
        // Clear up buffers to avoid mCamera.takePicture to be stuck because of a memory issue
        mCamera.setPreviewCallback(null);

        // PictureCallback is implemented by the current class
        mCamera.takePicture(null, null, this);
    }

    @Override
    public void onPictureTaken(byte[] data, Camera camera) {
        Log.i(TAG, "Saving a bitmap to file");
        // The camera preview was automatically stopped. Start it again.
        mCamera.startPreview();
        mCamera.setPreviewCallback(this);

        // Write the image in a file (in jpeg format)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            Bitmap bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);
            new Thread(new Runnable() {
                @Override
                public void run() {
                    ContentResolver resolver = getContext().getContentResolver();
                    ContentValues contentValues = new ContentValues();
                    contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, mPictureFileName);
                    contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "image/jpg");
                    contentValues.put(MediaStore.MediaColumns.RELATIVE_PATH, Environment.DIRECTORY_PICTURES);
                    Uri imageUri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues);
                    try {
                        OutputStream fos = resolver.openOutputStream(Objects.requireNonNull(imageUri));
                        bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
                        Objects.requireNonNull(fos).close();
                    } catch (java.io.IOException e) {
                        Log.e("PictureDemo", "Exception in photoCallback", e);
                    }
                }
            }).start();
        } else {
            mPictureFileName = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES).getPath()
                               + "/" + mPictureFileName;
            try {
                FileOutputStream fos = new FileOutputStream(mPictureFileName);
                fos.write(data);
                fos.close();
            } catch (java.io.IOException e) {
                Log.e("PictureDemo", "Exception in photoCallback", e);
            }
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

- **mCamera**: A class/struct defined in this file
- **Tutorial3View**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `android.util.AttributeSet`
- `android.content.Context`
- `android.hardware.Camera`
- `android.content.ContentValues`
- `java.io.FileOutputStream`
- `android.net.Uri`
- `java.util.Objects`
- `android.content.ContentResolver`
- `java.util.List`
- `android.os.Environment`
- `android.hardware.Camera.PictureCallback`
- `android.graphics.BitmapFactory`
- `android.os.Build`
- `android.provider.MediaStore`
- `android.util.Log`
- `android.graphics.Bitmap`
- `java.io.OutputStream`
- `android.hardware.Camera.Size`
- `org.opencv.android.JavaCameraView`


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

