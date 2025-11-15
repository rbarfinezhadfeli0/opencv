# Documentation for `docs/doc/js_tutorials/js_assets/js_trackbar.html_docs.md`

## File Metadata

- **Full Path**: `docs/doc/js_tutorials/js_assets/js_trackbar.html_docs.md`
- **File Name**: `js_trackbar.html_docs.md`
- **File Size**: 3,329 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/js_tutorials/js_assets/js_trackbar.html_docs.md](../../../../docs/doc/js_tutorials/js_assets/js_trackbar.html_docs.md)

## Purpose and Role

This file is located in the `docs/doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/js_tutorials/js_assets/js_trackbar.html`

## File Metadata

- **Full Path**: `doc/js_tutorials/js_assets/js_trackbar.html`
- **File Name**: `js_trackbar.html`
- **File Size**: 2,713 bytes
- **File Type**: .html
- **Link to Source**: [doc/js_tutorials/js_assets/js_trackbar.html](../../../doc/js_tutorials/js_assets/js_trackbar.html)

## Purpose and Role

This file is located in the `doc/js_tutorials/js_assets` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Trackbar Example</title>
<link href="js_example_style.css" rel="stylesheet" type="text/css" />
</head>
<body>
<h2>Trackbar Example</h2>
<p>
    &lt;canvas&gt; elements named <b>canvasInput1</b>, <b>canvasInput2</b> and <b>canvasOutput</b> have been prepared.<br>
    The code of &lt;textarea&gt; will be executed when &lt;input&gt; element named <b>trackbar</b> value changes.<br>
    You can change the code in the &lt;textarea&gt; to investigate more.
</p>
<div>
<textarea class="code" rows="12" cols="80" id="codeEditor" spellcheck="false">
</textarea>
<p class="err" id="errorMessage"></p>
</div>
<div>
    <b>trackbar</b>
    <input type="range" id="trackbar" disabled value="50" min="0" max="100" step="1">
    <label id="weightValue" ></label>
    <div>
        <table cellpadding="0" cellspacing="0" width="0" border="0">
        <tr>
            <td>
                <canvas id="canvasInput1" class="small"></canvas>
            </td>
            <td>
                <canvas id="canvasInput2" class="small"></canvas>
            </td>
            <td>
                <canvas id="canvasOutput" class="small"></canvas>
            </td>
        </tr>
        <tr>
            <td>
                <div class="caption">canvasInput1</div>
            </td>
            <td>
                <div class="caption">canvasInput2</div>
            </td>
            <td>
                <div class="caption">canvasOutput</div>
            </td>
        </tr>
        </table>
    </div>
</div>
<script src="utils.js" type="text/javascript"></script>
<script id="codeSnippet" type="text/code-snippet">
let trackbar = document.getElementById('trackbar');
let alpha = trackbar.value/trackbar.max;
let beta = ( 1.0 - alpha );
let src1 = cv.imread('canvasInput1');
let src2 = cv.imread('canvasInput2');
let dst = new cv.Mat();
cv.addWeighted( src1, alpha, src2, beta, 0.0, dst, -1);
cv.imshow('canvasOutput', dst);
dst.delete();
src1.delete();
src2.delete();
</script>
<script type="text/javascript">
let utils = new Utils('errorMessage');

utils.loadCode('codeSnippet', 'codeEditor');
utils.loadImageToCanvas('apple.jpg', 'canvasInput1');
utils.loadImageToCanvas('orange.jpg', 'canvasInput2');

let trackbar = document.getElementById('trackbar');
trackbar.addEventListener('input', () => {
    utils.executeCode('codeEditor');
});

let weightValue = document.getElementById('weightValue');
weightValue.innerText = trackbar.value;
trackbar.addEventListener('input', () => {
    weightValue.innerText = trackbar.value;
});

utils.loadOpenCv(() => {
    trackbar.removeAttribute('disabled');
    utils.executeCode('codeEditor');
});
</script>
</body>
</html>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

