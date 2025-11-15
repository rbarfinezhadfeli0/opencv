# Documentation for `modules/imgproc/include/opencv2/imgproc.hpp`

## File Metadata

- **Full Path**: `modules/imgproc/include/opencv2/imgproc.hpp`
- **File Name**: `imgproc.hpp`
- **File Size**: 264,515 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/imgproc/include/opencv2/imgproc.hpp](../../../../modules/imgproc/include/opencv2/imgproc.hpp)

## Purpose and Role

This file is located in the `modules/imgproc/include/opencv2` directory and serves as part of the OpenCV library infrastructure.

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
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
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
//   * The name of the copyright holders may not be used to endorse or promote products
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

#ifndef OPENCV_IMGPROC_HPP
#define OPENCV_IMGPROC_HPP

#include "opencv2/core.hpp"

/**
@defgroup imgproc Image Processing

This module offers a comprehensive suite of image processing functions, enabling tasks such as those listed above.

@{
    @defgroup imgproc_filter Image Filtering

    Functions and classes described in this section are used to perform various linear or non-linear
    filtering operations on 2D images (represented as Mat's). It means that for each pixel location
    \f$(x,y)\f$ in the source image (normally, rectangular), its neighborhood is considered and used to
    compute the response. In case of a linear filter, it is a weighted sum of pixel values. In case of
    morphological operations, it is the minimum or maximum values, and so on. The computed response is
    stored in the destination image at the same location \f$(x,y)\f$. It means that the output image
    will be of the same size as the input image. Normally, the functions support multi-channel arrays,
    in which case every channel is processed independently. Therefore, the output image will also have
    the same number of channels as the input one.

    Another common feature of the functions and classes described in this section is that, unlike
    simple arithmetic functions, they need to extrapolate values of some non-existing pixels. For
    example, if you want to smooth an image using a Gaussian \f$3 \times 3\f$ filter, then, when
    processing the left-most pixels in each row, you need pixels to the left of them, that is, outside
    of the image. You can let these pixels be the same as the left-most image pixels ("replicated
    border" extrapolation method), or assume that all the non-existing pixels are zeros ("constant
    border" extrapolation method), and so on. OpenCV enables you to specify the extrapolation method.
    For details, see #BorderTypes

    @anchor filter_depths
    ### Depth combinations
    Input depth (src.depth()) | Output depth (ddepth)
    --------------------------|----------------------
    CV_8U                     | -1/CV_16S/CV_32F/CV_64F
    CV_16U/CV_16S             | -1/CV_32F/CV_64F
    CV_32F                    | -1/CV_32F
    CV_64F                    | -1/CV_64F

    @note when ddepth=-1, the output image will have the same depth as the source.

    @note if you need double floating-point accuracy and using single floating-point input data
    (CV_32F input and CV_64F output depth combination), you can use @ref Mat.convertTo to convert
    the input data to the desired precision.

    @defgroup imgproc_transform Geometric Image Transformations

    The functions in this section perform various geometrical transformations of 2D images. They do not
    change the image content but deform the pixel grid and map this deformed grid to the destination
    image. In fact, to avoid sampling artifacts, the mapping is done in the reverse order, from
    destination to the source. That is, for each pixel \f$(x, y)\f$ of the destination image, the
    functions compute coordinates of the corresponding "donor" pixel in the source image and copy the
    pixel value:

    \f[\texttt{dst} (x,y)= \texttt{src} (f_x(x,y), f_y(x,y))\f]

    In case when you specify the forward mapping \f$\left<g_x, g_y\right>: \texttt{src} \rightarrow
    \texttt{dst}\f$, the OpenCV functions first compute the corresponding inverse mapping
    \f$\left<f_x, f_y\right>: \texttt{dst} \rightarrow \texttt{src}\f$ and then use the above formula.

    The actual implementations of the geometrical transformations, from the most generic remap and to
    the simplest and the fastest resize, need to solve two main problems with the above formula:

    - Extrapolation of non-existing pixels. Similarly to the filtering functions described in the
    previous section, for some \f$(x,y)\f$, either one of \f$f_x(x,y)\f$, or \f$f_y(x,y)\f$, or both
    of them may fall outside of the image. In this case, an extrapolation method needs to be used.
    OpenCV provides the same selection of extrapolation methods as in the filtering functions. In
    addition, it provides the method #BORDER_TRANSPARENT. This means that the corresponding pixels in
    the destination image will not be modified at all.

    - Interpolation of pixel values. Usually \f$f_x(x,y)\f$ and \f$f_y(x,y)\f$ are floating-point
    numbers. This means that \f$\left<f_x, f_y\right>\f$ can be either an affine or perspective
    transformation, or radial lens distortion correction, and so on. So, a pixel value at fractional
    coordinates needs to be retrieved. In the simplest case, the coordinates can be just rounded to the
    nearest integer coordinates and the corresponding pixel can be used. This is called a
    nearest-neighbor interpolation. However, a better result can be achieved by using more
    sophisticated [interpolation methods](https://en.wikipedia.org/wiki/Multivariate_interpolation) ,
    where a polynomial function is fit into some neighborhood of the computed pixel \f$(f_x(x,y),
    f_y(x,y))\f$, and then the value of the polynomial at \f$(f_x(x,y), f_y(x,y))\f$ is taken as the
    interpolated pixel value. In OpenCV, you can choose between several interpolation methods. See
    #resize for details.

    @note The geometrical transformations do not work with `CV_8S` or `CV_32S` images.

    @defgroup imgproc_misc Miscellaneous Image Transformations
    @defgroup imgproc_draw Drawing Functions

    Drawing functions work with matrices/images of arbitrary depth. The boundaries of the shapes can be
    rendered with antialiasing (implemented only for 8-bit images for now). All the functions include
    the parameter color that uses an RGB value (that may be constructed with the Scalar constructor )
    for color images and brightness for grayscale images. For color images, the channel ordering is
    normally *Blue, Green, Red*. This is what imshow, imread, and imwrite expect. So, if you form a
    color using the Scalar constructor, it should look like:

    \f[\texttt{Scalar} (blue \_ component, green \_ component, red \_ component[, alpha \_ component])\f]

    If you are using your own image rendering and I/O functions, you can use any channel ordering. The
    drawing functions process each channel independently and do not depend on the channel order or even
    on the used color space. The whole image can be converted from BGR to RGB or to a different color
    space using cvtColor .

    If a drawn figure is partially or completely outside the image, the drawing functions clip it. Also,
    many drawing functions can handle pixel coordinates specified with sub-pixel accuracy. This means
    that the coordinates can be passed as fixed-point numbers encoded as integers. The number of
    fractional bits is specified by the shift parameter and the real point coordinates are calculated as
    \f$\texttt{Point}(x,y)\rightarrow\texttt{Point2f}(x*2^{-shift},y*2^{-shift})\f$ . This feature is
    especially effective when rendering antialiased shapes.

    @note The functions do not support alpha-transparency when the target image is 4-channel. In this
    case, the color[3] is simply copied to the repainted pixels. Thus, if you want to paint
    semi-transparent shapes, you can paint them in a separate buffer and then blend it with the main
    image.

    @defgroup imgproc_color_conversions Color Space Conversions
    @defgroup imgproc_colormap ColorMaps in OpenCV

    The human perception isn't built for observing fine changes in grayscale images. Human eyes are more
    sensitive to observing changes between colors, so you often need to recolor your grayscale images to
    get a clue about them. OpenCV now comes with various colormaps to enhance the visualization in your
    computer vision application.

    In OpenCV you only need applyColorMap to apply a colormap on a given image. The following sample
    code reads the path to an image from command line, applies a Jet colormap on it and shows the
    result:

    @include snippets/imgproc_applyColorMap.cpp

    @see #ColormapTypes

    @defgroup imgproc_subdiv2d Planar Subdivision

    The Subdiv2D class described in this section is used to perform various planar subdivision on
    a set of 2D points (represented as vector of Point2f). OpenCV subdivides a plane into triangles
    using the Delaunay's algorithm, which corresponds to the dual graph of the Voronoi diagram.
    In the figure below, the Delaunay's triangulation is marked with black lines and the Voronoi
    diagram with red lines.

    ![Delaunay triangulation (black) and Voronoi (red)](pics/delaunay_voronoi.png)

    The subdivisions can be used for the 3D piece-wise transformation of a plane, morphing, fast
    location of points on the plane, building special graphs (such as NNG,RNG), and so forth.

    @defgroup imgproc_hist Histograms
    @defgroup imgproc_shape Structural Analysis and Shape Descriptors
    @defgroup imgproc_motion Motion Analysis and Object Tracking
    @defgroup imgproc_feature Feature Detection
    @defgroup imgproc_object Object Detection
    @defgroup imgproc_segmentation Image Segmentation
    @defgroup imgproc_hal Hardware Acceleration Layer
    @{
        @defgroup imgproc_hal_functions Functions
        @defgroup imgproc_hal_interface Interface
    @}
  @}
*/

