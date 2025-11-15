# Documentation for `modules/calib3d/include/opencv2/calib3d.hpp`

## File Metadata

- **Full Path**: `modules/calib3d/include/opencv2/calib3d.hpp`
- **File Name**: `calib3d.hpp`
- **File Size**: 241,703 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/calib3d/include/opencv2/calib3d.hpp](../../../../modules/calib3d/include/opencv2/calib3d.hpp)

## Purpose and Role

This file is located in the `modules/calib3d/include/opencv2` directory and serves as part of the OpenCV library infrastructure.

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
//                          License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
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

#ifndef OPENCV_CALIB3D_HPP
#define OPENCV_CALIB3D_HPP

#include "opencv2/core.hpp"
#include "opencv2/core/types.hpp"
#include "opencv2/features2d.hpp"
#include "opencv2/core/affine.hpp"
#include "opencv2/core/utils/logger.hpp"

/**
  @defgroup calib3d Camera Calibration and 3D Reconstruction

The functions in this section use a so-called pinhole camera model. The view of a scene
is obtained by projecting a scene's 3D point \f$P_w\f$ into the image plane using a perspective
transformation which forms the corresponding pixel \f$p\f$. Both \f$P_w\f$ and \f$p\f$ are
represented in homogeneous coordinates, i.e. as 3D and 2D homogeneous vector respectively. You will
find a brief introduction to projective geometry, homogeneous vectors and homogeneous
transformations at the end of this section's introduction. For more succinct notation, we often drop
the 'homogeneous' and say vector instead of homogeneous vector.

The distortion-free projective transformation given by a  pinhole camera model is shown below.

\f[s \; p = A \begin{bmatrix} R|t \end{bmatrix} P_w,\f]

where \f$P_w\f$ is a 3D point expressed with respect to the world coordinate system,
\f$p\f$ is a 2D pixel in the image plane, \f$A\f$ is the camera intrinsic matrix,
\f$R\f$ and \f$t\f$ are the rotation and translation that describe the change of coordinates from
world to camera coordinate systems (or camera frame) and \f$s\f$ is the projective transformation's
arbitrary scaling and not part of the camera model.

The camera intrinsic matrix \f$A\f$ (notation used as in @cite Zhang2000 and also generally notated
as \f$K\f$) projects 3D points given in the camera coordinate system to 2D pixel coordinates, i.e.

\f[p = A P_c.\f]

The camera intrinsic matrix \f$A\f$ is composed of the focal lengths \f$f_x\f$ and \f$f_y\f$, which are
expressed in pixel units, and the principal point \f$(c_x, c_y)\f$, that is usually close to the
image center:

\f[A = \vecthreethree{f_x}{0}{c_x}{0}{f_y}{c_y}{0}{0}{1},\f]

and thus

\f[s \vecthree{u}{v}{1} = \vecthreethree{f_x}{0}{c_x}{0}{f_y}{c_y}{0}{0}{1} \vecthree{X_c}{Y_c}{Z_c}.\f]

The matrix of intrinsic parameters does not depend on the scene viewed. So, once estimated, it can
be re-used as long as the focal length is fixed (in case of a zoom lens). Thus, if an image from the
camera is scaled by a factor, all of these parameters need to be scaled (multiplied/divided,
respectively) by the same factor.

The joint rotation-translation matrix \f$[R|t]\f$ is the matrix product of a projective
transformation and a homogeneous transformation. The 3-by-4 projective transformation maps 3D points
represented in camera coordinates to 2D points in the image plane and represented in normalized
camera coordinates \f$x' = X_c / Z_c\f$ and \f$y' = Y_c / Z_c\f$:

\f[Z_c \begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix} = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0
\end{bmatrix}
\begin{bmatrix}
X_c \\
Y_c \\
Z_c \\
1
\end{bmatrix}.\f]

The homogeneous transformation is encoded by the extrinsic parameters \f$R\f$ and \f$t\f$ and
represents the change of basis from world coordinate system \f$w\f$ to the camera coordinate sytem
\f$c\f$. Thus, given the representation of the point \f$P\f$ in world coordinates, \f$P_w\f$, we
obtain \f$P\f$'s representation in the camera coordinate system, \f$P_c\f$, by

\f[P_c = \begin{bmatrix}
R & t \\
0 & 1
\end{bmatrix} P_w,\f]

This homogeneous transformation is composed out of \f$R\f$, a 3-by-3 rotation matrix, and \f$t\f$, a
3-by-1 translation vector:

\f[\begin{bmatrix}
R & t \\
0 & 1
\end{bmatrix} = \begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z \\
0 & 0 & 0 & 1
\end{bmatrix},
\f]

and therefore

\f[\begin{bmatrix}
X_c \\
Y_c \\
Z_c \\
1
\end{bmatrix} = \begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
X_w \\
Y_w \\
Z_w \\
1
\end{bmatrix}.\f]

Combining the projective transformation and the homogeneous transformation, we obtain the projective
transformation that maps 3D points in world coordinates into 2D points in the image plane and in
normalized camera coordinates:

\f[Z_c \begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix} = \begin{bmatrix} R|t \end{bmatrix} \begin{bmatrix}
X_w \\
Y_w \\
Z_w \\
1
\end{bmatrix} = \begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z
\end{bmatrix}
\begin{bmatrix}
X_w \\
Y_w \\
Z_w \\
1
\end{bmatrix},\f]

with \f$x' = X_c / Z_c\f$ and \f$y' = Y_c / Z_c\f$. Putting the equations for instrincs and extrinsics together, we can write out
\f$s \; p = A \begin{bmatrix} R|t \end{bmatrix} P_w\f$ as

\f[s \vecthree{u}{v}{1} = \vecthreethree{f_x}{0}{c_x}{0}{f_y}{c_y}{0}{0}{1}
\begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_x \\
r_{21} & r_{22} & r_{23} & t_y \\
r_{31} & r_{32} & r_{33} & t_z
\end{bmatrix}
\begin{bmatrix}
X_w \\
Y_w \\
Z_w \\
1
\end{bmatrix}.\f]

If \f$Z_c \ne 0\f$, the transformation above is equivalent to the following,

\f[\begin{bmatrix}
u \\
v
\end{bmatrix} = \begin{bmatrix}
f_x X_c/Z_c + c_x \\
f_y Y_c/Z_c + c_y
\end{bmatrix}\f]

with

\f[\vecthree{X_c}{Y_c}{Z_c} = \begin{bmatrix}
R|t
\end{bmatrix} \begin{bmatrix}
X_w \\
Y_w \\
Z_w \\
1
\end{bmatrix}.\f]

The following figure illustrates the pinhole camera model.

![Pinhole camera model](pics/pinhole_camera_model.png)

Real lenses usually have some distortion, mostly radial distortion, and slight tangential distortion.
So, the above model is extended as:

\f[\begin{bmatrix}
u \\
v
\end{bmatrix} = \begin{bmatrix}
f_x x'' + c_x \\
f_y y'' + c_y
\end{bmatrix}\f]

where

\f[\begin{bmatrix}
x'' \\
y''
\end{bmatrix} = \begin{bmatrix}
x' \frac{1 + k_1 r^2 + k_2 r^4 + k_3 r^6}{1 + k_4 r^2 + k_5 r^4 + k_6 r^6} + 2 p_1 x' y' + p_2(r^2 + 2 x'^2) + s_1 r^2 + s_2 r^4 \\
y' \frac{1 + k_1 r^2 + k_2 r^4 + k_3 r^6}{1 + k_4 r^2 + k_5 r^4 + k_6 r^6} + p_1 (r^2 + 2 y'^2) + 2 p_2 x' y' + s_3 r^2 + s_4 r^4 \\
\end{bmatrix}\f]

with

\f[r^2 = x'^2 + y'^2\f]

and

\f[\begin{bmatrix}
x'\\
y'
\end{bmatrix} = \begin{bmatrix}
X_c/Z_c \\
Y_c/Z_c
\end{bmatrix},\f]

if \f$Z_c \ne 0\f$.

The distortion parameters are the radial coefficients \f$k_1\f$, \f$k_2\f$, \f$k_3\f$, \f$k_4\f$, \f$k_5\f$, and \f$k_6\f$
,\f$p_1\f$ and \f$p_2\f$ are the tangential distortion coefficients, and \f$s_1\f$, \f$s_2\f$, \f$s_3\f$, and \f$s_4\f$,
are the thin prism distortion coefficients. Higher-order coefficients are not considered in OpenCV.

The next figures show two common types of radial distortion: barrel distortion
(\f$ 1 + k_1 r^2 + k_2 r^4 + k_3 r^6 \f$ monotonically decreasing)
and pincushion distortion (\f$ 1 + k_1 r^2 + k_2 r^4 + k_3 r^6 \f$ monotonically increasing).
Radial distortion is always monotonic for real lenses,
and if the estimator produces a non-monotonic result,
this should be considered a calibration failure.
More generally, radial distortion must be monotonic and the distortion function must be bijective.
A failed estimation result may look deceptively good near the image center
but will work poorly in e.g. AR/SFM applications.
The optimization method used in OpenCV camera calibration does not include these constraints as
the framework does not support the required integer programming and polynomial inequalities.
See [issue #15992](https://github.com/opencv/opencv/issues/15992) for additional information.

![](pics/distortion_examples.png)
![](pics/distortion_examples2.png)

In some cases, the image sensor may be tilted in order to focus an oblique plane in front of the
camera (Scheimpflug principle). This can be useful for particle image velocimetry (PIV) or
triangulation with a laser fan. The tilt causes a perspective distortion of \f$x''\f$ and
\f$y''\f$. This distortion can be modeled in the following way, see e.g. @cite Louhichi07.

\f[\begin{bmatrix}
u \\
v
\end{bmatrix} = \begin{bmatrix}
f_x x''' + c_x \\
f_y y''' + c_y
\end{bmatrix},\f]

where

\f[s\vecthree{x'''}{y'''}{1} =
\vecthreethree{R_{33}(\tau_x, \tau_y)}{0}{-R_{13}(\tau_x, \tau_y)}
{0}{R_{33}(\tau_x, \tau_y)}{-R_{23}(\tau_x, \tau_y)}
{0}{0}{1} R(\tau_x, \tau_y) \vecthree{x''}{y''}{1}\f]

and the matrix \f$R(\tau_x, \tau_y)\f$ is defined by two rotations with angular parameter
\f$\tau_x\f$ and \f$\tau_y\f$, respectively,

\f[
R(\tau_x, \tau_y) =
\vecthreethree{\cos(\tau_y)}{0}{-\sin(\tau_y)}{0}{1}{0}{\sin(\tau_y)}{0}{\cos(\tau_y)}
\vecthreethree{1}{0}{0}{0}{\cos(\tau_x)}{\sin(\tau_x)}{0}{-\sin(\tau_x)}{\cos(\tau_x)} =
\vecthreethree{\cos(\tau_y)}{\sin(\tau_y)\sin(\tau_x)}{-\sin(\tau_y)\cos(\tau_x)}
{0}{\cos(\tau_x)}{\sin(\tau_x)}
{\sin(\tau_y)}{-\cos(\tau_y)\sin(\tau_x)}{\cos(\tau_y)\cos(\tau_x)}.
\f]

In the functions below the coefficients are passed or returned as

\f[(k_1, k_2, p_1, p_2[, k_3[, k_4, k_5, k_6 [, s_1, s_2, s_3, s_4[, \tau_x, \tau_y]]]])\f]

vector. That is, if the vector contains four elements, it means that \f$k_3=0\f$ . The distortion
coefficients do not depend on the scene viewed. Thus, they also belong to the intrinsic camera
parameters. And they remain the same regardless of the captured image resolution. If, for example, a
camera has been calibrated on images of 320 x 240 resolution, absolutely the same distortion
coefficients can be used for 640 x 480 images from the same camera while \f$f_x\f$, \f$f_y\f$,
\f$c_x\f$, and \f$c_y\f$ need to be scaled appropriately.

The functions below use the above model to do the following:

-   Project 3D points to the image plane given intrinsic and extrinsic parameters.
-   Compute extrinsic parameters given intrinsic parameters, a few 3D points, and their
projections.
-   Estimate intrinsic and extrinsic camera parameters from several views of a known calibration
pattern (every view is described by several 3D-2D point correspondences).
-   Estimate the relative position and orientation of the stereo camera "heads" and compute the
*rectification* transformation that makes the camera optical axes parallel.

<B> Homogeneous Coordinates </B><br>
Homogeneous Coordinates are a system of coordinates that are used in projective geometry. Their use
allows to represent points at infinity by finite coordinates and simplifies formulas when compared
to the cartesian counterparts, e.g. they have the advantage that affine transformations can be
expressed as linear homogeneous transformation.

One obtains the homogeneous vector \f$P_h\f$ by appending a 1 along an n-dimensional cartesian
vector \f$P\f$ e.g. for a 3D cartesian vector the mapping \f$P \rightarrow P_h\f$ is:

\f[\begin{bmatrix}
X \\
Y \\
Z
\end{bmatrix} \rightarrow \begin{bmatrix}
X \\
Y \\
Z \\
1
\end{bmatrix}.\f]

For the inverse mapping \f$P_h \rightarrow P\f$, one divides all elements of the homogeneous vector
by its last element, e.g. for a 3D homogeneous vector one gets its 2D cartesian counterpart by:

\f[\begin{bmatrix}
X \\
Y \\
W
\end{bmatrix} \rightarrow \begin{bmatrix}
X / W \\
Y / W
\end{bmatrix},\f]

if \f$W \ne 0\f$.

Due to this mapping, all multiples \f$k P_h\f$, for \f$k \ne 0\f$, of a homogeneous point represent
the same point \f$P_h\f$. An intuitive understanding of this property is that under a projective
transformation, all multiples of \f$P_h\f$ are mapped to the same point. This is the physical
observation one does for pinhole cameras, as all points along a ray through the camera's pinhole are
projected to the same image point, e.g. all points along the red ray in the image of the pinhole
camera model above would be mapped to the same image coordinate. This property is also the source
for the scale ambiguity s in the equation of the pinhole camera model.

As mentioned, by using homogeneous coordinates we can express any change of basis parameterized by
\f$R\f$ and \f$t\f$ as a linear transformation, e.g. for the change of basis from coordinate system
0 to coordinate system 1 becomes:

\f[P_1 = R P_0 + t \rightarrow P_{h_1} = \begin{bmatrix}
R & t \\
0 & 1
\end{bmatrix} P_{h_0}.\f]

<B> Homogeneous Transformations, Object frame / Camera frame </B><br>
Change of basis or computing the 3D coordinates from one frame to another frame can be achieved easily using
the following notation:

\f[
\mathbf{X}_c = \hspace{0.2em}
{}^{c}\mathbf{T}_o \hspace{0.2em} \mathbf{X}_o
\f]

\f[
\begin{bmatrix}
X_c \\
Y_c \\
Z_c \\
1
\end{bmatrix} =
\begin{bmatrix}
{}^{c}\mathbf{R}_o & {}^{c}\mathbf{t}_o \\
0_{1 \times 3} & 1
\end{bmatrix}
\begin{bmatrix}
X_o \\
Y_o \\
Z_o \\
1
\end{bmatrix}
\f]

For a 3D points (\f$ \mathbf{X}_o \f$) expressed in the object frame, the homogeneous transformation matrix
\f$ {}^{c}\mathbf{T}_o \f$ allows computing the corresponding coordinate (\f$ \mathbf{X}_c \f$) in the camera frame.
This transformation matrix is composed of a 3x3 rotation matrix \f$ {}^{c}\mathbf{R}_o \f$ and a 3x1 translation vector
\f$ {}^{c}\mathbf{t}_o \f$.
The 3x1 translation vector \f$ {}^{c}\mathbf{t}_o \f$ is the position of the object frame in the camera frame and the
3x3 rotation matrix \f$ {}^{c}\mathbf{R}_o \f$ the orientation of the object frame in the camera frame.

With this simple notation, it is easy to chain the transformations. For instance, to compute the 3D coordinates of a point
expressed in the object frame in the world frame can be done with:

\f[
\mathbf{X}_w = \hspace{0.2em}
{}^{w}\mathbf{T}_c \hspace{0.2em} {}^{c}\mathbf{T}_o \hspace{0.2em}
\mathbf{X}_o =
{}^{w}\mathbf{T}_o \hspace{0.2em} \mathbf{X}_o
\f]

Similarly, computing the inverse transformation can be done with:

\f[
\mathbf{X}_o = \hspace{0.2em}
{}^{o}\mathbf{T}_c \hspace{0.2em} \mathbf{X}_c =
\left( {}^{c}\mathbf{T}_o \right)^{-1} \hspace{0.2em} \mathbf{X}_c
\f]

The inverse of an homogeneous transformation matrix is then:

\f[
{}^{o}\mathbf{T}_c = \left( {}^{c}\mathbf{T}_o \right)^{-1} =
\begin{bmatrix}
{}^{c}\mathbf{R}^{\top}_o & - \hspace{0.2em} {}^{c}\mathbf{R}^{\top}_o \hspace{0.2em} {}^{c}\mathbf{t}_o \\
0_{1 \times 3} & 1
\end{bmatrix}
\f]

One can note that the inverse of a 3x3 rotation matrix is directly its matrix transpose.

![Perspective projection, from object to camera frame](pics/pinhole_homogeneous_transformation.png)

This figure summarizes the whole process. The object pose returned for instance by the @ref solvePnP function
or pose from fiducial marker detection is this \f$ {}^{c}\mathbf{T}_o \f$ transformation.

The camera intrinsic matrix \f$ \mathbf{K} \f$ allows projecting the 3D point expressed in the camera frame onto the image plane
assuming a perspective projection model (pinhole camera model). Image coordinates extracted from classical image processing functions
assume a (u,v) top-left coordinates frame.

\note
- for an online video course on this topic, see for instance:
  - ["3.3.1. Homogeneous Transformation Matrices", Modern Robotics, Kevin M. Lynch and Frank C. Park](https://modernrobotics.northwestern.edu/nu-gm-book-resource/3-3-1-homogeneous-transformation-matrices/)
- the 3x3 rotation matrix is composed of 9 values but describes a 3 dof transformation
- some additional properties of the 3x3 rotation matrix are:
  - \f$ \mathrm{det} \left( \mathbf{R} \right) = 1 \f$
  - \f$ \mathbf{R} \mathbf{R}^{\top} = \mathbf{R}^{\top} \mathbf{R} = \mathrm{I}_{3 \times 3} \f$
  - interpolating rotation can be done using the [Slerp (spherical linear interpolation)](https://en.wikipedia.org/wiki/Slerp) method
- quick conversions between the different rotation formalisms can be done using this [online tool](https://www.andre-gaschler.com/rotationconverter/)

<B> Intrinsic parameters from camera lens specifications </B><br>
When dealing with industrial cameras, the camera intrinsic matrix or more precisely \f$ \left(f_x, f_y \right) \f$
can be deduced, approximated from the camera specifications:

\f[
f_x = \frac{f_{\text{mm}}}{\text{pixel_size_in_mm}} = \frac{f_{\text{mm}}}{\text{sensor_size_in_mm} / \text{nb_pixels}}
\f]

In a same way, the physical focal length can be deduced from the angular field of view:

\f[
f_{\text{mm}} = \frac{\text{sensor_size_in_mm}}{2 \times \tan{\frac{\text{fov}}{2}}}
\f]

This latter conversion can be useful when using a rendering software to mimic a physical camera device.

@note
    -    See also #calibrationMatrixValues

<B> Additional references, notes </B><br>
@note
    -   Many functions in this module take a camera intrinsic matrix as an input parameter. Although all
        functions assume the same structure of this parameter, they may name it differently. The
        parameter's description, however, will be clear in that a camera intrinsic matrix with the structure
        shown above is required.
    -   A calibration sample for 3 cameras in a horizontal position can be found at
        opencv_source_code/samples/cpp/3calibration.cpp
    -   A calibration sample based on a sequence of images can be found at
        opencv_source_code/samples/cpp/calibration.cpp
    -   A calibration sample in order to do 3D reconstruction can be found at
        opencv_source_code/samples/cpp/build3dmodel.cpp
    -   A calibration example on stereo calibration can be found at
        opencv_source_code/samples/cpp/stereo_calib.cpp
    -   A calibration example on stereo matching can be found at
        opencv_source_code/samples/cpp/stereo_match.cpp
    -   (Python) A camera calibration sample can be found at
        opencv_source_code/samples/python/calibrate.py

  @{
    @defgroup calib3d_fisheye Fisheye camera model

    Definitions: Let P be a point in 3D of coordinates X in the world reference frame (stored in the
    matrix X) The coordinate vector of P in the camera reference frame is:

    \f[Xc = R X + T\f]

    where R is the rotation matrix corresponding to the rotation vector om: R = rodrigues(om); call x, y
    and z the 3 coordinates of Xc:

    \f[\begin{array}{l} x = Xc_1 \\ y = Xc_2 \\ z = Xc_3 \end{array} \f]

    The pinhole projection coordinates of P is [a; b] where

    \f[\begin{array}{l} a = x / z \ and \ b = y / z \\ r^2 = a^2 + b^2 \\ \theta = atan(r) \end{array} \f]

    Fisheye distortion:

    \f[\theta_d = \theta (1 + k_1 \theta^2 + k_2 \theta^4 + k_3 \theta^6 + k_4 \theta^8)\f]

    The distorted point coordinates are [x'; y'] where

    \f[\begin{array}{l} x' = (\theta_d / r) a \\ y' = (\theta_d / r) b \end{array} \f]

    Finally, conversion into pixel coordinates: The final pixel coordinates vector [u; v] where:

    \f[\begin{array}{l} u = f_x (x' + \alpha y') + c_x \\
    v = f_y y' + c_y \end{array} \f]

    Summary:
    Generic camera model @cite Kannala2006 with perspective projection and without distortion correction

  @}
 */

