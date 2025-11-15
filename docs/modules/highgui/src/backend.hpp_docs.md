# Documentation for `modules/highgui/src/backend.hpp`

## File Metadata

- **Full Path**: `modules/highgui/src/backend.hpp`
- **File Name**: `backend.hpp`
- **File Size**: 3,676 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/highgui/src/backend.hpp](../../../modules/highgui/src/backend.hpp)

## Purpose and Role

This file is located in the `modules/highgui/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
#ifndef OPENCV_HIGHGUI_BACKEND_HPP
#define OPENCV_HIGHGUI_BACKEND_HPP

#include <memory>
#include <map>

namespace cv { namespace highgui_backend {

class CV_EXPORTS UIWindowBase
{
public:
    typedef std::shared_ptr<UIWindowBase> Ptr;
    typedef std::weak_ptr<UIWindowBase> WeakPtr;

    virtual ~UIWindowBase();

    virtual const std::string& getID() const = 0;  // internal name, used for logging

    virtual bool isActive() const = 0;

    virtual void destroy() = 0;
};  // UIWindowBase

class UITrackbar;

class CV_EXPORTS UIWindow : public UIWindowBase
{
public:
    virtual ~UIWindow();

    virtual void imshow(InputArray image) = 0;

    virtual double getProperty(int prop) const = 0;
    virtual bool setProperty(int prop, double value) = 0;

    virtual void resize(int width, int height) = 0;
    virtual void move(int x, int y) = 0;

    virtual Rect getImageRect() const = 0;

    virtual void setTitle(const std::string& title) = 0;

    virtual void setMouseCallback(MouseCallback onMouse, void* userdata /*= 0*/) = 0;

    //TODO: handle both keys and mouse events (both with mouse coordinates)
    //virtual void setInputCallback(InputCallback onInputEvent, void* userdata /*= 0*/) = 0;

    virtual std::shared_ptr<UITrackbar> createTrackbar(
        const std::string& name,
        int count,
        TrackbarCallback onChange /*= 0*/,
        void* userdata /*= 0*/
    ) = 0;

    virtual std::shared_ptr<UITrackbar> findTrackbar(const std::string& name) = 0;

#if 0  // QT only
    virtual void displayOverlay(const std::string& text, int delayms = 0) = 0;
    virtual void displayStatusBar(const std::string& text, int delayms /*= 0*/) = 0;
    virtual int createButton(
        const std::string& bar_name, ButtonCallback on_change,
        void* userdata = 0, int type /*= QT_PUSH_BUTTON*/,
        bool initial_button_state /*= false*/
    ) = 0;
    // addText, QtFont stuff
#endif

#if 0  // OpenGL
    virtual void imshow(const ogl::Texture2D& tex) = 0;
    virtual void setOpenGlDrawCallback(OpenGlDrawCallback onOpenGlDraw, void* userdata = 0) = 0;
    virtual void setOpenGlContext() = 0;
    virtual void updateWindow() = 0;
#endif

};  // UIWindow


class CV_EXPORTS UITrackbar : public UIWindowBase
{
public:
    virtual ~UITrackbar();

    virtual int getPos() const = 0;
    virtual void setPos(int pos) = 0;

    virtual cv::Range getRange() const = 0;
    virtual void setRange(const cv::Range& range) = 0;
};  // UITrackbar


class CV_EXPORTS UIBackend
{
public:
    virtual ~UIBackend();

    virtual void destroyAllWindows() = 0;

    // namedWindow
    virtual std::shared_ptr<UIWindow> createWindow(
        const std::string& winname,
        int flags
    ) = 0;

    virtual int waitKeyEx(int delay /*= 0*/) = 0;
    virtual int pollKey() = 0;
    virtual const std::string getName() const = 0;
};

std::shared_ptr<UIBackend>& getCurrentUIBackend();
void setUIBackend(const std::shared_ptr<UIBackend>& api);
bool setUIBackend(const std::string& backendName);

#ifndef BUILD_PLUGIN

#ifdef HAVE_WIN32UI
std::shared_ptr<UIBackend> createUIBackendWin32UI();
#endif

#ifdef HAVE_GTK
std::shared_ptr<UIBackend> createUIBackendGTK();
#endif

#if 0  // TODO: defined HAVE_QT
std::shared_ptr<UIBackend> createUIBackendQT();
#endif

#ifdef HAVE_FRAMEBUFFER
std::shared_ptr<UIBackend> createUIBackendFramebuffer();
#endif

#endif  // BUILD_PLUGIN

}  // namespace highgui_backend

}  // namespace cv

#endif // OPENCV_HIGHGUI_BACKEND_HPP
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

### Classes and Structures

- **CV_EXPORTS**: A class/struct defined in this file
- **UITrackbar**: A class/struct defined in this file

### Functions and Methods

- **BUILD_PLUGIN()**: A function/method defined in this file
- **std()**: A function/method defined in this file
- **HAVE_FRAMEBUFFER()**: A function/method defined in this file
- **OPENCV_HIGHGUI_BACKEND_HPP()**: A function/method defined in this file
- **HAVE_GTK()**: A function/method defined in this file
- **HAVE_WIN32UI()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `memory`
- `map`


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

