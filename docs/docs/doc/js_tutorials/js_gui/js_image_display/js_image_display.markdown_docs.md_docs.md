# Documentation for `docs/doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown_docs.md`
- **File Name**: `js_image_display.markdown_docs.md`
- **File Size**: 3,737 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown_docs.md](../../../../../docs/doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/js_tutorials/js_gui/js_image_display` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown`
- **File Name**: `js_image_display.markdown`
- **File Size**: 2,999 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown](../../../../doc/js_tutorials/js_gui/js_image_display/js_image_display.markdown)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_gui/js_image_display` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Getting Started with Images {#tutorial_js_image_display}
===========================

Goals
-----

-   Learn how to read an image and how to display it in a web.

Read an image
-------------

OpenCV.js saves images as cv.Mat type. We use HTML canvas element to transfer cv.Mat to the web
or in reverse. The ImageData interface can represent or set the underlying pixel data of an area of a
canvas element.

@note Please refer to canvas docs for more details.

First, create an ImageData obj from canvas:
@code{.js}
let canvas = document.getElementById(canvasInputId);
let ctx = canvas.getContext('2d');
let imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
@endcode

Then, use cv.matFromImageData to construct a cv.Mat:
@code{.js}
let src = cv.matFromImageData(imgData);
@endcode

@note Because canvas only support 8-bit RGBA image with continuous storage, the cv.Mat type is cv.CV_8UC4.
It is different from native OpenCV because images returned and shown by the native **imread** and
**imshow** have the channels stored in BGR order.

Display an image
----------------

First, convert the type of src to cv.CV_8UC4:
@code{.js}
let dst = new cv.Mat();
// scale and shift are used to map the data to [0, 255].
src.convertTo(dst, cv.CV_8U, scale, shift);
// *** is GRAY, RGB, or RGBA, according to src.channels() is 1, 3 or 4.
cv.cvtColor(dst, dst, cv.COLOR_***2RGBA);
@endcode

Then, new an ImageData obj from dst:
@code{.js}
let imgData = new ImageData(new Uint8ClampedArray(dst.data), dst.cols, dst.rows);
@endcode

Finally, display it:
@code{.js}
let canvas = document.getElementById(canvasOutputId);
let ctx = canvas.getContext('2d');
ctx.clearRect(0, 0, canvas.width, canvas.height);
canvas.width = imgData.width;
canvas.height = imgData.height;
ctx.putImageData(imgData, 0, 0);
@endcode

In OpenCV.js
------------

OpenCV.js implements image reading and showing using the above method.

We use **cv.imread (imageSource)** to read an image from html canvas or img element.
@param imageSource   canvas element or id, or img element or id.
@return              mat with channels stored in RGBA order.

We use **cv.imshow (canvasSource, mat)** to display it. The function may scale the mat,
depending on its depth:
- If the mat is 8-bit unsigned, it is displayed as is.
- If the mat is 16-bit unsigned or 32-bit integer, the pixels are divided by 256. That
is, the value range [0,255*256] is mapped to [0,255].
- If the mat is 32-bit floating-point, the pixel values are multiplied by 255. That is,
the value range [0,1] is mapped to [0,255].

@param canvasSource  canvas element or id.
@param mat           mat to be shown.

The above code of image reading and showing could be simplified as below.
@code{.js}
let img = cv.imread(imageSource);
cv.imshow(canvasOutput, img);
img.delete();
@endcode

Try it
------

\htmlonly
<iframe src="../../js_image_display.html" width="100%"
        onload="this.style.height=this.contentDocument.body.scrollHeight +'px';">
</iframe>
\endhtmlonly

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

