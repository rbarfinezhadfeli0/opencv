# Documentation for `docs/doc/js_tutorials/js_assets/js_image_display.html_docs.md`

## File Metadata

- **Full Path**: `docs/doc/js_tutorials/js_assets/js_image_display.html_docs.md`
- **File Name**: `js_image_display.html_docs.md`
- **File Size**: 2,666 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/js_tutorials/js_assets/js_image_display.html_docs.md](../../../../docs/doc/js_tutorials/js_assets/js_image_display.html_docs.md)

## Purpose and Role

This file is located in the `docs/doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/js_tutorials/js_assets/js_image_display.html`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_image_display.html`
- **File Name**: `js_image_display.html`
- **File Size**: 2,025 bytes
- **File Type**: .html
- **Link to Source**: [doc/js_tutorials/js_assets/js_image_display.html](../../../doc/js_tutorials/js_assets/js_image_display.html)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Image Read and Show Example</title>
<link href="js_example_style.css" rel="stylesheet" type="text/css" />
</head>
<body>
<h2>Image Read and Show Example</h2>
<p>
    &lt;canvas&gt; elements named <b>canvasInput</b> and <b>canvasOutput</b> have been prepared.<br>
    Click <b>Try it</b> button to see the result. You can choose another image.<br>
    You can change the code in the &lt;textarea&gt; to investigate more.
</p>
<div>
<div class="control"><button id="tryIt" disabled>Try it</button></div>
<textarea class="code" rows="8" cols="80" id="codeEditor" spellcheck="false">
</textarea>
<p class="err" id="errorMessage"></p>
</div>
<div>
    <table cellpadding="0" cellspacing="0" width="0" border="0">
    <tr>
        <td>
            <canvas id="canvasInput"></canvas>
        </td>
        <td>
            <canvas id="canvasOutput"></canvas>
        </td>
    </tr>
    <tr>
        <td>
            <div class="caption">canvasInput <input type="file" id="fileInput" name="file" accept="image/*" /></div>
        </td>
        <td>
            <div class="caption">canvasOutput</div>
        </td>
    </tr>
    </table>
</div>
<script src="utils.js" type="text/javascript"></script>
<script id="codeSnippet" type="text/code-snippet">
let src = cv.imread('canvasInput');
let dst = new cv.Mat();
// To distinguish the input and output, we graying the image.
// You can try different conversions.
cv.cvtColor(src, dst, cv.COLOR_RGBA2GRAY);
cv.imshow('canvasOutput', dst);
src.delete();
dst.delete();
</script>
<script type="text/javascript">
let utils = new Utils('errorMessage');

utils.loadCode('codeSnippet', 'codeEditor');
utils.loadImageToCanvas('lena.jpg', 'canvasInput');
utils.addFileInputHandler('fileInput', 'canvasInput');

let tryIt = document.getElementById('tryIt');
tryIt.addEventListener('click', () => {
    utils.executeCode('codeEditor');
});

utils.loadOpenCv(() => {
    tryIt.removeAttribute('disabled');
});
</script>
</body>
</html>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

