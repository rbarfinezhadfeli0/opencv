# Documentation for `samples/android/tutorial-4-opencl/res/layout/activity.xml`

## File Metadata

- **Full Path**: `samples/android/tutorial-4-opencl/res/layout/activity.xml`
- **File Name**: `activity.xml`
- **File Size**: 960 bytes
- **File Type**: .xml
- **Link to Source**: [samples/android/tutorial-4-opencl/res/layout/activity.xml](../../../../../samples/android/tutorial-4-opencl/res/layout/activity.xml)

## Purpose and Role

This file is located in the `samples/android/tutorial-4-opencl/res/layout` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <org.opencv.samples.tutorial4.MyGLSurfaceView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/my_gl_surface_view" />

    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation = "vertical" >
	    <TextView
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/fps_text_view"
	        android:text="FPS:" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/proc_mode_text_view"
            android:text="Processing mode:" />
	    </LinearLayout>

</FrameLayout>

```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.