namespace cv
{

//! @addtogroup calib3d
//! @{

//! type of the robust estimation algorithm
enum { LMEDS  = 4,  //!< least-median of squares algorithm
       RANSAC = 8,  //!< RANSAC algorithm
       RHO    = 16, //!< RHO algorithm
       USAC_DEFAULT  = 32, //!< USAC algorithm, default settings
       USAC_PARALLEL = 33, //!< USAC, parallel version
       USAC_FM_8PTS = 34,  //!< USAC, fundamental matrix 8 points
       USAC_FAST = 35,     //!< USAC, fast settings
       USAC_ACCURATE = 36, //!< USAC, accurate settings
       USAC_PROSAC = 37,   //!< USAC, sorted points, runs PROSAC
       USAC_MAGSAC = 38    //!< USAC, runs MAGSAC++
     };

enum SolvePnPMethod {
    SOLVEPNP_ITERATIVE   = 0, //!< Pose refinement using non-linear Levenberg-Marquardt minimization scheme @cite Madsen04 @cite Eade13 \n
                              //!< Initial solution for non-planar "objectPoints" needs at least 6 points and uses the DLT algorithm. \n
                              //!< Initial solution for planar "objectPoints" needs at least 4 points and uses pose from homography decomposition.
    SOLVEPNP_EPNP        = 1, //!< EPnP: Efficient Perspective-n-Point Camera Pose Estimation @cite lepetit2009epnp
    SOLVEPNP_P3P         = 2, //!< Revisiting the P3P Problem @cite ding2023revisiting
    SOLVEPNP_DLS         = 3, //!< **Broken implementation. Using this flag will fallback to EPnP.** \n
                              //!< A Direct Least-Squares (DLS) Method for PnP @cite hesch2011direct
    SOLVEPNP_UPNP        = 4, //!< **Broken implementation. Using this flag will fallback to EPnP.** \n
                              //!< Exhaustive Linearization for Robust Camera Pose and Focal Length Estimation @cite penate2013exhaustive
    SOLVEPNP_AP3P        = 5, //!< An Efficient Algebraic Solution to the Perspective-Three-Point Problem @cite Ke17
    SOLVEPNP_IPPE        = 6, //!< Infinitesimal Plane-Based Pose Estimation @cite Collins14 \n
                              //!< Object points must be coplanar.
    SOLVEPNP_IPPE_SQUARE = 7, //!< Infinitesimal Plane-Based Pose Estimation @cite Collins14 \n
                              //!< This is a special case suitable for marker pose estimation.\n
                              //!< 4 coplanar object points must be defined in the following order:
                              //!<   - point 0: [-squareLength / 2,  squareLength / 2, 0]
                              //!<   - point 1: [ squareLength / 2,  squareLength / 2, 0]
                              //!<   - point 2: [ squareLength / 2, -squareLength / 2, 0]
                              //!<   - point 3: [-squareLength / 2, -squareLength / 2, 0]
    SOLVEPNP_SQPNP       = 8, //!< SQPnP: A Consistently Fast and Globally OptimalSolution to the Perspective-n-Point Problem @cite Terzakis2020SQPnP
#ifndef CV_DOXYGEN
    SOLVEPNP_MAX_COUNT        //!< Used for count
#endif
};

enum { CALIB_CB_ADAPTIVE_THRESH = 1,
       CALIB_CB_NORMALIZE_IMAGE = 2,
       CALIB_CB_FILTER_QUADS    = 4,
       CALIB_CB_FAST_CHECK      = 8,
       CALIB_CB_EXHAUSTIVE      = 16,
       CALIB_CB_ACCURACY        = 32,
       CALIB_CB_LARGER          = 64,
       CALIB_CB_MARKER          = 128,
       CALIB_CB_PLAIN           = 256
     };

enum { CALIB_CB_SYMMETRIC_GRID  = 1,
       CALIB_CB_ASYMMETRIC_GRID = 2,
       CALIB_CB_CLUSTERING      = 4
     };

enum { CALIB_NINTRINSIC          = 18,
       CALIB_USE_INTRINSIC_GUESS = 0x00001,
       CALIB_FIX_ASPECT_RATIO    = 0x00002,
       CALIB_FIX_PRINCIPAL_POINT = 0x00004,
       CALIB_ZERO_TANGENT_DIST   = 0x00008,
       CALIB_FIX_FOCAL_LENGTH    = 0x00010,
       CALIB_FIX_K1              = 0x00020,
       CALIB_FIX_K2              = 0x00040,
       CALIB_FIX_K3              = 0x00080,
       CALIB_FIX_K4              = 0x00800,
       CALIB_FIX_K5              = 0x01000,
       CALIB_FIX_K6              = 0x02000,
       CALIB_RATIONAL_MODEL      = 0x04000,
       CALIB_THIN_PRISM_MODEL    = 0x08000,
       CALIB_FIX_S1_S2_S3_S4     = 0x10000,
       CALIB_TILTED_MODEL        = 0x40000,
       CALIB_FIX_TAUX_TAUY       = 0x80000,
       CALIB_USE_QR              = 0x100000, //!< use QR instead of SVD decomposition for solving. Faster but potentially less precise
       CALIB_FIX_TANGENT_DIST    = 0x200000,
       // only for stereo
       CALIB_FIX_INTRINSIC       = 0x00100,
       CALIB_SAME_FOCAL_LENGTH   = 0x00200,
       // for stereo rectification
       CALIB_ZERO_DISPARITY      = 0x00400,
       CALIB_USE_LU              = (1 << 17), //!< use LU instead of SVD decomposition for solving. much faster but potentially less precise
       CALIB_USE_EXTRINSIC_GUESS = (1 << 22)  //!< for stereoCalibrate
     };

//! the algorithm for finding fundamental matrix
enum { FM_7POINT = 1, //!< 7-point algorithm
       FM_8POINT = 2, //!< 8-point algorithm
       FM_LMEDS  = 4, //!< least-median algorithm. 7-point algorithm is used.
       FM_RANSAC = 8  //!< RANSAC algorithm. It needs at least 15 points. 7-point algorithm is used.
     };

enum HandEyeCalibrationMethod
{
    CALIB_HAND_EYE_TSAI         = 0, //!< A New Technique for Fully Autonomous and Efficient 3D Robotics Hand/Eye Calibration @cite Tsai89
    CALIB_HAND_EYE_PARK         = 1, //!< Robot Sensor Calibration: Solving AX = XB on the Euclidean Group @cite Park94
    CALIB_HAND_EYE_HORAUD       = 2, //!< Hand-eye Calibration @cite Horaud95
    CALIB_HAND_EYE_ANDREFF      = 3, //!< On-line Hand-Eye Calibration @cite Andreff99
    CALIB_HAND_EYE_DANIILIDIS   = 4  //!< Hand-Eye Calibration Using Dual Quaternions @cite Daniilidis98
};

enum RobotWorldHandEyeCalibrationMethod
{
    CALIB_ROBOT_WORLD_HAND_EYE_SHAH = 0, //!< Solving the robot-world/hand-eye calibration problem using the kronecker product @cite Shah2013SolvingTR
    CALIB_ROBOT_WORLD_HAND_EYE_LI   = 1  //!< Simultaneous robot-world and hand-eye calibration using dual-quaternions and kronecker product @cite Li2010SimultaneousRA
};

enum SamplingMethod { SAMPLING_UNIFORM=0, SAMPLING_PROGRESSIVE_NAPSAC=1, SAMPLING_NAPSAC=2,
        SAMPLING_PROSAC=3 };
enum LocalOptimMethod {LOCAL_OPTIM_NULL=0, LOCAL_OPTIM_INNER_LO=1, LOCAL_OPTIM_INNER_AND_ITER_LO=2,
        LOCAL_OPTIM_GC=3, LOCAL_OPTIM_SIGMA=4};
enum ScoreMethod {SCORE_METHOD_RANSAC=0, SCORE_METHOD_MSAC=1, SCORE_METHOD_MAGSAC=2, SCORE_METHOD_LMEDS=3};
enum NeighborSearchMethod { NEIGH_FLANN_KNN=0, NEIGH_GRID=1, NEIGH_FLANN_RADIUS=2 };
enum PolishingMethod { NONE_POLISHER=0, LSQ_POLISHER=1, MAGSAC=2, COV_POLISHER=3 };

struct CV_EXPORTS_W_SIMPLE UsacParams
{ // in alphabetical order
    CV_WRAP UsacParams();
    CV_PROP_RW double confidence;
    CV_PROP_RW bool isParallel;
    CV_PROP_RW int loIterations;
    CV_PROP_RW LocalOptimMethod loMethod;
    CV_PROP_RW int loSampleSize;
    CV_PROP_RW int maxIterations;
    CV_PROP_RW NeighborSearchMethod neighborsSearch;
    CV_PROP_RW int randomGeneratorState;
    CV_PROP_RW SamplingMethod sampler;
    CV_PROP_RW ScoreMethod score;
    CV_PROP_RW double threshold;
    CV_PROP_RW PolishingMethod final_polisher;
    CV_PROP_RW int final_polisher_iterations;
};

/** @brief Converts a rotation matrix to a rotation vector or vice versa.

@param src Input rotation vector (3x1 or 1x3) or rotation matrix (3x3).
@param dst Output rotation matrix (3x3) or rotation vector (3x1 or 1x3), respectively.
@param jacobian Optional output Jacobian matrix, 3x9 or 9x3, which is a matrix of partial
derivatives of the output array components with respect to the input array components.

\f[\begin{array}{l} \theta \leftarrow norm(r) \\ r  \leftarrow r/ \theta \\ R =  \cos(\theta) I + (1- \cos{\theta} ) r r^T +  \sin(\theta) \vecthreethree{0}{-r_z}{r_y}{r_z}{0}{-r_x}{-r_y}{r_x}{0} \end{array}\f]

Inverse transformation can be also done easily, since

\f[\sin ( \theta ) \vecthreethree{0}{-r_z}{r_y}{r_z}{0}{-r_x}{-r_y}{r_x}{0} = \frac{R - R^T}{2}\f]

A rotation vector is a convenient and most compact representation of a rotation matrix (since any
rotation matrix has just 3 degrees of freedom). The representation is used in the global 3D geometry
optimization procedures like @ref calibrateCamera, @ref stereoCalibrate, or @ref solvePnP .

@note More information about the computation of the derivative of a 3D rotation matrix with respect to its exponential coordinate
can be found in:
    - A Compact Formula for the Derivative of a 3-D Rotation in Exponential Coordinates, Guillermo Gallego, Anthony J. Yezzi @cite Gallego2014ACF

@note Useful information on SE(3) and Lie Groups can be found in:
    - A tutorial on SE(3) transformation parameterizations and on-manifold optimization, Jose-Luis Blanco @cite blanco2010tutorial
    - Lie Groups for 2D and 3D Transformation, Ethan Eade @cite Eade17
    - A micro Lie theory for state estimation in robotics, Joan Solà, Jérémie Deray, Dinesh Atchuthan @cite Sol2018AML
 */
CV_EXPORTS_W void Rodrigues( InputArray src, OutputArray dst, OutputArray jacobian = noArray() );



/** Levenberg-Marquardt solver. Starting with the specified vector of parameters it
    optimizes the target vector criteria "err"
    (finds local minima of each target vector component absolute value).

    When needed, it calls user-provided callback.
*/
class CV_EXPORTS LMSolver : public Algorithm
{
public:
    class CV_EXPORTS Callback
    {
    public:
        virtual ~Callback() {}
        /**
         computes error and Jacobian for the specified vector of parameters

         @param param the current vector of parameters
         @param err output vector of errors: err_i = actual_f_i - ideal_f_i
         @param J output Jacobian: J_ij = d(ideal_f_i)/d(param_j)

         when J=noArray(), it means that it does not need to be computed.
         Dimensionality of error vector and param vector can be different.
         The callback should explicitly allocate (with "create" method) each output array
         (unless it's noArray()).
        */
        virtual bool compute(InputArray param, OutputArray err, OutputArray J) const = 0;
    };