namespace cv
{

/** @addtogroup imgproc
@{
*/

//! @addtogroup imgproc_filter
//! @{

enum SpecialFilter {
    FILTER_SCHARR = -1
};

//! type of morphological operation
enum MorphTypes{
    MORPH_ERODE    = 0, //!< see #erode
    MORPH_DILATE   = 1, //!< see #dilate
    MORPH_OPEN     = 2, //!< an opening operation
                        //!< \f[\texttt{dst} = \mathrm{open} ( \texttt{src} , \texttt{element} )= \mathrm{dilate} ( \mathrm{erode} ( \texttt{src} , \texttt{element} ))\f]
    MORPH_CLOSE    = 3, //!< a closing operation
                        //!< \f[\texttt{dst} = \mathrm{close} ( \texttt{src} , \texttt{element} )= \mathrm{erode} ( \mathrm{dilate} ( \texttt{src} , \texttt{element} ))\f]
    MORPH_GRADIENT = 4, //!< a morphological gradient
                        //!< \f[\texttt{dst} = \mathrm{morph\_grad} ( \texttt{src} , \texttt{element} )= \mathrm{dilate} ( \texttt{src} , \texttt{element} )- \mathrm{erode} ( \texttt{src} , \texttt{element} )\f]
    MORPH_TOPHAT   = 5, //!< "top hat"
                        //!< \f[\texttt{dst} = \mathrm{tophat} ( \texttt{src} , \texttt{element} )= \texttt{src} - \mathrm{open} ( \texttt{src} , \texttt{element} )\f]
    MORPH_BLACKHAT = 6, //!< "black hat"
                        //!< \f[\texttt{dst} = \mathrm{blackhat} ( \texttt{src} , \texttt{element} )= \mathrm{close} ( \texttt{src} , \texttt{element} )- \texttt{src}\f]
    MORPH_HITMISS  = 7  //!< "hit or miss"
                        //!<   .- Only supported for CV_8UC1 binary images. A tutorial can be found in the documentation
};

//! shape of the structuring element
enum MorphShapes {
    MORPH_RECT    = 0, //!< a rectangular structuring element:  \f[E_{ij}=1\f]
    MORPH_CROSS   = 1, //!< a cross-shaped structuring element:
                       //!< \f[E_{ij} = \begin{cases} 1 & \texttt{if } {i=\texttt{anchor.y } {or } {j=\texttt{anchor.x}}} \\0 & \texttt{otherwise} \end{cases}\f]
    MORPH_ELLIPSE = 2, //!< an elliptic structuring element, that is, a filled ellipse inscribed
                      //!< into the rectangle Rect(0, 0, esize.width, esize.height)
    MORPH_DIAMOND = 3  //!< a diamond structuring element defined by Manhattan distance
};

//! @} imgproc_filter

//! @addtogroup imgproc_transform
//! @{

//! interpolation algorithm
enum InterpolationFlags{
    /** nearest neighbor interpolation */
    INTER_NEAREST        = 0,
    /** bilinear interpolation */
    INTER_LINEAR         = 1,
    /** bicubic interpolation */
    INTER_CUBIC          = 2,
    /** resampling using pixel area relation. It may be a preferred method for image decimation, as
    it gives moire'-free results. But when the image is zoomed, it is similar to the INTER_NEAREST
    method. */
    INTER_AREA           = 3,
    /** Lanczos interpolation over 8x8 neighborhood */
    INTER_LANCZOS4       = 4,
    /** Bit exact bilinear interpolation */
    INTER_LINEAR_EXACT = 5,
    /** Bit exact nearest neighbor interpolation. This will produce same results as
    the nearest neighbor method in PIL, scikit-image or Matlab. */
    INTER_NEAREST_EXACT  = 6,
    /** mask for interpolation codes */
    INTER_MAX            = 7,
    /** flag, fills all of the destination image pixels. If some of them correspond to outliers in the
    source image, they are set to zero */
    WARP_FILL_OUTLIERS   = 8,
    /** flag, inverse transformation

    For example, #linearPolar or #logPolar transforms:
    - flag is __not__ set: \f$dst( \rho , \phi ) = src(x,y)\f$
    - flag is set: \f$dst(x,y) = src( \rho , \phi )\f$
    */
    WARP_INVERSE_MAP     = 16,
    WARP_RELATIVE_MAP    = 32
};

/** \brief Specify the polar mapping mode
@sa warpPolar
*/
enum WarpPolarMode
{
    WARP_POLAR_LINEAR = 0, ///< Remaps an image to/from polar space.
    WARP_POLAR_LOG = 256   ///< Remaps an image to/from semilog-polar space.
};

enum InterpolationMasks {
       INTER_BITS      = 5,
       INTER_BITS2     = INTER_BITS * 2,
       INTER_TAB_SIZE  = 1 << INTER_BITS,
       INTER_TAB_SIZE2 = INTER_TAB_SIZE * INTER_TAB_SIZE
     };

//! @} imgproc_transform

//! @addtogroup imgproc_misc
//! @{

//! Distance types for Distance Transform and M-estimators
//! @see distanceTransform, fitLine
enum DistanceTypes {
    DIST_USER    = -1,  //!< User defined distance
    DIST_L1      = 1,   //!< distance = |x1-x2| + |y1-y2|
    DIST_L2      = 2,   //!< the simple euclidean distance
    DIST_C       = 3,   //!< distance = max(|x1-x2|,|y1-y2|)
    DIST_L12     = 4,   //!< L1-L2 metric: distance = 2(sqrt(1+x*x/2) - 1))
    DIST_FAIR    = 5,   //!< distance = c^2(|x|/c-log(1+|x|/c)), c = 1.3998
    DIST_WELSCH  = 6,   //!< distance = c^2/2(1-exp(-(x/c)^2)), c = 2.9846
    DIST_HUBER   = 7    //!< distance = |x|<c ? x^2/2 : c(|x|-c/2), c=1.345
};

//! Mask size for distance transform
enum DistanceTransformMasks {
    DIST_MASK_3       = 3, //!< mask=3
    DIST_MASK_5       = 5, //!< mask=5
    DIST_MASK_PRECISE = 0  //!<
};

//! type of the threshold operation
//! ![threshold types](pics/threshold.png)
enum ThresholdTypes {
    THRESH_BINARY     = 0, //!< \f[\texttt{dst} (x,y) =  \fork{\texttt{maxval}}{if \(\texttt{src}(x,y) > \texttt{thresh}\)}{0}{otherwise}\f]
    THRESH_BINARY_INV = 1, //!< \f[\texttt{dst} (x,y) =  \fork{0}{if \(\texttt{src}(x,y) > \texttt{thresh}\)}{\texttt{maxval}}{otherwise}\f]
    THRESH_TRUNC      = 2, //!< \f[\texttt{dst} (x,y) =  \fork{\texttt{threshold}}{if \(\texttt{src}(x,y) > \texttt{thresh}\)}{\texttt{src}(x,y)}{otherwise}\f]
    THRESH_TOZERO     = 3, //!< \f[\texttt{dst} (x,y) =  \fork{\texttt{src}(x,y)}{if \(\texttt{src}(x,y) > \texttt{thresh}\)}{0}{otherwise}\f]
    THRESH_TOZERO_INV = 4, //!< \f[\texttt{dst} (x,y) =  \fork{0}{if \(\texttt{src}(x,y) > \texttt{thresh}\)}{\texttt{src}(x,y)}{otherwise}\f]
    THRESH_MASK       = 7,
    THRESH_OTSU       = 8, //!< flag, use Otsu algorithm to choose the optimal threshold value
    THRESH_TRIANGLE   = 16, //!< flag, use Triangle algorithm to choose the optimal threshold value
    THRESH_DRYRUN     = 128 //!< flag, compute threshold only (useful for OTSU/TRIANGLE) but does not actually run thresholding
};

//! adaptive threshold algorithm
//! @see adaptiveThreshold
enum AdaptiveThresholdTypes {
    /** the threshold value \f$T(x,y)\f$ is a mean of the \f$\texttt{blockSize} \times
    \texttt{blockSize}\f$ neighborhood of \f$(x, y)\f$ minus C */
    ADAPTIVE_THRESH_MEAN_C     = 0,
    /** the threshold value \f$T(x, y)\f$ is a weighted sum (cross-correlation with a Gaussian
    window) of the \f$\texttt{blockSize} \times \texttt{blockSize}\f$ neighborhood of \f$(x, y)\f$
    minus C . The default sigma (standard deviation) is used for the specified blockSize . See
    #getGaussianKernel*/
    ADAPTIVE_THRESH_GAUSSIAN_C = 1
};

//! class of the pixel in GrabCut algorithm
enum GrabCutClasses {
    GC_BGD    = 0,  //!< an obvious background pixels
    GC_FGD    = 1,  //!< an obvious foreground (object) pixel
    GC_PR_BGD = 2,  //!< a possible background pixel
    GC_PR_FGD = 3   //!< a possible foreground pixel
};

//! GrabCut algorithm flags
enum GrabCutModes {
    /** The function initializes the state and the mask using the provided rectangle. After that it
    runs iterCount iterations of the algorithm. */
    GC_INIT_WITH_RECT  = 0,
    /** The function initializes the state using the provided mask. Note that GC_INIT_WITH_RECT
    and GC_INIT_WITH_MASK can be combined. Then, all the pixels outside of the ROI are
    automatically initialized with GC_BGD .*/
    GC_INIT_WITH_MASK  = 1,
    /** The value means that the algorithm should just resume. */
    GC_EVAL            = 2,
    /** The value means that the algorithm should just run the grabCut algorithm (a single iteration) with the fixed model */
    GC_EVAL_FREEZE_MODEL = 3
};

//! distanceTransform algorithm flags
enum DistanceTransformLabelTypes {
    /** each connected component of zeros in src (as well as all the non-zero pixels closest to the
    connected component) will be assigned the same label */
    DIST_LABEL_CCOMP = 0,
    /** each zero pixel (and all the non-zero pixels closest to it) gets its own label. */
    DIST_LABEL_PIXEL = 1
};

//! floodfill algorithm flags
enum FloodFillFlags {
    /** If set, the difference between the current pixel and seed pixel is considered. Otherwise,
    the difference between neighbor pixels is considered (that is, the range is floating). */
    FLOODFILL_FIXED_RANGE = 1 << 16,
    /** If set, the function does not change the image ( newVal is ignored), and only fills the
    mask with the value specified in bits 8-16 of flags as described above. This option only make
    sense in function variants that have the mask parameter. */
    FLOODFILL_MASK_ONLY   = 1 << 17
};

//! @} imgproc_misc

//! @addtogroup imgproc_shape
//! @{

//! connected components statistics
enum ConnectedComponentsTypes {
    CC_STAT_LEFT   = 0, //!< The leftmost (x) coordinate which is the inclusive start of the bounding
                        //!< box in the horizontal direction.
    CC_STAT_TOP    = 1, //!< The topmost (y) coordinate which is the inclusive start of the bounding
                        //!< box in the vertical direction.
    CC_STAT_WIDTH  = 2, //!< The horizontal size of the bounding box
    CC_STAT_HEIGHT = 3, //!< The vertical size of the bounding box
    CC_STAT_AREA   = 4, //!< The total area (in pixels) of the connected component
#ifndef CV_DOXYGEN
    CC_STAT_MAX    = 5 //!< Max enumeration value. Used internally only for memory allocation
#endif
};

//! connected components algorithm
enum ConnectedComponentsAlgorithmsTypes {
    CCL_DEFAULT   = -1, //!< Spaghetti @cite Bolelli2019 algorithm for 8-way connectivity, Spaghetti4C @cite Bolelli2021 algorithm for 4-way connectivity.
    CCL_WU        = 0,  //!< SAUF @cite Wu2009 algorithm for 8-way connectivity, SAUF algorithm for 4-way connectivity. The parallel implementation described in @cite Bolelli2017 is available for SAUF.
    CCL_GRANA     = 1,  //!< BBDT @cite Grana2010 algorithm for 8-way connectivity, SAUF algorithm for 4-way connectivity. The parallel implementation described in @cite Bolelli2017 is available for both BBDT and SAUF.
    CCL_BOLELLI   = 2,  //!< Spaghetti @cite Bolelli2019 algorithm for 8-way connectivity, Spaghetti4C @cite Bolelli2021 algorithm for 4-way connectivity. The parallel implementation described in @cite Bolelli2017 is available for both Spaghetti and Spaghetti4C.
    CCL_SAUF      = 3,  //!< Same as CCL_WU. It is preferable to use the flag with the name of the algorithm (CCL_SAUF) rather than the one with the name of the first author (CCL_WU).
    CCL_BBDT      = 4,  //!< Same as CCL_GRANA. It is preferable to use the flag with the name of the algorithm (CCL_BBDT) rather than the one with the name of the first author (CCL_GRANA).
    CCL_SPAGHETTI = 5,  //!< Same as CCL_BOLELLI. It is preferable to use the flag with the name of the algorithm (CCL_SPAGHETTI) rather than the one with the name of the first author (CCL_BOLELLI).
};

//! mode of the contour retrieval algorithm
enum RetrievalModes {
    /** retrieves only the extreme outer contours. It sets `hierarchy[i][2]=hierarchy[i][3]=-1` for
    all the contours. */
    RETR_EXTERNAL  = 0,
    /** retrieves all of the contours without establishing any hierarchical relationships. */
    RETR_LIST      = 1,
    /** retrieves all of the contours and organizes them into a two-level hierarchy. At the top
    level, there are external boundaries of the components. At the second level, there are
    boundaries of the holes. If there is another contour inside a hole of a connected component, it
    is still put at the top level. */
    RETR_CCOMP     = 2,
    /** retrieves all of the contours and reconstructs a full hierarchy of nested contours.*/
    RETR_TREE      = 3,
    RETR_FLOODFILL = 4 //!<
};

//! the contour approximation algorithm
enum ContourApproximationModes {
    /** stores absolutely all the contour points. That is, any 2 subsequent points (x1,y1) and
    (x2,y2) of the contour will be either horizontal, vertical or diagonal neighbors, that is,
    max(abs(x1-x2),abs(y2-y1))==1. */
    CHAIN_APPROX_NONE      = 1,
    /** compresses horizontal, vertical, and diagonal segments and leaves only their end points.
    For example, an up-right rectangular contour is encoded with 4 points. */
    CHAIN_APPROX_SIMPLE    = 2,
    /** applies one of the flavors of the Teh-Chin chain approximation algorithm @cite TehChin89 */
    CHAIN_APPROX_TC89_L1   = 3,
    /** applies one of the flavors of the Teh-Chin chain approximation algorithm @cite TehChin89 */
    CHAIN_APPROX_TC89_KCOS = 4
};

/** @brief Shape matching methods

\f$A\f$ denotes object1,\f$B\f$ denotes object2

\f$\begin{array}{l} m^A_i =  \mathrm{sign} (h^A_i)  \cdot \log{h^A_i} \\ m^B_i =  \mathrm{sign} (h^B_i)  \cdot \log{h^B_i} \end{array}\f$

and \f$h^A_i, h^B_i\f$ are the Hu moments of \f$A\f$ and \f$B\f$ , respectively.
*/
enum ShapeMatchModes {
    CONTOURS_MATCH_I1  =1, //!< \f[I_1(A,B) =  \sum _{i=1...7}  \left |  \frac{1}{m^A_i} -  \frac{1}{m^B_i} \right |\f]
    CONTOURS_MATCH_I2  =2, //!< \f[I_2(A,B) =  \sum _{i=1...7}  \left | m^A_i - m^B_i  \right |\f]
    CONTOURS_MATCH_I3  =3  //!< \f[I_3(A,B) =  \max _{i=1...7}  \frac{ \left| m^A_i - m^B_i \right| }{ \left| m^A_i \right| }\f]
};

//! @} imgproc_shape

//! @addtogroup imgproc_feature
//! @{

//! Variants of a Hough transform
enum HoughModes {

    /** classical or standard Hough transform. Every line is represented by two floating-point
    numbers \f$(\rho, \theta)\f$ , where \f$\rho\f$ is a distance between (0,0) point and the line,
    and \f$\theta\f$ is the angle between x-axis and the normal to the line. Thus, the matrix must
    be (the created sequence will be) of CV_32FC2 type */
    HOUGH_STANDARD      = 0,
    /** probabilistic Hough transform (more efficient in case if the picture contains a few long
    linear segments). It returns line segments rather than the whole line. Each segment is
    represented by starting and ending points, and the matrix must be (the created sequence will
    be) of the CV_32SC4 type. */
    HOUGH_PROBABILISTIC = 1,
    /** multi-scale variant of the classical Hough transform. The lines are encoded the same way as
    HOUGH_STANDARD. */
    HOUGH_MULTI_SCALE   = 2,
    HOUGH_GRADIENT      = 3, //!< basically *21HT*, described in @cite Yuen90
    HOUGH_GRADIENT_ALT  = 4, //!< variation of HOUGH_GRADIENT to get better accuracy
};

//! Variants of Line Segment %Detector
enum LineSegmentDetectorModes {
    LSD_REFINE_NONE = 0, //!< No refinement applied
    LSD_REFINE_STD  = 1, //!< Standard refinement is applied. E.g. breaking arches into smaller straighter line approximations.
    LSD_REFINE_ADV  = 2  //!< Advanced refinement. Number of false alarms is calculated, lines are
                         //!< refined through increase of precision, decrement in size, etc.
};

//! @} imgproc_feature

/** Histogram comparison methods
  @ingroup imgproc_hist
*/
enum HistCompMethods {
    /** Correlation
    \f[d(H_1,H_2) =  \frac{\sum_I (H_1(I) - \bar{H_1}) (H_2(I) - \bar{H_2})}{\sqrt{\sum_I(H_1(I) - \bar{H_1})^2 \sum_I(H_2(I) - \bar{H_2})^2}}\f]
    where
    \f[\bar{H_k} =  \frac{1}{N} \sum _J H_k(J)\f]
    and \f$N\f$ is a total number of histogram bins. */
    HISTCMP_CORREL        = 0,
    /** Chi-Square
    \f[d(H_1,H_2) =  \sum _I  \frac{\left(H_1(I)-H_2(I)\right)^2}{H_1(I)}\f] */
    HISTCMP_CHISQR        = 1,
    /** Intersection
    \f[d(H_1,H_2) =  \sum _I  \min (H_1(I), H_2(I))\f] */
    HISTCMP_INTERSECT     = 2,
    /** Bhattacharyya distance
    (In fact, OpenCV computes Hellinger distance, which is related to Bhattacharyya coefficient.)
    \f[d(H_1,H_2) =  \sqrt{1 - \frac{1}{\sqrt{\bar{H_1} \bar{H_2} N^2}} \sum_I \sqrt{H_1(I) \cdot H_2(I)}}\f] */
    HISTCMP_BHATTACHARYYA = 3,
    HISTCMP_HELLINGER     = HISTCMP_BHATTACHARYYA, //!< Synonym for HISTCMP_BHATTACHARYYA
    /** Alternative Chi-Square
    \f[d(H_1,H_2) =  2 * \sum _I  \frac{\left(H_1(I)-H_2(I)\right)^2}{H_1(I)+H_2(I)}\f]
    This alternative formula is regularly used for texture comparison. See e.g. @cite Puzicha1997 */
    HISTCMP_CHISQR_ALT    = 4,
    /** Kullback-Leibler divergence
    \f[d(H_1,H_2) = \sum _I H_1(I) \log \left(\frac{H_1(I)}{H_2(I)}\right)\f] */
    HISTCMP_KL_DIV        = 5
};

/** the color conversion codes
@see @ref imgproc_color_conversions
@ingroup imgproc_color_conversions
 */
enum ColorConversionCodes {
    COLOR_BGR2BGRA     = 0, //!< add alpha channel to RGB or BGR image
    COLOR_RGB2RGBA     = COLOR_BGR2BGRA,

