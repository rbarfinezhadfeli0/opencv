# Documentation for `modules/js/perf/README.md`

## File Metadata

- **Full Path**: `modules/js/perf/README.md`
- **File Name**: `README.md`
- **File Size**: 1,124 bytes
- **File Type**: .md
- **Link to Source**: [modules/js/perf/README.md](../../../modules/js/perf/README.md)

## Purpose and Role

This file is located in the `modules/js/perf` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# OpenCV.js Performance Test

## Node.js Version

### Prerequisites

1. node.js, npm: Make sure you have installed these beforehand with the system package manager.

2. Benchmark.js: Make sure you have installed Benchmark.js by npm before use. Please run `npm install` in the directory `<build_dir>/bin/perf`.

### How to Use

For example, if you want to test the performance of cvtColor, please run `perf_cvtcolor.js` by node in terminal:

```sh
node perf_cvtcolor.js
```

All tests of cvtColor will be run by above command.

If you just want to run one specific case, please use `--test_param_filter="()"` flag, like:

```sh
node perf_cvtcolor.js --test_param_filter="(1920x1080, COLOR_BGR2GRAY)"
```

## Browser Version

### How to Use

To run performance tests, please launch a local web server in <build_dir>/bin folder. For example, node http-server which serves on localhost:8080.

Navigate the web browser to the kernel page you want to test, like http://localhost:8080/perf/imgproc/cvtcolor.html.

You can input the parameter, and then click the `Run` button to run the specific case, or it will run all the cases.


## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