    /**
       Runs Levenberg-Marquardt algorithm using the passed vector of parameters as the start point.
       The final vector of parameters (whether the algorithm converged or not) is stored at the same
       vector. The method returns the number of iterations used. If it's equal to the previously specified
       maxIters, there is a big chance the algorithm did not converge.

       @param param initial/final vector of parameters.

       Note that the dimensionality of parameter space is defined by the size of param vector,
       and the dimensionality of optimized criteria is defined by the size of err vector
       computed by the callback.
    */
    virtual int run(InputOutputArray param) const = 0;

    /**
       Sets the maximum number of iterations
       @param maxIters the number of iterations
    */
    virtual void setMaxIters(int maxIters) = 0;
    /**
       Retrieves the current maximum number of iterations
    */
    virtual int getMaxIters() const = 0;

    /**
       Creates Levenberg-Marquard solver

       @param cb callback
       @param maxIters maximum number of iterations that can be further
         modified using setMaxIters() method.
    */
    static Ptr<LMSolver> create(const Ptr<LMSolver::Callback>& cb, int maxIters);
    static Ptr<LMSolver> create(const Ptr<LMSolver::Callback>& cb, int maxIters, double eps);
};



/** @example samples/cpp/tutorial_code/features2D/Homography/pose_from_homography.cpp
An example program about pose estimation from coplanar points

Check @ref tutorial_homography "the corresponding tutorial" for more details
*/

/** @brief Finds a perspective transformation between two planes.

@param srcPoints Coordinates of the points in the original plane, a matrix of the type CV_32FC2
or vector\<Point2f\> .
@param dstPoints Coordinates of the points in the target plane, a matrix of the type CV_32FC2 or
a vector\<Point2f\> .
@param method Method used to compute a homography matrix. The following methods are possible:
-   **0** - a regular method using all the points, i.e., the least squares method
-   @ref RANSAC - RANSAC-based robust method
-   @ref LMEDS - Least-Median robust method
-   @ref RHO - PROSAC-based robust method
@param ransacReprojThreshold Maximum allowed reprojection error to treat a point pair as an inlier
(used in the RANSAC and RHO methods only). That is, if
\f[\| \texttt{dstPoints} _i -  \texttt{convertPointsHomogeneous} ( \texttt{H} \cdot \texttt{srcPoints} _i) \|_2  >  \texttt{ransacReprojThreshold}\f]
then the point \f$i\f$ is considered as an outlier. If srcPoints and dstPoints are measured in pixels,
it usually makes sense to set this parameter somewhere in the range of 1 to 10.
@param mask Optional output mask set by a robust method ( RANSAC or LMeDS ). Note that the input
mask values are ignored.
@param maxIters The maximum number of RANSAC iterations.
@param confidence Confidence level, between 0 and 1.

The function finds and returns the perspective transformation \f$H\f$ between the source and the
destination planes:

\f[s_i  \vecthree{x'_i}{y'_i}{1} \sim H  \vecthree{x_i}{y_i}{1}\f]

so that the back-projection error

\f[\sum _i \left ( x'_i- \frac{h_{11} x_i + h_{12} y_i + h_{13}}{h_{31} x_i + h_{32} y_i + h_{33}} \right )^2+ \left ( y'_i- \frac{h_{21} x_i + h_{22} y_i + h_{23}}{h_{31} x_i + h_{32} y_i + h_{33}} \right )^2\f]

is minimized. If the parameter method is set to the default value 0, the function uses all the point
pairs to compute an initial homography estimate with a simple least-squares scheme.

However, if not all of the point pairs ( \f$srcPoints_i\f$, \f$dstPoints_i\f$ ) fit the rigid perspective
transformation (that is, there are some outliers), this initial estimate will be poor. In this case,
you can use one of the three robust methods. The methods RANSAC, LMeDS and RHO try many different
random subsets of the corresponding point pairs (of four pairs each, collinear pairs are discarded), estimate the homography matrix
using this subset and a simple least-squares algorithm, and then compute the quality/goodness of the
computed homography (which is the number of inliers for RANSAC or the least median re-projection error for
LMeDS). The best subset is then used to produce the initial estimate of the homography matrix and
the mask of inliers/outliers.

Regardless of the method, robust or not, the computed homography matrix is refined further (using
inliers only in case of a robust method) with the Levenberg-Marquardt method to reduce the
re-projection error even more.

The methods RANSAC and RHO can handle practically any ratio of outliers but need a threshold to
distinguish inliers from outliers. The method LMeDS does not need any threshold but it works
correctly only when there are more than 50% of inliers. Finally, if there are no outliers and the
noise is rather small, use the default method (method=0).

The function is used to find initial intrinsic and extrinsic matrices. Homography matrix is
determined up to a scale. If \f$h_{33}\f$ is non-zero, the matrix is normalized so that \f$h_{33}=1\f$.
@note Whenever an \f$H\f$ matrix cannot be estimated, an empty one will be returned.

@sa
getAffineTransform, estimateAffine2D, estimateAffinePartial2D, getPerspectiveTransform, warpPerspective,
perspectiveTransform
 */
CV_EXPORTS_W Mat findHomography( InputArray srcPoints, InputArray dstPoints,
                                 int method = 0, double ransacReprojThreshold = 3,
                                 OutputArray mask=noArray(), const int maxIters = 2000,
                                 const double confidence = 0.995);

/** @overload */
CV_EXPORTS Mat findHomography( InputArray srcPoints, InputArray dstPoints,
                               OutputArray mask, int method = 0, double ransacReprojThreshold = 3 );


CV_EXPORTS_W Mat findHomography(InputArray srcPoints, InputArray dstPoints, OutputArray mask,
                   const UsacParams &params);

/** @brief Computes an RQ decomposition of 3x3 matrices.

@param src 3x3 input matrix.
@param mtxR Output 3x3 upper-triangular matrix.
@param mtxQ Output 3x3 orthogonal matrix.
@param Qx Optional output 3x3 rotation matrix around x-axis.
@param Qy Optional output 3x3 rotation matrix around y-axis.
@param Qz Optional output 3x3 rotation matrix around z-axis.

The function computes a RQ decomposition using the given rotations. This function is used in
#decomposeProjectionMatrix to decompose the left 3x3 submatrix of a projection matrix into a camera
and a rotation matrix.

It optionally returns three rotation matrices, one for each axis, and the three Euler angles in
degrees (as the return value) that could be used in OpenGL. Note, there is always more than one
sequence of rotations about the three principal axes that results in the same orientation of an
object, e.g. see @cite Slabaugh . Returned three rotation matrices and corresponding three Euler angles
are only one of the possible solutions.
 */
CV_EXPORTS_W Vec3d RQDecomp3x3( InputArray src, OutputArray mtxR, OutputArray mtxQ,
                                OutputArray Qx = noArray(),
                                OutputArray Qy = noArray(),
                                OutputArray Qz = noArray());

/** @brief Decomposes a projection matrix into a rotation matrix and a camera intrinsic matrix.

@param projMatrix 3x4 input projection matrix P.
@param cameraMatrix Output 3x3 camera intrinsic matrix \f$\cameramatrix{A}\f$.
@param rotMatrix Output 3x3 external rotation matrix R.
@param transVect Output 4x1 translation vector T.
@param rotMatrixX Optional 3x3 rotation matrix around x-axis.
@param rotMatrixY Optional 3x3 rotation matrix around y-axis.
@param rotMatrixZ Optional 3x3 rotation matrix around z-axis.
@param eulerAngles Optional three-element vector containing three Euler angles of rotation in
degrees.

The function computes a decomposition of a projection matrix into a calibration and a rotation
matrix and the position of a camera.

It optionally returns three rotation matrices, one for each axis, and three Euler angles that could
be used in OpenGL. Note, there is always more than one sequence of rotations about the three
principal axes that results in the same orientation of an object, e.g. see @cite Slabaugh . Returned
three rotation matrices and corresponding three Euler angles are only one of the possible solutions.

The function is based on #RQDecomp3x3 .
 */
CV_EXPORTS_W void decomposeProjectionMatrix( InputArray projMatrix, OutputArray cameraMatrix,
                                             OutputArray rotMatrix, OutputArray transVect,
                                             OutputArray rotMatrixX = noArray(),
                                             OutputArray rotMatrixY = noArray(),
                                             OutputArray rotMatrixZ = noArray(),
                                             OutputArray eulerAngles =noArray() );

/** @brief Computes partial derivatives of the matrix product for each multiplied matrix.

@param A First multiplied matrix.
@param B Second multiplied matrix.
@param dABdA First output derivative matrix d(A\*B)/dA of size
\f$\texttt{A.rows*B.cols} \times {A.rows*A.cols}\f$ .
@param dABdB Second output derivative matrix d(A\*B)/dB of size
\f$\texttt{A.rows*B.cols} \times {B.rows*B.cols}\f$ .

The function computes partial derivatives of the elements of the matrix product \f$A*B\f$ with regard to
the elements of each of the two input matrices. The function is used to compute the Jacobian
matrices in #stereoCalibrate but can also be used in any other similar optimization function.
 */
CV_EXPORTS_W void matMulDeriv( InputArray A, InputArray B, OutputArray dABdA, OutputArray dABdB );

/** @brief Combines two rotation-and-shift transformations.

@param rvec1 First rotation vector.
@param tvec1 First translation vector.
@param rvec2 Second rotation vector.
@param tvec2 Second translation vector.
@param rvec3 Output rotation vector of the superposition.
@param tvec3 Output translation vector of the superposition.
@param dr3dr1 Optional output derivative of rvec3 with regard to rvec1
@param dr3dt1 Optional output derivative of rvec3 with regard to tvec1
@param dr3dr2 Optional output derivative of rvec3 with regard to rvec2
@param dr3dt2 Optional output derivative of rvec3 with regard to tvec2
@param dt3dr1 Optional output derivative of tvec3 with regard to rvec1
@param dt3dt1 Optional output derivative of tvec3 with regard to tvec1
@param dt3dr2 Optional output derivative of tvec3 with regard to rvec2
@param dt3dt2 Optional output derivative of tvec3 with regard to tvec2

The functions compute:

\f[\begin{array}{l} \texttt{rvec3} =  \mathrm{rodrigues} ^{-1} \left ( \mathrm{rodrigues} ( \texttt{rvec2} )  \cdot \mathrm{rodrigues} ( \texttt{rvec1} ) \right )  \\ \texttt{tvec3} =  \mathrm{rodrigues} ( \texttt{rvec2} )  \cdot \texttt{tvec1} +  \texttt{tvec2} \end{array} ,\f]

where \f$\mathrm{rodrigues}\f$ denotes a rotation vector to a rotation matrix transformation, and
\f$\mathrm{rodrigues}^{-1}\f$ denotes the inverse transformation. See #Rodrigues for details.

Also, the functions can compute the derivatives of the output vectors with regards to the input
vectors (see #matMulDeriv ). The functions are used inside #stereoCalibrate but can also be used in
your own code where Levenberg-Marquardt or another gradient-based solver is used to optimize a
function that contains a matrix multiplication.
 */
CV_EXPORTS_W void composeRT( InputArray rvec1, InputArray tvec1,
                             InputArray rvec2, InputArray tvec2,
                             OutputArray rvec3, OutputArray tvec3,
                             OutputArray dr3dr1 = noArray(), OutputArray dr3dt1 = noArray(),
                             OutputArray dr3dr2 = noArray(), OutputArray dr3dt2 = noArray(),
                             OutputArray dt3dr1 = noArray(), OutputArray dt3dt1 = noArray(),
                             OutputArray dt3dr2 = noArray(), OutputArray dt3dt2 = noArray() );

/** @brief Projects 3D points to an image plane.

@param objectPoints Array of object points expressed wrt. the world coordinate frame. A 3xN/Nx3
1-channel or 1xN/Nx1 3-channel (or vector\<Point3f\> ), where N is the number of points in the view.
@param rvec The rotation vector (@ref Rodrigues) that, together with tvec, performs a change of
basis from world to camera coordinate system, see @ref calibrateCamera for details.
@param tvec The translation vector, see parameter description above.
@param cameraMatrix Camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$ . If the vector is empty, the zero distortion coefficients are assumed.
@param imagePoints Output array of image points, 1xN/Nx1 2-channel, or
vector\<Point2f\> .
@param jacobian Optional output 2Nx(10+\<numDistCoeffs\>) jacobian matrix of derivatives of image
points with respect to components of the rotation vector, translation vector, focal lengths,
coordinates of the principal point and the distortion coefficients. In the old interface different
components of the jacobian are returned via different output parameters.
@param aspectRatio Optional "fixed aspect ratio" parameter. If the parameter is not 0, the
function assumes that the aspect ratio (\f$f_x / f_y\f$) is fixed and correspondingly adjusts the
jacobian matrix.

The function computes the 2D projections of 3D points to the image plane, given intrinsic and
extrinsic camera parameters. Optionally, the function computes Jacobians -matrices of partial
derivatives of image points coordinates (as functions of all the input parameters) with respect to
the particular parameters, intrinsic and/or extrinsic. The Jacobians are used during the global
optimization in @ref calibrateCamera, @ref solvePnP, and @ref stereoCalibrate. The function itself
can also be used to compute a re-projection error, given the current intrinsic and extrinsic
parameters.

@note By setting rvec = tvec = \f$[0, 0, 0]\f$, or by setting cameraMatrix to a 3x3 identity matrix,
or by passing zero distortion coefficients, one can get various useful partial cases of the
function. This means, one can compute the distorted coordinates for a sparse set of points or apply
a perspective transformation (and also compute the derivatives) in the ideal zero-distortion setup.
 */
CV_EXPORTS_W void projectPoints( InputArray objectPoints,
                                 InputArray rvec, InputArray tvec,
                                 InputArray cameraMatrix, InputArray distCoeffs,
                                 OutputArray imagePoints,
                                 OutputArray jacobian = noArray(),
                                 double aspectRatio = 0 );

/** @example samples/cpp/tutorial_code/features2D/Homography/homography_from_camera_displacement.cpp
An example program about homography from the camera displacement

Check @ref tutorial_homography "the corresponding tutorial" for more details
*/

/** @brief Finds an object pose \f$ {}^{c}\mathbf{T}_o \f$ from 3D-2D point correspondences:

![Perspective projection, from object to camera frame](pics/pinhole_homogeneous_transformation.png){ width=50% }

@see @ref calib3d_solvePnP

This function returns the rotation and the translation vectors that transform a 3D point expressed in the object
coordinate frame to the camera coordinate frame, using different methods:
- P3P methods (@ref SOLVEPNP_P3P, @ref SOLVEPNP_AP3P): need 4 input points to return a unique solution.
- @ref SOLVEPNP_IPPE Input points must be >= 4 and object points must be coplanar.
- @ref SOLVEPNP_IPPE_SQUARE Special case suitable for marker pose estimation.
Number of input points must be 4. Object points must be defined in the following order:
  - point 0: [-squareLength / 2,  squareLength / 2, 0]
  - point 1: [ squareLength / 2,  squareLength / 2, 0]
  - point 2: [ squareLength / 2, -squareLength / 2, 0]
  - point 3: [-squareLength / 2, -squareLength / 2, 0]
- for all the other flags, number of input points must be >= 4 and object points can be in any configuration.

@param objectPoints Array of object points in the object coordinate space, Nx3 1-channel or
1xN/Nx1 3-channel, where N is the number of points. vector\<Point3d\> can be also passed here.
@param imagePoints Array of corresponding image points, Nx2 1-channel or 1xN/Nx1 2-channel,
where N is the number of points. vector\<Point2d\> can be also passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvec Output rotation vector (see @ref Rodrigues ) that, together with tvec, brings points from
the model coordinate system to the camera coordinate system.
@param tvec Output translation vector.
@param useExtrinsicGuess Parameter used for #SOLVEPNP_ITERATIVE. If true (1), the function uses
the provided rvec and tvec values as initial approximations of the rotation and translation
vectors, respectively, and further optimizes them.
@param flags Method for solving a PnP problem: see @ref calib3d_solvePnP_flags

More information about Perspective-n-Points is described in @ref calib3d_solvePnP

@note
   -   An example of how to use solvePnP for planar augmented reality can be found at
        opencv_source_code/samples/python/plane_ar.py
   -   If you are using Python:
        - Numpy array slices won't work as input because solvePnP requires contiguous
        arrays (enforced by the assertion using cv::Mat::checkVector() around line 55 of
        modules/calib3d/src/solvepnp.cpp version 2.4.9)
        - The P3P algorithm requires image points to be in an array of shape (N,1,2) due
        to its calling of #undistortPoints (around line 75 of modules/calib3d/src/solvepnp.cpp version 2.4.9)
        which requires 2-channel information.
        - Thus, given some data D = np.array(...) where D.shape = (N,M), in order to use a subset of
        it as, e.g., imagePoints, one must effectively copy it into a new array: imagePoints =
        np.ascontiguousarray(D[:,:2]).reshape((N,1,2))
   -   The methods @ref SOLVEPNP_DLS and @ref SOLVEPNP_UPNP cannot be used as the current implementations are
       unstable and sometimes give completely wrong results. If you pass one of these two
       flags, @ref SOLVEPNP_EPNP method will be used instead.
   -   The minimum number of points is 4 in the general case. In the case of @ref SOLVEPNP_P3P and @ref SOLVEPNP_AP3P
       methods, it is required to use exactly 4 points (the first 3 points are used to estimate all the solutions
       of the P3P problem, the last one is used to retain the best solution that minimizes the reprojection error).
   -   With @ref SOLVEPNP_ITERATIVE method and `useExtrinsicGuess=true`, the minimum number of points is 3 (3 points
       are sufficient to compute a pose but there are up to 4 solutions). The initial solution should be close to the
       global solution to converge.
   -   With @ref SOLVEPNP_IPPE input points must be >= 4 and object points must be coplanar.
   -   With @ref SOLVEPNP_IPPE_SQUARE this is a special case suitable for marker pose estimation.
       Number of input points must be 4. Object points must be defined in the following order:
         - point 0: [-squareLength / 2,  squareLength / 2, 0]
         - point 1: [ squareLength / 2,  squareLength / 2, 0]
         - point 2: [ squareLength / 2, -squareLength / 2, 0]
         - point 3: [-squareLength / 2, -squareLength / 2, 0]
   -   With @ref SOLVEPNP_SQPNP input points must be >= 3
 */
CV_EXPORTS_W bool solvePnP( InputArray objectPoints, InputArray imagePoints,
                            InputArray cameraMatrix, InputArray distCoeffs,
                            OutputArray rvec, OutputArray tvec,
                            bool useExtrinsicGuess = false, int flags = SOLVEPNP_ITERATIVE );

/** @brief Finds an object pose \f$ {}^{c}\mathbf{T}_o \f$ from 3D-2D point correspondences using the RANSAC scheme to deal with bad matches.

![Perspective projection, from object to camera frame](pics/pinhole_homogeneous_transformation.png){ width=50% }

@see @ref calib3d_solvePnP

@param objectPoints Array of object points in the object coordinate space, Nx3 1-channel or
1xN/Nx1 3-channel, where N is the number of points. vector\<Point3d\> can be also passed here.
@param imagePoints Array of corresponding image points, Nx2 1-channel or 1xN/Nx1 2-channel,
where N is the number of points. vector\<Point2d\> can be also passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvec Output rotation vector (see @ref Rodrigues ) that, together with tvec, brings points from
the model coordinate system to the camera coordinate system.
@param tvec Output translation vector.
@param useExtrinsicGuess Parameter used for @ref SOLVEPNP_ITERATIVE. If true (1), the function uses
the provided rvec and tvec values as initial approximations of the rotation and translation
vectors, respectively, and further optimizes them.
@param iterationsCount Number of iterations.
@param reprojectionError Inlier threshold value used by the RANSAC procedure. The parameter value
is the maximum allowed distance between the observed and computed point projections to consider it
an inlier.
@param confidence The probability that the algorithm produces a useful result.
@param inliers Output vector that contains indices of inliers in objectPoints and imagePoints .
@param flags Method for solving a PnP problem (see @ref solvePnP ).

The function estimates an object pose given a set of object points, their corresponding image
projections, as well as the camera intrinsic matrix and the distortion coefficients. This function finds such
a pose that minimizes reprojection error, that is, the sum of squared distances between the observed
projections imagePoints and the projected (using @ref projectPoints ) objectPoints. The use of RANSAC
makes the function resistant to outliers.

@note
   -   An example of how to use solvePnPRansac for object detection can be found at
        @ref tutorial_real_time_pose
   -   The default method used to estimate the camera pose for the Minimal Sample Sets step
       is #SOLVEPNP_EPNP. Exceptions are:
         - if you choose #SOLVEPNP_P3P or #SOLVEPNP_AP3P, these methods will be used.
         - if the number of input points is equal to 4, #SOLVEPNP_P3P is used.
   -   The method used to estimate the camera pose using all the inliers is defined by the
       flags parameters unless it is equal to #SOLVEPNP_P3P or #SOLVEPNP_AP3P. In this case,
       the method #SOLVEPNP_EPNP will be used instead.
 */
CV_EXPORTS_W bool solvePnPRansac( InputArray objectPoints, InputArray imagePoints,
                                  InputArray cameraMatrix, InputArray distCoeffs,
                                  OutputArray rvec, OutputArray tvec,
                                  bool useExtrinsicGuess = false, int iterationsCount = 100,
                                  float reprojectionError = 8.0, double confidence = 0.99,
                                  OutputArray inliers = noArray(), int flags = SOLVEPNP_ITERATIVE );


/*
Finds rotation and translation vector.
If cameraMatrix is given then run P3P. Otherwise run linear P6P and output cameraMatrix too.
*/
CV_EXPORTS_W bool solvePnPRansac( InputArray objectPoints, InputArray imagePoints,
                     InputOutputArray cameraMatrix, InputArray distCoeffs,
                     OutputArray rvec, OutputArray tvec, OutputArray inliers,
                     const UsacParams &params=UsacParams());

/** @brief Finds an object pose \f$ {}^{c}\mathbf{T}_o \f$ from **3** 3D-2D point correspondences.

![Perspective projection, from object to camera frame](pics/pinhole_homogeneous_transformation.png){ width=50% }

@see @ref calib3d_solvePnP

@param objectPoints Array of object points in the object coordinate space, 3x3 1-channel or
1x3/3x1 3-channel. vector\<Point3f\> can be also passed here.
@param imagePoints Array of corresponding image points, 3x2 1-channel or 1x3/3x1 2-channel.
 vector\<Point2f\> can be also passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvecs Output rotation vectors (see @ref Rodrigues ) that, together with tvecs, brings points from
the model coordinate system to the camera coordinate system. A P3P problem has up to 4 solutions.
@param tvecs Output translation vectors.
@param flags Method for solving a P3P problem:
-   @ref SOLVEPNP_P3P Method is based on the paper of Ding, Y., Yang, J., Larsson, V., Olsson, C., & Åstrom, K.
"Revisiting the P3P Problem" (@cite ding2023revisiting).
-   @ref SOLVEPNP_AP3P Method is based on the paper of T. Ke and S. Roumeliotis.
"An Efficient Algebraic Solution to the Perspective-Three-Point Problem" (@cite Ke17).

The function estimates the object pose given 3 object points, their corresponding image
projections, as well as the camera intrinsic matrix and the distortion coefficients.

@note
The solutions are sorted by reprojection errors (lowest to highest).
 */
CV_EXPORTS_W int solveP3P( InputArray objectPoints, InputArray imagePoints,
                           InputArray cameraMatrix, InputArray distCoeffs,
                           OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                           int flags );

/** @brief Refine a pose (the translation and the rotation that transform a 3D point expressed in the object coordinate frame
to the camera coordinate frame) from a 3D-2D point correspondences and starting from an initial solution.

@see @ref calib3d_solvePnP

@param objectPoints Array of object points in the object coordinate space, Nx3 1-channel or 1xN/Nx1 3-channel,
where N is the number of points. vector\<Point3d\> can also be passed here.
@param imagePoints Array of corresponding image points, Nx2 1-channel or 1xN/Nx1 2-channel,
where N is the number of points. vector\<Point2d\> can also be passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvec Input/Output rotation vector (see @ref Rodrigues ) that, together with tvec, brings points from
the model coordinate system to the camera coordinate system. Input values are used as an initial solution.
@param tvec Input/Output translation vector. Input values are used as an initial solution.
@param criteria Criteria when to stop the Levenberg-Marquard iterative algorithm.

The function refines the object pose given at least 3 object points, their corresponding image
projections, an initial solution for the rotation and translation vector,
as well as the camera intrinsic matrix and the distortion coefficients.
The function minimizes the projection error with respect to the rotation and the translation vectors, according
to a Levenberg-Marquardt iterative minimization @cite Madsen04 @cite Eade13 process.
 */
CV_EXPORTS_W void solvePnPRefineLM( InputArray objectPoints, InputArray imagePoints,
                                    InputArray cameraMatrix, InputArray distCoeffs,
                                    InputOutputArray rvec, InputOutputArray tvec,
                                    TermCriteria criteria = TermCriteria(TermCriteria::EPS + TermCriteria::COUNT, 20, FLT_EPSILON));

/** @brief Refine a pose (the translation and the rotation that transform a 3D point expressed in the object coordinate frame
to the camera coordinate frame) from a 3D-2D point correspondences and starting from an initial solution.

@see @ref calib3d_solvePnP

@param objectPoints Array of object points in the object coordinate space, Nx3 1-channel or 1xN/Nx1 3-channel,
where N is the number of points. vector\<Point3d\> can also be passed here.
@param imagePoints Array of corresponding image points, Nx2 1-channel or 1xN/Nx1 2-channel,
where N is the number of points. vector\<Point2d\> can also be passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvec Input/Output rotation vector (see @ref Rodrigues ) that, together with tvec, brings points from
the model coordinate system to the camera coordinate system. Input values are used as an initial solution.
@param tvec Input/Output translation vector. Input values are used as an initial solution.
@param criteria Criteria when to stop the Levenberg-Marquard iterative algorithm.
@param VVSlambda Gain for the virtual visual servoing control law, equivalent to the \f$\alpha\f$
gain in the Damped Gauss-Newton formulation.

The function refines the object pose given at least 3 object points, their corresponding image
projections, an initial solution for the rotation and translation vector,
as well as the camera intrinsic matrix and the distortion coefficients.
The function minimizes the projection error with respect to the rotation and the translation vectors, using a
virtual visual servoing (VVS) @cite Chaumette06 @cite Marchand16 scheme.
 */
CV_EXPORTS_W void solvePnPRefineVVS( InputArray objectPoints, InputArray imagePoints,
                                     InputArray cameraMatrix, InputArray distCoeffs,
                                     InputOutputArray rvec, InputOutputArray tvec,
                                     TermCriteria criteria = TermCriteria(TermCriteria::EPS + TermCriteria::COUNT, 20, FLT_EPSILON),
                                     double VVSlambda = 1);

/** @brief Finds an object pose \f$ {}^{c}\mathbf{T}_o \f$ from 3D-2D point correspondences.

![Perspective projection, from object to camera frame](pics/pinhole_homogeneous_transformation.png){ width=50% }

@see @ref calib3d_solvePnP

This function returns a list of all the possible solutions (a solution is a <rotation vector, translation vector>
couple), depending on the number of input points and the chosen method:
- P3P methods (@ref SOLVEPNP_P3P, @ref SOLVEPNP_AP3P): 3 or 4 input points. Number of returned solutions can be between 0 and 4 with 3 input points.
- @ref SOLVEPNP_IPPE Input points must be >= 4 and object points must be coplanar. Returns 2 solutions.
- @ref SOLVEPNP_IPPE_SQUARE Special case suitable for marker pose estimation.
Number of input points must be 4 and 2 solutions are returned. Object points must be defined in the following order:
  - point 0: [-squareLength / 2,  squareLength / 2, 0]
  - point 1: [ squareLength / 2,  squareLength / 2, 0]
  - point 2: [ squareLength / 2, -squareLength / 2, 0]
  - point 3: [-squareLength / 2, -squareLength / 2, 0]
- for all the other flags, number of input points must be >= 4 and object points can be in any configuration.
Only 1 solution is returned.

@param objectPoints Array of object points in the object coordinate space, Nx3 1-channel or
1xN/Nx1 3-channel, where N is the number of points. vector\<Point3d\> can be also passed here.
@param imagePoints Array of corresponding image points, Nx2 1-channel or 1xN/Nx1 2-channel,
where N is the number of points. vector\<Point2d\> can be also passed here.
@param cameraMatrix Input camera intrinsic matrix \f$\cameramatrix{A}\f$ .
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is NULL/empty, the zero distortion coefficients are
assumed.
@param rvecs Vector of output rotation vectors (see @ref Rodrigues ) that, together with tvecs, brings points from
the model coordinate system to the camera coordinate system.
@param tvecs Vector of output translation vectors.
@param useExtrinsicGuess Parameter used for #SOLVEPNP_ITERATIVE. If true (1), the function uses
the provided rvec and tvec values as initial approximations of the rotation and translation
vectors, respectively, and further optimizes them.
@param flags Method for solving a PnP problem: see @ref calib3d_solvePnP_flags
@param rvec Rotation vector used to initialize an iterative PnP refinement algorithm, when flag is @ref SOLVEPNP_ITERATIVE
and useExtrinsicGuess is set to true.
@param tvec Translation vector used to initialize an iterative PnP refinement algorithm, when flag is @ref SOLVEPNP_ITERATIVE
and useExtrinsicGuess is set to true.
@param reprojectionError Optional vector of reprojection error, that is the RMS error
(\f$ \text{RMSE} = \sqrt{\frac{\sum_{i}^{N} \left ( \hat{y_i} - y_i \right )^2}{N}} \f$) between the input image points
and the 3D object points projected with the estimated pose.

More information is described in @ref calib3d_solvePnP

@note
   -   An example of how to use solvePnP for planar augmented reality can be found at
        opencv_source_code/samples/python/plane_ar.py
   -   If you are using Python:
        - Numpy array slices won't work as input because solvePnP requires contiguous
        arrays (enforced by the assertion using cv::Mat::checkVector() around line 55 of
        modules/calib3d/src/solvepnp.cpp version 2.4.9)
        - The P3P algorithm requires image points to be in an array of shape (N,1,2) due
        to its calling of #undistortPoints (around line 75 of modules/calib3d/src/solvepnp.cpp version 2.4.9)
        which requires 2-channel information.
        - Thus, given some data D = np.array(...) where D.shape = (N,M), in order to use a subset of
        it as, e.g., imagePoints, one must effectively copy it into a new array: imagePoints =
        np.ascontiguousarray(D[:,:2]).reshape((N,1,2))
   -   The methods @ref SOLVEPNP_DLS and @ref SOLVEPNP_UPNP cannot be used as the current implementations are
       unstable and sometimes give completely wrong results. If you pass one of these two
       flags, @ref SOLVEPNP_EPNP method will be used instead.
   -   The minimum number of points is 4 in the general case. In the case of @ref SOLVEPNP_P3P and @ref SOLVEPNP_AP3P
       methods, it is required to use exactly 4 points (the first 3 points are used to estimate all the solutions
       of the P3P problem, the last one is used to retain the best solution that minimizes the reprojection error).
   -   With @ref SOLVEPNP_ITERATIVE method and `useExtrinsicGuess=true`, the minimum number of points is 3 (3 points
       are sufficient to compute a pose but there are up to 4 solutions). The initial solution should be close to the
       global solution to converge.
   -   With @ref SOLVEPNP_IPPE input points must be >= 4 and object points must be coplanar.
   -   With @ref SOLVEPNP_IPPE_SQUARE this is a special case suitable for marker pose estimation.
       Number of input points must be 4. Object points must be defined in the following order:
         - point 0: [-squareLength / 2,  squareLength / 2, 0]
         - point 1: [ squareLength / 2,  squareLength / 2, 0]
         - point 2: [ squareLength / 2, -squareLength / 2, 0]
         - point 3: [-squareLength / 2, -squareLength / 2, 0]
   -   With @ref SOLVEPNP_SQPNP input points must be >= 3
 */
CV_EXPORTS_W int solvePnPGeneric( InputArray objectPoints, InputArray imagePoints,
                                  InputArray cameraMatrix, InputArray distCoeffs,
                                  OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                  bool useExtrinsicGuess = false, SolvePnPMethod flags = SOLVEPNP_ITERATIVE,
                                  InputArray rvec = noArray(), InputArray tvec = noArray(),
                                  OutputArray reprojectionError = noArray() );

/** @brief Finds an initial camera intrinsic matrix from 3D-2D point correspondences.

@param objectPoints Vector of vectors of the calibration pattern points in the calibration pattern
coordinate space. In the old interface all the per-view vectors are concatenated. See
#calibrateCamera for details.
@param imagePoints Vector of vectors of the projections of the calibration pattern points. In the
old interface all the per-view vectors are concatenated.
@param imageSize Image size in pixels used to initialize the principal point.
@param aspectRatio If it is zero or negative, both \f$f_x\f$ and \f$f_y\f$ are estimated independently.
Otherwise, \f$f_x = f_y \cdot \texttt{aspectRatio}\f$ .

The function estimates and returns an initial camera intrinsic matrix for the camera calibration process.
Currently, the function only supports planar calibration patterns, which are patterns where each
object point has z-coordinate =0.
 */
CV_EXPORTS_W Mat initCameraMatrix2D( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints,
                                     Size imageSize, double aspectRatio = 1.0 );

/** @brief Finds the positions of internal corners of the chessboard.

@param image Source chessboard view. It must be an 8-bit grayscale or color image.
@param patternSize Number of inner corners per a chessboard row and column
( patternSize = cv::Size(points_per_row,points_per_colum) = cv::Size(columns,rows) ).
@param corners Output array of detected corners.
@param flags Various operation flags that can be zero or a combination of the following values:
-   @ref CALIB_CB_ADAPTIVE_THRESH Use adaptive thresholding to convert the image to black
and white, rather than a fixed threshold level (computed from the average image brightness).
-   @ref CALIB_CB_NORMALIZE_IMAGE Normalize the image gamma with #equalizeHist before
applying fixed or adaptive thresholding.
-   @ref CALIB_CB_FILTER_QUADS Use additional criteria (like contour area, perimeter,
square-like shape) to filter out false quads extracted at the contour retrieval stage.
-   @ref CALIB_CB_FAST_CHECK Run a fast check on the image that looks for chessboard corners,
and shortcut the call if none is found. This can drastically speed up the call in the
degenerate condition when no chessboard is observed.
-   @ref CALIB_CB_PLAIN All other flags are ignored. The input image is taken as is.
No image processing is done to improve to find the checkerboard. This has the effect of speeding up the
execution of the function but could lead to not recognizing the checkerboard if the image
is not previously binarized in the appropriate manner.

The function attempts to determine whether the input image is a view of the chessboard pattern and
locate the internal chessboard corners. The function returns a non-zero value if all of the corners
are found and they are placed in a certain order (row by row, left to right in every row).
Otherwise, if the function fails to find all the corners or reorder them, it returns 0. For example,
a regular chessboard has 8 x 8 squares and 7 x 7 internal corners, that is, points where the black
squares touch each other. The detected coordinates are approximate, and to determine their positions
more accurately, the function calls #cornerSubPix. You also may use the function #cornerSubPix with
different parameters if returned coordinates are not accurate enough.

Sample usage of detecting and drawing chessboard corners: :
@code
    Size patternsize(8,6); //interior number of corners
    Mat gray = ....; //source image
    vector<Point2f> corners; //this will be filled by the detected corners

    //CALIB_CB_FAST_CHECK saves a lot of time on images
    //that do not contain any chessboard corners
    bool patternfound = findChessboardCorners(gray, patternsize, corners,
            CALIB_CB_ADAPTIVE_THRESH + CALIB_CB_NORMALIZE_IMAGE
            + CALIB_CB_FAST_CHECK);

    if(patternfound)
      cornerSubPix(gray, corners, Size(11, 11), Size(-1, -1),
        TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));

    drawChessboardCorners(img, patternsize, Mat(corners), patternfound);
@endcode
@note The function requires white space (like a square-thick border, the wider the better) around
the board to make the detection more robust in various environments. Otherwise, if there is no
border and the background is dark, the outer black squares cannot be segmented properly and so the
square grouping and ordering algorithm fails.

Use the `generate_pattern.py` Python script (@ref tutorial_camera_calibration_pattern)
to create the desired checkerboard pattern.
 */
CV_EXPORTS_W bool findChessboardCorners( InputArray image, Size patternSize, OutputArray corners,
                                         int flags = CALIB_CB_ADAPTIVE_THRESH + CALIB_CB_NORMALIZE_IMAGE );

/*
   Checks whether the image contains chessboard of the specific size or not.
   If yes, nonzero value is returned.
*/
CV_EXPORTS_W bool checkChessboard(InputArray img, Size size);

/** @brief Finds the positions of internal corners of the chessboard using a sector based approach.

@param image Source chessboard view. It must be an 8-bit grayscale or color image.
@param patternSize Number of inner corners per a chessboard row and column
( patternSize = cv::Size(points_per_row,points_per_colum) = cv::Size(columns,rows) ).
@param corners Output array of detected corners.
@param flags Various operation flags that can be zero or a combination of the following values:
-   @ref CALIB_CB_NORMALIZE_IMAGE Normalize the image gamma with equalizeHist before detection.
-   @ref CALIB_CB_EXHAUSTIVE Run an exhaustive search to improve detection rate.
-   @ref CALIB_CB_ACCURACY Up sample input image to improve sub-pixel accuracy due to aliasing effects.
-   @ref CALIB_CB_LARGER The detected pattern is allowed to be larger than patternSize (see description).
-   @ref CALIB_CB_MARKER The detected pattern must have a marker (see description).
This should be used if an accurate camera calibration is required.
@param meta Optional output arrray of detected corners (CV_8UC1 and size = cv::Size(columns,rows)).
Each entry stands for one corner of the pattern and can have one of the following values:
-   0 = no meta data attached
-   1 = left-top corner of a black cell
-   2 = left-top corner of a white cell
-   3 = left-top corner of a black cell with a white marker dot
-   4 = left-top corner of a white cell with a black marker dot (pattern origin in case of markers otherwise first corner)

The function is analog to #findChessboardCorners but uses a localized radon
transformation approximated by box filters being more robust to all sort of
noise, faster on larger images and is able to directly return the sub-pixel
position of the internal chessboard corners. The Method is based on the paper
@cite duda2018 "Accurate Detection and Localization of Checkerboard Corners for
Calibration" demonstrating that the returned sub-pixel positions are more
accurate than the one returned by cornerSubPix allowing a precise camera
calibration for demanding applications.

In the case, the flags @ref CALIB_CB_LARGER or @ref CALIB_CB_MARKER are given,
the result can be recovered from the optional meta array. Both flags are
helpful to use calibration patterns exceeding the field of view of the camera.
These oversized patterns allow more accurate calibrations as corners can be
utilized, which are as close as possible to the image borders.  For a
consistent coordinate system across all images, the optional marker (see image
below) can be used to move the origin of the board to the location where the
black circle is located.

@note The function requires a white boarder with roughly the same width as one
of the checkerboard fields around the whole board to improve the detection in
various environments. In addition, because of the localized radon
transformation it is beneficial to use round corners for the field corners
which are located on the outside of the board. The following figure illustrates
a sample checkerboard optimized for the detection. However, any other checkerboard
can be used as well.

Use the `generate_pattern.py` Python script (@ref tutorial_camera_calibration_pattern)
to create the corresponding checkerboard pattern:
\image html pics/checkerboard_radon.png width=60%
 */
CV_EXPORTS_AS(findChessboardCornersSBWithMeta)
bool findChessboardCornersSB(InputArray image,Size patternSize, OutputArray corners,
                             int flags,OutputArray meta);
/** @overload */
CV_EXPORTS_W inline
bool findChessboardCornersSB(InputArray image, Size patternSize, OutputArray corners,
                             int flags = 0)
{
    return findChessboardCornersSB(image, patternSize, corners, flags, noArray());
}

/** @brief Estimates the sharpness of a detected chessboard.

Image sharpness, as well as brightness, are a critical parameter for accuracte
camera calibration. For accessing these parameters for filtering out
problematic calibraiton images, this method calculates edge profiles by traveling from
black to white chessboard cell centers. Based on this, the number of pixels is
calculated required to transit from black to white. This width of the
transition area is a good indication of how sharp the chessboard is imaged
and should be below ~3.0 pixels.

@param image Gray image used to find chessboard corners
@param patternSize Size of a found chessboard pattern
@param corners Corners found by #findChessboardCornersSB
@param rise_distance Rise distance 0.8 means 10% ... 90% of the final signal strength
@param vertical By default edge responses for horizontal lines are calculated
@param sharpness Optional output array with a sharpness value for calculated edge responses (see description)

The optional sharpness array is of type CV_32FC1 and has for each calculated
profile one row with the following five entries:
* 0 = x coordinate of the underlying edge in the image
* 1 = y coordinate of the underlying edge in the image
* 2 = width of the transition area (sharpness)
* 3 = signal strength in the black cell (min brightness)
* 4 = signal strength in the white cell (max brightness)

@return Scalar(average sharpness, average min brightness, average max brightness,0)
*/
CV_EXPORTS_W Scalar estimateChessboardSharpness(InputArray image, Size patternSize, InputArray corners,
                                                float rise_distance=0.8F,bool vertical=false,
                                                OutputArray sharpness=noArray());


//! finds subpixel-accurate positions of the chessboard corners
CV_EXPORTS_W bool find4QuadCornerSubpix( InputArray img, InputOutputArray corners, Size region_size );

/** @brief Renders the detected chessboard corners.

@param image Destination image. It must be an 8-bit color image.
@param patternSize Number of inner corners per a chessboard row and column
(patternSize = cv::Size(points_per_row,points_per_column)).
@param corners Array of detected corners, the output of #findChessboardCorners.
@param patternWasFound Parameter indicating whether the complete board was found or not. The
return value of #findChessboardCorners should be passed here.

The function draws individual chessboard corners detected either as red circles if the board was not
found, or as colored corners connected with lines if the board was found.
 */
CV_EXPORTS_W void drawChessboardCorners( InputOutputArray image, Size patternSize,
                                         InputArray corners, bool patternWasFound );

/** @brief Draw axes of the world/object coordinate system from pose estimation. @sa solvePnP

@param image Input/output image. It must have 1 or 3 channels. The number of channels is not altered.
@param cameraMatrix Input 3x3 floating-point matrix of camera intrinsic parameters.
\f$\cameramatrix{A}\f$
@param distCoeffs Input vector of distortion coefficients
\f$\distcoeffs\f$. If the vector is empty, the zero distortion coefficients are assumed.
@param rvec Rotation vector (see @ref Rodrigues ) that, together with tvec, brings points from
the model coordinate system to the camera coordinate system.
@param tvec Translation vector.
@param length Length of the painted axes in the same unit than tvec (usually in meters).
@param thickness Line thickness of the painted axes.

This function draws the axes of the world/object coordinate system w.r.t. to the camera frame.
OX is drawn in red, OY in green and OZ in blue.
 */
CV_EXPORTS_W void drawFrameAxes(InputOutputArray image, InputArray cameraMatrix, InputArray distCoeffs,
                                InputArray rvec, InputArray tvec, float length, int thickness=3);

struct CV_EXPORTS_W_SIMPLE CirclesGridFinderParameters
{
    CV_WRAP CirclesGridFinderParameters();
    CV_PROP_RW cv::Size2f densityNeighborhoodSize;
    CV_PROP_RW float minDensity;
    CV_PROP_RW int kmeansAttempts;
    CV_PROP_RW int minDistanceToAddKeypoint;
    CV_PROP_RW int keypointScale;
    CV_PROP_RW float minGraphConfidence;
    CV_PROP_RW float vertexGain;
    CV_PROP_RW float vertexPenalty;
    CV_PROP_RW float existingVertexGain;
    CV_PROP_RW float edgeGain;
    CV_PROP_RW float edgePenalty;
    CV_PROP_RW float convexHullFactor;
    CV_PROP_RW float minRNGEdgeSwitchDist;

