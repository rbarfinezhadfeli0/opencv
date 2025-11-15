# Documentation for `docs/doc/mymath.sty_docs.md`

## File Metadata

- **Full Path**: `docs/doc/mymath.sty_docs.md`
- **File Name**: `mymath.sty_docs.md`
- **File Size**: 1,617 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/mymath.sty_docs.md](../../docs/doc/mymath.sty_docs.md)

## Purpose and Role

This file is located in the `docs/doc` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/mymath.sty`

## File Metadata

- **Full Path**: `doc/mymath.sty`
- **File Name**: `mymath.sty`
- **File Size**: 1,153 bytes
- **File Type**: .sty
- **Link to Source**: [doc/mymath.sty](../doc/mymath.sty)

## Purpose and Role

This file is located in the `doc` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
\ProvidesPackage{mymath}

\usepackage{euler}
\usepackage{amssymb}
\usepackage{amsmath}
\usepackage{bbm}

\newcommand{\matTT}[9]{
\[
\left|\begin{array}{ccc}
 #1 & #2 & #3\\
 #4 & #5 & #6\\
 #7 & #8 & #9
\end{array}\right|
\]
}

\newcommand{\fork}[4]{
  \left\{
  \begin{array}{l l}
  #1 & \mbox{#2}\\
  #3 & \mbox{#4}\\
  \end{array} \right.}
\newcommand{\forkthree}[6]{
  \left\{
  \begin{array}{l l}
  #1 & \mbox{#2}\\
  #3 & \mbox{#4}\\
  #5 & \mbox{#6}\\
  \end{array} \right.}
\newcommand{\forkfour}[8]{
  \left\{
  \begin{array}{l l}
  #1 & \mbox{#2}\\
  #3 & \mbox{#4}\\
  #5 & \mbox{#6}\\
  #7 & \mbox{#8}\\
  \end{array} \right.}
\newcommand{\vecthree}[3]{
\begin{bmatrix}
 #1\\
 #2\\
 #3
\end{bmatrix}
}

\newcommand{\vecthreethree}[9]{
\begin{bmatrix}
 #1 & #2 & #3\\
 #4 & #5 & #6\\
 #7 & #8 & #9
\end{bmatrix}
}

\newcommand{\cameramatrix}[1]{
#1 =
\begin{bmatrix}
 f_x & 0 & c_x\\
 0 & f_y & c_y\\
 0 & 0 & 1
\end{bmatrix}
}

\newcommand{\distcoeffs}[]{
(k_1, k_2, p_1, p_2[, k_3[, k_4, k_5, k_6 [, s_1, s_2, s_3, s_4[, \tau_x, \tau_y]]]]) \text{ of 4, 5, 8, 12 or 14 elements}
}

\newcommand{\distcoeffsfisheye}[]{
(k_1, k_2, k_3, k_4)
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

