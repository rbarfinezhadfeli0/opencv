# Documentation for `modules/highgui/src/window_QT.h`

## File Metadata

- **Full Path**: `modules/highgui/src/window_QT.h`
- **File Name**: `window_QT.h`
- **File Size**: 17,257 bytes
- **File Type**: .h
- **Link to Source**: [modules/highgui/src/window_QT.h](../../../modules/highgui/src/window_QT.h)

## Purpose and Role

This file is located in the `modules/highgui/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
//IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.

// By downloading, copying, installing or using the software you agree to this license.
// If you do not agree to this license, do not download, install,
// copy or use the software.


//                          License Agreement
//               For Open Source Computer Vision Library

//Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
//Copyright (C) 2008-2010, Willow Garage Inc., all rights reserved.
//Third party copyrights are property of their respective owners.

//Redistribution and use in source and binary forms, with or without modification,
//are permitted provided that the following conditions are met:

//  * Redistribution's of source code must retain the above copyright notice,
//  this list of conditions and the following disclaimer.

//  * Redistribution's in binary form must reproduce the above copyright notice,
//  this list of conditions and the following disclaimer in the documentation
//  and/or other materials provided with the distribution.

//  * The name of the copyright holders may not be used to endorse or promote products
//  derived from this software without specific prior written permission.

//This software is provided by the copyright holders and contributors "as is" and
//any express or implied warranties, including, but not limited to, the implied
//warranties of merchantability and fitness for a particular purpose are disclaimed.
//In no event shall the Intel Corporation or contributors be liable for any direct,
//indirect, incidental, special, exemplary, or consequential damages
//(including, but not limited to, procurement of substitute goods or services;
//loss of use, data, or profits; or business interruption) however caused
//and on any theory of liability, whether in contract, strict liability,
//or tort (including negligence or otherwise) arising in any way out of
//the use of this software, even if advised of the possibility of such damage.

//--------------------Google Code 2010 -- Yannick Verdie--------------------//
#ifndef __OPENCV_HIGHGUI_QT_H__
#define __OPENCV_HIGHGUI_QT_H__

#include "precomp.hpp"

#ifndef _DEBUG
#define QT_NO_DEBUG_OUTPUT
#endif

#if defined( HAVE_QT_OPENGL )
#include <QtOpenGL>

  // QGLWidget deprecated and no longer functions with Qt6, use QOpenGLWidget instead
  #ifdef HAVE_QT6
  #include <QOpenGLWidget>
  #else
  #include <QGLWidget>
  #endif

#endif

#include <QAbstractEventDispatcher>
#include <QApplication>
#include <QFile>
#include <QPushButton>
#include <QGraphicsView>
#include <QSizePolicy>
#include <QInputDialog>
#include <QBoxLayout>
#include <QSettings>
#include <qtimer.h>
#include <QtConcurrentRun>
#include <QWaitCondition>
#include <QKeyEvent>
#include <QMetaObject>
#include <QPointer>
#include <QSlider>
#include <QLabel>
#include <QIODevice>
#include <QShortcut>
#include <QStatusBar>
#include <QVarLengthArray>
#include <QFileInfo>
#include <QDate>
#include <QFileDialog>
#include <QToolBar>
#include <QClipboard>

#include <QAction>
#include <QCheckBox>
#include <QRadioButton>
#include <QButtonGroup>
#include <QMenu>
#include <QTest>

//start private enum
enum { CV_MODE_NORMAL = 0, CV_MODE_OPENGL = 1 };

//we can change the keyboard shortcuts from here !
enum {	shortcut_zoom_normal 	= Qt::CTRL + Qt::Key_Z,
        shortcut_zoom_imgRegion = Qt::CTRL + Qt::Key_X,
        shortcut_save_img		= Qt::CTRL + Qt::Key_S,
        shortcut_copy_clipbrd   = Qt::CTRL + Qt::Key_C,
        shortcut_properties_win	= Qt::CTRL + Qt::Key_P,
        shortcut_zoom_in 		= Qt::CTRL + Qt::Key_Plus,//QKeySequence(QKeySequence::ZoomIn),
        shortcut_zoom_out		= Qt::CTRL + Qt::Key_Minus,//QKeySequence(QKeySequence::ZoomOut),
        shortcut_panning_left 	= Qt::CTRL + Qt::Key_Left,
        shortcut_panning_right 	= Qt::CTRL + Qt::Key_Right,
        shortcut_panning_up 	= Qt::CTRL + Qt::Key_Up,
        shortcut_panning_down 	= Qt::CTRL + Qt::Key_Down
    };
//end enum

class CvWindow;
class ViewPort;


class GuiReceiver : public QObject
{
    Q_OBJECT

public:
    GuiReceiver();
    ~GuiReceiver();

    int start();
    void isLastWindow();

    bool bTimeOut;
    QTimer* timer;

public slots:
    void createWindow( QString name, int flags = 0 );
    void destroyWindow(QString name);
    void destroyAllWindow();
    void addSlider(QString trackbar_name, QString window_name, void* value, int count, void* on_change);
    void addSlider2(QString trackbar_name, QString window_name, void* value, int count, void* on_change, void *userdata);
    void moveWindow(QString name, int x, int y);
    void resizeWindow(QString name, int width, int height);
    void showImage(QString name, void* arr);
    void displayInfo( QString name, QString text, int delayms );
    void displayStatusBar( QString name, QString text, int delayms );
    void timeOut();
    void toggleFullScreen(QString name, double flags );
    CvRect getWindowRect(QString name);
    double isFullScreen(QString name);
    double getPropWindow(QString name);
    void setPropWindow(QString name, double flags );
    void setWindowTitle(QString name, QString title);
    double getWindowVisible(QString name);
    double getRatioWindow(QString name);
    void setRatioWindow(QString name, double arg2 );
    void saveWindowParameters(QString name);
    void loadWindowParameters(QString name);
    void putText(void* arg1, QString text, QPoint org, void* font);
    void addButton(QString button_name, int button_type, int initial_button_state , void* on_change, void* userdata);
    void enablePropertiesButtonEachWindow();

    void setOpenGlDrawCallback(QString name, void* callback, void* userdata);
    void setOpenGlContext(QString name);
    void updateWindow(QString name);
    double isOpenGl(QString name);

private:
    int nb_windows;
    bool doesExternalQAppExist;
};


enum typeBar { type_CvTrackbar = 0, type_CvButtonbar = 1 };
class CvBar : public QHBoxLayout
{
public:
    typeBar type;
    QString name_bar;
    QPointer<QWidget> myparent;
};


class CvButtonbar : public CvBar
{
    Q_OBJECT
public:
    CvButtonbar(QWidget* arg, QString bar_name);

    void addButton(QString button_name, CvButtonCallback call, void* userdata,  int button_type, int initial_button_state);

private:
    void setLabel();

    QPointer<QLabel> label;
    QPointer<QButtonGroup> group_button;
};


class CvPushButton : public QPushButton
{
    Q_OBJECT
public:
    CvPushButton(CvButtonbar* par, QString button_name, CvButtonCallback call, void* userdata);

private:
    CvButtonbar* myparent;
    QString button_name ;
    CvButtonCallback callback;
    void* userdata;

private slots:
    void callCallBack(bool);
};


class CvCheckBox : public QCheckBox
{
    Q_OBJECT
public:
    CvCheckBox(CvButtonbar* par, QString button_name, CvButtonCallback call, void* userdata, int initial_button_state);

private:
    CvButtonbar* myparent;
    QString button_name ;
    CvButtonCallback callback;
    void* userdata;

private slots:
    void callCallBack(bool);
};


class CvRadioButton : public QRadioButton
{
    Q_OBJECT
public:
    CvRadioButton(CvButtonbar* par, QString button_name, CvButtonCallback call, void* userdata, int initial_button_state);

private:
    CvButtonbar* myparent;
    QString button_name ;
    CvButtonCallback callback;
    void* userdata;

private slots:
    void callCallBack(bool);
};


class CvTrackbar :  public CvBar
{
    Q_OBJECT
public:
    CvTrackbar(CvWindow* parent, QString name, int* value, int count, CvTrackbarCallback on_change);
    CvTrackbar(CvWindow* parent, QString name, int* value, int count, CvTrackbarCallback2 on_change, void* data);

    QPointer<QSlider> slider;

private slots:
    void createDialog();
    void update(int myvalue);

private:
    void setLabel(int myvalue);
    void create(CvWindow* arg, QString name, int* value, int count);
    QString createLabel();
    QPointer<QPushButton > label;
    CvTrackbarCallback callback;
    CvTrackbarCallback2 callback2;//look like it is use by python binding
    int* dataSlider;  // deprecated
    void* userdata;
};

//Both are top level window, so that a way to differentiate them.
//if (obj->metaObject ()->className () == "CvWindow") does not give me robust result

enum typeWindow { type_CvWindow = 1, type_CvWinProperties = 2 };
class CvWinModel : public QWidget
{
public:
typeWindow type;
};


class CvWinProperties : public CvWinModel
{
    Q_OBJECT
public:
    CvWinProperties(QString name, QObject* parent);
    ~CvWinProperties();
    QPointer<QBoxLayout> myLayout;

private:
    void closeEvent ( QCloseEvent * e ) CV_OVERRIDE;
    void showEvent ( QShowEvent * event ) CV_OVERRIDE;
    void hideEvent ( QHideEvent * event ) CV_OVERRIDE;
};


class CvWindow : public CvWinModel
{
    Q_OBJECT
public:
    CvWindow(QString arg2, int flag = CV_WINDOW_NORMAL);

    void setMouseCallBack(CvMouseCallback m, void* param);

    void writeSettings();
    void readSettings();

    double getRatio();
    void setRatio(int flags);

    CvRect getWindowRect();
    int getPropWindow();
    void setPropWindow(int flags);

    void toggleFullScreen(int flags);

    void updateImage(void* arr);

    void displayInfo(QString text, int delayms);
    void displayStatusBar(QString text, int delayms);

    void enablePropertiesButton();

    static CvButtonbar* createButtonBar(QString bar_name);

    static void addSlider(CvWindow* w, QString name, int* value, int count, CvTrackbarCallback on_change CV_DEFAULT(NULL));
    static void addSlider2(CvWindow* w, QString name, int* value, int count, CvTrackbarCallback2 on_change CV_DEFAULT(NULL), void* userdata CV_DEFAULT(0));

    void setOpenGlDrawCallback(CvOpenGlDrawCallback callback, void* userdata);
    void makeCurrentOpenGlContext();
    void updateGl();
    bool isOpenGl();

    void setViewportSize(QSize size);

    //parameters (will be save/load)
    int param_flags;
    int param_gui_mode;
    int param_ratio_mode;

    QPointer<QBoxLayout> myGlobalLayout; //All the widget (toolbar, view, LayoutBar, ...) are attached to it
    QPointer<QBoxLayout> myBarLayout;

    QVector<QAction*> vect_QActions;

    QPointer<QStatusBar> myStatusBar;
    QPointer<QToolBar> myToolBar;
    QPointer<QLabel> myStatusBar_msg;

protected:
    virtual void keyPressEvent(QKeyEvent* event) CV_OVERRIDE;
    virtual void closeEvent(QCloseEvent* event) CV_OVERRIDE;

private:

    int mode_display; //opengl or native
    ViewPort* myView;

    QVector<QShortcut*> vect_QShortcuts;

    void icvLoadTrackbars(QSettings *settings);
    void icvSaveTrackbars(QSettings *settings);
    void icvLoadControlPanel();
    void icvSaveControlPanel();
    void icvLoadButtonbar(CvButtonbar* t,QSettings *settings);
    void icvSaveButtonbar(CvButtonbar* t,QSettings *settings);

    void createActions();
    void createShortcuts();
    void createToolBar();
    void createView();
    void createStatusBar();
    void createGlobalLayout();
    void createBarLayout();
    CvWinProperties* createParameterWindow();

    void hideTools();
    void showTools();
    QSize getAvailableSize();

private slots:
    void displayPropertiesWin();
};


enum type_mouse_event { mouse_up = 0, mouse_down = 1, mouse_dbclick = 2, mouse_move = 3, mouse_wheel = 4 };
static const int tableMouseButtons[][3]={
    {CV_EVENT_LBUTTONUP, CV_EVENT_RBUTTONUP, CV_EVENT_MBUTTONUP},               //mouse_up
    {CV_EVENT_LBUTTONDOWN, CV_EVENT_RBUTTONDOWN, CV_EVENT_MBUTTONDOWN},         //mouse_down
    {CV_EVENT_LBUTTONDBLCLK, CV_EVENT_RBUTTONDBLCLK, CV_EVENT_MBUTTONDBLCLK},   //mouse_dbclick
    {CV_EVENT_MOUSEMOVE, CV_EVENT_MOUSEMOVE, CV_EVENT_MOUSEMOVE},               //mouse_move
    {0, 0, 0}                                                                   //mouse_wheel, to prevent exceptions in code
};


class ViewPort
{
public:
    virtual ~ViewPort() {}

    virtual QWidget* getWidget() = 0;

    virtual void setMouseCallBack(CvMouseCallback callback, void* param) = 0;

    virtual void writeSettings(QSettings& settings) = 0;
    virtual void readSettings(QSettings& settings) = 0;

    virtual double getRatio() = 0;
    virtual void setRatio(int flags) = 0;

    virtual void updateImage(const CvArr* arr) = 0;

    virtual void startDisplayInfo(QString text, int delayms) = 0;

    virtual void setOpenGlDrawCallback(CvOpenGlDrawCallback callback, void* userdata) = 0;
    virtual void makeCurrentOpenGlContext() = 0;
    virtual void updateGl() = 0;

    virtual void setSize(QSize size_) = 0;
};


class OCVViewPort : public ViewPort
{
public:
    explicit OCVViewPort();
    ~OCVViewPort() CV_OVERRIDE {};
    void setMouseCallBack(CvMouseCallback callback, void* param) CV_OVERRIDE;

protected:
    void icvmouseEvent(QMouseEvent* event, type_mouse_event category);
    void icvmouseHandler(QMouseEvent* event, type_mouse_event category, int& cv_event, int& flags);
    virtual void icvmouseProcessing(QPointF pt, int cv_event, int flags);

    CvMouseCallback mouseCallback;
    void* mouseData;
};


#ifdef HAVE_QT_OPENGL

// Use QOpenGLWidget for Qt6 (QGLWidget is deprecated)
#ifdef HAVE_QT6
typedef QOpenGLWidget OpenCVQtWidgetBase;
#else
typedef QGLWidget OpenCVQtWidgetBase;
#endif

class OpenGlViewPort : public OpenCVQtWidgetBase, public OCVViewPort
{
public:
    explicit OpenGlViewPort(QWidget* parent);
    ~OpenGlViewPort() CV_OVERRIDE;

    QWidget* getWidget() CV_OVERRIDE;

    void writeSettings(QSettings& settings) CV_OVERRIDE;
    void readSettings(QSettings& settings) CV_OVERRIDE;

    double getRatio() CV_OVERRIDE;
    void setRatio(int flags) CV_OVERRIDE;

    void updateImage(const CvArr* arr) CV_OVERRIDE;

    void startDisplayInfo(QString text, int delayms) CV_OVERRIDE;

    void setOpenGlDrawCallback(CvOpenGlDrawCallback callback, void* userdata) CV_OVERRIDE;
    void makeCurrentOpenGlContext() CV_OVERRIDE;
    void updateGl() CV_OVERRIDE;

    void setSize(QSize size_) CV_OVERRIDE;

protected:
    void initializeGL() CV_OVERRIDE;
    void resizeGL(int w, int h) CV_OVERRIDE;
    void paintGL() CV_OVERRIDE;

    void wheelEvent(QWheelEvent* event) CV_OVERRIDE;
    void mouseMoveEvent(QMouseEvent* event) CV_OVERRIDE;
    void mousePressEvent(QMouseEvent* event) CV_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent* event) CV_OVERRIDE;
    void mouseDoubleClickEvent(QMouseEvent* event) CV_OVERRIDE;

    QSize sizeHint() const CV_OVERRIDE;

private:
    QSize size;

    CvOpenGlDrawCallback glDrawCallback;
    void* glDrawData;
};

#endif // HAVE_QT_OPENGL


class DefaultViewPort : public QGraphicsView, public OCVViewPort
{
    Q_OBJECT

public:
    DefaultViewPort(CvWindow* centralWidget, int arg2);
    ~DefaultViewPort() CV_OVERRIDE;

    QWidget* getWidget() CV_OVERRIDE;

    void writeSettings(QSettings& settings) CV_OVERRIDE;
    void readSettings(QSettings& settings) CV_OVERRIDE;

    double getRatio() CV_OVERRIDE;
    void setRatio(int flags) CV_OVERRIDE;

    void updateImage(const CvArr* arr) CV_OVERRIDE;

    void startDisplayInfo(QString text, int delayms) CV_OVERRIDE;

    void setOpenGlDrawCallback(CvOpenGlDrawCallback callback, void* userdata) CV_OVERRIDE;
    void makeCurrentOpenGlContext() CV_OVERRIDE;
    void updateGl() CV_OVERRIDE;

    void setSize(QSize size_) CV_OVERRIDE;

public slots:
    //reference:
    //http://www.qtcentre.org/wiki/index.php?title=QGraphicsView:_Smooth_Panning_and_Zooming
    //http://doc.qt.nokia.com/4.6/gestures-imagegestures-imagewidget-cpp.html

    void siftWindowOnLeft();
    void siftWindowOnRight();
    void siftWindowOnUp() ;
    void siftWindowOnDown();

    void resetZoom();
    void imgRegion();
    void ZoomIn();
    void ZoomOut();

    void saveView();
    void copy2Clipbrd();

protected:
    void contextMenuEvent(QContextMenuEvent* event) CV_OVERRIDE;
    void resizeEvent(QResizeEvent* event) CV_OVERRIDE;
    void paintEvent(QPaintEvent* paintEventInfo) CV_OVERRIDE;

    void wheelEvent(QWheelEvent* event) CV_OVERRIDE;
    void mouseMoveEvent(QMouseEvent* event) CV_OVERRIDE;
    void mousePressEvent(QMouseEvent* event) CV_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent* event) CV_OVERRIDE;
    void mouseDoubleClickEvent(QMouseEvent* event) CV_OVERRIDE;

private:
    int param_keepRatio;

    //parameters (will be save/load)
    QTransform param_matrixWorld;

    CvMat* image2Draw_mat;
    QImage image2Draw_qt;
    int nbChannelOriginImage;


    void scaleView(qreal scaleFactor, QPointF center);
    void moveView(QPointF delta);

    QPoint  mouseCoordinate;
    QPointF positionGrabbing;
    QRect   positionCorners;
    QTransform matrixWorld_inv;
    float ratioX, ratioY;

    bool isSameSize(IplImage* img1,IplImage* img2);

    QSize sizeHint() const CV_OVERRIDE;
    QPointer<CvWindow> centralWidget;
    QPointer<QTimer> timerDisplay;
    bool drawInfo;
    QString infoText;
    QRectF target;

    void drawInstructions(QPainter *painter);
    void drawViewOverview(QPainter *painter);
    void drawImgRegion(QPainter *painter);
    void draw2D(QPainter *painter);
    void drawStatusBar();
    void controlImagePosition();

    void icvmouseProcessing(QPointF pt, int cv_event, int flags) CV_OVERRIDE;

private slots:
    void stopDisplayInfo();
};

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

### Classes and Structures

- **CvWindow**: A class/struct defined in this file
- **CvCheckBox**: A class/struct defined in this file
- **ViewPort**: A class/struct defined in this file
- **GuiReceiver**: A class/struct defined in this file
- **DefaultViewPort**: A class/struct defined in this file
- **OpenGlViewPort**: A class/struct defined in this file
- **OCVViewPort**: A class/struct defined in this file
- **CvBar**: A class/struct defined in this file
- **CvRadioButton**: A class/struct defined in this file
- **CvWinModel**: A class/struct defined in this file
- **CvTrackbar**: A class/struct defined in this file
- **CvButtonbar**: A class/struct defined in this file
- **CvPushButton**: A class/struct defined in this file
- **CvWinProperties**: A class/struct defined in this file

### Functions and Methods

- **__OPENCV_HIGHGUI_QT_H__()**: A function/method defined in this file
- **QOpenGLWidget()**: A function/method defined in this file
- **_DEBUG()**: A function/method defined in this file
- **HAVE_QT_OPENGL()**: A function/method defined in this file
- **QGLWidget()**: A function/method defined in this file
- **HAVE_QT6()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `QBoxLayout`
- `QToolBar`
- `QGLWidget`
- `QtOpenGL`
- `QGraphicsView`
- `QIODevice`
- `QDate`
- `QSizePolicy`
- `QTest`
- `QSlider`
- `qtimer.h`
- `QFile`
- `QKeyEvent`
- `precomp.hpp`
- `QApplication`
- `QAction`
- `QShortcut`
- `QClipboard`
- `QCheckBox`
- `QSettings`
- `QButtonGroup`
- `QWaitCondition`
- `QFileInfo`
- `QMenu`
- `QAbstractEventDispatcher`
- `QPointer`
- `QStatusBar`
- `QFileDialog`
- `QRadioButton`
- `QOpenGLWidget`
- `QPushButton`
- `QMetaObject`
- `QInputDialog`
- `QtConcurrentRun`
- `QLabel`
- `QVarLengthArray`

**Python Imports:**
- `here`
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