    enum GridType
    {
      SYMMETRIC_GRID, ASYMMETRIC_GRID
    };
    CV_PROP_RW GridType gridType;

    CV_PROP_RW float squareSize; //!< Distance between two adjacent points. Used by CALIB_CB_CLUSTERING.
    CV_PROP_RW float maxRectifiedDistance; //!< Max deviation from prediction. Used by CALIB_CB_CLUSTERING.
};

#ifndef DISABLE_OPENCV_3_COMPATIBILITY
typedef CirclesGridFinderParameters CirclesGridFinderParameters2;
#endif

/** @brief Finds centers in the grid of circles.

@param image grid view of input circles; it must be an 8-bit grayscale or color image.
@param patternSize number of circles per row and column
( patternSize = Size(points_per_row, points_per_colum) ).
@param centers output array of detected centers.
@param flags various operation flags that can be one of the following values:
-   @ref CALIB_CB_SYMMETRIC_GRID uses symmetric pattern of circles.
-   @ref CALIB_CB_ASYMMETRIC_GRID uses asymmetric pattern of circles.
-   @ref CALIB_CB_CLUSTERING uses a special algorithm for grid detection. It is more robust to
perspective distortions but much more sensitive to background clutter.
@param blobDetector feature detector that finds blobs like dark circles on light background.
                    If `blobDetector` is NULL then `image` represents Point2f array of candidates.
@param parameters struct for finding circles in a grid pattern.

The function attempts to determine whether the input image contains a grid of circles. If it is, the
function locates centers of the circles. The function returns a non-zero value if all of the centers
have been found and they have been placed in a certain order (row by row, left to right in every
row). Otherwise, if the function fails to find all the corners or reorder them, it returns 0.

Sample usage of detecting and drawing the centers of circles: :
@code
    Size patternsize(7,7); //number of centers
    Mat gray = ...; //source image
    vector<Point2f> centers; //this will be filled by the detected centers

    bool patternfound = findCirclesGrid(gray, patternsize, centers);

    drawChessboardCorners(img, patternsize, Mat(centers), patternfound);
@endcode
@note The function requires white space (like a square-thick border, the wider the better) around
the board to make the detection more robust in various environments.
 */
CV_EXPORTS_W bool findCirclesGrid( InputArray image, Size patternSize,
                                   OutputArray centers, int flags,
                                   const Ptr<FeatureDetector> &blobDetector,
                                   const CirclesGridFinderParameters& parameters);

/** @overload */
CV_EXPORTS_W bool findCirclesGrid( InputArray image, Size patternSize,
                                   OutputArray centers, int flags = CALIB_CB_SYMMETRIC_GRID,
                                   const Ptr<FeatureDetector> &blobDetector = SimpleBlobDetector::create());

/** @brief Finds the camera intrinsic and extrinsic parameters from several views of a calibration
pattern.

@param objectPoints In the new interface it is a vector of vectors of calibration pattern points in
the calibration pattern coordinate space (e.g. std::vector<std::vector<cv::Vec3f>>). The outer
vector contains as many elements as the number of pattern views. If the same calibration pattern
is shown in each view and it is fully visible, all the vectors will be the same. Although, it is
possible to use partially occluded patterns or even different patterns in different views. Then,
the vectors will be different. Although the points are 3D, they all lie in the calibration pattern's
XY coordinate plane (thus 0 in the Z-coordinate), if the used calibration pattern is a planar rig.
In the old interface all the vectors of object points from different views are concatenated
together.
@param imagePoints In the new interface it is a vector of vectors of the projections of calibration
pattern points (e.g. std::vector<std::vector<cv::Vec2f>>). imagePoints.size() and
objectPoints.size(), and imagePoints[i].size() and objectPoints[i].size() for each i, must be equal,
respectively. In the old interface all the vectors of object points from different views are
concatenated together.
@param imageSize Size of the image used only to initialize the camera intrinsic matrix.
@param cameraMatrix Input/output 3x3 floating-point camera intrinsic matrix
\f$\cameramatrix{A}\f$ . If @ref CALIB_USE_INTRINSIC_GUESS
and/or @ref CALIB_FIX_ASPECT_RATIO, @ref CALIB_FIX_PRINCIPAL_POINT or @ref CALIB_FIX_FOCAL_LENGTH
are specified, some or all of fx, fy, cx, cy must be initialized before calling the function.
@param distCoeffs Input/output vector of distortion coefficients
\f$\distcoeffs\f$.
@param rvecs Output vector of rotation vectors (@ref Rodrigues ) estimated for each pattern view
(e.g. std::vector<cv::Mat>>). That is, each i-th rotation vector together with the corresponding
i-th translation vector (see the next output parameter description) brings the calibration pattern
from the object coordinate space (in which object points are specified) to the camera coordinate
space. In more technical terms, the tuple of the i-th rotation and translation vector performs
a change of basis from object coordinate space to camera coordinate space. Due to its duality, this
tuple is equivalent to the position of the calibration pattern with respect to the camera coordinate
space.
@param tvecs Output vector of translation vectors estimated for each pattern view, see parameter
describtion above.
@param stdDeviationsIntrinsics Output vector of standard deviations estimated for intrinsic
parameters. Order of deviations values:
\f$(f_x, f_y, c_x, c_y, k_1, k_2, p_1, p_2, k_3, k_4, k_5, k_6 , s_1, s_2, s_3,
 s_4, \tau_x, \tau_y)\f$ If one of parameters is not estimated, it's deviation is equals to zero.
@param stdDeviationsExtrinsics Output vector of standard deviations estimated for extrinsic
parameters. Order of deviations values: \f$(R_0, T_0, \dotsc , R_{M - 1}, T_{M - 1})\f$ where M is
the number of pattern views. \f$R_i, T_i\f$ are concatenated 1x3 vectors.
 @param perViewErrors Output vector of the RMS re-projection error estimated for each pattern view.
@param flags Different flags that may be zero or a combination of the following values:
-   @ref CALIB_USE_INTRINSIC_GUESS cameraMatrix contains valid initial values of
fx, fy, cx, cy that are optimized further. Otherwise, (cx, cy) is initially set to the image
center ( imageSize is used), and focal distances are computed in a least-squares fashion.
Note, that if intrinsic parameters are known, there is no need to use this function just to
estimate extrinsic parameters. Use @ref solvePnP instead.
-   @ref CALIB_FIX_PRINCIPAL_POINT The principal point is not changed during the global
optimization. It stays at the center or at a different location specified when
 @ref CALIB_USE_INTRINSIC_GUESS is set too.
-   @ref CALIB_FIX_ASPECT_RATIO The functions consider only fy as a free parameter. The
ratio fx/fy stays the same as in the input cameraMatrix . When
 @ref CALIB_USE_INTRINSIC_GUESS is not set, the actual input values of fx and fy are
ignored, only their ratio is computed and used further.
-   @ref CALIB_ZERO_TANGENT_DIST Tangential distortion coefficients \f$(p_1, p_2)\f$ are set
to zeros and stay zero.
-   @ref CALIB_FIX_FOCAL_LENGTH The focal length is not changed during the global optimization if
 @ref CALIB_USE_INTRINSIC_GUESS is set.
-   @ref CALIB_FIX_K1,..., @ref CALIB_FIX_K6 The corresponding radial distortion
coefficient is not changed during the optimization. If @ref CALIB_USE_INTRINSIC_GUESS is
set, the coefficient from the supplied distCoeffs matrix is used. Otherwise, it is set to 0.
-   @ref CALIB_RATIONAL_MODEL Coefficients k4, k5, and k6 are enabled. To provide the
backward compatibility, this extra flag should be explicitly specified to make the
calibration function use the rational model and return 8 coefficients or more.
-   @ref CALIB_THIN_PRISM_MODEL Coefficients s1, s2, s3 and s4 are enabled. To provide the
backward compatibility, this extra flag should be explicitly specified to make the
calibration function use the thin prism model and return 12 coefficients or more.
-   @ref CALIB_FIX_S1_S2_S3_S4 The thin prism distortion coefficients are not changed during
the optimization. If @ref CALIB_USE_INTRINSIC_GUESS is set, the coefficient from the
supplied distCoeffs matrix is used. Otherwise, it is set to 0.
-   @ref CALIB_TILTED_MODEL Coefficients tauX and tauY are enabled. To provide the
backward compatibility, this extra flag should be explicitly specified to make the
calibration function use the tilted sensor model and return 14 coefficients.
-   @ref CALIB_FIX_TAUX_TAUY The coefficients of the tilted sensor model are not changed during
the optimization. If @ref CALIB_USE_INTRINSIC_GUESS is set, the coefficient from the
supplied distCoeffs matrix is used. Otherwise, it is set to 0.
@param criteria Termination criteria for the iterative optimization algorithm.

@return the overall RMS re-projection error.

The function estimates the intrinsic camera parameters and extrinsic parameters for each of the
views. The algorithm is based on @cite Zhang2000 and @cite BouguetMCT . The coordinates of 3D object
points and their corresponding 2D projections in each view must be specified. That may be achieved
by using an object with known geometry and easily detectable feature points. Such an object is
called a calibration rig or calibration pattern, and OpenCV has built-in support for a chessboard as
a calibration rig (see @ref findChessboardCorners). Currently, initialization of intrinsic
parameters (when @ref CALIB_USE_INTRINSIC_GUESS is not set) is only implemented for planar calibration
patterns (where Z-coordinates of the object points must be all zeros). 3D calibration rigs can also
be used as long as initial cameraMatrix is provided.

The algorithm performs the following steps:

-   Compute the initial intrinsic parameters (the option only available for planar calibration
    patterns) or read them from the input parameters. The distortion coefficients are all set to
    zeros initially unless some of CALIB_FIX_K? are specified.

-   Estimate the initial camera pose as if the intrinsic parameters have been already known. This is
    done using @ref solvePnP .

-   Run the global Levenberg-Marquardt optimization algorithm to minimize the reprojection error,
    that is, the total sum of squared distances between the observed feature points imagePoints and
    the projected (using the current estimates for camera parameters and the poses) object points
    objectPoints. See @ref projectPoints for details.

@note
    If you use a non-square (i.e. non-N-by-N) grid and @ref findChessboardCorners for calibration,
    and @ref calibrateCamera returns bad values (zero distortion coefficients, \f$c_x\f$ and
    \f$c_y\f$ very far from the image center, and/or large differences between \f$f_x\f$ and
    \f$f_y\f$ (ratios of 10:1 or more)), then you are probably using patternSize=cvSize(rows,cols)
    instead of using patternSize=cvSize(cols,rows) in @ref findChessboardCorners.

@note
    The function may throw exceptions, if unsupported combination of parameters is provided or
    the system is underconstrained.

@sa
   calibrateCameraRO, findChessboardCorners, solvePnP, initCameraMatrix2D, stereoCalibrate,
   undistort
 */
CV_EXPORTS_AS(calibrateCameraExtended) double calibrateCamera( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints, Size imageSize,
                                     InputOutputArray cameraMatrix, InputOutputArray distCoeffs,
                                     OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                     OutputArray stdDeviationsIntrinsics,
                                     OutputArray stdDeviationsExtrinsics,
                                     OutputArray perViewErrors,
                                     int flags = 0, TermCriteria criteria = TermCriteria(
                                        TermCriteria::COUNT + TermCriteria::EPS, 30, DBL_EPSILON) );

/** @overload */
CV_EXPORTS_W double calibrateCamera( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints, Size imageSize,
                                     InputOutputArray cameraMatrix, InputOutputArray distCoeffs,
                                     OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                     int flags = 0, TermCriteria criteria = TermCriteria(
                                        TermCriteria::COUNT + TermCriteria::EPS, 30, DBL_EPSILON) );

/** @brief Finds the camera intrinsic and extrinsic parameters from several views of a calibration pattern.

This function is an extension of #calibrateCamera with the method of releasing object which was
proposed in @cite strobl2011iccv. In many common cases with inaccurate, unmeasured, roughly planar
targets (calibration plates), this method can dramatically improve the precision of the estimated
camera parameters. Both the object-releasing method and standard method are supported by this
function. Use the parameter **iFixedPoint** for method selection. In the internal implementation,
#calibrateCamera is a wrapper for this function.

@param objectPoints Vector of vectors of calibration pattern points in the calibration pattern
coordinate space. See #calibrateCamera for details. If the method of releasing object to be used,
the identical calibration board must be used in each view and it must be fully visible, and all
objectPoints[i] must be the same and all points should be roughly close to a plane. **The calibration
target has to be rigid, or at least static if the camera (rather than the calibration target) is
shifted for grabbing images.**
@param imagePoints Vector of vectors of the projections of calibration pattern points. See
#calibrateCamera for details.
@param imageSize Size of the image used only to initialize the intrinsic camera matrix.
@param iFixedPoint The index of the 3D object point in objectPoints[0] to be fixed. It also acts as
a switch for calibration method selection. If object-releasing method to be used, pass in the
parameter in the range of [1, objectPoints[0].size()-2], otherwise a value out of this range will
make standard calibration method selected. Usually the top-right corner point of the calibration
board grid is recommended to be fixed when object-releasing method being utilized. According to
\cite strobl2011iccv, two other points are also fixed. In this implementation, objectPoints[0].front
and objectPoints[0].back.z are used. With object-releasing method, accurate rvecs, tvecs and
newObjPoints are only possible if coordinates of these three fixed points are accurate enough.
@param cameraMatrix Output 3x3 floating-point camera matrix. See #calibrateCamera for details.
@param distCoeffs Output vector of distortion coefficients. See #calibrateCamera for details.
@param rvecs Output vector of rotation vectors estimated for each pattern view. See #calibrateCamera
for details.
@param tvecs Output vector of translation vectors estimated for each pattern view.
@param newObjPoints The updated output vector of calibration pattern points. The coordinates might
be scaled based on three fixed points. The returned coordinates are accurate only if the above
mentioned three fixed points are accurate. If not needed, noArray() can be passed in. This parameter
is ignored with standard calibration method.
@param stdDeviationsIntrinsics Output vector of standard deviations estimated for intrinsic parameters.
See #calibrateCamera for details.
@param stdDeviationsExtrinsics Output vector of standard deviations estimated for extrinsic parameters.
See #calibrateCamera for details.
@param stdDeviationsObjPoints Output vector of standard deviations estimated for refined coordinates
of calibration pattern points. It has the same size and order as objectPoints[0] vector. This
parameter is ignored with standard calibration method.
 @param perViewErrors Output vector of the RMS re-projection error estimated for each pattern view.
@param flags Different flags that may be zero or a combination of some predefined values. See
#calibrateCamera for details. If the method of releasing object is used, the calibration time may
be much longer. CALIB_USE_QR or CALIB_USE_LU could be used for faster calibration with potentially
less precise and less stable in some rare cases.
@param criteria Termination criteria for the iterative optimization algorithm.

@return the overall RMS re-projection error.

The function estimates the intrinsic camera parameters and extrinsic parameters for each of the
views. The algorithm is based on @cite Zhang2000, @cite BouguetMCT and @cite strobl2011iccv. See
#calibrateCamera for other detailed explanations.
@sa
   calibrateCamera, findChessboardCorners, solvePnP, initCameraMatrix2D, stereoCalibrate, undistort
 */
CV_EXPORTS_AS(calibrateCameraROExtended) double calibrateCameraRO( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints, Size imageSize, int iFixedPoint,
                                     InputOutputArray cameraMatrix, InputOutputArray distCoeffs,
                                     OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                     OutputArray newObjPoints,
                                     OutputArray stdDeviationsIntrinsics,
                                     OutputArray stdDeviationsExtrinsics,
                                     OutputArray stdDeviationsObjPoints,
                                     OutputArray perViewErrors,
                                     int flags = 0, TermCriteria criteria = TermCriteria(
                                        TermCriteria::COUNT + TermCriteria::EPS, 30, DBL_EPSILON) );

/** @overload */
CV_EXPORTS_W double calibrateCameraRO( InputArrayOfArrays objectPoints,
                                     InputArrayOfArrays imagePoints, Size imageSize, int iFixedPoint,
                                     InputOutputArray cameraMatrix, InputOutputArray distCoeffs,
                                     OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs,
                                     OutputArray newObjPoints,
                                     int flags = 0, TermCriteria criteria = TermCriteria(
                                        TermCriteria::COUNT + TermCriteria::EPS, 30, DBL_EPSILON) );

/** @brief Computes useful camera characteristics from the camera intrinsic matrix.

@param cameraMatrix Input camera intrinsic matrix that can be estimated by #calibrateCamera or
#stereoCalibrate .
@param imageSize Input image size in pixels.
@param apertureWidth Physical width in mm of the sensor.
@param apertureHeight Physical height in mm of the sensor.
@param fovx Output field of view in degrees along the horizontal sensor axis.
@param fovy Output field of view in degrees along the vertical sensor axis.
@param focalLength Focal length of the lens in mm.
@param principalPoint Principal point in mm.
@param aspectRatio \f$f_y/f_x\f$

The function computes various useful camera characteristics from the previously estimated camera
matrix.

@note
   Do keep in mind that the unity measure 'mm' stands for whatever unit of measure one chooses for
    the chessboard pitch (it can thus be any value).
 */
CV_EXPORTS_W void calibrationMatrixValues( InputArray cameraMatrix, Size imageSize,
                                           double apertureWidth, double apertureHeight,
     
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

- **CV_EXPORTS**: A class/struct defined in this file
- **it**: A class/struct defined in this file
- **CV_EXPORTS_W**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **different**: A class/struct defined in this file
- **its**: A class/struct defined in this file
- **of**: A class/struct defined in this file
- **CV_EXPORTS_W_SIMPLE**: A class/struct defined in this file
- **all**: A class/struct defined in this file
- **implements**: A class/struct defined in this file

### Functions and Methods

- **itself()**: A function/method defined in this file
- **computes()**: A function/method defined in this file
- **requires()**: A function/method defined in this file
- **converts()**: A function/method defined in this file
- **finds()**: A function/method defined in this file
- **outputs()**: A function/method defined in this file
- **can()**: A function/method defined in this file
- **only()**: A function/method defined in this file
- **CV_DOXYGEN()**: A function/method defined in this file
- **returns()**: A function/method defined in this file
- **calculates()**: A function/method defined in this file
- **locates()**: A function/method defined in this file
- **distinguishes()**: A function/method defined in this file
- **CirclesGridFinderParameters()**: A function/method defined in this file
- **is()**: A function/method defined in this file
- **estimates()**: A function/method defined in this file
- **makes()**: A function/method defined in this file
- **attempts()**: A function/method defined in this file
- **resistant()**: A function/method defined in this file
- **uses()**: A function/method defined in this file
- **but()**: A function/method defined in this file
- **use()**: A function/method defined in this file
- **overload()**: A function/method defined in this file
- **differs()**: A function/method defined in this file
- **performs()**: A function/method defined in this file
- **DISABLE_OPENCV_3_COMPATIBILITY()**: A function/method defined in this file
- **that()**: A function/method defined in this file
- **implements()**: A function/method defined in this file
- **does()**: A function/method defined in this file
- **along()**: A function/method defined in this file
- **draws()**: A function/method defined in this file
- **OPENCV_CALIB3D_HPP()**: A function/method defined in this file
- **must()**: A function/method defined in this file
- **minimizes()**: A function/method defined in this file
- **fails()**: A function/method defined in this file
- **just()**: A function/method defined in this file
- **may()**: A function/method defined in this file
- **takes()**: A function/method defined in this file
- **calls()**: A function/method defined in this file
- **assumes()**: A function/method defined in this file
- **refines()**: A function/method defined in this file
- **or()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/affine.hpp`
- `opencv2/core/utils/logger.hpp`
- `opencv2/features2d.hpp`
- `opencv2/core/types.hpp`
- `opencv2/core.hpp`

**Python Imports:**
- `one`
- `prediction.`
- `fiducial`
- `object`
- `several`
- `this`
- `homography`
- `coplanar`
- `coordinate`
- `a`
- `camera`
- `the`
- `3D`
- `different`
- `pose`
- `an`
- `black`
- `classical`
- `outliers.`
- `world`


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