    COLOR_BGRA2BGR     = 1, //!< remove alpha channel from RGB or BGR image
    COLOR_RGBA2RGB     = COLOR_BGRA2BGR,

    COLOR_BGR2RGBA     = 2, //!< convert between RGB and BGR color spaces (with or without alpha channel)
    COLOR_RGB2BGRA     = COLOR_BGR2RGBA,

    COLOR_RGBA2BGR     = 3,
    COLOR_BGRA2RGB     = COLOR_RGBA2BGR,

    COLOR_BGR2RGB      = 4,
    COLOR_RGB2BGR      = COLOR_BGR2RGB,

    COLOR_BGRA2RGBA    = 5,
    COLOR_RGBA2BGRA    = COLOR_BGRA2RGBA,

    COLOR_BGR2GRAY     = 6, //!< convert between RGB/BGR and grayscale, @ref color_convert_rgb_gray "color conversions"
    COLOR_RGB2GRAY     = 7,
    COLOR_GRAY2BGR     = 8,
    COLOR_GRAY2RGB     = COLOR_GRAY2BGR,
    COLOR_GRAY2BGRA    = 9,
    COLOR_GRAY2RGBA    = COLOR_GRAY2BGRA,
    COLOR_BGRA2GRAY    = 10,
    COLOR_RGBA2GRAY    = 11,

    COLOR_BGR2BGR565   = 12, //!< convert between RGB/BGR and BGR565 (16-bit images)
    COLOR_RGB2BGR565   = 13,
    COLOR_BGR5652BGR   = 14,
    COLOR_BGR5652RGB   = 15,
    COLOR_BGRA2BGR565  = 16,
    COLOR_RGBA2BGR565  = 17,
    COLOR_BGR5652BGRA  = 18,
    COLOR_BGR5652RGBA  = 19,

    COLOR_GRAY2BGR565  = 20, //!< convert between grayscale to BGR565 (16-bit images)
    COLOR_BGR5652GRAY  = 21,

    COLOR_BGR2BGR555   = 22,  //!< convert between RGB/BGR and BGR555 (16-bit images)
    COLOR_RGB2BGR555   = 23,
    COLOR_BGR5552BGR   = 24,
    COLOR_BGR5552RGB   = 25,
    COLOR_BGRA2BGR555  = 26,
    COLOR_RGBA2BGR555  = 27,
    COLOR_BGR5552BGRA  = 28,
    COLOR_BGR5552RGBA  = 29,

    COLOR_GRAY2BGR555  = 30, //!< convert between grayscale and BGR555 (16-bit images)
    COLOR_BGR5552GRAY  = 31,

    COLOR_BGR2XYZ      = 32, //!< convert RGB/BGR to CIE XYZ, @ref color_convert_rgb_xyz "color conversions"
    COLOR_RGB2XYZ      = 33,
    COLOR_XYZ2BGR      = 34,
    COLOR_XYZ2RGB      = 35,

    COLOR_BGR2YCrCb    = 36, //!< convert RGB/BGR to luma-chroma (aka YCC), @ref color_convert_rgb_ycrcb "color conversions"
    COLOR_RGB2YCrCb    = 37,
    COLOR_YCrCb2BGR    = 38,
    COLOR_YCrCb2RGB    = 39,

    COLOR_BGR2HSV      = 40, //!< convert RGB/BGR to HSV (hue saturation value) with H range 0..180 if 8 bit image, @ref color_convert_rgb_hsv "color conversions"
    COLOR_RGB2HSV      = 41,

    COLOR_BGR2Lab      = 44, //!< convert RGB/BGR to CIE Lab, @ref color_convert_rgb_lab "color conversions"
    COLOR_RGB2Lab      = 45,

    COLOR_BGR2Luv      = 50, //!< convert RGB/BGR to CIE Luv, @ref color_convert_rgb_luv "color conversions"
    COLOR_RGB2Luv      = 51,
    COLOR_BGR2HLS      = 52, //!< convert RGB/BGR to HLS (hue lightness saturation) with H range 0..180 if 8 bit image, @ref color_convert_rgb_hls "color conversions"
    COLOR_RGB2HLS      = 53,

    COLOR_HSV2BGR      = 54, //!< backward conversions HSV to RGB/BGR with H range 0..180 if 8 bit image
    COLOR_HSV2RGB      = 55,

    COLOR_Lab2BGR      = 56,
    COLOR_Lab2RGB      = 57,
    COLOR_Luv2BGR      = 58,
    COLOR_Luv2RGB      = 59,
    COLOR_HLS2BGR      = 60, //!< backward conversions HLS to RGB/BGR with H range 0..180 if 8 bit image
    COLOR_HLS2RGB      = 61,

    COLOR_BGR2HSV_FULL = 66, //!< convert RGB/BGR to HSV (hue saturation value) with H range 0..255 if 8 bit image, @ref color_convert_rgb_hsv "color conversions"
    COLOR_RGB2HSV_FULL = 67,
    COLOR_BGR2HLS_FULL = 68, //!< convert RGB/BGR to HLS (hue lightness saturation) with H range 0..255 if 8 bit image, @ref color_convert_rgb_hls "color conversions"
    COLOR_RGB2HLS_FULL = 69,

    COLOR_HSV2BGR_FULL = 70, //!< backward conversions HSV to RGB/BGR with H range 0..255 if 8 bit image
    COLOR_HSV2RGB_FULL = 71,
    COLOR_HLS2BGR_FULL = 72, //!< backward conversions HLS to RGB/BGR with H range 0..255 if 8 bit image
    COLOR_HLS2RGB_FULL = 73,

    COLOR_LBGR2Lab     = 74,
    COLOR_LRGB2Lab     = 75,
    COLOR_LBGR2Luv     = 76,
    COLOR_LRGB2Luv     = 77,

    COLOR_Lab2LBGR     = 78,
    COLOR_Lab2LRGB     = 79,
    COLOR_Luv2LBGR     = 80,
    COLOR_Luv2LRGB     = 81,

    COLOR_BGR2YUV      = 82, //!< convert between RGB/BGR and YUV
    COLOR_RGB2YUV      = 83,
    COLOR_YUV2BGR      = 84,
    COLOR_YUV2RGB      = 85,

    COLOR_YUV2RGB_NV12  = 90, //!< convert between 4:2:0-subsampled YUV NV12 and RGB, two planes (in one or separate arrays): Y and U/V interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_NV12  = 91, //!< convert between 4:2:0-subsampled YUV NV12 and BGR, two planes (in one or separate arrays): Y and U/V interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_NV21  = 92, //!< convert between 4:2:0-subsampled YUV NV21 and RGB, two planes (in one or separate arrays): Y and V/U interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_NV21  = 93, //!< convert between 4:2:0-subsampled YUV NV21 and BGR, two planes (in one or separate arrays): Y and V/U interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV420sp2RGB  = COLOR_YUV2RGB_NV21, //!< synonym to NV21
    COLOR_YUV420sp2BGR  = COLOR_YUV2BGR_NV21, //!< synonym to NV21

    COLOR_YUV2RGBA_NV12 = 94, //!< convert between 4:2:0-subsampled YUV NV12 and RGBA, two planes (in one or separate arrays): Y and U/V interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_NV12 = 95, //!< convert between 4:2:0-subsampled YUV NV12 and BGRA, two planes (in one or separate arrays): Y and U/V interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_NV21 = 96, //!< convert between 4:2:0-subsampled YUV NV21 and RGBA, two planes (in one or separate arrays): Y and V/U interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_NV21 = 97, //!< convert between 4:2:0-subsampled YUV NV21 and BGRA, two planes (in one or separate arrays): Y and V/U interleaved, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV420sp2RGBA = COLOR_YUV2RGBA_NV21, //!< synonym to NV21
    COLOR_YUV420sp2BGRA = COLOR_YUV2BGRA_NV21, //!< synonym to NV21

    COLOR_YUV2RGB_YV12  =  98, //!< convert between 4:2:0-subsampled YUV YV12 and RGB, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_YV12  =  99, //!< convert between 4:2:0-subsampled YUV YV12 and BGR, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_IYUV  = 100, //!< convert between 4:2:0-subsampled YUV IYUV and RGB, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_IYUV  = 101, //!< convert between 4:2:0-subsampled YUV IYUV and BGR, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_I420  = COLOR_YUV2RGB_IYUV, //!< synonym to IYUV
    COLOR_YUV2BGR_I420  = COLOR_YUV2BGR_IYUV, //!< synonym to IYUV
    COLOR_YUV420p2RGB   = COLOR_YUV2RGB_YV12, //!< synonym to YV12
    COLOR_YUV420p2BGR   = COLOR_YUV2BGR_YV12, //!< synonym to YV12

    COLOR_YUV2RGBA_YV12 = 102, //!< convert between 4:2:0-subsampled YUV YV12 and RGBA, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_YV12 = 103, //!< convert between 4:2:0-subsampled YUV YV12 and BGRA, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_IYUV = 104, //!< convert between 4:2:0-subsampled YUV YV12 and RGBA, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_IYUV = 105, //!< convert between 4:2:0-subsampled YUV YV12 and BGRA, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_I420 = COLOR_YUV2RGBA_IYUV, //!< synonym to IYUV
    COLOR_YUV2BGRA_I420 = COLOR_YUV2BGRA_IYUV, //!< synonym to IYUV
    COLOR_YUV420p2RGBA  = COLOR_YUV2RGBA_YV12, //!< synonym to YV12
    COLOR_YUV420p2BGRA  = COLOR_YUV2BGRA_YV12, //!< synonym to YV12

    COLOR_YUV2GRAY_420  = 106, //!< extract Y channel from YUV 4:2:0 image
    COLOR_YUV2GRAY_NV21 = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV2GRAY_NV12 = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV2GRAY_YV12 = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV2GRAY_IYUV = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV2GRAY_I420 = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV420sp2GRAY = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420
    COLOR_YUV420p2GRAY  = COLOR_YUV2GRAY_420, //!< synonym to COLOR_YUV2GRAY_420

    COLOR_YUV2RGB_UYVY = 107, //!< convert between YUV UYVY and RGB, YUV is 4:2:2-subsampled and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_UYVY = 108, //!< convert between YUV UYVY and BGR, YUV is 4:2:2-subsampled and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    //COLOR_YUV2RGB_VYUY = 109, //!< convert between YUV VYUY and RGB, YUV is 4:2:2-subsampled and interleaved as V/Y1/U/Y2, see @ref color_convert_rgb_yuv_42x
    //COLOR_YUV2BGR_VYUY = 110, //!< convert between YUV VYUY and BGR, YUV is 4:2:2-subsampled and interleaved as V/Y1/U/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_Y422 = COLOR_YUV2RGB_UYVY, //!< synonym to UYVY
    COLOR_YUV2BGR_Y422 = COLOR_YUV2BGR_UYVY, //!< synonym to UYVY
    COLOR_YUV2RGB_UYNV = COLOR_YUV2RGB_UYVY, //!< synonym to UYVY
    COLOR_YUV2BGR_UYNV = COLOR_YUV2BGR_UYVY, //!< synonym to UYVY

