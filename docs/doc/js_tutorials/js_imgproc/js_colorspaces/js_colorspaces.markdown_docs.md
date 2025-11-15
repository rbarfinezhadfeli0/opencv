# Documentation for `doc/js_tutorials/js_imgproc/js_colorspaces/js_colorspaces.markdown`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_imgproc/js_colorspaces/js_colorspaces.markdown`
- **File Name**: `js_colorspaces.markdown`
- **File Size**: 1,769 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/js_tutorials/js_imgproc/js_colorspaces/js_colorspaces.markdown](../../../../doc/js_tutorials/js_imgproc/js_colorspaces/js_colorspaces.markdown)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_imgproc/js_colorspaces` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Changing Colorspaces {#tutorial_js_colorspaces}
====================

Goal
----

-   In this tutorial, you will learn how to convert images from one color-space to another, like
    RGB \f$\leftrightarrow\f$ Gray, RGB \f$\leftrightarrow\f$ HSV etc.
-   You will learn following functions : **cv.cvtColor()**, **cv.inRange()** etc.

cvtColor
--------------------

There are more than 150 color-space conversion methods available in OpenCV. But we will look into
the most widely used one: RGB \f$\leftrightarrow\f$ Gray.

We use the function: **cv.cvtColor (src, dst, code, dstCn = 0)**
@param src    input image.
@param dst    output image of the same size and depth as src
@param code   color space conversion code(see **cv.ColorConversionCodes**).
@param dstCn  number of channels in the destination image; if the parameter is 0, the number of the channels is derived automatically from src and code.

For RGB \f$\rightarrow\f$ Gray conversion we use the code cv.COLOR_RGBA2GRAY.

Try it
------

\htmlonly
<iframe src="../../js_colorspaces_cvtColor.html" width="100%"
        onload="this.style.height=this.contentDocument.body.scrollHeight +'px';">
</iframe>
\endhtmlonly

inRange
---------------

Checks if array elements lie between the elements of two other arrays.

We use the function: **cv.inRange (src, lowerb, upperb, dst)**
@param src     first input image.
@param lowerb  inclusive lower boundary Mat of the same size as src.
@param upperb  inclusive upper boundary Mat of the same size as src.
@param dst     output image of the same size as src and cv.CV_8U type.

Try it
------

\htmlonly
<iframe src="../../js_colorspaces_inRange.html" width="100%"
        onload="this.style.height=this.contentDocument.body.scrollHeight +'px';">
</iframe>
\endhtmlonly
```

## General Information

This file is part of the OpenCV repository infrastructure.

