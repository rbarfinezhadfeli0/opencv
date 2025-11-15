# Documentation for `docs/doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown_docs.md`
- **File Name**: `js_houghcircles.markdown_docs.md`
- **File Size**: 3,117 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown_docs.md](../../../../../docs/doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/js_tutorials/js_imgproc/js_houghcircles` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown`
- **File Name**: `js_houghcircles.markdown`
- **File Size**: 2,369 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown](../../../../doc/js_tutorials/js_imgproc/js_houghcircles/js_houghcircles.markdown)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_imgproc/js_houghcircles` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Hough Circle Transform {#tutorial_js_houghcircles}
======================

Goal
----

-   We will learn to use Hough Transform to find circles in an image.
-   We will learn these functions: **cv.HoughCircles()**

Theory
------

A circle is represented mathematically as \f$(x-x_{center})^2 + (y - y_{center})^2 = r^2\f$ where
\f$(x_{center},y_{center})\f$ is the center of the circle, and \f$r\f$ is the radius of the circle. From
equation, we can see we have 3 parameters, so we need a 3D accumulator for hough transform, which
would be highly ineffective. So OpenCV uses more trickier method, **Hough Gradient Method** which
uses the gradient information of edges.

We use the function: **cv.HoughCircles (image, circles, method, dp, minDist, param1 = 100, param2 = 100, minRadius = 0, maxRadius = 0)**

@param image       8-bit, single-channel, grayscale input image.
@param circles     output vector of found circles(cv.CV_32FC3 type). Each vector is encoded as a 3-element floating-point vector (x,y,radius) .
@param method      detection method(see cv.HoughModes). Currently, the only implemented method is HOUGH_GRADIENT
@param dp      	   inverse ratio of the accumulator resolution to the image resolution. For example, if dp = 1 , the accumulator has the same resolution as the input image. If dp = 2 , the accumulator has half as big width and height.
@param minDist     minimum distance between the centers of the detected circles. If the parameter is too small, multiple neighbor circles may be falsely detected in addition to a true one. If it is too large, some circles may be missed.
@param param1      first method-specific parameter. In case of HOUGH_GRADIENT , it is the higher threshold of the two passed to the Canny edge detector (the lower one is twice smaller).
@param param2      second method-specific parameter. In case of HOUGH_GRADIENT , it is the accumulator threshold for the circle centers at the detection stage. The smaller it is, the more false circles may be detected. Circles, corresponding to the larger accumulator values, will be returned first.
@param minRadius   minimum circle radius.
@param maxRadius   maximum circle radius.

Try it
------

\htmlonly
<iframe src="../../js_houghcircles_HoughCirclesP.html" width="100%"
        onload="this.style.height=this.contentDocument.body.scrollHeight +'px';">
</iframe>
\endhtmlonly
```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