    COLOR_YUV2RGBA_UYVY = 111, //!< convert between YUV UYVY and RGBA, YUV is 4:2:2-subsampled and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_UYVY = 112, //!< convert between YUV UYVY and BGRA, YUV is 4:2:2-subsampled and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    //COLOR_YUV2RGBA_VYUY = 113, //!< convert between YUV VYUY and RGBA, YUV is 4:2:2-subsampled and interleaved as V/Y1/U/Y2, see @ref color_convert_rgb_yuv_42x
    //COLOR_YUV2BGRA_VYUY = 114, //!< convert between YUV VYUY and BGRA, YUV is 4:2:2-subsampled and interleaved as V/Y1/U/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_Y422 = COLOR_YUV2RGBA_UYVY, //!< synonym to UYVY
    COLOR_YUV2BGRA_Y422 = COLOR_YUV2BGRA_UYVY, //!< synonym to UYVY
    COLOR_YUV2RGBA_UYNV = COLOR_YUV2RGBA_UYVY, //!< synonym to UYVY
    COLOR_YUV2BGRA_UYNV = COLOR_YUV2BGRA_UYVY, //!< synonym to UYVY

    COLOR_YUV2RGB_YUY2 = 115, //!< convert between YUV YUY2 and RGB, YUV is 4:2:2-subsampled and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_YUY2 = 116, //!< convert between YUV YUY2 and BGR, YUV is 4:2:2-subsampled and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_YVYU = 117, //!< convert between YUV YVYU and RGB, YUV is 4:2:2-subsampled and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGR_YVYU = 118, //!< convert between YUV YVYU and BGR, YUV is 4:2:2-subsampled and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGB_YUYV = COLOR_YUV2RGB_YUY2, //!< synonym to YUY2
    COLOR_YUV2BGR_YUYV = COLOR_YUV2BGR_YUY2, //!< synonym to YUY2
    COLOR_YUV2RGB_YUNV = COLOR_YUV2RGB_YUY2, //!< synonym to YUY2
    COLOR_YUV2BGR_YUNV = COLOR_YUV2BGR_YUY2, //!< synonym to YUY2

    COLOR_YUV2RGBA_YUY2 = 119, //!< convert between YUV YUY2 and RGBA, YUV is 4:2:2-subsampled and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_YUY2 = 120, //!< convert between YUV YUY2 and BGRA, YUV is 4:2:2-subsampled and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_YVYU = 121, //!< convert between YUV YVYU and RGBA, YUV is 4:2:2-subsampled and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2BGRA_YVYU = 122, //!< convert between YUV YVYU and BGRA, YUV is 4:2:2-subsampled and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_YUV2RGBA_YUYV = COLOR_YUV2RGBA_YUY2, //!< synonym to YUY2
    COLOR_YUV2BGRA_YUYV = COLOR_YUV2BGRA_YUY2, //!< synonym to YUY2
    COLOR_YUV2RGBA_YUNV = COLOR_YUV2RGBA_YUY2, //!< synonym to YUY2
    COLOR_YUV2BGRA_YUNV = COLOR_YUV2BGRA_YUY2, //!< synonym to YUY2

    COLOR_YUV2GRAY_UYVY = 123, //!< extract Y channel from YUV 4:2:2 image
    COLOR_YUV2GRAY_YUY2 = 124, //!< extract Y channel from YUV 4:2:2 image
    //CV_YUV2GRAY_VYUY  = CV_YUV2GRAY_UYVY, //!< synonym to COLOR_YUV2GRAY_UYVY
    COLOR_YUV2GRAY_Y422 = COLOR_YUV2GRAY_UYVY, //!< synonym to COLOR_YUV2GRAY_UYVY
    COLOR_YUV2GRAY_UYNV = COLOR_YUV2GRAY_UYVY, //!< synonym to COLOR_YUV2GRAY_UYVY
    COLOR_YUV2GRAY_YVYU = COLOR_YUV2GRAY_YUY2, //!< synonym to COLOR_YUV2GRAY_YUY2
    COLOR_YUV2GRAY_YUYV = COLOR_YUV2GRAY_YUY2, //!< synonym to COLOR_YUV2GRAY_YUY2
    COLOR_YUV2GRAY_YUNV = COLOR_YUV2GRAY_YUY2, //!< synonym to COLOR_YUV2GRAY_YUY2

    //! alpha premultiplication
    COLOR_RGBA2mRGBA    = 125,
    COLOR_mRGBA2RGBA    = 126,

    COLOR_RGB2YUV_I420  = 127, //!< convert between RGB and 4:2:0-subsampled YUV I420, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_BGR2YUV_I420  = 128, //!< convert between BGR and 4:2:0-subsampled YUV I420, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_RGB2YUV_IYUV  = COLOR_RGB2YUV_I420, //!< synonym to I420
    COLOR_BGR2YUV_IYUV  = COLOR_BGR2YUV_I420, //!< synonym to I420

    COLOR_RGBA2YUV_I420 = 129, //!< convert between RGBA and 4:2:0-subsampled YUV I420, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_BGRA2YUV_I420 = 130, //!< convert between BGRA and 4:2:0-subsampled YUV I420, three planes in one array: Y, U and V, see @ref color_convert_rgb_yuv_42x
    COLOR_RGBA2YUV_IYUV = COLOR_RGBA2YUV_I420, //!< synonym to I420
    COLOR_BGRA2YUV_IYUV = COLOR_BGRA2YUV_I420, //!< synonym to I420
    COLOR_RGB2YUV_YV12  = 131, //!< convert between RGB and 4:2:0-subsampled YUV YV12, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_BGR2YUV_YV12  = 132, //!< convert between BGR and 4:2:0-subsampled YUV YV12, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_RGBA2YUV_YV12 = 133, //!< convert between RGBA and 4:2:0-subsampled YUV YV12, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x
    COLOR_BGRA2YUV_YV12 = 134, //!< convert between BGRA and 4:2:0-subsampled YUV YV12, three planes in one array: Y, V and U, see @ref color_convert_rgb_yuv_42x

    //! Demosaicing, see @ref color_convert_bayer "color conversions" for additional information
    COLOR_BayerBG2BGR = 46, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2BGR = 47, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2BGR = 48, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2BGR = 49, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerRGGB2BGR = COLOR_BayerBG2BGR,
    COLOR_BayerGRBG2BGR = COLOR_BayerGB2BGR,
    COLOR_BayerBGGR2BGR = COLOR_BayerRG2BGR,
    COLOR_BayerGBRG2BGR = COLOR_BayerGR2BGR,

    COLOR_BayerRGGB2RGB = COLOR_BayerBGGR2BGR,
    COLOR_BayerGRBG2RGB = COLOR_BayerGBRG2BGR,
    COLOR_BayerBGGR2RGB = COLOR_BayerRGGB2BGR,
    COLOR_BayerGBRG2RGB = COLOR_BayerGRBG2BGR,

    COLOR_BayerBG2RGB = COLOR_BayerRG2BGR, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2RGB = COLOR_BayerGR2BGR, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2RGB = COLOR_BayerBG2BGR, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2RGB = COLOR_BayerGB2BGR, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerBG2GRAY = 86, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2GRAY = 87, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2GRAY = 88, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2GRAY = 89, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerRGGB2GRAY = COLOR_BayerBG2GRAY,
    COLOR_BayerGRBG2GRAY = COLOR_BayerGB2GRAY,
    COLOR_BayerBGGR2GRAY = COLOR_BayerRG2GRAY,
    COLOR_BayerGBRG2GRAY = COLOR_BayerGR2GRAY,

    //! Demosaicing using Variable Number of Gradients
    COLOR_BayerBG2BGR_VNG = 62, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2BGR_VNG = 63, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2BGR_VNG = 64, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2BGR_VNG = 65, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerRGGB2BGR_VNG = COLOR_BayerBG2BGR_VNG,
    COLOR_BayerGRBG2BGR_VNG = COLOR_BayerGB2BGR_VNG,
    COLOR_BayerBGGR2BGR_VNG = COLOR_BayerRG2BGR_VNG,
    COLOR_BayerGBRG2BGR_VNG = COLOR_BayerGR2BGR_VNG,

    COLOR_BayerRGGB2RGB_VNG = COLOR_BayerBGGR2BGR_VNG,
    COLOR_BayerGRBG2RGB_VNG = COLOR_BayerGBRG2BGR_VNG,
    COLOR_BayerBGGR2RGB_VNG = COLOR_BayerRGGB2BGR_VNG,
    COLOR_BayerGBRG2RGB_VNG = COLOR_BayerGRBG2BGR_VNG,

    COLOR_BayerBG2RGB_VNG = COLOR_BayerRG2BGR_VNG, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2RGB_VNG = COLOR_BayerGR2BGR_VNG, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2RGB_VNG = COLOR_BayerBG2BGR_VNG, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2RGB_VNG = COLOR_BayerGB2BGR_VNG, //!< equivalent to GBRG Bayer pattern

    //! Edge-Aware Demosaicing
    COLOR_BayerBG2BGR_EA  = 135, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2BGR_EA  = 136, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2BGR_EA  = 137, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2BGR_EA  = 138, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerRGGB2BGR_EA  = COLOR_BayerBG2BGR_EA,
    COLOR_BayerGRBG2BGR_EA  = COLOR_BayerGB2BGR_EA,
    COLOR_BayerBGGR2BGR_EA  = COLOR_BayerRG2BGR_EA,
    COLOR_BayerGBRG2BGR_EA  = COLOR_BayerGR2BGR_EA,

    COLOR_BayerRGGB2RGB_EA  = COLOR_BayerBGGR2BGR_EA,
    COLOR_BayerGRBG2RGB_EA  = COLOR_BayerGBRG2BGR_EA,
    COLOR_BayerBGGR2RGB_EA  = COLOR_BayerRGGB2BGR_EA,
    COLOR_BayerGBRG2RGB_EA  = COLOR_BayerGRBG2BGR_EA,

    COLOR_BayerBG2RGB_EA  = COLOR_BayerRG2BGR_EA, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2RGB_EA  = COLOR_BayerGR2BGR_EA, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2RGB_EA  = COLOR_BayerBG2BGR_EA, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2RGB_EA  = COLOR_BayerGB2BGR_EA, //!< equivalent to GBRG Bayer pattern

    //! Demosaicing with alpha channel
    COLOR_BayerBG2BGRA = 139, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2BGRA = 140, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2BGRA = 141, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2BGRA = 142, //!< equivalent to GBRG Bayer pattern

    COLOR_BayerRGGB2BGRA = COLOR_BayerBG2BGRA,
    COLOR_BayerGRBG2BGRA = COLOR_BayerGB2BGRA,
    COLOR_BayerBGGR2BGRA = COLOR_BayerRG2BGRA,
    COLOR_BayerGBRG2BGRA = COLOR_BayerGR2BGRA,

    COLOR_BayerRGGB2RGBA = COLOR_BayerBGGR2BGRA,
    COLOR_BayerGRBG2RGBA = COLOR_BayerGBRG2BGRA,
    COLOR_BayerBGGR2RGBA = COLOR_BayerRGGB2BGRA,
    COLOR_BayerGBRG2RGBA = COLOR_BayerGRBG2BGRA,

    COLOR_BayerBG2RGBA = COLOR_BayerRG2BGRA, //!< equivalent to RGGB Bayer pattern
    COLOR_BayerGB2RGBA = COLOR_BayerGR2BGRA, //!< equivalent to GRBG Bayer pattern
    COLOR_BayerRG2RGBA = COLOR_BayerBG2BGRA, //!< equivalent to BGGR Bayer pattern
    COLOR_BayerGR2RGBA = COLOR_BayerGB2BGRA, //!< equivalent to GBRG Bayer pattern

    COLOR_RGB2YUV_UYVY = 143, //!< convert between RGB and YUV UYVU, YUV is 4:2:2 and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_BGR2YUV_UYVY = 144, //!< convert between BGR and YUV UYVU, YUV is 4:2:2 and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_RGB2YUV_Y422 = COLOR_RGB2YUV_UYVY, //!< synonym to UYVY
    COLOR_BGR2YUV_Y422 = COLOR_BGR2YUV_UYVY, //!< synonym to UYVY
    COLOR_RGB2YUV_UYNV = COLOR_RGB2YUV_UYVY, //!< synonym to UYVY
    COLOR_BGR2YUV_UYNV = COLOR_BGR2YUV_UYVY, //!< synonym to UYVY

    COLOR_RGBA2YUV_UYVY = 145, //!< convert between RGBA and YUV UYVU, YUV is 4:2:2 and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_BGRA2YUV_UYVY = 146, //!< convert between BGRA and YUV UYVU, YUV is 4:2:2 and interleaved as U/Y1/V/Y2, see @ref color_convert_rgb_yuv_42x
    COLOR_RGBA2YUV_Y422 = COLOR_RGBA2YUV_UYVY, //!< synonym to UYVY
    COLOR_BGRA2YUV_Y422 = COLOR_BGRA2YUV_UYVY, //!< synonym to UYVY
    COLOR_RGBA2YUV_UYNV = COLOR_RGBA2YUV_UYVY, //!< synonym to UYVY
    COLOR_BGRA2YUV_UYNV = COLOR_BGRA2YUV_UYVY, //!< synonym to UYVY

    COLOR_RGB2YUV_YUY2 = 147, //!< convert between RGB and YUV YUY2, YUV is 4:2:2 and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_BGR2YUV_YUY2 = 148, //!< convert between BGR and YUV YUY2, YUV is 4:2:2 and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_RGB2YUV_YVYU = 149, //!< convert between RGB and YUV YVYU, YUV is 4:2:2 and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_BGR2YUV_YVYU = 150, //!< convert between BGR and YUV YVYU, YUV is 4:2:2 and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_RGB2YUV_YUYV = COLOR_RGB2YUV_YUY2, //!< synonym to YUY2
    COLOR_BGR2YUV_YUYV = COLOR_BGR2YUV_YUY2, //!< synonym to YUY2
    COLOR_RGB2YUV_YUNV = COLOR_RGB2YUV_YUY2, //!< synonym to YUY2
    COLOR_BGR2YUV_YUNV = COLOR_BGR2YUV_YUY2, //!< synonym to YUY2

