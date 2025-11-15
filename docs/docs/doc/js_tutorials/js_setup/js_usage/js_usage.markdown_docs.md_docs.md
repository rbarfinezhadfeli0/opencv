# Documentation for `docs/doc/js_tutorials/js_setup/js_usage/js_usage.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/js_tutorials/js_setup/js_usage/js_usage.markdown_docs.md`
- **File Name**: `js_usage.markdown_docs.md`
- **File Size**: 5,677 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/js_tutorials/js_setup/js_usage/js_usage.markdown_docs.md](../../../../../docs/doc/js_tutorials/js_setup/js_usage/js_usage.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/js_tutorials/js_setup/js_usage` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/js_tutorials/js_setup/js_usage/js_usage.markdown`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_setup/js_usage/js_usage.markdown`
- **File Name**: `js_usage.markdown`
- **File Size**: 5,009 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/js_tutorials/js_setup/js_usage/js_usage.markdown](../../../../doc/js_tutorials/js_setup/js_usage/js_usage.markdown)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_setup/js_usage` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Using OpenCV.js {#tutorial_js_usage}
===============================

Steps
-----

In this tutorial, you will learn how to include and start to use `opencv.js` inside a web page.
You can get a copy of `opencv.js` from `opencv-{VERSION_NUMBER}-docs.zip` in each [release](https://github.com/opencv/opencv/releases), or simply download the prebuilt script from the online documentations at "https://docs.opencv.org/{VERSION_NUMBER}/opencv.js" (For example, [https://docs.opencv.org/4.5.0/opencv.js](https://docs.opencv.org/4.5.0/opencv.js). Use `4.x` if you want the latest build).
You can also build your own copy by following the tutorial @ref tutorial_js_setup.

### Create a web page

First, let's create a simple web page that is able to upload an image.

@code{.js}
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Hello OpenCV.js</title>
</head>
<body>
<h2>Hello OpenCV.js</h2>
<div>
  <div class="inputoutput">
    <img id="imageSrc" alt="No Image" />
    <div class="caption">imageSrc <input type="file" id="fileInput" name="file" /></div>
  </div>
</div>
<script type="text/javascript">
let imgElement = document.getElementById("imageSrc")
let inputElement = document.getElementById("fileInput");
inputElement.addEventListener("change", (e) => {
  imgElement.src = URL.createObjectURL(e.target.files[0]);
}, false);
</script>
</body>
</html>
@endcode

To run this web page, copy the content above and save to a local index.html file. To run it, open it using your web browser.

@note It is a better practice to use a local web server to host the index.html.

### Include OpenCV.js

Set the URL of `opencv.js` to `src` attribute of \<script\> tag.

@note For this tutorial, we host `opencv.js` at same folder as index.html. You can also choose to use the URL of the prebuilt `opencv.js` in our online documentation.

Example for synchronous loading:
@code{.js}
<script src="opencv.js" type="text/javascript"></script>
@endcode

You may want to load `opencv.js` asynchronously by `async` attribute in \<script\> tag. To be notified when `opencv.js` is ready, you can register a callback to `onload` attribute.

Example for asynchronous loading
@code{.js}
<script async src="opencv.js" onload="onOpenCvReady();" type="text/javascript"></script>
@endcode

### Use OpenCV.js

Once `opencv.js` is ready, you can access OpenCV objects and functions through `cv` object.
The promise-typed `cv` object should be unwrap with `await` operator.
See https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await .

For example, you can create a cv.Mat from an image by cv.imread.

@note Because image loading is asynchronous, you need to put cv.Mat creation inside the `onload` callback.

@code{.js}
imgElement.onload = await function() {
  cv = (cv instanceof Promise) ? await cv : cv;
  let mat = cv.imread(imgElement);
}
@endcode

Many OpenCV functions can be used to process cv.Mat. You can refer to other tutorials, such as @ref tutorial_js_table_of_contents_imgproc, for details.

In this tutorial, we just show a cv.Mat on screen. To show a cv.Mat, you need a canvas element.

@code{.js}
<canvas id="outputCanvas"></canvas>
@endcode

You can use cv.imshow to show cv.Mat on the canvas.
@code{.js}
cv.imshow("outputCanvas", mat);
@endcode

Putting all of the steps together, the final index.html is shown below.

@code{.js}
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Hello OpenCV.js</title>
</head>
<body>
<h2>Hello OpenCV.js</h2>
<p id="status">OpenCV.js is loading...</p>
<div>
  <div class="inputoutput">
    <img id="imageSrc" alt="No Image" />
    <div class="caption">imageSrc <input type="file" id="fileInput" name="file" /></div>
  </div>
  <div class="inputoutput">
    <canvas id="canvasOutput" ></canvas>
    <div class="caption">canvasOutput</div>
  </div>
</div>
<script type="text/javascript">
let imgElement = document.getElementById('imageSrc');
let inputElement = document.getElementById('fileInput');
inputElement.addEventListener('change', (e) => {
  imgElement.src = URL.createObjectURL(e.target.files[0]);
}, false);

imgElement.onload = async function() {
  cv = (cv instanceof Promise) ? await cv : cv;
  let mat = cv.imread(imgElement);
  cv.imshow('canvasOutput', mat);
  mat.delete();
};

var Module = {
  // https://emscripten.org/docs/api_reference/module.html#Module.onRuntimeInitialized
  onRuntimeInitialized() {
    document.getElementById('status').innerHTML = 'OpenCV.js is ready.';
  }
};
</script>
<script async src="opencv.js" type="text/javascript"></script>
</body>
</html>
@endcode

@note You have to call delete method of cv.Mat to free memory allocated in Emscripten's heap. Please refer to [Memory management of Emscripten](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html#memory-management) for details.

Try it
------
\htmlonly
<iframe src="../../js_setup_usage.html" width="100%"
        onload="this.style.height=this.contentDocument.body.scrollHeight +'px';">
</iframe>
\endhtmlonly

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

