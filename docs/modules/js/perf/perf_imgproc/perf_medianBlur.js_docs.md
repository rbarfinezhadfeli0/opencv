# Documentation for `modules/js/perf/perf_imgproc/perf_medianBlur.js`

## File Metadata

- **Full Path**: `modules/js/perf/perf_imgproc/perf_medianBlur.js`
- **File Name**: `perf_medianBlur.js`
- **File Size**: 4,239 bytes
- **File Type**: .js
- **Link to Source**: [modules/js/perf/perf_imgproc/perf_medianBlur.js](../../../../modules/js/perf/perf_imgproc/perf_medianBlur.js)

## Purpose and Role

This file is located in the `modules/js/perf/perf_imgproc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
var isNodeJs = (typeof window) === 'undefined'? true : false;

if　(isNodeJs)　{
  var Benchmark = require('benchmark');
  var cv = require('../../opencv');
  var HelpFunc = require('../perf_helpfunc');
  var Base = require('../base');
} else {
  var paramsElement = document.getElementById('params');
  var runButton = document.getElementById('runButton');
  var logElement = document.getElementById('log');
}

function perf() {

    console.log('opencv.js loaded');
    if (isNodeJs) {
      global.cv = cv;
      global.combine = HelpFunc.combine;
      global.log = HelpFunc.log;
      global.decodeParams2Case = HelpFunc.decodeParams2Case;
      global.setBenchmarkSuite = HelpFunc.setBenchmarkSuite;
      global.addKernelCase = HelpFunc.addKernelCase;
      global.cvSize = Base.getCvSize();
    } else {
      enableButton();
      cvSize = getCvSize();
    }
    let totalCaseNum, currentCaseId;

    const MedianBlurSize = [cvSize.szODD, cvSize.szQVGA, cvSize.szVGA, cvSize.sz720p];
    const MedianBlurType = ["CV_8UC1", "CV_8UC4", "CV_16UC1", "CV_16SC1", "CV_32FC1"];
    const combiMedianBlur = combine(MedianBlurSize, MedianBlurType, [3,5]);

    function addMedianBlurCase(suite, type) {
        suite.add('medianBlur', function() {
            cv.medianBlur(src, dst, ksize);
          }, {
              'setup': function() {
                let size = this.params.size;
                let matType = cv[this.params.matType];
                let ksize = this.params.ksize;
                let src = new cv.Mat(size, matType);
                let dst = new cv.Mat(size, matType);
                },
              'teardown': function() {
                src.delete();
                dst.delete();
              }
          });
    }

    function addMedianBlurModeCase(suite, combination, type) {
      totalCaseNum += combination.length;
      for (let i = 0; i < combination.length; ++i) {
        let size =  combination[i][0];
        let matType = combination[i][1];
        let ksize = combination[i][2];

        let params = {size: size, matType:matType, ksize: ksize};
        addKernelCase(suite, params, type, addMedianBlurCase);
      }
    }

    function genBenchmarkCase(paramsContent) {
      let suite = new Benchmark.Suite;
      totalCaseNum = 0;
      currentCaseId = 0;

      if (/\([0-9]+x[0-9]+,[\ ]*CV\_\w+,[\ ]*(3|5)\)/g.test(paramsContent.toString())) {
          let params = paramsContent.toString().match(/\([0-9]+x[0-9]+,[\ ]*CV\_\w+,[\ ]*(3|5)\)/g)[0];
          let paramObjs = [];
          paramObjs.push({name:"size", value:"", reg:[""], index:0});
          paramObjs.push({name:"matType", value:"", reg:["/CV\_[0-9]+[FSUfsu]C[0-9]/"], index:1});
          paramObjs.push({name:"ksize", value: "", reg:["/\\b[0-9]\\b/"], index:2});
          let locationList = decodeParams2Case(params, paramObjs, medianBlurCombinations);

          for (let i = 0; i < locationList.length; i++){
              let first = locationList[i][0];
              let second = locationList[i][1];
              addMedianBlurModeCase(suite, [medianBlurCombinations[first][second]], first);
            }
      } else {
        log("no filter or getting invalid params, run all the cases");
        addMedianBlurModeCase(suite, combiMedianBlur, 0);
      }
      setBenchmarkSuite(suite, "medianBlur", currentCaseId);
      log(`Running ${totalCaseNum} tests from medianBlur`);
      suite.run({ 'async': true }); // run the benchmark
  }

    let medianBlurCombinations = [combiMedianBlur];

    if (isNodeJs) {
      const args = process.argv.slice(2);
      let paramsContent = '';
      if (/--test_param_filter=\([0-9]+x[0-9]+,[\ ]*CV\_\w+,[\ ]*(3|5)\)/g.test(args.toString())) {
        paramsContent = args.toString().match(/\([0-9]+x[0-9]+,[\ ]*CV\_\w+,[\ ]*(3|5)\)/g)[0];
      }
      genBenchmarkCase(paramsContent);
    } else {
      runButton.onclick = function()　{
        let paramsContent = paramsElement.value;
        genBenchmarkCase(paramsContent);
        if (totalCaseNum !== 0) {
          disableButton();
        }
      }
    }
};

async function main() {
  if (cv instanceof Promise) {
    cv = await cv;
    perf();
  } else {
    cv.onRuntimeInitialized = perf;
  }
}

main();```

## High-Level Overview

This is a .js source code file.


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Functions and Methods

- **addMedianBlurCase()**: A function/method defined in this file
- **perf()**: A function/method defined in this file
- **genBenchmarkCase()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **addMedianBlurModeCase()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `medianBlur`


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