    COLOR_RGBA2YUV_YUY2 = 151, //!< convert between RGBA and YUV YUY2, YUV is 4:2:2 and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_BGRA2YUV_YUY2 = 152, //!< convert between BGRA and YUV YUY2, YUV is 4:2:2 and interleaved as Y1/U/Y2/V, see @ref color_convert_rgb_yuv_42x
    COLOR_RGBA2YUV_YVYU = 153, //!< convert between RGBA and YUV YVYU, YUV is 4:2:2 and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_BGRA2YUV_YVYU = 154, //!< convert between BGRA and YUV YVYU, YUV is 4:2:2 and interleaved as Y1/V/Y2/U, see @ref color_convert_rgb_yuv_42x
    COLOR_RGBA2YUV_YUYV = COLOR_RGBA2YUV_YUY2, //!< synonym to YUY2
    COLOR_BGRA2YUV_YUYV = COLOR_BGRA2YUV_YUY2, //!< synonym to YUY2
    COLOR_RGBA2YUV_YUNV = COLOR_RGBA2YUV_YUY2, //!< synonym to YUY2
    COLOR_BGRA2YUV_YUNV = COLOR_BGRA2YUV_YUY2, //!< synonym to YUY2

    COLOR_COLORCVT_MAX  = 155
};

//! @addtogroup imgproc_shape
//! @{

//! types of intersection between rectangles
enum RectanglesIntersectTypes {
    INTERSECT_NONE = 0, //!< No intersection
    INTERSECT_PARTIAL  = 1, //!< There is a partial intersection
    INTERSECT_FULL  = 2 //!< One of the rectangle is fully enclosed in the other
};

/** types of line
@ingroup imgproc_draw
*/
enum LineTypes {
    FILLED  = -1,
    LINE_4  = 4, //!< 4-connected line
    LINE_8  = 8, //!< 8-connected line
    LINE_AA = 16 //!< antialiased line
};

/** Only a subset of Hershey fonts <https://en.wikipedia.org/wiki/Hershey_fonts> are supported
@ingroup imgproc_draw
*/
enum HersheyFonts {
    FONT_HERSHEY_SIMPLEX        = 0, //!< normal size sans-serif font
    FONT_HERSHEY_PLAIN          = 1, //!< small size sans-serif font
    FONT_HERSHEY_DUPLEX         = 2, //!< normal size sans-serif font (more complex than FONT_HERSHEY_SIMPLEX)
    FONT_HERSHEY_COMPLEX        = 3, //!< normal size serif font
    FONT_HERSHEY_TRIPLEX        = 4, //!< normal size serif font (more complex than FONT_HERSHEY_COMPLEX)
    FONT_HERSHEY_COMPLEX_SMALL  = 5, //!< smaller version of FONT_HERSHEY_COMPLEX
    FONT_HERSHEY_SCRIPT_SIMPLEX = 6, //!< hand-writing style font
    FONT_HERSHEY_SCRIPT_COMPLEX = 7, //!< more complex variant of FONT_HERSHEY_SCRIPT_SIMPLEX
    FONT_ITALIC                 = 16 //!< flag for italic font
};

/** Possible set of marker types used for the cv::drawMarker function
@ingroup imgproc_draw
*/
enum MarkerTypes
{
    MARKER_CROSS = 0,           //!< A crosshair marker shape
    MARKER_TILTED_CROSS = 1,    //!< A 45 degree tilted crosshair marker shape
    MARKER_STAR = 2,            //!< A star marker shape, combination of cross and tilted cross
    MARKER_DIAMOND = 3,         //!< A diamond marker shape
    MARKER_SQUARE = 4,          //!< A square marker shape
    MARKER_TRIANGLE_UP = 5,     //!< An upwards pointing triangle marker shape
    MARKER_TRIANGLE_DOWN = 6    //!< A downwards pointing triangle marker shape
};

/** @brief finds arbitrary template in the grayscale image using Generalized Hough Transform
*/
class CV_EXPORTS_W GeneralizedHough : public Algorithm
{
public:
    //! set template to search
    CV_WRAP virtual void setTemplate(InputArray templ, Point templCenter = Point(-1, -1)) = 0;
    CV_WRAP virtual void setTemplate(InputArray edges, InputArray dx, InputArray dy, Point templCenter = Point(-1, -1)) = 0;

    //! find template on image
    CV_WRAP virtual void detect(InputArray image, OutputArray positions, OutputArray votes = noArray()) = 0;
    CV_WRAP virtual void detect(InputArray edges, InputArray dx, InputArray dy, OutputArray positions, OutputArray votes = noArray()) = 0;

    //! Canny low threshold.
    CV_WRAP virtual void setCannyLowThresh(int cannyLowThresh) = 0;
    CV_WRAP virtual int getCannyLowThresh() const = 0;

    //! Canny high threshold.
    CV_WRAP virtual void setCannyHighThresh(int cannyHighThresh) = 0;
    CV_WRAP virtual int getCannyHighThresh() const = 0;

    //! Minimum distance between the centers of the detected objects.
    CV_WRAP virtual void setMinDist(double minDist) = 0;
    CV_WRAP virtual double getMinDist() const = 0;

    //! Inverse ratio of the accumulator resolution to the image resolution.
    CV_WRAP virtual void setDp(double dp) = 0;
    CV_WRAP virtual double getDp() const = 0;

    //! Maximal size of inner buffers.
    CV_WRAP virtual void setMaxBufferSize(int maxBufferSize) = 0;
    CV_WRAP virtual int getMaxBufferSize() const = 0;
};

/** @brief finds arbitrary template in the grayscale image using Generalized Hough Transform

Detects position only without translation and rotation @cite Ballard1981 .
*/
class CV_EXPORTS_W GeneralizedHoughBallard : public GeneralizedHough
{
public:
    //! R-Table levels.
    CV_WRAP virtual void setLevels(int levels) = 0;
    CV_WRAP virtual int getLevels() const = 0;

    //! The accumulator threshold for the template centers at the detection stage. The smaller it is, the more false positions may be detected.
    CV_WRAP virtual void setVotesThreshold(int votesThreshold) = 0;
    CV_WRAP virtual int getVotesThreshold() const = 0;
};

/** @brief finds arbitrary template in the grayscale image using Generalized Hough Transform

Detects position, translation and rotation @cite Guil1999 .
*/
class CV_EXPORTS_W GeneralizedHoughGuil : public GeneralizedHough
{
public:
    //! Angle difference in degrees between two points in feature.
    CV_WRAP virtual void setXi(double xi) = 0;
    CV_WRAP virtual double getXi() const = 0;

    //! Feature table levels.
    CV_WRAP virtual void setLevels(int levels) = 0;
    CV_WRAP virtual int getLevels() const = 0;

    //! Maximal difference between angles that treated as equal.
    CV_WRAP virtual void setAngleEpsilon(double angleEpsilon) = 0;
    CV_WRAP virtual double getAngleEpsilon() const = 0;

    //! Minimal rotation angle to detect in degrees.
    CV_WRAP virtual void setMinAngle(double minAngle) = 0;
    CV_WRAP virtual double getMinAngle() const = 0;

    //! Maximal rotation angle to detect in degrees.
    CV_WRAP virtual void setMaxAngle(double maxAngle) = 0;
    CV_WRAP virtual double getMaxAngle() const = 0;

    //! Angle step in degrees.
    CV_WRAP virtual void setAngleStep(double angleStep) = 0;
    CV_WRAP virtual double getAngleStep() const = 0;

    //! Angle votes threshold.
    CV_WRAP virtual void setAngleThresh(int angleThresh) = 0;
    CV_WRAP virtual int getAngleThresh() const = 0;

    //! Minimal scale to detect.
    CV_WRAP virtual void setMinScale(double minScale) = 0;
    CV_WRAP virtual double getMinScale() const = 0;

    //! Maximal scale to detect.
    CV_WRAP virtual void setMaxScale(double maxScale) = 0;
    CV_WRAP virtual double getMaxScale() const = 0;

    //! Scale step.
    CV_WRAP virtual void setScaleStep(double scaleStep) = 0;
    CV_WRAP virtual double getScaleStep() const = 0;

    //! Scale votes threshold.
    CV_WRAP virtual void setScaleThresh(int scaleThresh) = 0;
    CV_WRAP virtual int getScaleThresh() const = 0;

    //! Position votes threshold.
    CV_WRAP virtual void setPosThresh(int posThresh) = 0;
    CV_WRAP virtual int getPosThresh() const = 0;
};

//! @} imgproc_shape

//! @addtogroup imgproc_hist
//! @{

/** @brief Base class for Contrast Limited Adaptive Histogram Equalization.
*/
class CV_EXPORTS_W CLAHE : public Algorithm
{
public:
    /** @brief Equalizes the histogram of a grayscale image using Contrast Limited Adaptive Histogram Equalization.

    @param src Source image of type CV_8UC1 or CV_16UC1.
    @param dst Destination image.
     */
    CV_WRAP virtual void apply(InputArray src, OutputArray dst) = 0;

    /** @brief Sets threshold for contrast limiting.

    @param clipLimit threshold value.
    */
    CV_WRAP virtual void setClipLimit(double clipLimit) = 0;

    //! Returns threshold value for contrast limiting.
    CV_WRAP virtual double getClipLimit() const = 0;

    /** @brief Sets size of grid for histogram equalization. Input image will be divided into
    equally sized rectangular tiles.

    @param tileGridSize defines the number of tiles in row and column.
    */
    CV_WRAP virtual void setTilesGridSize(Size tileGridSize) = 0;

    //!@brief Returns Size defines the number of tiles in row and column.
    CV_WRAP virtual Size getTilesGridSize() const = 0;

    CV_WRAP virtual void collectGarbage() = 0;
};

//! @} imgproc_hist

//! @addtogroup imgproc_subdiv2d
//! @{

class CV_EXPORTS_W Subdiv2D
{
public:
    /** Subdiv2D point location cases */
    enum { PTLOC_ERROR        = -2, //!< Point location error
           PTLOC_OUTSIDE_RECT = -1, //!< Point outside the subdivision bounding rect
           PTLOC_INSIDE       = 0, //!< Point inside some facet
           PTLOC_VERTEX       = 1, //!< Point coincides with one of the subdivision vertices
           PTLOC_ON_EDGE      = 2  //!< Point on some edge
         };

    /** Subdiv2D edge type navigation (see: getEdge()) */
    enum { NEXT_AROUND_ORG   = 0x00,
           NEXT_AROUND_DST   = 0x22,
           PREV_AROUND_ORG   = 0x11,
           PREV_AROUND_DST   = 0x33,
           NEXT_AROUND_LEFT  = 0x13,
           NEXT_AROUND_RIGHT = 0x31,
           PREV_AROUND_LEFT  = 0x20,
           PREV_AROUND_RIGHT = 0x02
         };

    /** creates an empty Subdiv2D object.
    To create a new empty Delaunay subdivision you need to use the #initDelaunay function.
     */
    CV_WRAP Subdiv2D();

    /** @overload

    @param rect Rectangle that includes all of the 2D points that are to be added to the subdivision.

    The function creates an empty Delaunay subdivision where 2D points can be added using the function
    insert() . All of the points to be added must be within the specified rectangle, otherwise a runtime
    error is raised.
     */
    CV_WRAP Subdiv2D(Rect rect);

    /** @overload */
    CV_WRAP Subdiv2D(Rect2f rect2f);

    /** @overload

    @brief Creates a new empty Delaunay subdivision

    @param rect Rectangle that includes all of the 2D points that are to be added to the subdivision.

     */
    CV_WRAP void initDelaunay(Rect rect);

    /** @overload

    @brief Creates a new empty Delaunay subdivision

    @param rect Rectangle that includes all of the 2d points that are to be added to the subdivision.

     */
    CV_WRAP_AS(initDelaunay2f) CV_WRAP void initDelaunay(Rect2f rect);

    /** @brief Insert a single point into a Delaunay triangulation.

    @param pt Point to insert.

    The function inserts a single point into a subdivision and modifies the subdivision topology
    appropriately. If a point with the same coordinates exists already, no new point is added.
    @returns the ID of the point.

    @note If the point is outside of the triangulation specified rect a runtime error is raised.
     */
    CV_WRAP int insert(Point2f pt);

    /** @brief Insert multiple points into a Delaunay triangulation.

    @param ptvec Points to insert.

    The function inserts a vector of points into a subdivision and modifies the subdivision topology
    appropriately.
     */
    CV_WRAP void insert(const std::vector<Point2f>& ptvec);

    /** @brief Returns the location of a point within a Delaunay triangulation.

    @param pt Point to locate.
    @param edge Output edge that the point belongs to or is located to the right of it.
    @param vertex Optional output vertex the input point coincides with.

    The function locates the input point within the subdivision and gives one of the triangle edges
    or vertices.

    @returns an integer which specify one of the following five cases for point location:
    -  The point falls into some facet. The function returns #PTLOC_INSIDE and edge will contain one of
       edges of the facet.
    -  The point falls onto the edge. The function returns #PTLOC_ON_EDGE and edge will contain this edge.
    -  The point coincides with one of the subdivision vertices. The function returns #PTLOC_VERTEX and
       vertex will contain a pointer to the vertex.
    -  The point is outside the subdivision reference rectangle. The function returns #PTLOC_OUTSIDE_RECT
       and no pointers are filled.
    -  One of input arguments is invalid. A runtime error is raised or, if silent or "parent" error
       processing mode is selected, #PTLOC_ERROR is returned.
     */
    CV_WRAP int locate(Point2f pt, CV_OUT int& edge, CV_OUT int& vertex);

