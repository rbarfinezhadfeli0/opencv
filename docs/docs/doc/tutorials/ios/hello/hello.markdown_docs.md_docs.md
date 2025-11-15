# Documentation for `docs/doc/tutorials/ios/hello/hello.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/ios/hello/hello.markdown_docs.md`
- **File Name**: `hello.markdown_docs.md`
- **File Size**: 2,759 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/ios/hello/hello.markdown_docs.md](../../../../../docs/doc/tutorials/ios/hello/hello.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/ios/hello` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/ios/hello/hello.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/ios/hello/hello.markdown`
- **File Name**: `hello.markdown`
- **File Size**: 2,161 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/ios/hello/hello.markdown](../../../../doc/tutorials/ios/hello/hello.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/ios/hello` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
OpenCV iOS Hello {#tutorial_hello}
================

@tableofcontents

@prev_tutorial{tutorial_ios_install}
@next_tutorial{tutorial_image_manipulation}

|    |    |
| -: | :- |
| Original author | Charu Hans |
| Compatibility | OpenCV >= 3.0 |

Goal
----

In this tutorial we will learn how to:

-   Link OpenCV framework with Xcode
-   How to write simple Hello World application using OpenCV and Xcode.

Linking OpenCV iOS
------------------

Follow this step by step guide to link OpenCV to iOS.

-#  Create a new XCode project.
-#  Now we need to link *opencv2.framework* with Xcode. Select the project Navigator in the left
    hand panel and click on project name.
-#  Under the TARGETS click on Build Phases. Expand Link Binary With Libraries option.
-#  Click on Add others and go to directory where *opencv2.framework* is located and click open
-#  Now you can start writing your application.

![](images/linking_opencv_ios.png)

Hello OpenCV iOS Application
----------------------------

Now we will learn how to write a simple Hello World Application in Xcode using OpenCV.

-   Link your project with OpenCV as shown in previous section.
-   Open the file named *NameOfProject-Prefix.pch* ( replace NameOfProject with name of your
    project) and add the following lines of code.
    @code{.m}
    #ifdef __cplusplus
    #import <opencv2/opencv.hpp>
    #endif
    @endcode
    ![](images/header_directive.png)

-   Add the following lines of code to viewDidLoad method in ViewController.m.
    @code{.m}
    UIAlertView * alert = [[UIAlertView alloc] initWithTitle:@"Hello!" message:@"Welcome to OpenCV" delegate:self cancelButtonTitle:@"Continue" otherButtonTitles:nil];
    [alert show];
    @endcode
    ![](images/view_did_load.png)

-   You are good to run the project.

Output
------

![](images/ios_hello_output.png)

Changes for XCode5+ and iOS8+
-----------------------------

With the newer XCode and iOS versions you need to watch out for some specific details

-   The *.m file in your project should be renamed to *.mm.
-   You have to manually include AssetsLibrary.framework into your project, which is not done anymore by default.

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

