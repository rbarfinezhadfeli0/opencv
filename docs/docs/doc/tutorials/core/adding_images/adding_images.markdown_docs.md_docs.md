# Documentation for `docs/doc/tutorials/core/adding_images/adding_images.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/core/adding_images/adding_images.markdown_docs.md`
- **File Name**: `adding_images.markdown_docs.md`
- **File Size**: 4,534 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/core/adding_images/adding_images.markdown_docs.md](../../../../../docs/doc/tutorials/core/adding_images/adding_images.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/core/adding_images` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/core/adding_images/adding_images.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/core/adding_images/adding_images.markdown`
- **File Name**: `adding_images.markdown`
- **File Size**: 3,851 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/core/adding_images/adding_images.markdown](../../../../doc/tutorials/core/adding_images/adding_images.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/core/adding_images` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Adding (blending) two images using OpenCV {#tutorial_adding_images}
=========================================

@tableofcontents

@prev_tutorial{tutorial_mat_operations}
@next_tutorial{tutorial_basic_linear_transform}

|    |    |
| -: | :- |
| Original author | Ana Huamán |
| Compatibility | OpenCV >= 3.0 |

We will learn how to blend two images!
Goal
----

In this tutorial you will learn:

-   what is *linear blending* and why it is useful;
-   how to add two images using **addWeighted()**

Theory
------

@note
   The explanation below belongs to the book [Computer Vision: Algorithms and
    Applications](http://szeliski.org/Book/) by Richard Szeliski

From our previous tutorial, we know already a bit of *Pixel operators*. An interesting dyadic
(two-input) operator is the *linear blend operator*:

\f[g(x) = (1 - \alpha)f_{0}(x) + \alpha f_{1}(x)\f]

By varying \f$\alpha\f$ from \f$0 \rightarrow 1\f$ this operator can be used to perform a temporal
*cross-dissolve* between two images or videos, as seen in slide shows and film productions (cool,
eh?)

Source Code
-----------

@add_toggle_cpp
Download the source code from
[here](https://raw.githubusercontent.com/opencv/opencv/4.x/samples/cpp/tutorial_code/core/AddingImages/AddingImages.cpp).
@include cpp/tutorial_code/core/AddingImages/AddingImages.cpp
@end_toggle

@add_toggle_java
Download the source code from
[here](https://raw.githubusercontent.com/opencv/opencv/4.x/samples/java/tutorial_code/core/AddingImages/AddingImages.java).
@include java/tutorial_code/core/AddingImages/AddingImages.java
@end_toggle

@add_toggle_python
Download the source code from
[here](https://raw.githubusercontent.com/opencv/opencv/4.x/samples/python/tutorial_code/core/AddingImages/adding_images.py).
@include python/tutorial_code/core/AddingImages/adding_images.py
@end_toggle

Explanation
-----------

Since we are going to perform:

\f[g(x) = (1 - \alpha)f_{0}(x) + \alpha f_{1}(x)\f]

We need two source images (\f$f_{0}(x)\f$ and \f$f_{1}(x)\f$). So, we load them in the usual way:
@add_toggle_cpp
@snippet cpp/tutorial_code/core/AddingImages/AddingImages.cpp load
@end_toggle

@add_toggle_java
@snippet java/tutorial_code/core/AddingImages/AddingImages.java load
@end_toggle

@add_toggle_python
@snippet python/tutorial_code/core/AddingImages/adding_images.py load
@end_toggle

We used the following images: [LinuxLogo.jpg](https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/LinuxLogo.jpg) and [WindowsLogo.jpg](https://raw.githubusercontent.com/opencv/opencv/4.x/samples/data/WindowsLogo.jpg)

@warning Since we are *adding* *src1* and *src2*, they both have to be of the same size
(width and height) and type.

Now we need to generate the `g(x)` image. For this, the function **addWeighted()** comes quite handy:

@add_toggle_cpp
@snippet cpp/tutorial_code/core/AddingImages/AddingImages.cpp blend_images
@end_toggle

@add_toggle_java
@snippet java/tutorial_code/core/AddingImages/AddingImages.java blend_images
@end_toggle

@add_toggle_python
@snippet python/tutorial_code/core/AddingImages/adding_images.py blend_images
Numpy version of above line (but cv function is around 2x faster):
\code{.py}
    dst = np.uint8(alpha*(img1)+beta*(img2))
\endcode
@end_toggle

since **addWeighted()**  produces:
\f[dst = \alpha \cdot src1 + \beta \cdot src2 + \gamma\f]
In this case, `gamma` is the argument \f$0.0\f$ in the code above.

Create windows, show the images and wait for the user to end the program.
@add_toggle_cpp
@snippet cpp/tutorial_code/core/AddingImages/AddingImages.cpp display
@end_toggle

@add_toggle_java
@snippet java/tutorial_code/core/AddingImages/AddingImages.java display
@end_toggle

@add_toggle_python
@snippet python/tutorial_code/core/AddingImages/adding_images.py display
@end_toggle

Result
------

![](images/Adding_Images_Tutorial_Result_Big.jpg)

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