    /** @brief Finds the subdivision vertex closest to the given point.

    @param pt Input point.
    @param nearestPt Output subdivision vertex point.

    The function is another function that locates the input point within the subdivision. It finds the
    subdivision vertex that is the closest to the input point. It is not necessarily one of vertices
    of the facet containing the input point, though the facet (located using locate() ) is used as a
    starting point.

    @returns vertex ID.
     */
    CV_WRAP int findNearest(Point2f pt, CV_OUT Point2f* nearestPt = 0);

    /** @brief Returns a list of all edges.

    @param edgeList Output vector.

    The function gives each edge as a 4 numbers vector, where each two are one of the edge
    vertices. i.e. org_x = v[0], org_y = v[1], dst_x = v[2], dst_y = v[3].
     */
    CV_WRAP void getEdgeList(CV_OUT std::vector<Vec4f>& edgeList) const;

    /** @brief Returns a list of the leading edge ID connected to each triangle.

    @param leadingEdgeList Output vector.

    The function gives one edge ID for each triangle.
     */
    CV_WRAP void getLeadingEdgeList(CV_OUT std::vector<int>& leadingEdgeList) const;

    /** @brief Returns a list of all triangles.

    @param triangleList Output vector.

    The function gives each triangle as a 6 numbers vector, where each two are one of the triangle
    vertices. i.e. p1_x = v[0], p1_y = v[1], p2_x = v[2], p2_y = v[3], p3_x = v[4], p3_y = v[5].
     */
    CV_WRAP void getTriangleList(CV_OUT std::vector<Vec6f>& triangleList) const;

    /** @brief Returns a list of all Voronoi facets.

    @param idx Vector of vertices IDs to consider. For all vertices you can pass empty vector.
    @param facetList Output vector of the Voronoi facets.
    @param facetCenters Output vector of the Voronoi facets center points.

     */
    CV_WRAP void getVoronoiFacetList(const std::vector<int>& idx, CV_OUT std::vector<std::vector<Point2f> >& facetList,
                                     CV_OUT std::vector<Point2f>& facetCenters);

    /** @brief Returns vertex location from vertex ID.

    @param vertex vertex ID.
    @param firstEdge Optional. The first edge ID which is connected to the vertex.
    @returns vertex (x,y)

     */
    CV_WRAP Point2f getVertex(int vertex, CV_OUT int* firstEdge = 0) const;

    /** @brief Returns one of the edges related to the given edge.

    @param edge Subdivision edge ID.
    @param nextEdgeType Parameter specifying which of the related edges to return.
    The following values are possible:
    -   NEXT_AROUND_ORG next around the edge origin ( eOnext on the picture below if e is the input edge)
    -   NEXT_AROUND_DST next around the edge vertex ( eDnext )
    -   PREV_AROUND_ORG previous around the edge origin (reversed eRnext )
    -   PREV_AROUND_DST previous around the edge destination (reversed eLnext )
    -   NEXT_AROUND_LEFT next around the left facet ( eLnext )
    -   NEXT_AROUND_RIGHT next around the right facet ( eRnext )
    -   PREV_AROUND_LEFT previous around the left facet (reversed eOnext )
    -   PREV_AROUND_RIGHT previous around the right facet (reversed eDnext )

    ![sample output](pics/quadedge.png)

    @returns edge ID related to the input edge.
     */
    CV_WRAP int getEdge( int edge, int nextEdgeType ) const;

    /** @brief Returns next edge around the edge origin.

    @param edge Subdivision edge ID.

    @returns an integer which is next edge ID around the edge origin: eOnext on the
    picture above if e is the input edge).
     */
    CV_WRAP int nextEdge(int edge) const;

    /** @brief Returns another edge of the same quad-edge.

    @param edge Subdivision edge ID.
    @param rotate Parameter specifying which of the edges of the same quad-edge as the input
    one to return. The following values are possible:
    -   0 - the input edge ( e on the picture below if e is the input edge)
    -   1 - the rotated edge ( eRot )
    -   2 - the reversed edge (reversed e (in green))
    -   3 - the reversed rotated edge (reversed eRot (in green))

    @returns one of the edges ID of the same quad-edge as the input edge.
     */
    CV_WRAP int rotateEdge(int edge, int rotate) const;
    CV_WRAP int symEdge(int edge) const;

    /** @brief Returns the edge origin.

    @param edge Subdivision edge ID.
    @param orgpt Output vertex location.

    @returns vertex ID.
     */
    CV_WRAP int edgeOrg(int edge, CV_OUT Point2f* orgpt = 0) const;

    /** @brief Returns the edge destination.

    @param edge Subdivision edge ID.
    @param dstpt Output vertex location.

    @returns vertex ID.
     */
    CV_WRAP int edgeDst(int edge, CV_OUT Point2f* dstpt = 0) const;

protected:
    int newEdge();
    void deleteEdge(int edge);
    int newPoint(Point2f pt, bool isvirtual, int firstEdge = 0);
    void deletePoint(int vtx);
    void setEdgePoints( int edge, int orgPt, int dstPt );
    void splice( int edgeA, int edgeB );
    int connectEdges( int edgeA, int edgeB );
    void swapEdges( int edge );
    int isRightOf(Point2f pt, int edge) const;
    void calcVoronoi();
    void clearVoronoi();
    void checkSubdiv() const;

    struct CV_EXPORTS Vertex
    {
        Vertex();
        Vertex(Point2f pt, bool isvirtual, int firstEdge=0);
        bool isvirtual() const;
        bool isfree() const;

        int firstEdge;
        int type;
        Point2f pt;
    };

    struct CV_EXPORTS QuadEdge
    {
        QuadEdge();
        QuadEdge(int edgeidx);
        bool isfree() const;

        int next[4];
        int pt[4];
    };

    //! All of the vertices
    std::vector<Vertex> vtx;
    //! All of the edges
    std::vector<QuadEdge> qedges;
    int freeQEdge;
    int freePoint;
    bool validGeometry;

    int recentEdge;
    //! Top left corner of the bounding rect
    Point2f topLeft;
    //! Bottom right corner of the bounding rect
    Point2f bottomRight;
};

//! @} imgproc_subdiv2d

//! @addtogroup imgproc_feature
//! @{

/** @example samples/cpp/lsd_lines.cpp
An example using the LineSegmentDetector
\image html building_lsd.png "Sample output image" width=434 height=300
*/

/** @brief Line segment detector class

following the algorithm described at @cite Rafael12 .

@note Implementation has been removed from OpenCV version 3.4.6 to 3.4.15 and version 4.1.0 to 4.5.3 due original code license conflict.
restored again after [Computation of a NFA](https://github.com/rafael-grompone-von-gioi/binomial_nfa) code published under the MIT license.
*/
class CV_EXPORTS_W LineSegmentDetector : public Algorithm
{
public:

    /** @brief Finds lines in the input image.

    This is the output of the default parameters of the algorithm on the above shown image.

    ![image](pics/building_lsd.png)

    @param image A grayscale (CV_8UC1) input image. If only a roi needs to be selected, use:
    `lsd_ptr-\>detect(image(roi), lines, ...); lines += Scalar(roi.x, roi.y, roi.x, roi.y);`
    @param lines A vector of Vec4f elements specifying the beginning and ending point of a line. Where
    Vec4f is (x1, y1, x2, y2), point 1 is the start, point 2 - end. Returned lines are strictly
    oriented depending on the gradient.
    @param width Vector of widths of the regions, where the lines are found. E.g. Width of line.
    @param prec Vector of precisions with which the lines are found.
    @param nfa Vector containing number of false alarms in the line region, with precision of 10%. The
    bigger the value, logarithmically better the detection.
    - -1 corresponds to 10 mean false alarms
    - 0 corresponds to 1 mean false alarm
    - 1 corresponds to 0.1 mean false alarms
    This vector will be calculated only when the objects type is #LSD_REFINE_ADV.
    */
    CV_WRAP virtual void detect(InputArray image, OutputArray lines,
                        OutputArray width = noArray(), OutputArray prec = noArray(),
                        OutputArray nfa = noArray()) = 0;

    /** @brief Draws the line segments on a given image.
    @param image The image, where the lines will be drawn. Should be bigger or equal to the image,
    where the lines were found.
    @param lines A vector of the lines that needed to be drawn.
     */
    CV_WRAP virtual void drawSegments(InputOutputArray image, InputArray lines) = 0;

    /** @brief Draws two groups of lines in blue and red, counting the non overlapping (mismatching) pixels.

    @param size The size of the image, where lines1 and lines2 were found.
    @param lines1 The first group of lines that needs to be drawn. It is visualized in blue color.
    @param lines2 The second group of lines. They visualized in red color.
    @param image Optional image, where the lines will be drawn. The image should be color(3-channel)
    in order for lines1 and lines2 to be drawn in the above mentioned colors.
     */
    CV_WRAP virtual int compareSegments(const Size& size, InputArray lines1, InputArray lines2, InputOutputArray image = noArray()) = 0;

