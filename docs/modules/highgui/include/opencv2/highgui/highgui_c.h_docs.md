# Documentation for `modules/highgui/include/opencv2/highgui/highgui_c.h`

## File Metadata

- **Full Path**: `modules/highgui/include/opencv2/highgui/highgui_c.h`
- **File Name**: `highgui_c.h`
- **File Size**: 10,492 bytes
- **File Type**: .h
- **Link to Source**: [modules/highgui/include/opencv2/highgui/highgui_c.h](../../../../../modules/highgui/include/opencv2/highgui/highgui_c.h)

## Purpose and Role

This file is located in the `modules/highgui/include/opencv2/highgui` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of Intel Corporation may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#ifndef OPENCV_HIGHGUI_H
#define OPENCV_HIGHGUI_H

#include "opencv2/core/core_c.h"
#include "opencv2/imgproc/imgproc_c.h"

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/** @addtogroup highgui_c
  @{
  */

/****************************************************************************************\
*                                  Basic GUI functions                                   *
\****************************************************************************************/
//YV
//-----------New for Qt
/* For font */
enum {  CV_FONT_LIGHT           = 25,//QFont::Light,
        CV_FONT_NORMAL          = 50,//QFont::Normal,
        CV_FONT_DEMIBOLD        = 63,//QFont::DemiBold,
        CV_FONT_BOLD            = 75,//QFont::Bold,
        CV_FONT_BLACK           = 87 //QFont::Black
};

enum {  CV_STYLE_NORMAL         = 0,//QFont::StyleNormal,
        CV_STYLE_ITALIC         = 1,//QFont::StyleItalic,
        CV_STYLE_OBLIQUE        = 2 //QFont::StyleOblique
};
/* ---------*/

//for color cvScalar(blue_component, green_component, red_component[, alpha_component])
//and alpha= 0 <-> 0xFF (not transparent <-> transparent)
CVAPI(CvFont) cvFontQt(const char* nameFont, int pointSize CV_DEFAULT(-1), CvScalar color CV_DEFAULT(cvScalarAll(0)), int weight CV_DEFAULT(CV_FONT_NORMAL),  int style CV_DEFAULT(CV_STYLE_NORMAL), int spacing CV_DEFAULT(0));

CVAPI(void) cvAddText(const CvArr* img, const char* text, CvPoint org, CvFont *arg2);

CVAPI(void) cvDisplayOverlay(const char* name, const char* text, int delayms CV_DEFAULT(0));
CVAPI(void) cvDisplayStatusBar(const char* name, const char* text, int delayms CV_DEFAULT(0));

CVAPI(void) cvSaveWindowParameters(const char* name);
CVAPI(void) cvLoadWindowParameters(const char* name);
CVAPI(int) cvStartLoop(int (*pt2Func)(int argc, char *argv[]), int argc, char* argv[]);
CVAPI(void) cvStopLoop( void );

typedef void (CV_CDECL *CvButtonCallback)(int state, void* userdata);
enum {CV_PUSH_BUTTON = 0, CV_CHECKBOX = 1, CV_RADIOBOX = 2};
CVAPI(int) cvCreateButton( const char* button_name CV_DEFAULT(NULL),CvButtonCallback on_change CV_DEFAULT(NULL), void* userdata CV_DEFAULT(NULL) , int button_type CV_DEFAULT(CV_PUSH_BUTTON), int initial_button_state CV_DEFAULT(0));
//----------------------


/* this function is used to set some external parameters in case of X Window */
CVAPI(int) cvInitSystem( int argc, char** argv );

CVAPI(int) cvStartWindowThread( void );

// ---------  YV ---------
enum
{
    //These 3 flags are used by cvSet/GetWindowProperty
    CV_WND_PROP_FULLSCREEN = 0, //to change/get window's fullscreen property
    CV_WND_PROP_AUTOSIZE   = 1, //to change/get window's autosize property
    CV_WND_PROP_ASPECTRATIO= 2, //to change/get window's aspectratio property
    CV_WND_PROP_OPENGL     = 3, //to change/get window's opengl support
    CV_WND_PROP_VISIBLE    = 4,

    //These 2 flags are used by cvNamedWindow and cvSet/GetWindowProperty
    CV_WINDOW_NORMAL       = 0x00000000, //the user can resize the window (no constraint)  / also use to switch a fullscreen window to a normal size
    CV_WINDOW_AUTOSIZE     = 0x00000001, //the user cannot resize the window, the size is constrainted by the image displayed
    CV_WINDOW_OPENGL       = 0x00001000, //window with opengl support

    //Those flags are only for Qt
    CV_GUI_EXPANDED         = 0x00000000, //status bar and tool bar
    CV_GUI_NORMAL           = 0x00000010, //old fashious way

    //These 3 flags are used by cvNamedWindow and cvSet/GetWindowProperty
    CV_WINDOW_FULLSCREEN   = 1,//change the window to fullscreen
    CV_WINDOW_FREERATIO    = 0x00000100,//the image expends as much as it can (no ratio constraint)
    CV_WINDOW_KEEPRATIO    = 0x00000000//the ration image is respected.
};

/* create window */
CVAPI(int) cvNamedWindow( const char* name, int flags CV_DEFAULT(CV_WINDOW_AUTOSIZE) );

/* Set and Get Property of the window */
CVAPI(void) cvSetWindowProperty(const char* name, int prop_id, double prop_value);
CVAPI(double) cvGetWindowProperty(const char* name, int prop_id);

/* display image within window (highgui windows remember their content) */
CVAPI(void) cvShowImage( const char* name, const CvArr* image );

