# Documentation for `modules/java/generator/android/java/org/opencv/android/FpsMeter.java`

## File Metadata

- **Full Path**: `modules/java/generator/android/java/org/opencv/android/FpsMeter.java`
- **File Name**: `FpsMeter.java`
- **File Size**: 2,154 bytes
- **File Type**: .java
- **Link to Source**: [modules/java/generator/android/java/org/opencv/android/FpsMeter.java](../../../../../../../../modules/java/generator/android/java/org/opencv/android/FpsMeter.java)

## Purpose and Role

This file is located in the `modules/java/generator/android/java/org/opencv/android` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
package org.opencv.android;

import java.text.DecimalFormat;

import org.opencv.core.Core;

import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.util.Log;

public class FpsMeter {
    private static final String TAG               = "FpsMeter";
    private static final int    STEP              = 20;
    private static final DecimalFormat FPS_FORMAT = new DecimalFormat("0.00");

    private int                 mFramesCounter;
    private double              mFrequency;
    private long                mprevFrameTime;
    private String              mStrfps;
    Paint                       mPaint;
    boolean                     mIsInitialized = false;
    int                         mWidth = 0;
    int                         mHeight = 0;

    public void init() {
        mFramesCounter = 0;
        mFrequency = Core.getTickFrequency();
        mprevFrameTime = Core.getTickCount();
        mStrfps = "";

        mPaint = new Paint();
        mPaint.setColor(Color.BLUE);
        mPaint.setTextSize(20);
    }

    public void measure() {
        if (!mIsInitialized) {
            init();
            mIsInitialized = true;
        } else {
            mFramesCounter++;
            if (mFramesCounter % STEP == 0) {
                long time = Core.getTickCount();
                double fps = STEP * mFrequency / (time - mprevFrameTime);
                mprevFrameTime = time;
                if (mWidth != 0 && mHeight != 0)
                    mStrfps = FPS_FORMAT.format(fps) + " FPS@" + Integer.valueOf(mWidth) + "x" + Integer.valueOf(mHeight);
                else
                    mStrfps = FPS_FORMAT.format(fps) + " FPS";
                Log.i(TAG, mStrfps);
            }
        }
    }

    public void setResolution(int width, int height) {
        Log.d(TAG, "FpsMeter.setResolution " + Integer.valueOf(mWidth) + "x" + Integer.valueOf(mHeight));
        mWidth = width;
        mHeight = height;
    }

    public void draw(Canvas canvas, float offsetx, float offsety) {
        Log.d(TAG, mStrfps);
        canvas.drawText(mStrfps, offsetx, offsety, mPaint);
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

- **FpsMeter**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `android.util.Log`
- `java.text.DecimalFormat`
- `android.graphics.Canvas`
- `android.graphics.Paint`
- `android.graphics.Color`
- `org.opencv.core.Core`


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

