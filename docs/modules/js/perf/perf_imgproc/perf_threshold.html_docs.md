# Documentation for `modules/js/perf/perf_imgproc/perf_threshold.html`

## File Metadata

- **Full Path**: `modules/js/perf/perf_imgproc/perf_threshold.html`
- **File Name**: `perf_threshold.html`
- **File Size**: 2,223 bytes
- **File Type**: .html
- **Link to Source**: [modules/js/perf/perf_imgproc/perf_threshold.html](../../../../modules/js/perf/perf_imgproc/perf_threshold.html)

## Purpose and Role

This file is located in the `modules/js/perf/perf_imgproc` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>OpenCV.js Performance Test</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
    <style>
      body {
        font-size: 13px;
      }
      .top-margin {
        margin-top:10px;
      }
      h1, h4 {
        margin: 24px 0 0;
      }
      h1 {
        font-size: 2.0em;
      }
      h4 {
        font-size: 1.2em;
      }
      pre {
        font-family: 'Consolas', 'Monaco', monospace, serif;
        font-size: 12px;
        tab-size: 2;
      }
      input[type=checkbox] {
        vertical-align: middle;
      }
    </style>
  </head>
  <body>
    <div class="container" id="container">
      <div class="row">
        <div class="col-12">
          <h1>OpenCV.js Performance Test</h1>
          <div>
            <h4>Modules</h4>
              <h7>Image Processing</h7>
          </div>
          <div>
            <h4>Kernels</h4>
              <h7>Threshold</h7>
          </div>
          <div>
            <h4>Parameters Filter</h4>
            <input type="text" id="params" min="1" size="40" placeholder="default: run all the case"/>  for example: (1920x1080, CV_8UC1, THRESH_BINARY)
          </div>
          <div class='row labels-wrapper' id='labelitem'></div>
          <div class="row top-margin">
          </div>
          <div>
          <button type="button" id="runButton" class="btn btn-primary disabled" disabled="disabled">Loading</button>
            (It will take several minutes)</div>
          <div class="row top-margin">
          </div>
          <div>
            <pre id="log"></pre>
          </div>
        </div>
      </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/platform/1.3.5/platform.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/benchmark/2.1.4/benchmark.js"></script>
    <script src="../../opencv.js" type="text/javascript"></script>
    <script src="../base.js"></script>
    <script src="../perf_helpfunc.js"></script>
    <script src="./perf_threshold.js"></script>
  </body>
</html>
```

## General Information

This file is part of the OpenCV repository infrastructure.