/* resize/move window */
CVAPI(void) cvResizeWindow( const char* name, int width, int height );
CVAPI(void) cvMoveWindow( const char* name, int x, int y );


/* destroy window and all the trackers associated with it */
CVAPI(void) cvDestroyWindow( const char* name );

CVAPI(void) cvDestroyAllWindows(void);

/* get native window handle (HWND in case of Win32 and Widget in case of X Window) */
CVAPI(void*) cvGetWindowHandle( const char* name );

/* get name of highgui window given its native handle */
CVAPI(const char*) cvGetWindowName( void* window_handle );


typedef void (CV_CDECL *CvTrackbarCallback)(int pos);

/* create trackbar and display it on top of given window, set callback */
CVAPI(int) cvCreateTrackbar( const char* trackbar_name, const char* window_name,
                             int* value, int count, CvTrackbarCallback on_change CV_DEFAULT(NULL));

typedef void (CV_CDECL *CvTrackbarCallback2)(int pos, void* userdata);

CVAPI(int) cvCreateTrackbar2( const char* trackbar_name, const char* window_name,
                              int* value, int count, CvTrackbarCallback2 on_change,
                              void* userdata CV_DEFAULT(0));

/* retrieve or set trackbar position */
CVAPI(int) cvGetTrackbarPos( const char* trackbar_name, const char* window_name );
CVAPI(void) cvSetTrackbarPos( const char* trackbar_name, const char* window_name, int pos );
CVAPI(void) cvSetTrackbarMax(const char* trackbar_name, const char* window_name, int maxval);
CVAPI(void) cvSetTrackbarMin(const char* trackbar_name, const char* window_name, int minval);

enum
{
    CV_EVENT_MOUSEMOVE      =0,
    CV_EVENT_LBUTTONDOWN    =1,
    CV_EVENT_RBUTTONDOWN    =2,
    CV_EVENT_MBUTTONDOWN    =3,
    CV_EVENT_LBUTTONUP      =4,
    CV_EVENT_RBUTTONUP      =5,
    CV_EVENT_MBUTTONUP      =6,
    CV_EVENT_LBUTTONDBLCLK  =7,
    CV_EVENT_RBUTTONDBLCLK  =8,
    CV_EVENT_MBUTTONDBLCLK  =9,
    CV_EVENT_MOUSEWHEEL     =10,
    CV_EVENT_MOUSEHWHEEL    =11
};

enum
{
    CV_EVENT_FLAG_LBUTTON   =1,
    CV_EVENT_FLAG_RBUTTON   =2,
    CV_EVENT_FLAG_MBUTTON   =4,
    CV_EVENT_FLAG_CTRLKEY   =8,
    CV_EVENT_FLAG_SHIFTKEY  =16,
    CV_EVENT_FLAG_ALTKEY    =32
};


#define CV_GET_WHEEL_DELTA(flags) ((short)((flags >> 16) & 0xffff)) // upper 16 bits

typedef void (CV_CDECL *CvMouseCallback )(int event, int x, int y, int flags, void* param);

/* assign callback for mouse events */
CVAPI(void) cvSetMouseCallback( const char* window_name, CvMouseCallback on_mouse,
                                void* param CV_DEFAULT(NULL));

/* wait for key event infinitely (delay<=0) or for "delay" milliseconds */
CVAPI(int) cvWaitKey(int delay CV_DEFAULT(0));

// OpenGL support

typedef void (CV_CDECL *CvOpenGlDrawCallback)(void* userdata);
CVAPI(void) cvSetOpenGlDrawCallback(const char* window_name, CvOpenGlDrawCallback callback, void* userdata CV_DEFAULT(NULL));

CVAPI(void) cvSetOpenGlContext(const char* window_name);
CVAPI(void) cvUpdateWindow(const char* window_name);


/****************************************************************************************\

*                              Obsolete functions/synonyms                               *
\****************************************************************************************/

#define cvAddSearchPath(path)
#define cvvInitSystem cvInitSystem
#define cvvNamedWindow cvNamedWindow
#define cvvShowImage cvShowImage
#define cvvResizeWindow cvResizeWindow
#define cvvDestroyWindow cvDestroyWindow
#define cvvCreateTrackbar cvCreateTrackbar
#define cvvAddSearchPath cvAddSearchPath
#define cvvWaitKey(name) cvWaitKey(0)
#define cvvWaitKeyEx(name,delay) cvWaitKey(delay)
#define HG_AUTOSIZE CV_WINDOW_AUTOSIZE
#define set_preprocess_func cvSetPreprocessFuncWin32
#define set_postprocess_func cvSetPostprocessFuncWin32

#if defined _WIN32

CVAPI(void) cvSetPreprocessFuncWin32_(const void* callback);
CVAPI(void) cvSetPostprocessFuncWin32_(const void* callback);
#define cvSetPreprocessFuncWin32(callback) cvSetPreprocessFuncWin32_((const void*)(callback))
#define cvSetPostprocessFuncWin32(callback) cvSetPostprocessFuncWin32_((const void*)(callback))

#endif

/** @} highgui_c */

#ifdef __cplusplus
}
#endif

#endif
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **is()**: A function/method defined in this file
- **cvSetPreprocessFuncWin32()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **__cplusplus()**: A function/method defined in this file
- **cvSetPostprocessFuncWin32()**: A function/method defined in this file
- **OPENCV_HIGHGUI_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/imgproc/imgproc_c.h`
- `opencv2/core/core_c.h`

**Python Imports:**
- `this`


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