    virtual ~LineSegmentDetector() { }
};

/** @brief Creates a smart pointer to a LineSegmentDetector object and initializes it.

The LineSegmentDetector algorithm is defined using the standard values. Only advanced users may want
to edit those, as to tailor it for their own application.

@param refine The way found lines will be refined, see #LineSegmentDetectorModes
@param scale The scale of the image that will be used to find the lines. Range (0..1].
@param sigma_scale Sigma for Gaussian filter. It is computed as sigma = sigma_scale/scale.
@param quant Bound to the quantization error on the gradient norm.
@param ang_th Gradient angle tolerance in degrees.
@param log_eps Detection threshold: -log10(NFA) \> log_eps. Used only when advance refinement is chosen.
@param density_th Minimal density of aligned region points in the enclosing rectangle.
@param n_bins Number of bins in pseudo-ordering of gradient modulus.
 */
CV_EXPORTS_W Ptr<LineSegmentDetector> createLineSegmentDetector(
    int refine = LSD_REFINE_STD, double scale = 0.8,
    double sigma_scale = 0.6, double quant = 2.0, double ang_th = 22.5,
    double log_eps = 0, double density_th = 0.7, int n_bins = 1024);

//! @} imgproc_feature

//! @addtogroup imgproc_filter
//! @{

/** @brief Returns Gaussian filter coefficients.

The function computes and returns the \f$\texttt{ksize} \times 1\f$ matrix of Gaussian filter
coefficients:

\f[G_i= \alpha *e^{-(i-( \texttt{ksize} -1)/2)^2/(2* \texttt{sigma}^2)},\f]

where \f$i=0..\texttt{ksize}-1\f$ and \f$\alpha\f$ is the scale factor chosen so that \f$\sum_i G_i=1\f$.

Two of such generated kernels can be passed to sepFilter2D. Those functions automatically recognize
smoothing kernels (a symmetrical kernel with sum of weights equal to 1) and handle them accordingly.
You may also use the higher-level GaussianBlur.
@param ksize Aperture size. It should be odd ( \f$\texttt{ksize} \mod 2 = 1\f$ ) and positive.
@param sigma Gaussian standard deviation. If it is non-positive, it is computed from ksize as
`sigma = 0.3*((ksize-1)*0.5 - 1) + 0.8`.
@param ktype Type of filter coefficients. It can be CV_32F or CV_64F .
@sa  sepFilter2D, getDerivKernels, getStructuringElement, GaussianBlur
 */
CV_EXPORTS_W Mat getGaussianKernel( int ksize, double sigma, int ktype = CV_64F );

/** @brief Returns filter coefficients for computing spatial image derivatives.

The function computes and returns the filter coefficients for spatial image derivatives. When
`ksize=FILTER_SCHARR`, the Scharr \f$3 \times 3\f$ kernels are generated (see #Scharr). Otherwise, Sobel
kernels are generated (see #Sobel). The filters are normally passed to #sepFilter2D or to

@param kx Output matrix of row filter coefficients. It has the type ktype .
@param ky Output matrix of column filter coefficients. It has the type ktype .
@param dx Derivative order in respect of x.
@param dy Derivative order in respect of y.
@param ksize Aperture size. It can be FILTER_SCHARR, 1, 3, 5, or 7.
@param normalize Flag indicating whether to normalize (scale down) the filter coefficients or not.
Theoretically, the coefficients should have the denominator \f$=2^{ksize*2-dx-dy-2}\f$. If you are
going to filter floating-point images, you are likely to use the normalized kernels. But if you
compute derivatives of an 8-bit image, store the results in a 16-bit image, and wish to preserve
all the fractional bits, you may want to set normalize=false .
@param ktype Type of filter coefficients. It can be CV_32f or CV_64F .
 */
CV_EXPORTS_W void getDerivKernels( OutputArray kx, OutputArray ky,
                                   int dx, int dy, int ksize,
                                   bool normalize = false, int ktype = CV_32F );

/** @brief Returns Gabor filter coefficients.

For more details about gabor filter equations and parameters, see: [Gabor
Filter](https://en.wikipedia.org/wiki/Gabor_filter).

@param ksize Size of the filter returned.
@param sigma Standard deviation of the gaussian envelope.
@param theta Orientation of the normal to the parallel stripes of a Gabor function.
@param lambd Wavelength of the sinusoidal factor.
@param gamma Spatial aspect ratio.
@param psi Phase offset.
@param ktype Type of filter coefficients. It can be CV_32F or CV_64F .
 */
CV_EXPORTS_W Mat getGaborKernel( Size ksize, double sigma, double theta, double lambd,
                                 double gamma, double psi = CV_PI*0.5, int ktype = CV_64F );

//! returns "magic" border value for erosion and dilation. It is automatically transformed to Scalar::all(-DBL_MAX) for dilation.
static inline Scalar morphologyDefaultBorderValue() { return Scalar::all(DBL_MAX); }

/** @brief Returns a structuring element of the specified size and shape for morphological operations.

The function constructs and returns the structuring element that can be further passed to #erode,
#dilate or #morphologyEx. But you can also construct an arbitrary binary mask yourself and use it as
the structuring element.

@param shape Element shape that could be one of #MorphShapes
@param ksize Size of the structuring element.
@param anchor Anchor position within the element. The default value \f$(-1, -1)\f$ means that the
anchor is at the center. Note that only the shape of a cross-shaped element depends on the anchor
position. In other cases the anchor just regulates how much the result of the morphological
operation is shifted.
 */
CV_EXPORTS_W Mat getStructuringElement(int shape, Size ksize, Point anchor = Point(-1,-1));

/** @example samples/cpp/tutorial_code/ImgProc/Smoothing/Smoothing.cpp
Sample code for simple filters
![Sample screenshot](Smoothing_Tutorial_Result_Median_Filter.jpg)
Check @ref tutorial_gausian_median_blur_bilateral_filter "the corresponding tutorial" for more details
 */

/** @brief Blurs an image using the median filter.

The function smoothes an image using the median filter with the \f$\texttt{ksize} \times
\texttt{ksize}\f$ aperture. Each channel of a multi-channel image is processed independently.
In-place operation is supported.

@note The median filter uses #BORDER_REPLICATE internally to cope with border pixels, see #BorderTypes

@param src input 1-, 3-, or 4-channel image; when ksize is 3 or 5, the image depth should be
CV_8U, CV_16U, or CV_32F, for larger aperture sizes, it can only be CV_8U.
@param dst destination array of the same size and type as src.
@param ksize aperture linear size; it must be odd and greater than 1, for example: 3, 5, 7 ...
@sa  bilateralFilter, blur, boxFilter, GaussianBlur
 */
CV_EXPORTS_W void medianBlur( InputArray src, OutputArray dst, int ksize );

/** @brief Blurs an image using a Gaussian filter.

The function convolves the source image with the specified Gaussian kernel. In-place filtering is
supported.

@param src input image; the image can have any number of channels, which are processed
independently, but the depth should be CV_8U, CV_16U, CV_16S, CV_32F or CV_64F.
@param dst output image of the same size and type as src.
@param ksize Gaussian kernel size. ksize.width and ksize.height can differ but they both must be
positive and odd. Or, they can be zero's and then they are computed from sigma.
@param sigmaX Gaussian kernel standard deviation in X direction.
@param sigmaY Gaussian kernel standard deviation in Y direction; if sigmaY is zero, it is set to be
equal to sigmaX, if both sigmas are zeros, they are computed from ksize.width and ksize.height,
respectively (see #getGaussianKernel for details); to fully control the result regardless of
possible future modifications of all this semantics, it is recommended to specify all of ksize,
sigmaX, and sigmaY.
@param borderType pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@param hint Implementation modfication flags. See #AlgorithmHint

@sa  sepFilter2D, filter2D, blur, boxFilter, bilateralFilter, medianBlur
 */
CV_EXPORTS_W void GaussianBlur( InputArray src, OutputArray dst, Size ksize,
                                double sigmaX, double sigmaY = 0,
                                int borderType = BORDER_DEFAULT,
                                AlgorithmHint hint = cv::ALGO_HINT_DEFAULT );

/** @brief Applies the bilateral filter to an image.

The function applies bilateral filtering to the input image, as described in
https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/MANDUCHI1/Bilateral_Filtering.html
bilateralFilter can reduce unwanted noise very well while keeping edges fairly sharp. However, it is
very slow compared to most filters.

_Sigma values_: For simplicity, you can set the 2 sigma values to be the same. If they are small (\<
10), the filter will not have much effect, whereas if they are large (\> 150), they will have a very
strong effect, making the image look "cartoonish".

_Filter size_: Large filters (d \> 5) are very slow, so it is recommended to use d=5 for real-time
applications, and perhaps d=9 for offline applications that need heavy noise filtering.

This filter does not work inplace.
@param src Source 8-bit or floating-point, 1-channel or 3-channel image.
@param dst Destination image of the same size and type as src .
@param d Diameter of each pixel neighborhood that is used during filtering. If it is non-positive,
it is computed from sigmaSpace.
@param sigmaColor Filter sigma in the color space. A larger value of the parameter means that
farther colors within the pixel neighborhood (see sigmaSpace) will be mixed together, resulting
in larger areas of semi-equal color.
@param sigmaSpace Filter sigma in the coordinate space. A larger value of the parameter means that
farther pixels will influence each other as long as their colors are close enough (see sigmaColor
). When d\>0, it specifies the neighborhood size regardless of sigmaSpace. Otherwise, d is
proportional to sigmaSpace.
@param borderType border mode used to extrapolate pixels outside of the image, see #BorderTypes
 */
CV_EXPORTS_W void bilateralFilter( InputArray src, OutputArray dst, int d,
                                   double sigmaColor, double sigmaSpace,
                                   int borderType = BORDER_DEFAULT );

/** @brief Blurs an image using the box filter.

The function smooths an image using the kernel:

\f[\texttt{K} =  \alpha \begin{bmatrix} 1 & 1 & 1 &  \cdots & 1 & 1  \\ 1 & 1 & 1 &  \cdots & 1 & 1  \\ \hdotsfor{6} \\ 1 & 1 & 1 &  \cdots & 1 & 1 \end{bmatrix}\f]

where

\f[\alpha = \begin{cases} \frac{1}{\texttt{ksize.width*ksize.height}} & \texttt{when } \texttt{normalize=true}  \\1 & \texttt{otherwise}\end{cases}\f]

Unnormalized box filter is useful for computing various integral characteristics over each pixel
neighborhood, such as covariance matrices of image derivatives (used in dense optical flow
algorithms, and so on). If you need to compute pixel sums over variable-size windows, use #integral.

@param src input image.
@param dst output image of the same size and type as src.
@param ddepth the output image depth (-1 to use src.depth()).
@param ksize blurring kernel size.
@param anchor anchor point; default value Point(-1,-1) means that the anchor is at the kernel
center.
@param normalize flag, specifying whether the kernel is normalized by its area or not.
@param borderType border mode used to extrapolate pixels outside of the image, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  blur, bilateralFilter, GaussianBlur, medianBlur, integral
 */
CV_EXPORTS_W void boxFilter( InputArray src, OutputArray dst, int ddepth,
                             Size ksize, Point anchor = Point(-1,-1),
                             bool normalize = true,
                             int borderType = BORDER_DEFAULT );

/** @brief Calculates the normalized sum of squares of the pixel values overlapping the filter.

For every pixel \f$ (x, y) \f$ in the source image, the function calculates the sum of squares of those neighboring
pixel values which overlap the filter placed over the pixel \f$ (x, y) \f$.

The unnormalized square box filter can be useful in computing local image statistics such as the local
variance and standard deviation around the neighborhood of a pixel.

@param src input image
@param dst output image of the same size and type as src
@param ddepth the output image depth (-1 to use src.depth())
@param ksize kernel size
@param anchor kernel anchor point. The default value of Point(-1, -1) denotes that the anchor is at the kernel
center.
@param normalize flag, specifying whether the kernel is to be normalized by it's area or not.
@param borderType border mode used to extrapolate pixels outside of the image, see #BorderTypes. #BORDER_WRAP is not supported.
@sa boxFilter
*/
CV_EXPORTS_W void sqrBoxFilter( InputArray src, OutputArray dst, int ddepth,
                                Size ksize, Point anchor = Point(-1, -1),
                                bool normalize = true,
                                int borderType = BORDER_DEFAULT );

/** @brief Blurs an image using the normalized box filter.

The function smooths an image using the kernel:

\f[\texttt{K} =  \frac{1}{\texttt{ksize.width*ksize.height}} \begin{bmatrix} 1 & 1 & 1 &  \cdots & 1 & 1  \\ 1 & 1 & 1 &  \cdots & 1 & 1  \\ \hdotsfor{6} \\ 1 & 1 & 1 &  \cdots & 1 & 1  \\ \end{bmatrix}\f]

The call `blur(src, dst, ksize, anchor, borderType)` is equivalent to `boxFilter(src, dst, src.type(), ksize,
anchor, true, borderType)`.

@param src input image; it can have any number of channels, which are processed independently, but
the depth should be CV_8U, CV_16U, CV_16S, CV_32F or CV_64F.
@param dst output image of the same size and type as src.
@param ksize blurring kernel size.
@param anchor anchor point; default value Point(-1,-1) means that the anchor is at the kernel
center.
@param borderType border mode used to extrapolate pixels outside of the image, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  boxFilter, bilateralFilter, GaussianBlur, medianBlur
 */
CV_EXPORTS_W void blur( InputArray src, OutputArray dst,
                        Size ksize, Point anchor = Point(-1,-1),
                        int borderType = BORDER_DEFAULT );

/** @brief Blurs an image using the stackBlur.

The function applies and stackBlur to an image.
stackBlur can generate similar results as Gaussian blur, and the time consumption does not increase with the increase of kernel size.
It creates a kind of moving stack of colors whilst scanning through the image. Thereby it just has to add one new block of color to the right side
of the stack and remove the leftmost color. The remaining colors on the topmost layer of the stack are either added on or reduced by one,
depending on if they are on the right or on the left side of the stack. The only supported borderType is BORDER_REPLICATE.
Original paper was proposed by Mario Klingemann, which can be found https://underdestruction.com/2004/02/25/stackblur-2004.

@param src input image. The number of channels can be arbitrary, but the depth should be one of
CV_8U, CV_16U, CV_16S or CV_32F.
@param dst output image of the same size and type as src.
@param ksize stack-blurring kernel size. The ksize.width and ksize.height can differ but they both must be
positive and odd.
*/
CV_EXPORTS_W void stackBlur(InputArray src, OutputArray dst, Size ksize);

/** @brief Convolves an image with the kernel.

The function applies an arbitrary linear filter to an image. In-place operation is supported. When
the aperture is partially outside the image, the function interpolates outlier pixel values
according to the specified border mode.

The function does actually compute correlation, not the convolution:

\f[\texttt{dst} (x,y) =  \sum _{ \substack{0\leq x' < \texttt{kernel.cols}\\{0\leq y' < \texttt{kernel.rows}}}}  \texttt{kernel} (x',y')* \texttt{src} (x+x'- \texttt{anchor.x} ,y+y'- \texttt{anchor.y} )\f]

That is, the kernel is not mirrored around the anchor point. If you need a real convolution, flip
the kernel using #flip and set the new anchor to `(kernel.cols - anchor.x - 1, kernel.rows -
anchor.y - 1)`.

The function uses the DFT-based algorithm in case of sufficiently large kernels (~`11 x 11` or
larger) and the direct algorithm for small kernels.

@param src input image.
@param dst output image of the same size and the same number of channels as src.
@param ddepth desired depth of the destination image, see @ref filter_depths "combinations"
@param kernel convolution kernel (or rather a correlation kernel), a single-channel floating point
matrix; if you want to apply different kernels to different channels, split the image into
separate color planes using split and process them individually.
@param anchor anchor of the kernel that indicates the relative position of a filtered point within
the kernel; the anchor should lie within the kernel; default value (-1,-1) means that the anchor
is at the kernel center.
@param delta optional value added to the filtered pixels before storing them in dst.
@param borderType pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  sepFilter2D, dft, matchTemplate
 */
CV_EXPORTS_W void filter2D( InputArray src, OutputArray dst, int ddepth,
                            InputArray kernel, Point anchor = Point(-1,-1),
                            double delta = 0, int borderType = BORDER_DEFAULT );

/** @brief Applies a separable linear filter to an image.

The function applies a separable linear filter to the image. That is, first, every row of src is
filtered with the 1D kernel kernelX. Then, every column of the result is filtered with the 1D
kernel kernelY. The final result shifted by delta is stored in dst .

@param src Source image.
@param dst Destination image of the same size and the same number of channels as src .
@param ddepth Destination image depth, see @ref filter_depths "combinations"
@param kernelX Coefficients for filtering each row.
@param kernelY Coefficients for filtering each column.
@param anchor Anchor position within the kernel. The default value \f$(-1,-1)\f$ means that the anchor
is at the kernel center.
@param delta Value added to the filtered results before storing them.
@param borderType Pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  filter2D, Sobel, GaussianBlur, boxFilter, blur
 */
CV_EXPORTS_W void sepFilter2D( InputArray src, OutputArray dst, int ddepth,
                               InputArray kernelX, InputArray kernelY,
                               Point anchor = Point(-1,-1),
                               double delta = 0, int borderType = BORDER_DEFAULT );

/** @example samples/cpp/tutorial_code/ImgTrans/Sobel_Demo.cpp
Sample code using Sobel and/or Scharr OpenCV functions to make a simple Edge Detector
![Sample screenshot](Sobel_Derivatives_Tutorial_Result.jpg)
Check @ref tutorial_sobel_derivatives "the corresponding tutorial" for more details
*/

/** @brief Calculates the first, second, third, or mixed image derivatives using an extended Sobel operator.

In all cases except one, the \f$\texttt{ksize} \times \texttt{ksize}\f$ separable kernel is used to
calculate the derivative. When \f$\texttt{ksize = 1}\f$, the \f$3 \times 1\f$ or \f$1 \times 3\f$
kernel is used (that is, no Gaussian smoothing is done). `ksize = 1` can only be used for the first
or the second x- or y- derivatives.

There is also the special value `ksize = #FILTER_SCHARR (-1)` that corresponds to the \f$3\times3\f$ Scharr
filter that may give more accurate results than the \f$3\times3\f$ Sobel. The Scharr aperture is

\f[\vecthreethree{-3}{0}{3}{-10}{0}{10}{-3}{0}{3}\f]

for the x-derivative, or transposed for the y-derivative.

The function calculates an image derivative by convolving the image with the appropriate kernel:

\f[\texttt{dst} =  \frac{\partial^{xorder+yorder} \texttt{src}}{\partial x^{xorder} \partial y^{yorder}}\f]

The Sobel operators combine Gaussian smoothing and differentiation, so the result is more or less
resistant to the noise. Most often, the function is called with ( xorder = 1, yorder = 0, ksize = 3)
or ( xorder = 0, yorder = 1, ksize = 3) to calculate the first x- or y- image derivative. The first
case corresponds to a kernel of:

\f[\vecthreethree{-1}{0}{1}{-2}{0}{2}{-1}{0}{1}\f]

The second case corresponds to a kernel of:

\f[\vecthreethree{-1}{-2}{-1}{0}{0}{0}{1}{2}{1}\f]

@param src input image.
@param dst output image of the same size and the same number of channels as src .
@param ddepth output image depth, see @ref filter_depths "combinations"; in the case of
    8-bit input images it will result in truncated derivatives.
@param dx order of the derivative x.
@param dy order of the derivative y.
@param ksize size of the extended Sobel kernel; it must be 1, 3, 5, or 7.
@param scale optional scale factor for the computed derivative values; by default, no scaling is
applied (see #getDerivKernels for details).
@param delta optional delta value that is added to the results prior to storing them in dst.
@param borderType pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  Scharr, Laplacian, sepFilter2D, filter2D, GaussianBlur, cartToPolar
 */
CV_EXPORTS_W void Sobel( InputArray src, OutputArray dst, int ddepth,
                         int dx, int dy, int ksize = 3,
                         double scale = 1, double delta = 0,
                         int borderType = BORDER_DEFAULT );

/** @brief Calculates the first order image derivative in both x and y using a Sobel operator

Equivalent to calling:

@code
Sobel( src, dx, CV_16SC1, 1, 0, 3 );
Sobel( src, dy, CV_16SC1, 0, 1, 3 );
@endcode

@param src input image.
@param dx output image with first-order derivative in x.
@param dy output image with first-order derivative in y.
@param ksize size of Sobel kernel. It must be 3.
@param borderType pixel extrapolation method, see #BorderTypes.
                  Only #BORDER_DEFAULT=#BORDER_REFLECT_101 and #BORDER_REPLICATE are supported.

@sa Sobel
 */

CV_EXPORTS_W void spatialGradient( InputArray src, OutputArray dx,
                                   OutputArray dy, int ksize = 3,
                                   int borderType = BORDER_DEFAULT );

/** @brief Calculates the first x- or y- image derivative using Scharr operator.

The function computes the first x- or y- spatial image derivative using the Scharr operator. The
call

\f[\texttt{Scharr(src, dst, ddepth, dx, dy, scale, delta, borderType)}\f]

is equivalent to

\f[\texttt{Sobel(src, dst, ddepth, dx, dy, FILTER_SCHARR, scale, delta, borderType)} .\f]

@param src input image.
@param dst output image of the same size and the same number of channels as src.
@param ddepth output image depth, see @ref filter_depths "combinations"
@param dx order of the derivative x.
@param dy order of the derivative y.
@param scale optional scale factor for the computed derivative values; by default, no scaling is
applied (see #getDerivKernels for details).
@param delta optional delta value that is added to the results prior to storing them in dst.
@param borderType pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  cartToPolar
 */
CV_EXPORTS_W void Scharr( InputArray src, OutputArray dst, int ddepth,
                          int dx, int dy, double scale = 1, double delta = 0,
                          int borderType = BORDER_DEFAULT );

/** @example samples/cpp/laplace.cpp
An example using Laplace transformations for edge detection
*/

/** @brief Calculates the Laplacian of an image.

The function calculates the Laplacian of the source image by adding up the second x and y
derivatives calculated using the Sobel operator:

\f[\texttt{dst} =  \Delta \texttt{src} =  \frac{\partial^2 \texttt{src}}{\partial x^2} +  \frac{\partial^2 \texttt{src}}{\partial y^2}\f]

This is done when `ksize > 1`. When `ksize == 1`, the Laplacian is computed by filtering the image
with the following \f$3 \times 3\f$ aperture:

\f[\vecthreethree {0}{1}{0}{1}{-4}{1}{0}{1}{0}\f]

@param src Source image.
@param dst Destination image of the same size and the same number of channels as src .
@param ddepth Desired depth of the destination image, see @ref filter_depths "combinations".
@param ksize Aperture size used to compute the second-derivative filters. See #getDerivKernels for
details. The size must be positive and odd.
@param scale Optional scale factor for the computed Laplacian values. By default, no scaling is
applied. See #getDerivKernels for details.
@param delta Optional delta value that is added to the results prior to storing them in dst .
@param borderType Pixel extrapolation method, see #BorderTypes. #BORDER_WRAP is not supported.
@sa  Sobel, Scharr
 */
CV_EXPORTS_W void Laplacian( InputArray src, OutputArray dst, int ddepth,
                             int ksize = 1, double scale = 1, double delta = 0,
                             int borderType = BORDER_DEFAULT );

//! @} imgproc_filter

//! @addtogroup imgproc_feature
//! @{

/** @example samples/cpp/edge.cpp
This program demonstrates usage of the Canny edge detector

Check @ref tutorial_canny_detector "the corresponding tutorial" for more details
*/

/** @brief Finds edges in an image using the Canny algorithm @cite Canny86 .

The function finds edges in the input image and marks them in the output map edges using the
Canny algorithm. The smallest value between threshold1 and threshold2 is used for edge linking. The
largest value is used to find initial segments of strong edges. See
<https://en.wikipedia.org/wiki/Canny_edge_detector>

@param image 8-bit input image.
@param edges output edge map; single channels 8-bit image, which has the same size as image .
@param threshold1 first threshold for the hysteresis procedure.
@param threshold2 second threshold for the hysteresis procedure.
@param apertureSize aperture size for the Sobel operator.
@param L2gradient a flag, indicating whether a more accurate \f$L_2\f$ norm
\f$=\sqrt{(dI/dx)^2 + (dI/dy)^2}\f$ should be used to calculate the image gradient magnitude (
L2gradient=true ), or whether the default \f$L_1\f$ norm \f$=|dI/dx|+|dI/dy|\f$ is enough (
L2gradient=false ).
 */
CV_EXPORTS_W void Canny( InputArray image, OutputArray edges,
                         double threshold1, double threshold2,
                         int apertureSize = 3, bool L2gradient = false );

/** \overload

Finds edges in an image using the Canny algorithm with custom image gradient.

@param dx 16-bit x derivative of input image (CV_16SC1 or CV_16SC3).
@param dy 16-bit y derivative of input image (same type as dx).
@param edges output edge map; single channels 8-bit image, which has the same size as image .
@param threshold1 first threshold for the hysteresis procedure.
@param threshold2 second threshold for the hysteresis procedure.
@param L2gradient a flag, indicating whether a more accurate \f$L_2\f$ norm
\f$=\sqrt{(dI/dx)^2 + (dI/dy)^2}\f$ should be used to calculate the image gradient magnitude (
L2gradient=true ), or whether the default \f$L_1\f$ norm \f$=|dI/dx|+|dI/dy|\f$ is enough (
L2gradient=false ).
 */
CV_EXPORTS_W void Canny( InputArray dx, InputArray dy,
                         OutputArray edges,
                         double threshold1, double threshold2,
                         bool L2gradient = false );

/** @brief Calculates the minimal eigenvalue of gradient matrices for corner detection.

The function is similar to cornerEigenValsAndVecs but it calculates and stores only the minimal
eigenvalue of the covariance matrix of derivatives, that is, \f$\min(\lambda_1, \lambda_2)\f$ in terms
of the formulae in the cornerEigenValsAndVecs description.

@param src Input single-channel 8-bit or floating-point image.
@param dst Image to store the minimal eigenvalues. It has the type CV_32FC1 and the same size as
src .
@param blockSize Neighborhood size (see the details on #cornerEigenValsAndVecs ).
@param ksize Aperture parameter for the Sobel operator.
@param borderType Pixel extrapolation method. See #BorderTypes. #BORDER_WRAP is not supported.
 */
CV_EXPORTS_W void cornerMinEigenVal( InputArray src, OutputArray dst,
                                     int blockSize, int ksize = 3,
                                     int borderType = BORDER_DEFAULT );

/** @brief Harris corner detector.

The function runs the Harris corner detector on the image. Similarly to cornerMinEigenVal and
cornerEigenValsAndVecs , for each pixel \f$(x, y)\f$ it calculates a \f$2\times2\f$ gradient covariance
matrix \f$M^{(x,y)}\f$ over a \f$\texttt{blockSize} \times \texttt{blockSize}\f$ neighborhood. Then, it
computes the following characteristic:

\f[\texttt{dst} (x,y) =  \mathrm{det} M^{(x,y)} - k  \cdot \left ( \mathrm{tr} M^{(x,y)} \right )^2\f]

Corners in the image can be found as the local maxima of this response map.

@param src Input single-channel 8-bit or floating-point image.
@param dst Image to store the Harris detector responses. It has the type CV_32FC1 and the same
size as src .
@param blockSize Neighborhood size (see the details on #cornerEigenValsAndVecs ).
@param ksize Aperture parameter for the Sobel operator.
@param k Harris detector free parameter. See the formula above.
@param borderType Pixel extrapolation method. See #BorderTypes. #BORDER_WRAP is not supported.
 */
CV_EXPORTS_W void cornerHarris( InputArray src, OutputArray dst, int blockSize,
                                int ksize, double k,
                                int borderType = BORDER_DEFAULT );

/*
... [Content truncated - file is very large]
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

- **CV_EXPORTS_W**: A class/struct defined in this file
- **CV_EXPORTS**: A class/struct defined in this file
- **each**: A class/struct defined in this file
- **the**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **described**: A class/struct defined in this file
- **an**: A class/struct defined in this file
- **Interface**: A class/struct defined in this file
- **LineIterator**: A class/struct defined in this file
- **following**: A class/struct defined in this file
- **of**: A class/struct defined in this file
- **and**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_IMGPROC_HPP()**: A function/method defined in this file
- **initializes()**: A function/method defined in this file
- **computes()**: A function/method defined in this file
- **converts()**: A function/method defined in this file
- **interpolates()**: A function/method defined in this file
- **finds()**: A function/method defined in this file
- **iterates()**: A function/method defined in this file
- **calculates()**: A function/method defined in this file
- **variants()**: A function/method defined in this file
- **CV_DOXYGEN()**: A function/method defined in this file
- **returns()**: A function/method defined in this file
- **can()**: A function/method defined in this file
- **locates()**: A function/method defined in this file
- **parameters()**: A function/method defined in this file
- **supports()**: A function/method defined in this file
- **smooths()**: A function/method defined in this file
- **this()**: A function/method defined in this file
- **runs()**: A function/method defined in this file
- **produces()**: A function/method defined in this file
- **is()**: A function/method defined in this file
- **cannot()**: A function/method defined in this file
- **smoothes()**: A function/method defined in this file
- **cv()**: A function/method defined in this file
- **uses()**: A function/method defined in this file
- **detects()**: A function/method defined in this file
- **insert()**: A function/method defined in this file
- **constructs()**: A function/method defined in this file
- **warpAffine()**: A function/method defined in this file
- **performs()**: A function/method defined in this file
- **gives()**: A function/method defined in this file
- **that()**: A function/method defined in this file
- **cornerEigenValsAndVecs()**: A function/method defined in this file
- **implements()**: A function/method defined in this file
- **warpPerspective()**: A function/method defined in this file
- **does()**: A function/method defined in this file
- **creates()**: A function/method defined in this file
- **convolves()**: A function/method defined in this file
- **remap()**: A function/method defined in this file
- **adds()**: A function/method defined in this file
- **of()**: A function/method defined in this file
- **by()**: A function/method defined in this file
- **emulates()**: A function/method defined in this file
- **resize()**: A function/method defined in this file
- **erodes()**: A function/method defined in this file
- **as()**: A function/method defined in this file
- **compute()**: A function/method defined in this file
- **response()**: A function/method defined in this file
- **dilates()**: A function/method defined in this file
- **getRectSubPix()**: A function/method defined in this file
- **inserts()**: A function/method defined in this file
- **applies()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `./imgproc/segmentation.hpp`

**Python Imports:**
- `four`
- `that.`
- `one`
- `sigmaSpace.`
- `several`
- `images`
- `this`
- `polar`
- `ksize`
- `floating`
- `YUV`
- `RGB`
- `BGR`
- `OpenCV`
- `command`
- `a`
- `every`
- `semilog`
- `destination`
- `the`
- `where`
- `an`
- `ksize.width`
- `src.size`
- `three`
- `src`
- `vertex`
- `0`
- `sigma.`


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

